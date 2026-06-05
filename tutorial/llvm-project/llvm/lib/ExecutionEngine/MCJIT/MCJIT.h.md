# MCJIT.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/MCJIT/MCJIT.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Class definition for the MCJIT.
  - **CN**: 实现传统的 MCJIT 执行引擎，通过 LLVM 代码生成与 RuntimeDyld 把模块编译成本地代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- MCJIT.h - Class definition for the MCJIT ----------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_EXECUTIONENGINE_MCJIT_MCJIT_H
#define LLVM_LIB_EXECUTIONENGINE_MCJIT_MCJIT_H

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 12-22
```cpp
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ExecutionEngine/ExecutionEngine.h"
#include "llvm/ExecutionEngine/RTDyldMemoryManager.h"
#include "llvm/ExecutionEngine/RuntimeDyld.h"

namespace llvm {
class MCJIT;
class Module;
class ObjectCache;

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ExecutionEngine/ExecutionEngine.h`, `llvm/ExecutionEngine/RTDyldMemoryManager.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ExecutionEngine/ExecutionEngine.h`, `llvm/ExecutionEngine/RTDyldMemoryManager.h`。

### Lines 23-32
```cpp
// This is a helper class that the MCJIT execution engine uses for linking
// functions across modules that it owns.  It aggregates the memory manager
// that is passed in to the MCJIT constructor and defers most functionality
// to that object.
class LinkingSymbolResolver : public LegacyJITSymbolResolver {
public:
  LinkingSymbolResolver(MCJIT &Parent,
                        std::shared_ptr<LegacyJITSymbolResolver> Resolver)
      : ParentEngine(Parent), ClientResolver(std::move(Resolver)) {}

```
- **EN**: Introduces declarations for `that`, `LinkingSymbolResolver`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `that`, `LinkingSymbolResolver` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 33-45
```cpp
  JITSymbol findSymbol(const std::string &Name) override;

  // MCJIT doesn't support logical dylibs.
  JITSymbol findSymbolInLogicalDylib(const std::string &Name) override {
    return nullptr;
  }

private:
  MCJIT &ParentEngine;
  std::shared_ptr<LegacyJITSymbolResolver> ClientResolver;
  void anchor() override;
};

```
- **EN**: Implements logic around `findSymbol`, `findSymbolInLogicalDylib`, `anchor`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `findSymbol`, `findSymbolInLogicalDylib`, `anchor` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 46-63
```cpp
// About Module states: added->loaded->finalized.
//
// The purpose of the "added" state is having modules in standby. (added=known
// but not compiled). The idea is that you can add a module to provide function
// definitions but if nothing in that module is referenced by a module in which
// a function is executed (note the wording here because it's not exactly the
// ideal case) then the module never gets compiled. This is sort of lazy
// compilation.
//
// The purpose of the "loaded" state (loaded=compiled and required sections
// copied into local memory but not yet ready for execution) is to have an
// intermediate state wherein clients can remap the addresses of sections, using
// MCJIT::mapSectionAddress, (in preparation for later copying to a new location
// or an external process) before relocations and page permissions are applied.
//
// It might not be obvious at first glance, but the "remote-mcjit" case in the
// lli tool does this.  In that case, the intermediate action is taken by the
// RemoteMemoryManager in response to the notifyObjectLoaded function being
```
- **EN**: Documents the next declaration group or explains a subsystem-specific rule.
- **CN**: 为后续声明分组提供说明，或解释某个子系统的专用规则。

### Lines 64-72
```cpp
// called.

class MCJIT : public ExecutionEngine {
  MCJIT(std::unique_ptr<Module> M, std::unique_ptr<TargetMachine> tm,
        std::shared_ptr<MCJITMemoryManager> MemMgr,
        std::shared_ptr<LegacyJITSymbolResolver> Resolver);

  typedef llvm::SmallPtrSet<Module *, 4> ModulePtrSet;

```
- **EN**: Introduces declarations for `MCJIT`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `MCJIT` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 73-81
```cpp
  class OwningModuleContainer {
  public:
    OwningModuleContainer() = default;
    ~OwningModuleContainer() {
      freeModulePtrSet(AddedModules);
      freeModulePtrSet(LoadedModules);
      freeModulePtrSet(FinalizedModules);
    }

```
- **EN**: Introduces declarations for `OwningModuleContainer`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `OwningModuleContainer` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 82-90
```cpp
    ModulePtrSet::iterator begin_added() { return AddedModules.begin(); }
    ModulePtrSet::iterator end_added() { return AddedModules.end(); }
    iterator_range<ModulePtrSet::iterator> added() {
      return make_range(begin_added(), end_added());
    }

    ModulePtrSet::iterator begin_loaded() { return LoadedModules.begin(); }
    ModulePtrSet::iterator end_loaded() { return LoadedModules.end(); }

```
- **EN**: Implements logic around `begin_added`, `end_added`, `added`, `make_range`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `begin_added`, `end_added`, `added`, `make_range`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 91-102
```cpp
    ModulePtrSet::iterator begin_finalized() { return FinalizedModules.begin(); }
    ModulePtrSet::iterator end_finalized() { return FinalizedModules.end(); }

    void addModule(std::unique_ptr<Module> M) {
      AddedModules.insert(M.release());
    }

    bool removeModule(Module *M) {
      return AddedModules.erase(M) || LoadedModules.erase(M) ||
             FinalizedModules.erase(M);
    }

```
- **EN**: Implements logic around `begin_finalized`, `end_finalized`, `addModule`, `insert`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `begin_finalized`, `end_finalized`, `addModule`, `insert`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 103-112
```cpp
    bool hasModuleBeenAddedButNotLoaded(Module *M) {
      return AddedModules.contains(M);
    }

    bool hasModuleBeenLoaded(Module *M) {
      // If the module is in either the "loaded" or "finalized" sections it
      // has been loaded.
      return LoadedModules.contains(M) || FinalizedModules.contains(M);
    }

```
- **EN**: Implements logic around `hasModuleBeenAddedButNotLoaded`, `contains`, `hasModuleBeenLoaded`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `hasModuleBeenAddedButNotLoaded`, `contains`, `hasModuleBeenLoaded` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 113-121
```cpp
    bool hasModuleBeenFinalized(Module *M) {
      return FinalizedModules.contains(M);
    }

    bool ownsModule(Module* M) {
      return AddedModules.contains(M) || LoadedModules.contains(M) ||
             FinalizedModules.contains(M);
    }

```
- **EN**: Implements logic around `hasModuleBeenFinalized`, `contains`, `ownsModule`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `hasModuleBeenFinalized`, `contains`, `ownsModule` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 122-131
```cpp
    void markModuleAsLoaded(Module *M) {
      // This checks against logic errors in the MCJIT implementation.
      // This function should never be called with either a Module that MCJIT
      // does not own or a Module that has already been loaded and/or finalized.
      assert(AddedModules.count(M) &&
             "markModuleAsLoaded: Module not found in AddedModules");

      // Remove the module from the "Added" set.
      AddedModules.erase(M);

```
- **EN**: Implements logic around `markModuleAsLoaded`, `assert`, `erase`.
- **CN**: 围绕 `markModuleAsLoaded`, `assert`, `erase` 实现具体逻辑。

### Lines 132-143
```cpp
      // Add the Module to the "Loaded" set.
      LoadedModules.insert(M);
    }

    void markModuleAsFinalized(Module *M) {
      // This checks against logic errors in the MCJIT implementation.
      // This function should never be called with either a Module that MCJIT
      // does not own, a Module that has not been loaded or a Module that has
      // already been finalized.
      assert(LoadedModules.count(M) &&
             "markModuleAsFinalized: Module not found in LoadedModules");

```
- **EN**: Implements logic around `insert`, `markModuleAsFinalized`, `assert`.
- **CN**: 围绕 `insert`, `markModuleAsFinalized`, `assert` 实现具体逻辑。

### Lines 144-156
```cpp
      // Remove the module from the "Loaded" section of the list.
      LoadedModules.erase(M);

      // Add the Module to the "Finalized" section of the list by inserting it
      // before the 'end' iterator.
      FinalizedModules.insert(M);
    }

    void markAllLoadedModulesAsFinalized() {
      FinalizedModules.insert_range(LoadedModules);
      LoadedModules.clear();
    }

```
- **EN**: Implements logic around `erase`, `insert`, `markAllLoadedModulesAsFinalized`, `insert_range`, and 1 more symbols.
- **CN**: 围绕 `erase`, `insert`, `markAllLoadedModulesAsFinalized`, `insert_range`, and 1 more symbols 实现具体逻辑。

### Lines 157-169
```cpp
  private:
    ModulePtrSet AddedModules;
    ModulePtrSet LoadedModules;
    ModulePtrSet FinalizedModules;

    void freeModulePtrSet(ModulePtrSet& MPS) {
      // Go through the module set and delete everything.
      for (Module *M : MPS)
        delete M;
      MPS.clear();
    }
  };

```
- **EN**: Implements logic around `freeModulePtrSet`, `clear`.
- **CN**: 围绕 `freeModulePtrSet`, `clear` 实现具体逻辑。

### Lines 170-178
```cpp
  std::unique_ptr<TargetMachine> TM;
  MCContext *Ctx;
  std::shared_ptr<MCJITMemoryManager> MemMgr;
  LinkingSymbolResolver Resolver;
  RuntimeDyld Dyld;
  std::vector<JITEventListener*> EventListeners;

  OwningModuleContainer OwnedModules;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 179-187
```cpp
  SmallVector<object::OwningBinary<object::Archive>, 2> Archives;
  SmallVector<std::unique_ptr<MemoryBuffer>, 2> Buffers;

  SmallVector<std::unique_ptr<object::ObjectFile>, 2> LoadedObjects;

  // An optional ObjectCache to be notified of compiled objects and used to
  // perform lookup of pre-compiled code to avoid re-compilation.
  ObjectCache *ObjCache;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 188-196
```cpp
  Function *FindFunctionNamedInModulePtrSet(StringRef FnName,
                                            ModulePtrSet::iterator I,
                                            ModulePtrSet::iterator E);

  GlobalVariable *FindGlobalVariableNamedInModulePtrSet(StringRef Name,
                                                        bool AllowInternal,
                                                        ModulePtrSet::iterator I,
                                                        ModulePtrSet::iterator E);

```
- **EN**: Implements logic around `FindFunctionNamedInModulePtrSet`, `FindGlobalVariableNamedInModulePtrSet`.
- **CN**: 围绕 `FindFunctionNamedInModulePtrSet`, `FindGlobalVariableNamedInModulePtrSet` 实现具体逻辑。

### Lines 197-205
```cpp
  void runStaticConstructorsDestructorsInModulePtrSet(bool isDtors,
                                                      ModulePtrSet::iterator I,
                                                      ModulePtrSet::iterator E);

public:
  ~MCJIT() override;

  /// @name ExecutionEngine interface implementation
  /// @{
```
- **EN**: Implements logic around `runStaticConstructorsDestructorsInModulePtrSet`, `~MCJIT`.
- **CN**: 围绕 `runStaticConstructorsDestructorsInModulePtrSet`, `~MCJIT` 实现具体逻辑。

### Lines 206-214
```cpp
  void addModule(std::unique_ptr<Module> M) override;
  void addObjectFile(std::unique_ptr<object::ObjectFile> O) override;
  void addObjectFile(object::OwningBinary<object::ObjectFile> O) override;
  void addArchive(object::OwningBinary<object::Archive> O) override;
  bool removeModule(Module *M) override;

  /// FindFunctionNamed - Search all of the active modules to find the function that
  /// defines FnName.  This is very slow operation and shouldn't be used for
  /// general code.
```
- **EN**: Implements logic around `addModule`, `addObjectFile`, `addArchive`, `removeModule`.
- **CN**: 围绕 `addModule`, `addObjectFile`, `addArchive`, `removeModule` 实现具体逻辑。

### Lines 215-223
```cpp
  Function *FindFunctionNamed(StringRef FnName) override;

  /// FindGlobalVariableNamed - Search all of the active modules to find the
  /// global variable that defines Name.  This is very slow operation and
  /// shouldn't be used for general code.
  GlobalVariable *FindGlobalVariableNamed(StringRef Name,
                                          bool AllowInternal = false) override;

  /// Sets the object manager that MCJIT should use to avoid compilation.
```
- **EN**: Implements logic around `FindFunctionNamed`, `FindGlobalVariableNamed`.
- **CN**: 围绕 `FindFunctionNamed`, `FindGlobalVariableNamed` 实现具体逻辑。

### Lines 224-232
```cpp
  void setObjectCache(ObjectCache *manager) override;

  void setProcessAllSections(bool ProcessAllSections) override {
    Dyld.setProcessAllSections(ProcessAllSections);
  }

  void generateCodeForModule(Module *M) override;

  /// finalizeObject - ensure the module is fully processed and is usable.
```
- **EN**: Implements logic around `setObjectCache`, `setProcessAllSections`, `generateCodeForModule`.
- **CN**: 围绕 `setObjectCache`, `setProcessAllSections`, `generateCodeForModule` 实现具体逻辑。

### Lines 233-244
```cpp
  ///
  /// It is the user-level function for completing the process of making the
  /// object usable for execution. It should be called after sections within an
  /// object have been relocated using mapSectionAddress.  When this method is
  /// called the MCJIT execution engine will reapply relocations for a loaded
  /// object.
  /// Is it OK to finalize a set of modules, add modules and finalize again.
  // FIXME: Do we really need both of these?
  void finalizeObject() override;
  virtual void finalizeModule(Module *);
  void finalizeLoadedModules();

```
- **EN**: Implements logic around `finalizeObject`, `finalizeModule`, `finalizeLoadedModules`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `finalizeObject`, `finalizeModule`, `finalizeLoadedModules` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 245-255
```cpp
  /// runStaticConstructorsDestructors - This method is used to execute all of
  /// the static constructors or destructors for a program.
  ///
  /// \param isDtors - Run the destructors instead of constructors.
  void runStaticConstructorsDestructors(bool isDtors) override;

  void *getPointerToFunction(Function *F) override;

  GenericValue runFunction(Function *F,
                           ArrayRef<GenericValue> ArgValues) override;

```
- **EN**: Implements logic around `runStaticConstructorsDestructors`, `getPointerToFunction`, `runFunction`; this block executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `runStaticConstructorsDestructors`, `getPointerToFunction`, `runFunction` 实现具体逻辑；这一段执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 256-266
```cpp
  /// getPointerToNamedFunction - This method returns the address of the
  /// specified function by using the dlsym function call.  As such it is only
  /// useful for resolving library symbols, not code generated symbols.
  ///
  /// If AbortOnFailure is false and no function with the given name is
  /// found, this function silently returns a null pointer. Otherwise,
  /// it prints a message to stderr and aborts.
  ///
  void *getPointerToNamedFunction(StringRef Name,
                                  bool AbortOnFailure = true) override;

```
- **EN**: Implements logic around `getPointerToNamedFunction`.
- **CN**: 围绕 `getPointerToNamedFunction` 实现具体逻辑。

### Lines 267-277
```cpp
  /// mapSectionAddress - map a section to its target address space value.
  /// Map the address of a JIT section as returned from the memory manager
  /// to the address in the target process as the running code will see it.
  /// This is the address which will be used for relocation resolution.
  void mapSectionAddress(const void *LocalAddress,
                         uint64_t TargetAddress) override {
    Dyld.mapSectionAddress(LocalAddress, TargetAddress);
  }
  void RegisterJITEventListener(JITEventListener *L) override;
  void UnregisterJITEventListener(JITEventListener *L) override;

```
- **EN**: Implements logic around `mapSectionAddress`, `RegisterJITEventListener`, `UnregisterJITEventListener`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `mapSectionAddress`, `RegisterJITEventListener`, `UnregisterJITEventListener` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 278-286
```cpp
  // If successful, these function will implicitly finalize all loaded objects.
  // To get a function address within MCJIT without causing a finalize, use
  // getSymbolAddress.
  uint64_t getGlobalValueAddress(const std::string &Name) override;
  uint64_t getFunctionAddress(const std::string &Name) override;

  TargetMachine *getTargetMachine() override { return TM.get(); }

  /// @}
```
- **EN**: Implements logic around `getGlobalValueAddress`, `getFunctionAddress`, `getTargetMachine`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getGlobalValueAddress`, `getFunctionAddress`, `getTargetMachine` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 287-299
```cpp
  /// @name (Private) Registration Interfaces
  /// @{

  static void Register() {
    MCJITCtor = createJIT;
  }

  static ExecutionEngine *
  createJIT(std::unique_ptr<Module> M, std::string *ErrorStr,
            std::shared_ptr<MCJITMemoryManager> MemMgr,
            std::shared_ptr<LegacyJITSymbolResolver> Resolver,
            std::unique_ptr<TargetMachine> TM);

```
- **EN**: Implements logic around `Register`, `createJIT`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `Register`, `createJIT` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 300-315
```cpp
  // @}

  // Takes a mangled name and returns the corresponding JITSymbol (if a
  // definition of that mangled name has been added to the JIT).
  JITSymbol findSymbol(const std::string &Name, bool CheckFunctionsOnly);

  // DEPRECATED - Please use findSymbol instead.
  //
  // This is not directly exposed via the ExecutionEngine API, but it is
  // used by the LinkingMemoryManager.
  //
  // getSymbolAddress takes an unmangled name and returns the corresponding
  // JITSymbol if a definition of the name has been added to the JIT.
  uint64_t getSymbolAddress(const std::string &Name,
                            bool CheckFunctionsOnly);

```
- **EN**: Implements logic around `findSymbol`, `getSymbolAddress`.
- **CN**: 围绕 `findSymbol`, `getSymbolAddress` 实现具体逻辑。

### Lines 316-327
```cpp
protected:
  /// emitObject -- Generate a JITed object in memory from the specified module
  /// Currently, MCJIT only supports a single module and the module passed to
  /// this function call is expected to be the contained module.  The module
  /// is passed as a parameter here to prepare for multiple module support in
  /// the future.
  std::unique_ptr<MemoryBuffer> emitObject(Module *M);

  void notifyObjectLoaded(const object::ObjectFile &Obj,
                          const RuntimeDyld::LoadedObjectInfo &L);
  void notifyFreeingObject(const object::ObjectFile &Obj);

```
- **EN**: Implements logic around `emitObject`, `notifyObjectLoaded`, `notifyFreeingObject`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `emitObject`, `notifyObjectLoaded`, `notifyFreeingObject` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 328-334
```cpp
  JITSymbol findExistingSymbol(const std::string &Name);
  Module *findModuleForSymbol(const std::string &Name, bool CheckFunctionsOnly);
};

} // end llvm namespace

#endif // LLVM_LIB_EXECUTIONENGINE_MCJIT_MCJIT_H
```
- **EN**: Implements logic around `findExistingSymbol`, `findModuleForSymbol`.
- **CN**: 围绕 `findExistingSymbol`, `findModuleForSymbol` 实现具体逻辑。

## Key Concepts / 关键概念

- **MCJIT compilation / MCJIT 编译**:
  - **EN**: Bridges LLVM code generation, object emission, and RuntimeDyld-based loading
  - **CN**: 连接 LLVM 代码生成、目标输出与基于 RuntimeDyld 的装载
- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ExecutionEngine/ExecutionEngine.h`, `llvm/ExecutionEngine/RTDyldMemoryManager.h`, `llvm/ExecutionEngine/RuntimeDyld.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine
