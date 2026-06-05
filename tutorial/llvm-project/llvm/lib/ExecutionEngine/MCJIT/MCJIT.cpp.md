# MCJIT.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/MCJIT/MCJIT.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements MC-based Just-in-Time Compiler.
  - **CN**: 实现传统的 MCJIT 执行引擎，通过 LLVM 代码生成与 RuntimeDyld 把模块编译成本地代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```cpp
//===-- MCJIT.cpp - MC-based Just-in-Time Compiler ------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "MCJIT.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ExecutionEngine/GenericValue.h"
#include "llvm/ExecutionEngine/JITEventListener.h"
#include "llvm/ExecutionEngine/MCJIT.h"
#include "llvm/ExecutionEngine/ObjectCache.h"
#include "llvm/ExecutionEngine/SectionMemoryManager.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/DerivedTypes.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/LegacyPassManager.h"
#include "llvm/IR/Mangler.h"
#include "llvm/IR/Module.h"
#include "llvm/Object/Archive.h"
#include "llvm/Object/ObjectFile.h"
#include "llvm/Support/DynamicLibrary.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `MCJIT.h`, `llvm/ADT/STLExtras.h`, `llvm/ExecutionEngine/GenericValue.h`, `llvm/ExecutionEngine/JITEventListener.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCJIT.h`, `llvm/ADT/STLExtras.h`, `llvm/ExecutionEngine/GenericValue.h`, `llvm/ExecutionEngine/JITEventListener.h`。

### Lines 25-37
```cpp
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Support/SmallVectorMemoryBuffer.h"
#include <mutex>

using namespace llvm;

namespace {

static struct RegisterJIT {
  RegisterJIT() { MCJIT::Register(); }
} JITRegistrator;

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/Support/ErrorHandling.h`, `llvm/Support/MemoryBuffer.h`, `llvm/Support/SmallVectorMemoryBuffer.h`, `mutex`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/Support/ErrorHandling.h`, `llvm/Support/MemoryBuffer.h`, `llvm/Support/SmallVectorMemoryBuffer.h`, `mutex`。

### Lines 38-52
```cpp
}

extern "C" void LLVMLinkInMCJIT() {
}

ExecutionEngine *
MCJIT::createJIT(std::unique_ptr<Module> M, std::string *ErrorStr,
                 std::shared_ptr<MCJITMemoryManager> MemMgr,
                 std::shared_ptr<LegacyJITSymbolResolver> Resolver,
                 std::unique_ptr<TargetMachine> TM) {
  // Try to register the program as a source of symbols to resolve against.
  //
  // FIXME: Don't do this here.
  sys::DynamicLibrary::LoadLibraryPermanently(nullptr, nullptr);

```
- **EN**: Implements logic around `createJIT`, `LoadLibraryPermanently`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `createJIT`, `LoadLibraryPermanently` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 53-64
```cpp
  if (!MemMgr || !Resolver) {
    auto RTDyldMM = std::make_shared<SectionMemoryManager>();
    if (!MemMgr)
      MemMgr = RTDyldMM;
    if (!Resolver)
      Resolver = RTDyldMM;
  }

  return new MCJIT(std::move(M), std::move(TM), std::move(MemMgr),
                   std::move(Resolver));
}

```
- **EN**: Implements logic around `make_shared<SectionMemoryManager>`, `MCJIT`, `move`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_shared<SectionMemoryManager>`, `MCJIT`, `move` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 65-84
```cpp
MCJIT::MCJIT(std::unique_ptr<Module> M, std::unique_ptr<TargetMachine> TM,
             std::shared_ptr<MCJITMemoryManager> MemMgr,
             std::shared_ptr<LegacyJITSymbolResolver> Resolver)
    : ExecutionEngine(TM->createDataLayout(), std::move(M)), TM(std::move(TM)),
      Ctx(nullptr), MemMgr(std::move(MemMgr)),
      Resolver(*this, std::move(Resolver)), Dyld(*this->MemMgr, this->Resolver),
      ObjCache(nullptr) {
  // FIXME: We are managing our modules, so we do not want the base class
  // ExecutionEngine to manage them as well. To avoid double destruction
  // of the first (and only) module added in ExecutionEngine constructor
  // we remove it from EE and will destruct it ourselves.
  //
  // It may make sense to move our module manager (based on SmallStPtr) back
  // into EE if the JIT and Interpreter can live with it.
  // If so, additional functions: addModule, removeModule, FindFunctionNamed,
  // runStaticConstructorsDestructors could be moved back to EE as well.
  //
  std::unique_ptr<Module> First = std::move(Modules[0]);
  Modules.clear();

```
- **EN**: Implements logic around `MCJIT`, `ExecutionEngine`, `Ctx`, `Resolver`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `MCJIT`, `ExecutionEngine`, `Ctx`, `Resolver`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为。

### Lines 85-96
```cpp
  if (First->getDataLayout().isDefault())
    First->setDataLayout(getDataLayout());

  OwnedModules.addModule(std::move(First));
  RegisterJITEventListener(JITEventListener::createGDBRegistrationListener());
}

MCJIT::~MCJIT() {
  std::lock_guard<sys::Mutex> locked(lock);

  Dyld.deregisterEHFrames();

```
- **EN**: Implements logic around `setDataLayout`, `addModule`, `RegisterJITEventListener`, `~MCJIT`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `setDataLayout`, `addModule`, `RegisterJITEventListener`, `~MCJIT`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 97-109
```cpp
  for (auto &Obj : LoadedObjects)
    if (Obj)
      notifyFreeingObject(*Obj);

  Archives.clear();
}

void MCJIT::addModule(std::unique_ptr<Module> M) {
  std::lock_guard<sys::Mutex> locked(lock);

  if (M->getDataLayout().isDefault())
    M->setDataLayout(getDataLayout());

```
- **EN**: Implements logic around `notifyFreeingObject`, `clear`, `addModule`, `locked`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `notifyFreeingObject`, `clear`, `addModule`, `locked`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 110-122
```cpp
  OwnedModules.addModule(std::move(M));
}

bool MCJIT::removeModule(Module *M) {
  std::lock_guard<sys::Mutex> locked(lock);
  return OwnedModules.removeModule(M);
}

void MCJIT::addObjectFile(std::unique_ptr<object::ObjectFile> Obj) {
  std::unique_ptr<RuntimeDyld::LoadedObjectInfo> L = Dyld.loadObject(*Obj);
  if (Dyld.hasError())
    report_fatal_error(Dyld.getErrorString());

```
- **EN**: Implements logic around `addModule`, `removeModule`, `locked`, `addObjectFile`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `addModule`, `removeModule`, `locked`, `addObjectFile`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 123-135
```cpp
  notifyObjectLoaded(*Obj, *L);

  LoadedObjects.push_back(std::move(Obj));
}

void MCJIT::addObjectFile(object::OwningBinary<object::ObjectFile> Obj) {
  std::unique_ptr<object::ObjectFile> ObjFile;
  std::unique_ptr<MemoryBuffer> MemBuf;
  std::tie(ObjFile, MemBuf) = Obj.takeBinary();
  addObjectFile(std::move(ObjFile));
  Buffers.push_back(std::move(MemBuf));
}

```
- **EN**: Implements logic around `notifyObjectLoaded`, `push_back`, `addObjectFile`, `tie`.
- **CN**: 围绕 `notifyObjectLoaded`, `push_back`, `addObjectFile`, `tie` 实现具体逻辑。

### Lines 136-147
```cpp
void MCJIT::addArchive(object::OwningBinary<object::Archive> A) {
  Archives.push_back(std::move(A));
}

void MCJIT::setObjectCache(ObjectCache* NewCache) {
  std::lock_guard<sys::Mutex> locked(lock);
  ObjCache = NewCache;
}

std::unique_ptr<MemoryBuffer> MCJIT::emitObject(Module *M) {
  assert(M && "Can not emit a null module");

```
- **EN**: Implements logic around `addArchive`, `push_back`, `setObjectCache`, `locked`, and 2 more symbols.
- **CN**: 围绕 `addArchive`, `push_back`, `setObjectCache`, `locked`, and 2 more symbols 实现具体逻辑。

### Lines 148-159
```cpp
  std::lock_guard<sys::Mutex> locked(lock);

  // Materialize all globals in the module if they have not been
  // materialized already.
  cantFail(M->materializeAll());

  // This must be a module which has already been added but not loaded to this
  // MCJIT instance, since these conditions are tested by our caller,
  // generateCodeForModule.

  legacy::PassManager PM;

```
- **EN**: Implements logic around `locked`, `cantFail`.
- **CN**: 围绕 `locked`, `cantFail` 实现具体逻辑。

### Lines 160-172
```cpp
  // The RuntimeDyld will take ownership of this shortly
  SmallVector<char, 4096> ObjBufferSV;
  raw_svector_ostream ObjStream(ObjBufferSV);

  // Turn the machine code intermediate representation into bytes in memory
  // that may be executed.
  if (TM->addPassesToEmitMC(PM, Ctx, ObjStream, !getVerifyModules()))
    report_fatal_error("Target does not support MC emission!");

  // Initialize passes.
  PM.run(*M);
  // Flush the output buffer to get the generated code into memory

```
- **EN**: Implements logic around `ObjStream`, `report_fatal_error`, `run`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `ObjStream`, `report_fatal_error`, `run` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 173-184
```cpp
  auto CompiledObjBuffer = std::make_unique<SmallVectorMemoryBuffer>(
      std::move(ObjBufferSV), /*RequiresNullTerminator=*/false);

  // If we have an object cache, tell it about the new object.
  // Note that we're using the compiled image, not the loaded image (as below).
  if (ObjCache) {
    // MemoryBuffer is a thin wrapper around the actual memory, so it's OK
    // to create a temporary object here and delete it after the call.
    MemoryBufferRef MB = CompiledObjBuffer->getMemBufferRef();
    ObjCache->notifyObjectCompiled(M, MB);
  }

```
- **EN**: Implements logic around `make_unique<SmallVectorMemoryBuffer>`, `move`, `getMemBufferRef`, `notifyObjectCompiled`.
- **CN**: 围绕 `make_unique<SmallVectorMemoryBuffer>`, `move`, `getMemBufferRef`, `notifyObjectCompiled` 实现具体逻辑。

### Lines 185-199
```cpp
  return CompiledObjBuffer;
}

void MCJIT::generateCodeForModule(Module *M) {
  // Get a thread lock to make sure we aren't trying to load multiple times
  std::lock_guard<sys::Mutex> locked(lock);

  // This must be a module which has already been added to this MCJIT instance.
  assert(OwnedModules.ownsModule(M) &&
         "MCJIT::generateCodeForModule: Unknown module.");

  // Re-compilation is not supported
  if (OwnedModules.hasModuleBeenLoaded(M))
    return;

```
- **EN**: Implements logic around `generateCodeForModule`, `locked`, `assert`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `generateCodeForModule`, `locked`, `assert` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 200-212
```cpp
  std::unique_ptr<MemoryBuffer> ObjectToLoad;
  // Try to load the pre-compiled object from cache if possible
  if (ObjCache)
    ObjectToLoad = ObjCache->getObject(M);

  assert(M->getDataLayout() == getDataLayout() && "DataLayout Mismatch");

  // If the cache did not contain a suitable object, compile the object
  if (!ObjectToLoad) {
    ObjectToLoad = emitObject(M);
    assert(ObjectToLoad && "Compilation did not produce an object.");
  }

```
- **EN**: Implements logic around `getObject`, `assert`, `emitObject`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `getObject`, `assert`, `emitObject` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 213-225
```cpp
  // Load the object into the dynamic linker.
  // MCJIT now owns the ObjectImage pointer (via its LoadedObjects list).
  Expected<std::unique_ptr<object::ObjectFile>> LoadedObject =
    object::ObjectFile::createObjectFile(ObjectToLoad->getMemBufferRef());
  if (!LoadedObject) {
    std::string Buf;
    raw_string_ostream OS(Buf);
    logAllUnhandledErrors(LoadedObject.takeError(), OS);
    report_fatal_error(Twine(Buf));
  }
  std::unique_ptr<RuntimeDyld::LoadedObjectInfo> L =
    Dyld.loadObject(*LoadedObject.get());

```
- **EN**: Implements logic around `createObjectFile`, `OS`, `logAllUnhandledErrors`, `report_fatal_error`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `createObjectFile`, `OS`, `logAllUnhandledErrors`, `report_fatal_error`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 226-239
```cpp
  if (Dyld.hasError())
    report_fatal_error(Dyld.getErrorString());

  notifyObjectLoaded(*LoadedObject.get(), *L);

  Buffers.push_back(std::move(ObjectToLoad));
  LoadedObjects.push_back(std::move(*LoadedObject));

  OwnedModules.markModuleAsLoaded(M);
}

void MCJIT::finalizeLoadedModules() {
  std::lock_guard<sys::Mutex> locked(lock);

```
- **EN**: Implements logic around `report_fatal_error`, `notifyObjectLoaded`, `push_back`, `markModuleAsLoaded`, and 2 more symbols.
- **CN**: 围绕 `report_fatal_error`, `notifyObjectLoaded`, `push_back`, `markModuleAsLoaded`, and 2 more symbols 实现具体逻辑。

### Lines 240-251
```cpp
  // Resolve any outstanding relocations.
  Dyld.resolveRelocations();

  // Check for Dyld error.
  if (Dyld.hasError())
    ErrMsg = Dyld.getErrorString().str();

  OwnedModules.markAllLoadedModulesAsFinalized();

  // Register EH frame data for any module we own which has been loaded
  Dyld.registerEHFrames();

```
- **EN**: Implements logic around `resolveRelocations`, `getErrorString`, `markAllLoadedModulesAsFinalized`, `registerEHFrames`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `resolveRelocations`, `getErrorString`, `markAllLoadedModulesAsFinalized`, `registerEHFrames` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 252-263
```cpp
  // Set page permissions.
  MemMgr->finalizeMemory();
}

// FIXME: Rename this.
void MCJIT::finalizeObject() {
  std::lock_guard<sys::Mutex> locked(lock);

  // Generate code for module is going to move objects out of the 'added' list,
  // so we need to copy that out before using it:
  SmallVector<Module *, 16> ModsToAdd(OwnedModules.added());

```
- **EN**: Implements logic around `finalizeMemory`, `finalizeObject`, `locked`, `ModsToAdd`.
- **CN**: 围绕 `finalizeMemory`, `finalizeObject`, `locked`, `ModsToAdd` 实现具体逻辑。

### Lines 264-275
```cpp
  for (auto *M : ModsToAdd)
    generateCodeForModule(M);

  finalizeLoadedModules();
}

void MCJIT::finalizeModule(Module *M) {
  std::lock_guard<sys::Mutex> locked(lock);

  // This must be a module which has already been added to this MCJIT instance.
  assert(OwnedModules.ownsModule(M) && "MCJIT::finalizeModule: Unknown module.");

```
- **EN**: Implements logic around `generateCodeForModule`, `finalizeLoadedModules`, `finalizeModule`, `locked`, and 1 more symbols.
- **CN**: 围绕 `generateCodeForModule`, `finalizeLoadedModules`, `finalizeModule`, `locked`, and 1 more symbols 实现具体逻辑。

### Lines 276-288
```cpp
  // If the module hasn't been compiled, just do that.
  if (!OwnedModules.hasModuleBeenLoaded(M))
    generateCodeForModule(M);

  finalizeLoadedModules();
}

JITSymbol MCJIT::findExistingSymbol(const std::string &Name) {
  if (void *Addr = getPointerToGlobalIfAvailable(Name))
    return JITSymbol(static_cast<uint64_t>(
                         reinterpret_cast<uintptr_t>(Addr)),
                     JITSymbolFlags::Exported);

```
- **EN**: Implements logic around `generateCodeForModule`, `finalizeLoadedModules`, `findExistingSymbol`, `JITSymbol`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `generateCodeForModule`, `finalizeLoadedModules`, `findExistingSymbol`, `JITSymbol`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 289-312
```cpp
  return Dyld.getSymbol(Name);
}

Module *MCJIT::findModuleForSymbol(const std::string &Name,
                                   bool CheckFunctionsOnly) {
  StringRef DemangledName = Name;
  if (DemangledName[0] == getDataLayout().getGlobalPrefix())
    DemangledName = DemangledName.substr(1);

  std::lock_guard<sys::Mutex> locked(lock);

  // If it hasn't already been generated, see if it's in one of our modules.
  for (ModulePtrSet::iterator I = OwnedModules.begin_added(),
                              E = OwnedModules.end_added();
       I != E; ++I) {
    Module *M = *I;
    Function *F = M->getFunction(DemangledName);
    if (F && !F->isDeclaration())
      return M;
    if (!CheckFunctionsOnly) {
      GlobalVariable *G = M->getGlobalVariable(DemangledName);
      if (G && !G->isDeclaration())
        return M;
      // FIXME: Do we need to worry about global aliases?
```
- **EN**: Implements logic around `getSymbol`, `findModuleForSymbol`, `substr`, `locked`, and 3 more symbols; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `getSymbol`, `findModuleForSymbol`, `substr`, `locked`, and 3 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 313-335
```cpp
    }
  }
  // We didn't find the symbol in any of our modules.
  return nullptr;
}

uint64_t MCJIT::getSymbolAddress(const std::string &Name,
                                 bool CheckFunctionsOnly) {
  std::string MangledName;
  {
    raw_string_ostream MangledNameStream(MangledName);
    Mangler::getNameWithPrefix(MangledNameStream, Name, getDataLayout());
  }
  if (auto Sym = findSymbol(MangledName, CheckFunctionsOnly)) {
    if (auto AddrOrErr = Sym.getAddress())
      return *AddrOrErr;
    else
      report_fatal_error(AddrOrErr.takeError());
  } else if (auto Err = Sym.takeError())
    report_fatal_error(Sym.takeError());
  return 0;
}

```
- **EN**: Implements logic around `getSymbolAddress`, `MangledNameStream`, `getNameWithPrefix`, `report_fatal_error`; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `getSymbolAddress`, `MangledNameStream`, `getNameWithPrefix`, `report_fatal_error` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 336-359
```cpp
JITSymbol MCJIT::findSymbol(const std::string &Name,
                            bool CheckFunctionsOnly) {
  std::lock_guard<sys::Mutex> locked(lock);

  // First, check to see if we already have this symbol.
  if (auto Sym = findExistingSymbol(Name))
    return Sym;

  for (object::OwningBinary<object::Archive> &OB : Archives) {
    object::Archive *A = OB.getBinary();
    // Look for our symbols in each Archive
    auto OptionalChildOrErr = A->findSym(Name);
    if (!OptionalChildOrErr)
      report_fatal_error(OptionalChildOrErr.takeError());
    auto &OptionalChild = *OptionalChildOrErr;
    if (OptionalChild) {
      // FIXME: Support nested archives?
      Expected<std::unique_ptr<object::Binary>> ChildBinOrErr =
          OptionalChild->getAsBinary();
      if (!ChildBinOrErr) {
        // TODO: Actually report errors helpfully.
        consumeError(ChildBinOrErr.takeError());
        continue;
      }
```
- **EN**: Implements logic around `findSymbol`, `locked`, `getBinary`, `findSym`, and 3 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `findSymbol`, `locked`, `getBinary`, `findSym`, and 3 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 360-372
```cpp
      std::unique_ptr<object::Binary> &ChildBin = ChildBinOrErr.get();
      if (ChildBin->isObject()) {
        std::unique_ptr<object::ObjectFile> OF(
            static_cast<object::ObjectFile *>(ChildBin.release()));
        // This causes the object file to be loaded.
        addObjectFile(std::move(OF));
        // The address should be here now.
        if (auto Sym = findExistingSymbol(Name))
          return Sym;
      }
    }
  }

```
- **EN**: Implements logic around `get`, `OF`, `release`, `addObjectFile`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `get`, `OF`, `release`, `addObjectFile` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 373-389
```cpp
  // If it hasn't already been generated, see if it's in one of our modules.
  Module *M = findModuleForSymbol(Name, CheckFunctionsOnly);
  if (M) {
    generateCodeForModule(M);

    // Check the RuntimeDyld table again, it should be there now.
    return findExistingSymbol(Name);
  }

  // If a LazyFunctionCreator is installed, use it to get/create the function.
  // FIXME: Should we instead have a LazySymbolCreator callback?
  if (LazyFunctionCreator) {
    auto Addr = static_cast<uint64_t>(
                  reinterpret_cast<uintptr_t>(LazyFunctionCreator(Name)));
    return JITSymbol(Addr, JITSymbolFlags::Exported);
  }

```
- **EN**: Implements logic around `findModuleForSymbol`, `generateCodeForModule`, `findExistingSymbol`, `static_cast<uint64_t>`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `findModuleForSymbol`, `generateCodeForModule`, `findExistingSymbol`, `static_cast<uint64_t>`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 390-408
```cpp
  return nullptr;
}

uint64_t MCJIT::getGlobalValueAddress(const std::string &Name) {
  std::lock_guard<sys::Mutex> locked(lock);
  uint64_t Result = getSymbolAddress(Name, false);
  if (Result != 0)
    finalizeLoadedModules();
  return Result;
}

uint64_t MCJIT::getFunctionAddress(const std::string &Name) {
  std::lock_guard<sys::Mutex> locked(lock);
  uint64_t Result = getSymbolAddress(Name, true);
  if (Result != 0)
    finalizeLoadedModules();
  return Result;
}

```
- **EN**: Implements logic around `getGlobalValueAddress`, `locked`, `getSymbolAddress`, `finalizeLoadedModules`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getGlobalValueAddress`, `locked`, `getSymbolAddress`, `finalizeLoadedModules`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 409-423
```cpp
// Deprecated.  Use getFunctionAddress instead.
void *MCJIT::getPointerToFunction(Function *F) {
  std::lock_guard<sys::Mutex> locked(lock);

  Mangler Mang;
  SmallString<128> Name;
  TM->getNameWithPrefix(Name, F, Mang);

  if (F->isDeclaration() || F->hasAvailableExternallyLinkage()) {
    bool AbortOnFailure = !F->hasExternalWeakLinkage();
    void *Addr = getPointerToNamedFunction(Name, AbortOnFailure);
    updateGlobalMapping(F, Addr);
    return Addr;
  }

```
- **EN**: Implements logic around `getPointerToFunction`, `locked`, `getNameWithPrefix`, `hasExternalWeakLinkage`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getPointerToFunction`, `locked`, `getNameWithPrefix`, `hasExternalWeakLinkage`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 424-437
```cpp
  Module *M = F->getParent();
  bool HasBeenAddedButNotLoaded = OwnedModules.hasModuleBeenAddedButNotLoaded(M);

  // Make sure the relevant module has been compiled and loaded.
  if (HasBeenAddedButNotLoaded)
    generateCodeForModule(M);
  else if (!OwnedModules.hasModuleBeenLoaded(M)) {
    // If this function doesn't belong to one of our modules, we're done.
    // FIXME: Asking for the pointer to a function that hasn't been registered,
    //        and isn't a declaration (which is handled above) should probably
    //        be an assertion.
    return nullptr;
  }

```
- **EN**: Implements logic around `getParent`, `hasModuleBeenAddedButNotLoaded`, `generateCodeForModule`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getParent`, `hasModuleBeenAddedButNotLoaded`, `generateCodeForModule` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 438-451
```cpp
  // FIXME: Should the Dyld be retaining module information? Probably not.
  //
  // This is the accessor for the target address, so make sure to check the
  // load address of the symbol, not the local address.
  return (void*)Dyld.getSymbol(Name).getAddress();
}

void MCJIT::runStaticConstructorsDestructorsInModulePtrSet(
    bool isDtors, ModulePtrSet::iterator I, ModulePtrSet::iterator E) {
  for (; I != E; ++I) {
    ExecutionEngine::runStaticConstructorsDestructors(**I, isDtors);
  }
}

```
- **EN**: Implements logic around `runStaticConstructorsDestructorsInModulePtrSet`, `runStaticConstructorsDestructors`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `runStaticConstructorsDestructorsInModulePtrSet`, `runStaticConstructorsDestructors` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 452-472
```cpp
void MCJIT::runStaticConstructorsDestructors(bool isDtors) {
  // Execute global ctors/dtors for each module in the program.
  runStaticConstructorsDestructorsInModulePtrSet(
      isDtors, OwnedModules.begin_added(), OwnedModules.end_added());
  runStaticConstructorsDestructorsInModulePtrSet(
      isDtors, OwnedModules.begin_loaded(), OwnedModules.end_loaded());
  runStaticConstructorsDestructorsInModulePtrSet(
      isDtors, OwnedModules.begin_finalized(), OwnedModules.end_finalized());
}

Function *MCJIT::FindFunctionNamedInModulePtrSet(StringRef FnName,
                                                 ModulePtrSet::iterator I,
                                                 ModulePtrSet::iterator E) {
  for (; I != E; ++I) {
    Function *F = (*I)->getFunction(FnName);
    if (F && !F->isDeclaration())
      return F;
  }
  return nullptr;
}

```
- **EN**: Implements logic around `runStaticConstructorsDestructors`, `runStaticConstructorsDestructorsInModulePtrSet`, `begin_added`, `begin_loaded`, and 3 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `runStaticConstructorsDestructors`, `runStaticConstructorsDestructorsInModulePtrSet`, `begin_added`, `begin_loaded`, and 3 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 473-484
```cpp
GlobalVariable *MCJIT::FindGlobalVariableNamedInModulePtrSet(StringRef Name,
                                                             bool AllowInternal,
                                                             ModulePtrSet::iterator I,
                                                             ModulePtrSet::iterator E) {
  for (; I != E; ++I) {
    GlobalVariable *GV = (*I)->getGlobalVariable(Name, AllowInternal);
    if (GV && !GV->isDeclaration())
      return GV;
  }
  return nullptr;
}

```
- **EN**: Implements logic around `FindGlobalVariableNamedInModulePtrSet`, `getGlobalVariable`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `FindGlobalVariableNamedInModulePtrSet`, `getGlobalVariable` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 485-497
```cpp

Function *MCJIT::FindFunctionNamed(StringRef FnName) {
  Function *F = FindFunctionNamedInModulePtrSet(
      FnName, OwnedModules.begin_added(), OwnedModules.end_added());
  if (!F)
    F = FindFunctionNamedInModulePtrSet(FnName, OwnedModules.begin_loaded(),
                                        OwnedModules.end_loaded());
  if (!F)
    F = FindFunctionNamedInModulePtrSet(FnName, OwnedModules.begin_finalized(),
                                        OwnedModules.end_finalized());
  return F;
}

```
- **EN**: Implements logic around `FindFunctionNamed`, `FindFunctionNamedInModulePtrSet`, `begin_added`, `end_loaded`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `FindFunctionNamed`, `FindFunctionNamedInModulePtrSet`, `begin_added`, `end_loaded`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 498-509
```cpp
GlobalVariable *MCJIT::FindGlobalVariableNamed(StringRef Name, bool AllowInternal) {
  GlobalVariable *GV = FindGlobalVariableNamedInModulePtrSet(
      Name, AllowInternal, OwnedModules.begin_added(), OwnedModules.end_added());
  if (!GV)
    GV = FindGlobalVariableNamedInModulePtrSet(Name, AllowInternal, OwnedModules.begin_loaded(),
                                        OwnedModules.end_loaded());
  if (!GV)
    GV = FindGlobalVariableNamedInModulePtrSet(Name, AllowInternal, OwnedModules.begin_finalized(),
                                        OwnedModules.end_finalized());
  return GV;
}

```
- **EN**: Implements logic around `FindGlobalVariableNamed`, `FindGlobalVariableNamedInModulePtrSet`, `begin_added`, `end_loaded`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `FindGlobalVariableNamed`, `FindGlobalVariableNamedInModulePtrSet`, `begin_added`, `end_loaded`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 510-524
```cpp
GenericValue MCJIT::runFunction(Function *F, ArrayRef<GenericValue> ArgValues) {
  assert(F && "Function *F was null at entry to run()");

  void *FPtr = getPointerToFunction(F);
  finalizeModule(F->getParent());
  assert(FPtr && "Pointer to fn's code was null after getPointerToFunction");
  FunctionType *FTy = F->getFunctionType();
  Type *RetTy = FTy->getReturnType();

  assert((FTy->getNumParams() == ArgValues.size() ||
          (FTy->isVarArg() && FTy->getNumParams() <= ArgValues.size())) &&
         "Wrong number of arguments passed into function!");
  assert(FTy->getNumParams() == ArgValues.size() &&
         "This doesn't support passing arguments through varargs (yet)!");

```
- **EN**: Implements logic around `runFunction`, `assert`, `getPointerToFunction`, `finalizeModule`, and 4 more symbols; this block executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `runFunction`, `assert`, `getPointerToFunction`, `finalizeModule`, and 4 more symbols 实现具体逻辑；这一段执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 525-548
```cpp
  // Handle some common cases first.  These cases correspond to common `main'
  // prototypes.
  if (RetTy->isIntegerTy(32) || RetTy->isVoidTy()) {
    switch (ArgValues.size()) {
    case 3:
      if (FTy->getParamType(0)->isIntegerTy(32) &&
          FTy->getParamType(1)->isPointerTy() &&
          FTy->getParamType(2)->isPointerTy()) {
        int (*PF)(int, char **, const char **) =
          (int(*)(int, char **, const char **))(intptr_t)FPtr;

        // Call the function.
        GenericValue rv;
        rv.IntVal = APInt(32, PF(ArgValues[0].IntVal.getZExtValue(),
                                 (char **)GVTOP(ArgValues[1]),
                                 (const char **)GVTOP(ArgValues[2])));
        return rv;
      }
      break;
    case 2:
      if (FTy->getParamType(0)->isIntegerTy(32) &&
          FTy->getParamType(1)->isPointerTy()) {
        int (*PF)(int, char **) = (int(*)(int, char **))(intptr_t)FPtr;

```
- **EN**: Implements logic around `getParamType`, `int`, `APInt`, `GVTOP`; this block executes or prepares runtime behavior for LLVM IR/JIT code; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getParamType`, `int`, `APInt`, `GVTOP` 实现具体逻辑；这一段执行或准备 LLVM IR/JIT 代码的运行时行为，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 549-567
```cpp
        // Call the function.
        GenericValue rv;
        rv.IntVal = APInt(32, PF(ArgValues[0].IntVal.getZExtValue(),
                                 (char **)GVTOP(ArgValues[1])));
        return rv;
      }
      break;
    case 1:
      if (FTy->getNumParams() == 1 &&
          FTy->getParamType(0)->isIntegerTy(32)) {
        GenericValue rv;
        int (*PF)(int) = (int(*)(int))(intptr_t)FPtr;
        rv.IntVal = APInt(32, PF(ArgValues[0].IntVal.getZExtValue()));
        return rv;
      }
      break;
    }
  }

```
- **EN**: Implements logic around `APInt`, `GVTOP`, `getParamType`, `int`; this block executes or prepares runtime behavior for LLVM IR/JIT code; returns subsystem-specific computed results.
- **CN**: 围绕 `APInt`, `GVTOP`, `getParamType`, `int` 实现具体逻辑；这一段执行或准备 LLVM IR/JIT 代码的运行时行为，返回子系统相关的计算结果。

### Lines 568-591
```cpp
  // Handle cases where no arguments are passed first.
  if (ArgValues.empty()) {
    GenericValue rv;
    switch (RetTy->getTypeID()) {
    default: llvm_unreachable("Unknown return type for function call!");
    case Type::IntegerTyID: {
      unsigned BitWidth = cast<IntegerType>(RetTy)->getBitWidth();
      if (BitWidth == 1)
        rv.IntVal = APInt(BitWidth, ((bool(*)())(intptr_t)FPtr)());
      else if (BitWidth <= 8)
        rv.IntVal = APInt(BitWidth, ((char(*)())(intptr_t)FPtr)());
      else if (BitWidth <= 16)
        rv.IntVal = APInt(BitWidth, ((short(*)())(intptr_t)FPtr)());
      else if (BitWidth <= 32)
        rv.IntVal = APInt(BitWidth, ((int(*)())(intptr_t)FPtr)());
      else if (BitWidth <= 64)
        rv.IntVal = APInt(BitWidth, ((int64_t(*)())(intptr_t)FPtr)());
      else
        llvm_unreachable("Integer types > 64 bits not supported");
      return rv;
    }
    case Type::VoidTyID:
      rv.IntVal = APInt(32, ((int (*)())(intptr_t)FPtr)(), true);
      return rv;
```
- **EN**: Implements logic around `llvm_unreachable`, `cast<IntegerType>`, `APInt`; this block executes or prepares runtime behavior for LLVM IR/JIT code; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `llvm_unreachable`, `cast<IntegerType>`, `APInt` 实现具体逻辑；这一段执行或准备 LLVM IR/JIT 代码的运行时行为，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 592-606
```cpp
    case Type::FloatTyID:
      rv.FloatVal = ((float(*)())(intptr_t)FPtr)();
      return rv;
    case Type::DoubleTyID:
      rv.DoubleVal = ((double(*)())(intptr_t)FPtr)();
      return rv;
    case Type::X86_FP80TyID:
    case Type::FP128TyID:
    case Type::PPC_FP128TyID:
      llvm_unreachable("long double not supported yet");
    case Type::PointerTyID:
      return PTOGV(((void*(*)())(intptr_t)FPtr)());
    }
  }

```
- **EN**: Implements logic around `float`, `double`, `llvm_unreachable`, `PTOGV`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `float`, `double`, `llvm_unreachable`, `PTOGV` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 607-622
```cpp
  report_fatal_error("MCJIT::runFunction does not support full-featured "
                     "argument passing. Please use "
                     "ExecutionEngine::getFunctionAddress and cast the result "
                     "to the desired function pointer type.");
}

void *MCJIT::getPointerToNamedFunction(StringRef Name, bool AbortOnFailure) {
  if (!isSymbolSearchingDisabled()) {
    if (auto Sym = Resolver.findSymbol(std::string(Name))) {
      if (auto AddrOrErr = Sym.getAddress())
        return reinterpret_cast<void*>(
                 static_cast<uintptr_t>(*AddrOrErr));
    } else if (auto Err = Sym.takeError())
      report_fatal_error(std::move(Err));
  }

```
- **EN**: Implements logic around `report_fatal_error`, `getPointerToNamedFunction`, `static_cast<uintptr_t>`; this block handles relocation, fixup, or symbol-resolution work; executes or prepares runtime behavior for LLVM IR/JIT code; returns subsystem-specific computed results.
- **CN**: 围绕 `report_fatal_error`, `getPointerToNamedFunction`, `static_cast<uintptr_t>` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，执行或准备 LLVM IR/JIT 代码的运行时行为，返回子系统相关的计算结果。

### Lines 623-634
```cpp
  /// If a LazyFunctionCreator is installed, use it to get/create the function.
  if (LazyFunctionCreator)
    if (void *RP = LazyFunctionCreator(std::string(Name)))
      return RP;

  if (AbortOnFailure) {
    report_fatal_error("Program used external function '"+Name+
                       "' which could not be resolved!");
  }
  return nullptr;
}

```
- **EN**: Implements logic around `report_fatal_error`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `report_fatal_error` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 635-652
```cpp
void MCJIT::RegisterJITEventListener(JITEventListener *L) {
  if (!L)
    return;
  std::lock_guard<sys::Mutex> locked(lock);
  EventListeners.push_back(L);
}

void MCJIT::UnregisterJITEventListener(JITEventListener *L) {
  if (!L)
    return;
  std::lock_guard<sys::Mutex> locked(lock);
  auto I = find(reverse(EventListeners), L);
  if (I != EventListeners.rend()) {
    std::swap(*I, EventListeners.back());
    EventListeners.pop_back();
  }
}

```
- **EN**: Implements logic around `RegisterJITEventListener`, `locked`, `push_back`, `UnregisterJITEventListener`, and 3 more symbols.
- **CN**: 围绕 `RegisterJITEventListener`, `locked`, `push_back`, `UnregisterJITEventListener`, and 3 more symbols 实现具体逻辑。

### Lines 653-670
```cpp
void MCJIT::notifyObjectLoaded(const object::ObjectFile &Obj,
                               const RuntimeDyld::LoadedObjectInfo &L) {
  uint64_t Key =
      static_cast<uint64_t>(reinterpret_cast<uintptr_t>(Obj.getData().data()));
  std::lock_guard<sys::Mutex> locked(lock);
  MemMgr->notifyObjectLoaded(this, Obj);
  for (JITEventListener *EL : EventListeners)
    EL->notifyObjectLoaded(Key, Obj, L);
}

void MCJIT::notifyFreeingObject(const object::ObjectFile &Obj) {
  uint64_t Key =
      static_cast<uint64_t>(reinterpret_cast<uintptr_t>(Obj.getData().data()));
  std::lock_guard<sys::Mutex> locked(lock);
  for (JITEventListener *L : EventListeners)
    L->notifyFreeingObject(Key);
}

```
- **EN**: Implements logic around `notifyObjectLoaded`, `static_cast<uint64_t>`, `locked`, `notifyFreeingObject`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `notifyObjectLoaded`, `static_cast<uint64_t>`, `locked`, `notifyFreeingObject` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 671-681
```cpp
JITSymbol
LinkingSymbolResolver::findSymbol(const std::string &Name) {
  auto Result = ParentEngine.findSymbol(Name, false);
  if (Result)
    return Result;
  if (ParentEngine.isSymbolSearchingDisabled())
    return nullptr;
  return ClientResolver->findSymbol(Name);
}

void LinkingSymbolResolver::anchor() {}
```
- **EN**: Implements logic around `findSymbol`, `anchor`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `findSymbol`, `anchor` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **MCJIT compilation / MCJIT 编译**:
  - **EN**: Bridges LLVM code generation, object emission, and RuntimeDyld-based loading
  - **CN**: 连接 LLVM 代码生成、目标输出与基于 RuntimeDyld 的装载
- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCJIT.h`, `llvm/ADT/STLExtras.h`, `llvm/ExecutionEngine/GenericValue.h`, `llvm/ExecutionEngine/JITEventListener.h`, `llvm/ExecutionEngine/MCJIT.h`, `llvm/ExecutionEngine/ObjectCache.h`, `llvm/ExecutionEngine/SectionMemoryManager.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Function.h`, `llvm/IR/LegacyPassManager.h`, `llvm/IR/Mangler.h` ... (+8 more)
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object, IR, Support
