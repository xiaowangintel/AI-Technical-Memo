# AppleObjCRuntimeV2.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/LanguageRuntime/ObjC/AppleObjCRuntime/AppleObjCRuntimeV2.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Declares the LLDB interfaces, data structures, and helper APIs associated with `AppleObjCRuntimeV2`.
  - **CN**: 声明与 `AppleObjCRuntimeV2` 相关的 LLDB 接口、数据结构以及辅助 API。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- AppleObjCRuntimeV2.h ------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLDB_SOURCE_PLUGINS_LANGUAGERUNTIME_OBJC_APPLEOBJCRUNTIME_APPLEOBJCRUNTIMEV2_H
#define LLDB_SOURCE_PLUGINS_LANGUAGERUNTIME_OBJC_APPLEOBJCRUNTIME_APPLEOBJCRUNTIMEV2_H

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 12-21
```cpp
#include <map>
#include <memory>
#include <mutex>
#include <optional>

#include "AppleObjCRuntime.h"
#include "lldb/lldb-private.h"

#include "Plugins/LanguageRuntime/ObjC/ObjCLanguageRuntime.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `map`, `memory`, `mutex`, `optional`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `map`, `memory`, `mutex`, `optional`。

### Lines 22-32
```cpp
#include "llvm/ADT/BitVector.h"
#include "llvm/ADT/SmallSet.h"

class RemoteNXMapTable;

namespace lldb_private {

class AppleObjCRuntimeV2 : public AppleObjCRuntime {
public:
  ~AppleObjCRuntimeV2() override = default;

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/ADT/BitVector.h`, `llvm/ADT/SmallSet.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/ADT/BitVector.h`, `llvm/ADT/SmallSet.h`。

### Lines 33-43
```cpp
  static void Initialize();

  static void Terminate();

  static lldb_private::LanguageRuntime *
  CreateInstance(Process *process, lldb::LanguageType language);

  static llvm::StringRef GetPluginNameStatic() { return "apple-objc-v2"; }

  LanguageRuntime *GetPreferredLanguageRuntime(ValueObject &in_value) override;

```
- **EN**: Implements logic around `Initialize`, `Terminate`, `CreateInstance`, `GetPluginNameStatic`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `Initialize`, `Terminate`, `CreateInstance`, `GetPluginNameStatic`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 44-53
```cpp
  static char ID;

  bool isA(const void *ClassID) const override {
    return ClassID == &ID || AppleObjCRuntime::isA(ClassID);
  }

  static bool classof(const LanguageRuntime *runtime) {
    return runtime->isA(&ID);
  }

```
- **EN**: Implements logic around `isA`, `classof`.
- **CN**: 围绕 `isA`, `classof` 实现具体逻辑。

### Lines 54-64
```cpp
  bool GetDynamicTypeAndAddress(ValueObject &in_value,
                                lldb::DynamicValueType use_dynamic,
                                TypeAndOrName &class_type_or_name,
                                Address &address, Value::ValueType &value_type,
                                llvm::ArrayRef<uint8_t> &local_buffer) override;

  llvm::Expected<std::unique_ptr<UtilityFunction>>
  CreateObjectChecker(std::string name, ExecutionContext &exe_ctx) override;

  llvm::StringRef GetPluginName() override { return GetPluginNameStatic(); }

```
- **EN**: Implements logic around `GetDynamicTypeAndAddress`, `CreateObjectChecker`, `GetPluginName`; this block propagates recoverable errors, status objects, or diagnostics; maps executable state back to modules, symbols, sections, or addresses; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetDynamicTypeAndAddress`, `CreateObjectChecker`, `GetPluginName` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并将可执行状态映射回模块、符号、节区或地址，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 65-75
```cpp
  ObjCRuntimeVersions GetRuntimeVersion() const override {
    return ObjCRuntimeVersions::eAppleObjC_V2;
  }

  size_t GetByteOffsetForIvar(CompilerType &parent_ast_type,
                              const char *ivar_name) override;

  void UpdateISAToDescriptorMapIfNeeded() override;

  ClassDescriptorSP GetClassDescriptor(ValueObject &valobj) override;

```
- **EN**: Implements logic around `GetRuntimeVersion`, `GetByteOffsetForIvar`, `UpdateISAToDescriptorMapIfNeeded`, `GetClassDescriptor`; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetRuntimeVersion`, `GetByteOffsetForIvar`, `UpdateISAToDescriptorMapIfNeeded`, `GetClassDescriptor` 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 76-85
```cpp
  ClassDescriptorSP GetClassDescriptorFromISA(ObjCISA isa) override;

  DeclVendor *GetDeclVendor() override;

  lldb::addr_t LookupRuntimeSymbol(ConstString name) override;

  EncodingToTypeSP GetEncodingToType() override;

  bool IsTaggedPointer(lldb::addr_t ptr) override;

```
- **EN**: Declares APIs around `GetClassDescriptorFromISA`, `GetDeclVendor`, `LookupRuntimeSymbol`, `GetEncodingToType`, and 1 more symbols.
- **CN**: 声明与 `GetClassDescriptorFromISA`, `GetDeclVendor`, `LookupRuntimeSymbol`, `GetEncodingToType`, and 1 more symbols 相关的 API。

### Lines 86-96
```cpp
  TaggedPointerVendor *GetTaggedPointerVendor() override {
    return m_tagged_pointer_vendor_up.get();
  }

  lldb::addr_t GetTaggedPointerObfuscator();

  /// Returns the base address for relative method list selector strings.
  lldb::addr_t GetRelativeSelectorBaseAddr() {
    return m_relative_selector_base;
  }

```
- **EN**: Implements logic around `GetTaggedPointerVendor`, `get`, `GetTaggedPointerObfuscator`, `GetRelativeSelectorBaseAddr`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetTaggedPointerVendor`, `get`, `GetTaggedPointerObfuscator`, `GetRelativeSelectorBaseAddr` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 97-107
```cpp
  void SetRelativeSelectorBaseAddr(lldb::addr_t relative_selector_base) {
    m_relative_selector_base = relative_selector_base;
  }

  void GetValuesForGlobalCFBooleans(lldb::addr_t &cf_true,
                                    lldb::addr_t &cf_false) override;

  void ModulesDidLoad(const ModuleList &module_list) override;

  bool IsSharedCacheImageLoaded(uint16_t image_index);

```
- **EN**: Implements logic around `SetRelativeSelectorBaseAddr`, `GetValuesForGlobalCFBooleans`, `ModulesDidLoad`, `IsSharedCacheImageLoaded`.
- **CN**: 围绕 `SetRelativeSelectorBaseAddr`, `GetValuesForGlobalCFBooleans`, `ModulesDidLoad`, `IsSharedCacheImageLoaded` 实现具体逻辑。

### Lines 108-117
```cpp
  std::optional<uint64_t> GetSharedCacheImageHeaderVersion();

  StructuredData::ObjectSP GetLanguageSpecificData(SymbolContext sc) override;

protected:
  lldb::BreakpointResolverSP
  CreateExceptionResolver(const lldb::BreakpointSP &bkpt, bool catch_bp,
                          bool throw_bp) override;

private:
```
- **EN**: Declares APIs around `GetSharedCacheImageHeaderVersion`, `GetLanguageSpecificData`, `CreateExceptionResolver`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 声明与 `GetSharedCacheImageHeaderVersion`, `GetLanguageSpecificData`, `CreateExceptionResolver` 相关的 API；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 118-127
```cpp
  class HashTableSignature {
  public:
    HashTableSignature();

    bool NeedsUpdate(Process *process, AppleObjCRuntimeV2 *runtime,
                     RemoteNXMapTable &hash_table);

    void UpdateSignature(const RemoteNXMapTable &hash_table);

  protected:
```
- **EN**: Introduces declarations for `HashTableSignature`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `HashTableSignature` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 128-138
```cpp
    uint32_t m_count = 0;
    uint32_t m_num_buckets = 0;
    lldb::addr_t m_buckets_ptr = 0;
  };

  class NonPointerISACache {
  public:
    static NonPointerISACache *
    CreateInstance(AppleObjCRuntimeV2 &runtime,
                   const lldb::ModuleSP &objc_module_sp);

```
- **EN**: Introduces declarations for `NonPointerISACache`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `NonPointerISACache` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 139-152
```cpp
    ObjCLanguageRuntime::ClassDescriptorSP GetClassDescriptor(ObjCISA isa);

  private:
    NonPointerISACache(AppleObjCRuntimeV2 &runtime,
                       const lldb::ModuleSP &objc_module_sp,
                       uint64_t objc_debug_isa_class_mask,
                       uint64_t objc_debug_isa_magic_mask,
                       uint64_t objc_debug_isa_magic_value,
                       uint64_t objc_debug_indexed_isa_magic_mask,
                       uint64_t objc_debug_indexed_isa_magic_value,
                       uint64_t objc_debug_indexed_isa_index_mask,
                       uint64_t objc_debug_indexed_isa_index_shift,
                       lldb::addr_t objc_indexed_classes);

```
- **EN**: Declares APIs around `GetClassDescriptor`, `NonPointerISACache`.
- **CN**: 声明与 `GetClassDescriptor`, `NonPointerISACache` 相关的 API。

### Lines 153-167
```cpp
    bool EvaluateNonPointerISA(ObjCISA isa, ObjCISA &ret_isa);

    AppleObjCRuntimeV2 &m_runtime;
    std::map<ObjCISA, ObjCLanguageRuntime::ClassDescriptorSP> m_cache;
    lldb::ModuleWP m_objc_module_wp;
    uint64_t m_objc_debug_isa_class_mask;
    uint64_t m_objc_debug_isa_magic_mask;
    uint64_t m_objc_debug_isa_magic_value;

    uint64_t m_objc_debug_indexed_isa_magic_mask;
    uint64_t m_objc_debug_indexed_isa_magic_value;
    uint64_t m_objc_debug_indexed_isa_index_mask;
    uint64_t m_objc_debug_indexed_isa_index_shift;
    lldb::addr_t m_objc_indexed_classes;

```
- **EN**: Declares APIs around `EvaluateNonPointerISA`.
- **CN**: 声明与 `EvaluateNonPointerISA` 相关的 API。

### Lines 168-178
```cpp
    std::vector<lldb::addr_t> m_indexed_isa_cache;

    friend class AppleObjCRuntimeV2;

    NonPointerISACache(const NonPointerISACache &) = delete;
    const NonPointerISACache &operator=(const NonPointerISACache &) = delete;
  };

  class TaggedPointerVendorV2
      : public ObjCLanguageRuntime::TaggedPointerVendor {
  public:
```
- **EN**: Introduces declarations for `TaggedPointerVendorV2`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `TaggedPointerVendorV2` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 179-190
```cpp
    ~TaggedPointerVendorV2() override = default;

    static TaggedPointerVendorV2 *
    CreateInstance(AppleObjCRuntimeV2 &runtime,
                   const lldb::ModuleSP &objc_module_sp);

  protected:
    AppleObjCRuntimeV2 &m_runtime;

    TaggedPointerVendorV2(AppleObjCRuntimeV2 &runtime)
        : TaggedPointerVendor(), m_runtime(runtime) {}

```
- **EN**: Implements logic around `~TaggedPointerVendorV2`, `CreateInstance`, `TaggedPointerVendorV2`, `TaggedPointerVendor`.
- **CN**: 围绕 `~TaggedPointerVendorV2`, `CreateInstance`, `TaggedPointerVendorV2`, `TaggedPointerVendor` 实现具体逻辑。

### Lines 191-200
```cpp
  private:
    TaggedPointerVendorV2(const TaggedPointerVendorV2 &) = delete;
    const TaggedPointerVendorV2 &
    operator=(const TaggedPointerVendorV2 &) = delete;
  };

  class TaggedPointerVendorRuntimeAssisted : public TaggedPointerVendorV2 {
  public:
    bool IsPossibleTaggedPointer(lldb::addr_t ptr) override;

```
- **EN**: Introduces declarations for `TaggedPointerVendorRuntimeAssisted`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `TaggedPointerVendorRuntimeAssisted` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 201-212
```cpp
    ObjCLanguageRuntime::ClassDescriptorSP
    GetClassDescriptor(lldb::addr_t ptr) override;

  protected:
    TaggedPointerVendorRuntimeAssisted(
        AppleObjCRuntimeV2 &runtime, uint64_t objc_debug_taggedpointer_mask,
        uint32_t objc_debug_taggedpointer_slot_shift,
        uint32_t objc_debug_taggedpointer_slot_mask,
        uint32_t objc_debug_taggedpointer_payload_lshift,
        uint32_t objc_debug_taggedpointer_payload_rshift,
        lldb::addr_t objc_debug_taggedpointer_classes);

```
- **EN**: Declares APIs around `GetClassDescriptor`, `TaggedPointerVendorRuntimeAssisted`.
- **CN**: 声明与 `GetClassDescriptor`, `TaggedPointerVendorRuntimeAssisted` 相关的 API。

### Lines 213-222
```cpp
    typedef std::map<uint8_t, ObjCLanguageRuntime::ClassDescriptorSP> Cache;
    typedef Cache::iterator CacheIterator;
    Cache m_cache;
    uint64_t m_objc_debug_taggedpointer_mask;
    uint32_t m_objc_debug_taggedpointer_slot_shift;
    uint32_t m_objc_debug_taggedpointer_slot_mask;
    uint32_t m_objc_debug_taggedpointer_payload_lshift;
    uint32_t m_objc_debug_taggedpointer_payload_rshift;
    lldb::addr_t m_objc_debug_taggedpointer_classes;

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 223-233
```cpp
    friend class AppleObjCRuntimeV2::TaggedPointerVendorV2;

    TaggedPointerVendorRuntimeAssisted(
        const TaggedPointerVendorRuntimeAssisted &) = delete;
    const TaggedPointerVendorRuntimeAssisted &
    operator=(const TaggedPointerVendorRuntimeAssisted &) = delete;
  };

  class TaggedPointerVendorExtended
      : public TaggedPointerVendorRuntimeAssisted {
  public:
```
- **EN**: Introduces declarations for `TaggedPointerVendorExtended`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `TaggedPointerVendorExtended` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 234-251
```cpp
    ObjCLanguageRuntime::ClassDescriptorSP
    GetClassDescriptor(lldb::addr_t ptr) override;

  protected:
    TaggedPointerVendorExtended(
        AppleObjCRuntimeV2 &runtime, uint64_t objc_debug_taggedpointer_mask,
        uint64_t objc_debug_taggedpointer_ext_mask,
        uint32_t objc_debug_taggedpointer_slot_shift,
        uint32_t objc_debug_taggedpointer_ext_slot_shift,
        uint32_t objc_debug_taggedpointer_slot_mask,
        uint32_t objc_debug_taggedpointer_ext_slot_mask,
        uint32_t objc_debug_taggedpointer_payload_lshift,
        uint32_t objc_debug_taggedpointer_payload_rshift,
        uint32_t objc_debug_taggedpointer_ext_payload_lshift,
        uint32_t objc_debug_taggedpointer_ext_payload_rshift,
        lldb::addr_t objc_debug_taggedpointer_classes,
        lldb::addr_t objc_debug_taggedpointer_ext_classes);

```
- **EN**: Declares APIs around `GetClassDescriptor`, `TaggedPointerVendorExtended`.
- **CN**: 声明与 `GetClassDescriptor`, `TaggedPointerVendorExtended` 相关的 API。

### Lines 252-263
```cpp
    bool IsPossibleExtendedTaggedPointer(lldb::addr_t ptr);

    typedef std::map<uint8_t, ObjCLanguageRuntime::ClassDescriptorSP> Cache;
    typedef Cache::iterator CacheIterator;
    Cache m_ext_cache;
    uint64_t m_objc_debug_taggedpointer_ext_mask;
    uint32_t m_objc_debug_taggedpointer_ext_slot_shift;
    uint32_t m_objc_debug_taggedpointer_ext_slot_mask;
    uint32_t m_objc_debug_taggedpointer_ext_payload_lshift;
    uint32_t m_objc_debug_taggedpointer_ext_payload_rshift;
    lldb::addr_t m_objc_debug_taggedpointer_ext_classes;

```
- **EN**: Declares APIs around `IsPossibleExtendedTaggedPointer`.
- **CN**: 声明与 `IsPossibleExtendedTaggedPointer` 相关的 API。

### Lines 264-274
```cpp
    friend class AppleObjCRuntimeV2::TaggedPointerVendorV2;

    TaggedPointerVendorExtended(const TaggedPointerVendorExtended &) = delete;
    const TaggedPointerVendorExtended &
    operator=(const TaggedPointerVendorExtended &) = delete;
  };

  class TaggedPointerVendorLegacy : public TaggedPointerVendorV2 {
  public:
    bool IsPossibleTaggedPointer(lldb::addr_t ptr) override;

```
- **EN**: Introduces declarations for `TaggedPointerVendorLegacy`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `TaggedPointerVendorLegacy` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 275-288
```cpp
    ObjCLanguageRuntime::ClassDescriptorSP
    GetClassDescriptor(lldb::addr_t ptr) override;

  protected:
    TaggedPointerVendorLegacy(AppleObjCRuntimeV2 &runtime)
        : TaggedPointerVendorV2(runtime) {}

    friend class AppleObjCRuntimeV2::TaggedPointerVendorV2;

    TaggedPointerVendorLegacy(const TaggedPointerVendorLegacy &) = delete;
    const TaggedPointerVendorLegacy &
    operator=(const TaggedPointerVendorLegacy &) = delete;
  };

```
- **EN**: Implements logic around `GetClassDescriptor`, `TaggedPointerVendorLegacy`, `TaggedPointerVendorV2`.
- **CN**: 围绕 `GetClassDescriptor`, `TaggedPointerVendorLegacy`, `TaggedPointerVendorV2` 实现具体逻辑。

### Lines 289-298
```cpp
  struct DescriptorMapUpdateResult {
    bool m_update_ran;
    bool m_retry_update;
    uint32_t m_num_found;

    DescriptorMapUpdateResult(bool ran, bool retry, uint32_t found) {
      m_update_ran = ran;

      m_retry_update = retry;

```
- **EN**: Introduces declarations for `DescriptorMapUpdateResult`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `DescriptorMapUpdateResult` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 299-310
```cpp
      m_num_found = found;
    }

    static DescriptorMapUpdateResult Fail() { return {false, false, 0}; }

    static DescriptorMapUpdateResult Success(uint32_t found) {
      return {true, false, found};
    }

    static DescriptorMapUpdateResult Retry() { return {false, true, 0}; }
  };

```
- **EN**: Implements logic around `Fail`, `Success`, `Retry`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `Fail`, `Success`, `Retry` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 311-322
```cpp
  /// Abstraction to read the Objective-C class info.
  class ClassInfoExtractor {
  public:
    ClassInfoExtractor(AppleObjCRuntimeV2 &runtime) : m_runtime(runtime) {}
    std::mutex &GetMutex() { return m_mutex; }

  protected:
    /// The lifetime of this object is tied to that of the runtime.
    AppleObjCRuntimeV2 &m_runtime;
    std::mutex m_mutex;
  };

```
- **EN**: Introduces declarations for `ClassInfoExtractor`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ClassInfoExtractor` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 323-332
```cpp
  /// We can read the class info from the Objective-C runtime using
  /// gdb_objc_realized_classes, objc_copyRealizedClassList or
  /// objc_getRealizedClassList_trylock. The RealizedClassList variants are
  /// preferred because they include lazily named classes, but they are not
  /// always available or safe to call.
  ///
  /// We potentially need more than one helper for the same process, because we
  /// may need to use gdb_objc_realized_classes until dyld is initialized and
  /// then switch over to objc_copyRealizedClassList or
  /// objc_getRealizedClassList_trylock for lazily named classes.
```
- **EN**: Documents the surrounding declarations, example workflow, or design constraints for this LLDB component.
- **CN**: 说明周边声明、示例流程，或记录该 LLDB 组件的设计约束。

### Lines 333-342
```cpp
  class DynamicClassInfoExtractor : public ClassInfoExtractor {
  public:
    DynamicClassInfoExtractor(AppleObjCRuntimeV2 &runtime)
        : ClassInfoExtractor(runtime) {}

    DescriptorMapUpdateResult
    UpdateISAToDescriptorMap(RemoteNXMapTable &hash_table);

  private:
    enum Helper {
```
- **EN**: Introduces declarations for `DynamicClassInfoExtractor`, `Helper`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `DynamicClassInfoExtractor`, `Helper` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 343-353
```cpp
      gdb_objc_realized_classes,
      objc_copyRealizedClassList,
      objc_getRealizedClassList_trylock
    };

    /// Compute which helper to use. If dyld is not yet fully initialized we
    /// must use gdb_objc_realized_classes. Otherwise, we prefer
    /// objc_getRealizedClassList_trylock and objc_copyRealizedClassList
    /// respectively, depending on availability.
    Helper ComputeHelper(ExecutionContext &exe_ctx) const;

```
- **EN**: Declares APIs around `ComputeHelper`.
- **CN**: 声明与 `ComputeHelper` 相关的 API。

### Lines 354-366
```cpp
    UtilityFunction *GetClassInfoUtilityFunction(ExecutionContext &exe_ctx,
                                                 Helper helper);
    lldb::addr_t &GetClassInfoArgs(Helper helper);

    std::unique_ptr<UtilityFunction>
    GetClassInfoUtilityFunctionImpl(ExecutionContext &exe_ctx, Helper helper,
                                    std::string code, std::string name);

    struct UtilityFunctionHelper {
      std::unique_ptr<UtilityFunction> utility_function;
      lldb::addr_t args = LLDB_INVALID_ADDRESS;
    };

```
- **EN**: Introduces declarations for `UtilityFunctionHelper`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `UtilityFunctionHelper` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 367-377
```cpp
    UtilityFunctionHelper m_gdb_objc_realized_classes_helper;
    UtilityFunctionHelper m_objc_copyRealizedClassList_helper;
    UtilityFunctionHelper m_objc_getRealizedClassList_trylock_helper;
  };

  /// Abstraction to read the Objective-C class info from the shared cache.
  class SharedCacheClassInfoExtractor : public ClassInfoExtractor {
  public:
    SharedCacheClassInfoExtractor(AppleObjCRuntimeV2 &runtime)
        : ClassInfoExtractor(runtime) {}

```
- **EN**: Introduces declarations for `SharedCacheClassInfoExtractor`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `SharedCacheClassInfoExtractor` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 378-389
```cpp
    DescriptorMapUpdateResult UpdateISAToDescriptorMap();

  private:
    UtilityFunction *GetClassInfoUtilityFunction(ExecutionContext &exe_ctx);

    std::unique_ptr<UtilityFunction>
    GetClassInfoUtilityFunctionImpl(ExecutionContext &exe_ctx);

    std::unique_ptr<UtilityFunction> m_utility_function;
    lldb::addr_t m_args = LLDB_INVALID_ADDRESS;
  };

```
- **EN**: Declares APIs around `UpdateISAToDescriptorMap`, `GetClassInfoUtilityFunction`, `GetClassInfoUtilityFunctionImpl`.
- **CN**: 声明与 `UpdateISAToDescriptorMap`, `GetClassInfoUtilityFunction`, `GetClassInfoUtilityFunctionImpl` 相关的 API。

### Lines 390-400
```cpp
  class SharedCacheImageHeaders {
  public:
    static std::unique_ptr<SharedCacheImageHeaders>
    CreateSharedCacheImageHeaders(AppleObjCRuntimeV2 &runtime);

    void SetNeedsUpdate() { m_needs_update = true; }

    bool IsImageLoaded(uint16_t image_index);

    uint64_t GetVersion();

```
- **EN**: Introduces declarations for `SharedCacheImageHeaders`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `SharedCacheImageHeaders` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 401-418
```cpp
  private:
    SharedCacheImageHeaders(AppleObjCRuntimeV2 &runtime,
                            lldb::addr_t headerInfoRWs_ptr, uint32_t count,
                            uint32_t entsize)
        : m_runtime(runtime), m_headerInfoRWs_ptr(headerInfoRWs_ptr),
          m_loaded_images(count, false), m_version(0), m_count(count),
          m_entsize(entsize), m_needs_update(true) {}
    llvm::Error UpdateIfNeeded();

    AppleObjCRuntimeV2 &m_runtime;
    lldb::addr_t m_headerInfoRWs_ptr;
    llvm::BitVector m_loaded_images;
    uint64_t m_version;
    uint32_t m_count;
    uint32_t m_entsize;
    bool m_needs_update;
  };

```
- **EN**: Implements logic around `SharedCacheImageHeaders`, `m_runtime`, `m_loaded_images`, `m_entsize`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `SharedCacheImageHeaders`, `m_runtime`, `m_loaded_images`, `m_entsize`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 419-428
```cpp
  AppleObjCRuntimeV2(Process *process, const lldb::ModuleSP &objc_module_sp);

  ObjCISA GetPointerISA(ObjCISA isa);

  lldb::addr_t GetISAHashTablePointer();

  using ValueObjectSet = llvm::SmallPtrSet<ValueObject *, 8>;
  ClassDescriptorSP GetClassDescriptorImpl(ValueObject &valobj,
                                           ValueObjectSet &seen);

```
- **EN**: Declares APIs around `AppleObjCRuntimeV2`, `GetPointerISA`, `GetISAHashTablePointer`, `GetClassDescriptorImpl`; this block coordinates debugger runtime objects, events, or asynchronous control flow; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 声明与 `AppleObjCRuntimeV2`, `GetPointerISA`, `GetISAHashTablePointer`, `GetClassDescriptorImpl` 相关的 API；该代码块协调调试器运行时对象、事件或异步控制流，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 429-438
```cpp
  /// Update the generation count of realized classes. This is not an exact
  /// count but rather a value that is incremented when new classes are realized
  /// or destroyed. Unlike the count in gdb_objc_realized_classes, it will
  /// change when lazily named classes get realized.
  bool RealizedClassGenerationCountChanged();

  uint32_t ParseClassInfoArray(const lldb_private::DataExtractor &data,
                               uint32_t num_class_infos);

  enum class SharedCacheWarningReason {
```
- **EN**: Introduces declarations for `SharedCacheWarningReason`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `SharedCacheWarningReason` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 439-449
```cpp
    eExpressionUnableToRun,
    eExpressionExecutionFailure,
    eNotEnoughClassesRead
  };

  void WarnIfNoClassesCached(SharedCacheWarningReason reason);
  void WarnIfNoExpandedSharedCache();

  lldb::addr_t GetSharedCacheReadOnlyAddress();
  lldb::addr_t GetSharedCacheBaseAddress();

```
- **EN**: Declares APIs around `WarnIfNoClassesCached`, `WarnIfNoExpandedSharedCache`, `GetSharedCacheReadOnlyAddress`, `GetSharedCacheBaseAddress`.
- **CN**: 声明与 `WarnIfNoClassesCached`, `WarnIfNoExpandedSharedCache`, `GetSharedCacheReadOnlyAddress`, `GetSharedCacheBaseAddress` 相关的 API。

### Lines 450-460
```cpp
  bool GetCFBooleanValuesIfNeeded();

  bool HasSymbol(ConstString Name);

  NonPointerISACache *GetNonPointerIsaCache() {
    if (!m_non_pointer_isa_cache_up)
      m_non_pointer_isa_cache_up.reset(
          NonPointerISACache::CreateInstance(*this, m_objc_module_sp));
    return m_non_pointer_isa_cache_up.get();
  }

```
- **EN**: Implements logic around `GetCFBooleanValuesIfNeeded`, `HasSymbol`, `GetNonPointerIsaCache`, `reset`, and 2 more symbols.
- **CN**: 围绕 `GetCFBooleanValuesIfNeeded`, `HasSymbol`, `GetNonPointerIsaCache`, `reset`, and 2 more symbols 实现具体逻辑。

### Lines 461-480
```cpp
  friend class ClassDescriptorV2;

  lldb::ModuleSP m_objc_module_sp;

  DynamicClassInfoExtractor m_dynamic_class_info_extractor;
  SharedCacheClassInfoExtractor m_shared_cache_class_info_extractor;

  std::unique_ptr<DeclVendor> m_decl_vendor_up;
  lldb::addr_t m_tagged_pointer_obfuscator;
  lldb::addr_t m_isa_hash_table_ptr;
  lldb::addr_t m_relative_selector_base;
  HashTableSignature m_hash_signature;
  bool m_has_object_getClass;
  bool m_has_objc_copyRealizedClassList;
  bool m_has_objc_getRealizedClassList_trylock;
  bool m_loaded_objc_opt;
  std::unique_ptr<NonPointerISACache> m_non_pointer_isa_cache_up;
  std::unique_ptr<TaggedPointerVendor> m_tagged_pointer_vendor_up;
  EncodingToTypeSP m_encoding_to_type_sp;
  std::once_flag m_no_classes_cached_warning;
```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 481-489
```cpp
  std::once_flag m_no_expanded_cache_warning;
  std::optional<std::pair<lldb::addr_t, lldb::addr_t>> m_CFBoolean_values;
  uint64_t m_realized_class_generation_count;
  std::unique_ptr<SharedCacheImageHeaders> m_shared_cache_image_headers_up;
};

} // namespace lldb_private

#endif // LLDB_SOURCE_PLUGINS_LANGUAGERUNTIME_OBJC_APPLEOBJCRUNTIME_APPLEOBJCRUNTIMEV2_H
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Stop control / 停机控制**:
  - **EN**: Represents stop reasons, conditions, callbacks, and breakpoint resolution state.
  - **CN**: 表示停机原因、条件、回调以及断点解析状态。
- **Symbolication / 符号化**:
  - **EN**: Relates raw addresses back to modules, sections, symbols, and source locations.
  - **CN**: 将原始地址关联回模块、节区、符号与源码位置。
- **Runtime control / 运行时控制**:
  - **EN**: Models debugger ownership over targets, processes, threads, and event delivery.
  - **CN**: 建模调试器对目标、进程、线程以及事件投递的控制。
- **Value presentation / 值展示**:
  - **EN**: Controls how variables are rendered, summarized, and expanded in debugger views.
  - **CN**: 控制变量在调试器视图中的渲染、摘要与展开方式。
- **User expressions / 用户表达式**:
  - **EN**: Wraps and evaluates source snippets inside the debuggee context.
  - **CN**: 在被调试进程上下文中包装并求值源码片段。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `AppleObjCRuntime.h`, `lldb/lldb-private.h`, `Plugins/LanguageRuntime/ObjC/ObjCLanguageRuntime.h`, `llvm/ADT/BitVector.h`, `llvm/ADT/SmallSet.h`
- **Standard-library headers / 标准库头文件**: `<map>`, `<memory>`, `<mutex>`, `<optional>`
- **Subsystem categories / 子系统类别**: LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (2), shared LLDB public/private type declarations / 共享 LLDB 公共/私有类型声明 (1)
