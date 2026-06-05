# OnDiskCAS.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/CAS/OnDiskCAS.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements content-addressable storage, on-disk caches, and related schema support.
  - **CN**: 实现内容寻址存储、磁盘缓存以及相关 schema 支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===----------------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-21
```cpp

#include "BuiltinCAS.h"
#include "OnDiskCommon.h"
#include "llvm/ADT/ScopeExit.h"
#include "llvm/CAS/BuiltinCASContext.h"
#include "llvm/CAS/BuiltinObjectHasher.h"
#include "llvm/CAS/OnDiskCASLogger.h"
#include "llvm/CAS/OnDiskGraphDB.h"
#include "llvm/CAS/UnifiedOnDiskCache.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/Error.h"
#include "llvm/Support/IOSandbox.h"
#include "llvm/Support/Path.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `BuiltinCAS.h`, `OnDiskCommon.h`, `llvm/ADT/ScopeExit.h`, `llvm/CAS/BuiltinCASContext.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `BuiltinCAS.h`, `OnDiskCommon.h`, `llvm/ADT/ScopeExit.h`, `llvm/CAS/BuiltinCASContext.h`。

### Lines 22-28
```cpp
using namespace llvm;
using namespace llvm::cas;
using namespace llvm::cas::builtin;

namespace {

class OnDiskCAS : public BuiltinCAS {
```
- **EN**: Introduces declarations for `llvm`, `llvm::cas`, `llvm::cas::builtin`, `OnDiskCAS`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `llvm`, `llvm::cas`, `llvm::cas::builtin`, `OnDiskCAS` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 29-35
```cpp
public:
  Expected<ObjectRef> storeImpl(ArrayRef<uint8_t> ComputedHash,
                                ArrayRef<ObjectRef> Refs,
                                ArrayRef<char> Data) final;

  Expected<std::optional<ObjectHandle>> loadIfExists(ObjectRef Ref) final;

```
- **EN**: Implements logic around `storeImpl`, `loadIfExists`; this block works with hashed storage or cache state.
- **CN**: 围绕 `storeImpl`, `loadIfExists` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 36-43
```cpp
  CASID getID(ObjectRef Ref) const final;

  std::optional<ObjectRef> getReference(const CASID &ID) const final;

  Expected<bool> isMaterialized(ObjectRef Ref) const final;

  ArrayRef<char> getDataConst(ObjectHandle Node) const final;

```
- **EN**: Implements logic around `getID`, `getReference`, `isMaterialized`, `getDataConst`; this block works with hashed storage or cache state.
- **CN**: 围绕 `getID`, `getReference`, `isMaterialized`, `getDataConst` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 44-50
```cpp
  Expected<ObjectRef> storeFromFile(StringRef Path) final;

  Error exportDataToFile(ObjectHandle Node, StringRef Path) const final;

  void print(raw_ostream &OS) const final;
  Error validate(bool CheckHash) const final;

```
- **EN**: Implements logic around `storeFromFile`, `exportDataToFile`, `print`, `validate`; this block propagates recoverable errors through LLVM error utilities; emits or serializes data to an external representation; works with hashed storage or cache state.
- **CN**: 围绕 `storeFromFile`, `exportDataToFile`, `print`, `validate` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并把数据输出或序列化为外部表示，并处理基于哈希的存储或缓存状态。

### Lines 51-60
```cpp
  static Expected<std::unique_ptr<OnDiskCAS>> open(StringRef Path);

  OnDiskCAS(std::shared_ptr<ondisk::UnifiedOnDiskCache> UniDB)
      : UnifiedDB(std::move(UniDB)), DB(&UnifiedDB->getGraphDB()) {}

private:
  ObjectHandle convertHandle(ondisk::ObjectHandle Node) const {
    return makeObjectHandle(Node.getOpaqueData());
  }

```
- **EN**: Implements logic around `open`, `OnDiskCAS`, `UnifiedDB`, `convertHandle`, and 1 more symbols; this block works with hashed storage or cache state.
- **CN**: 围绕 `open`, `OnDiskCAS`, `UnifiedDB`, `convertHandle`, and 1 more symbols 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 61-68
```cpp
  ondisk::ObjectHandle convertHandle(ObjectHandle Node) const {
    return ondisk::ObjectHandle(Node.getInternalRef(*this));
  }

  ObjectRef convertRef(ondisk::ObjectID Ref) const {
    return makeObjectRef(Ref.getOpaqueData());
  }

```
- **EN**: Implements logic around `convertHandle`, `ObjectHandle`, `convertRef`, `makeObjectRef`; this block works with hashed storage or cache state.
- **CN**: 围绕 `convertHandle`, `ObjectHandle`, `convertRef`, `makeObjectRef` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 69-77
```cpp
  ondisk::ObjectID convertRef(ObjectRef Ref) const {
    return ondisk::ObjectID::fromOpaqueData(Ref.getInternalRef(*this));
  }

  size_t getNumRefs(ObjectHandle Node) const final {
    auto RefsRange = DB->getObjectRefs(convertHandle(Node));
    return llvm::size(RefsRange);
  }

```
- **EN**: Implements logic around `convertRef`, `fromOpaqueData`, `getNumRefs`, `getObjectRefs`, and 1 more symbols; this block works with hashed storage or cache state.
- **CN**: 围绕 `convertRef`, `fromOpaqueData`, `getNumRefs`, `getObjectRefs`, and 1 more symbols 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 78-85
```cpp
  ObjectRef readRef(ObjectHandle Node, size_t I) const final {
    auto RefsRange = DB->getObjectRefs(convertHandle(Node));
    return convertRef(RefsRange.begin()[I]);
  }

  Error forEachRef(ObjectHandle Node,
                   function_ref<Error(ObjectRef)> Callback) const final;

```
- **EN**: Implements logic around `readRef`, `getObjectRefs`, `convertRef`, `forEachRef`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `readRef`, `getObjectRefs`, `convertRef`, `forEachRef`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 86-92
```cpp
  Error setSizeLimit(std::optional<uint64_t> SizeLimit) final;
  Expected<std::optional<uint64_t>> getStorageSize() const final;
  Error pruneStorageData() final;

  OnDiskCAS(std::unique_ptr<ondisk::OnDiskGraphDB> GraphDB)
      : OwnedDB(std::move(GraphDB)), DB(OwnedDB.get()) {}

```
- **EN**: Implements logic around `setSizeLimit`, `getStorageSize`, `pruneStorageData`, `OnDiskCAS`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `setSizeLimit`, `getStorageSize`, `pruneStorageData`, `OnDiskCAS`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 93-99
```cpp
  std::unique_ptr<ondisk::OnDiskGraphDB> OwnedDB;
  std::shared_ptr<ondisk::UnifiedOnDiskCache> UnifiedDB;
  ondisk::OnDiskGraphDB *DB;
};

} // end anonymous namespace

```
- **EN**: Contains supporting implementation details for the surrounding LLVM library component.
- **CN**: 包含周边 LLVM 库组件所需的辅助实现细节。

### Lines 100-107
```cpp
void OnDiskCAS::print(raw_ostream &OS) const { DB->print(OS); }
Error OnDiskCAS::validate(bool CheckHash) const {
  if (auto E = DB->validate(CheckHash, builtin::hashingFunc))
    return E;

  return Error::success();
}

```
- **EN**: Implements logic around `print`, `validate`, `success`; this block propagates recoverable errors through LLVM error utilities; emits or serializes data to an external representation; works with hashed storage or cache state.
- **CN**: 围绕 `print`, `validate`, `success` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并把数据输出或序列化为外部表示，并处理基于哈希的存储或缓存状态。

### Lines 108-120
```cpp
CASID OnDiskCAS::getID(ObjectRef Ref) const {
  ArrayRef<uint8_t> Hash = DB->getDigest(convertRef(Ref));
  return CASID::create(&getContext(), toStringRef(Hash));
}

std::optional<ObjectRef> OnDiskCAS::getReference(const CASID &ID) const {
  std::optional<ondisk::ObjectID> ObjID =
      DB->getExistingReference(ID.getHash());
  if (!ObjID)
    return std::nullopt;
  return convertRef(*ObjID);
}

```
- **EN**: Implements logic around `getID`, `getDigest`, `create`, `getReference`, and 2 more symbols; this block works with hashed storage or cache state.
- **CN**: 围绕 `getID`, `getDigest`, `create`, `getReference`, and 2 more symbols 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 121-128
```cpp
Expected<bool> OnDiskCAS::isMaterialized(ObjectRef ExternalRef) const {
  return DB->isMaterialized(convertRef(ExternalRef));
}

ArrayRef<char> OnDiskCAS::getDataConst(ObjectHandle Node) const {
  return DB->getObjectData(convertHandle(Node));
}

```
- **EN**: Implements logic around `isMaterialized`, `getDataConst`, `getObjectData`; this block works with hashed storage or cache state.
- **CN**: 围绕 `isMaterialized`, `getDataConst`, `getObjectData` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 129-139
```cpp
Expected<std::optional<ObjectHandle>>
OnDiskCAS::loadIfExists(ObjectRef ExternalRef) {
  Expected<std::optional<ondisk::ObjectHandle>> ObjHnd =
      DB->load(convertRef(ExternalRef));
  if (!ObjHnd)
    return ObjHnd.takeError();
  if (!*ObjHnd)
    return std::nullopt;
  return convertHandle(**ObjHnd);
}

```
- **EN**: Implements logic around `loadIfExists`, `load`, `takeError`, `convertHandle`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `loadIfExists`, `load`, `takeError`, `convertHandle` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 140-148
```cpp
Expected<ObjectRef> OnDiskCAS::storeImpl(ArrayRef<uint8_t> ComputedHash,
                                         ArrayRef<ObjectRef> Refs,
                                         ArrayRef<char> Data) {
  SmallVector<ondisk::ObjectID, 64> IDs;
  IDs.reserve(Refs.size());
  for (ObjectRef Ref : Refs) {
    IDs.push_back(convertRef(Ref));
  }

```
- **EN**: Implements logic around `storeImpl`, `reserve`, `push_back`; this block works with hashed storage or cache state.
- **CN**: 围绕 `storeImpl`, `reserve`, `push_back` 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 149-156
```cpp
  auto StoredID = DB->getReference(ComputedHash);
  if (LLVM_UNLIKELY(!StoredID))
    return StoredID.takeError();
  if (Error E = DB->store(*StoredID, IDs, Data))
    return std::move(E);
  return convertRef(*StoredID);
}

```
- **EN**: Implements logic around `getReference`, `takeError`, `store`, `move`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `getReference`, `takeError`, `store`, `move`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 157-168
```cpp
Expected<ObjectRef> OnDiskCAS::storeFromFile(StringRef Path) {
  auto Hash = BuiltinObjectHasher<HasherT>::hashFile(Path);
  if (LLVM_UNLIKELY(!Hash))
    return Hash.takeError();
  auto StoredID = DB->getReference(*Hash);
  if (LLVM_UNLIKELY(!StoredID))
    return StoredID.takeError();
  if (Error E = DB->storeFile(*StoredID, Path))
    return E;
  return convertRef(*StoredID);
}

```
- **EN**: Implements logic around `storeFromFile`, `hashFile`, `takeError`, `getReference`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `storeFromFile`, `hashFile`, `takeError`, `getReference`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 169-176
```cpp
Error OnDiskCAS::exportDataToFile(ObjectHandle Node, StringRef Path) const {
  auto FBData = DB->getInternalFileBackedObjectData(convertHandle(Node));
  if (!FBData.FileInfo.has_value())
    return BuiltinCAS::exportDataToFile(Node, Path);

  // Optimized version using the underlying database file.
  assert(FBData.FileInfo.has_value());

```
- **EN**: Implements logic around `exportDataToFile`, `getInternalFileBackedObjectData`, `has_value`, `assert`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `exportDataToFile`, `getInternalFileBackedObjectData`, `has_value`, `assert` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 177-185
```cpp
  auto BypassSandbox = sys::sandbox::scopedDisable();

  ondisk::UniqueTempFile UniqueTmp;
  auto ExpectedPath = UniqueTmp.createAndCopyFrom(sys::path::parent_path(Path),
                                                  FBData.FileInfo->FilePath);
  if (!ExpectedPath)
    return ExpectedPath.takeError();
  StringRef TmpPath = *ExpectedPath;

```
- **EN**: Implements logic around `scopedDisable`, `createAndCopyFrom`, `takeError`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `scopedDisable`, `createAndCopyFrom`, `takeError` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 186-199
```cpp
  if (FBData.FileInfo->IsFileNulTerminated) {
    // Remove the nul terminator.
    int FD;
    if (std::error_code EC =
            sys::fs::openFileForWrite(TmpPath, FD, sys::fs::CD_OpenExisting))
      return createFileError(TmpPath, EC);
    auto CloseFile = scope_exit([&FD] {
      sys::fs::file_t File = sys::fs::convertFDToNativeFile(FD);
      sys::fs::closeFile(File);
    });
    if (std::error_code EC = sys::fs::resize_file(FD, FBData.Data.size()))
      return createFileError(TmpPath, EC);
  }

```
- **EN**: Implements logic around `openFileForWrite`, `createFileError`, `scope_exit`, `convertFDToNativeFile`, and 2 more symbols; this block works with hashed storage or cache state.
- **CN**: 围绕 `openFileForWrite`, `createFileError`, `scope_exit`, `convertFDToNativeFile`, and 2 more symbols 实现具体逻辑；该代码块处理基于哈希的存储或缓存状态。

### Lines 200-213
```cpp
  if (Error E = UniqueTmp.renameTo(Path))
    return E;

  return Error::success();
}

Error OnDiskCAS::forEachRef(ObjectHandle Node,
                            function_ref<Error(ObjectRef)> Callback) const {
  auto RefsRange = DB->getObjectRefs(convertHandle(Node));
  for (ondisk::ObjectID Ref : RefsRange) {
    if (Error E = Callback(convertRef(Ref)))
      return E;
  }
  return Error::success();
```
- **EN**: Implements logic around `renameTo`, `success`, `forEachRef`, `function_ref`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `renameTo`, `success`, `forEachRef`, `function_ref`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 214-220
```cpp
}

Error OnDiskCAS::setSizeLimit(std::optional<uint64_t> SizeLimit) {
  UnifiedDB->setSizeLimit(SizeLimit);
  return Error::success();
}

```
- **EN**: Implements logic around `setSizeLimit`, `success`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `setSizeLimit`, `success` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 221-234
```cpp
Expected<std::optional<uint64_t>> OnDiskCAS::getStorageSize() const {
  return UnifiedDB->getStorageSize();
}

Error OnDiskCAS::pruneStorageData() { return UnifiedDB->collectGarbage(); }

Expected<std::unique_ptr<OnDiskCAS>> OnDiskCAS::open(StringRef AbsPath) {
  std::shared_ptr<ondisk::OnDiskCASLogger> Logger;
#ifndef _WIN32
  if (Error E =
          ondisk::OnDiskCASLogger::openIfEnabled(AbsPath).moveInto(Logger))
    return std::move(E);
#endif

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 235-243
```cpp
  Expected<std::unique_ptr<ondisk::OnDiskGraphDB>> DB =
      ondisk::OnDiskGraphDB::open(AbsPath, BuiltinCASContext::getHashName(),
                                  sizeof(HashType), /*UpstreamDB=*/nullptr,
                                  std::move(Logger));
  if (!DB)
    return DB.takeError();
  return std::unique_ptr<OnDiskCAS>(new OnDiskCAS(std::move(*DB)));
}

```
- **EN**: Implements logic around `open`, `move`, `takeError`, `unique_ptr`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `open`, `move`, `takeError`, `unique_ptr` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

### Lines 244-251
```cpp
bool cas::isOnDiskCASEnabled() {
#if LLVM_ENABLE_ONDISK_CAS
  return true;
#else
  return false;
#endif
}

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 252-259
```cpp
Expected<std::unique_ptr<ObjectStore>> cas::createOnDiskCAS(const Twine &Path) {
#if LLVM_ENABLE_ONDISK_CAS
  // FIXME: An absolute path isn't really good enough. Should open a directory
  // and use openat() for files underneath.
  SmallString<256> AbsPath;
  Path.toVector(AbsPath);
  sys::fs::make_absolute(AbsPath);

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 260-270
```cpp
  return OnDiskCAS::open(AbsPath);
#else
  return createStringError(inconvertibleErrorCode(), "OnDiskCAS is disabled");
#endif /* LLVM_ENABLE_ONDISK_CAS */
}

std::unique_ptr<ObjectStore>
cas::builtin::createObjectStoreFromUnifiedOnDiskCache(
    std::shared_ptr<ondisk::UnifiedOnDiskCache> UniDB) {
  return std::make_unique<OnDiskCAS>(std::move(UniDB));
}
```
- **EN**: Implements logic around `open`, `createStringError`, `createObjectStoreFromUnifiedOnDiskCache`, `make_unique`; this block propagates recoverable errors through LLVM error utilities; works with hashed storage or cache state.
- **CN**: 围绕 `open`, `createStringError`, `createObjectStoreFromUnifiedOnDiskCache`, `make_unique` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并处理基于哈希的存储或缓存状态。

## Key Concepts / 关键概念

- **Content-addressable storage / 内容寻址存储**:
  - **EN**: Represents objects by hash and manages in-memory or on-disk persistence.
  - **CN**: 通过哈希表示对象，并管理内存或磁盘持久化。
- **Structured error handling / 结构化错误处理**:
  - **EN**: Uses `Expected`, `Error`, or related helpers to make failures explicit.
  - **CN**: 使用 `Expected`、`Error` 或相关辅助工具显式表示失败。
- **LLVM container usage / LLVM 容器使用**:
  - **EN**: Relies on LLVM ADT containers for performance-conscious in-memory data management.
  - **CN**: 依赖 LLVM ADT 容器来进行注重性能的内存数据管理。
- **Streaming output / 流式输出**:
  - **EN**: Writes diagnostics, serialized data, or textual representations through LLVM stream APIs.
  - **CN**: 通过 LLVM 流式 API 输出诊断、序列化数据或文本表示。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `BuiltinCAS.h`, `OnDiskCommon.h`, `llvm/ADT/ScopeExit.h`, `llvm/CAS/BuiltinCASContext.h`, `llvm/CAS/BuiltinObjectHasher.h`, `llvm/CAS/OnDiskCASLogger.h`, `llvm/CAS/OnDiskGraphDB.h`, `llvm/CAS/UnifiedOnDiskCache.h`, `llvm/Support/Compiler.h`, `llvm/Support/Error.h` ... (+2 more)
- **Subsystem categories / 子系统类别**: content-addressable storage interfaces / 内容寻址存储接口 (5), support-library helpers / Support 库辅助功能 (4), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
