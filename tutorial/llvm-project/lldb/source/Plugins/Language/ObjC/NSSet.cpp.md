# NSSet.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Language/ObjC/NSSet.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `NSSet`.
  - **CN**: 实现与 `NSSet` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-23
```cpp
//===-- NSSet.cpp ---------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "NSSet.h"
#include "CFBasicHash.h"

#include "Plugins/LanguageRuntime/ObjC/AppleObjCRuntime/AppleObjCRuntime.h"
#include "Plugins/TypeSystem/Clang/TypeSystemClang.h"
#include "lldb/DataFormatters/FormattersHelpers.h"
#include "lldb/Target/Language.h"
#include "lldb/Target/Target.h"
#include "lldb/Utility/DataBufferHeap.h"
#include "lldb/Utility/Endian.h"
#include "lldb/Utility/Status.h"
#include "lldb/Utility/Stream.h"
#include "lldb/ValueObject/ValueObject.h"
#include "lldb/ValueObject/ValueObjectConstResult.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `NSSet.h`, `CFBasicHash.h`, `Plugins/LanguageRuntime/ObjC/AppleObjCRuntime/AppleObjCRuntime.h`, `Plugins/TypeSystem/Clang/TypeSystemClang.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `NSSet.h`, `CFBasicHash.h`, `Plugins/LanguageRuntime/ObjC/AppleObjCRuntime/AppleObjCRuntime.h`, `Plugins/TypeSystem/Clang/TypeSystemClang.h`。

### Lines 24-40
```cpp
using namespace lldb;
using namespace lldb_private;
using namespace lldb_private::formatters;

std::map<ConstString, CXXFunctionSummaryFormat::Callback> &
NSSet_Additionals::GetAdditionalSummaries() {
  static std::map<ConstString, CXXFunctionSummaryFormat::Callback> g_map;
  return g_map;
}

std::map<ConstString, CXXSyntheticChildren::CreateFrontEndCallback> &
NSSet_Additionals::GetAdditionalSynthetics() {
  static std::map<ConstString, CXXSyntheticChildren::CreateFrontEndCallback>
      g_map;
  return g_map;
}

```
- **EN**: Implements logic around `GetAdditionalSummaries`, `GetAdditionalSynthetics`.
- **CN**: 围绕 `GetAdditionalSummaries`, `GetAdditionalSynthetics` 实现具体逻辑。

### Lines 41-54
```cpp
namespace lldb_private {
namespace formatters {
class NSSetISyntheticFrontEnd : public SyntheticChildrenFrontEnd {
public:
  NSSetISyntheticFrontEnd(lldb::ValueObjectSP valobj_sp);

  ~NSSetISyntheticFrontEnd() override;

  llvm::Expected<uint32_t> CalculateNumChildren() override;

  lldb::ValueObjectSP GetChildAtIndex(uint32_t idx) override;

  lldb::ChildCacheState Update() override;

```
- **EN**: Introduces declarations for `lldb_private`, `formatters`, `NSSetISyntheticFrontEnd`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `lldb_private`, `formatters`, `NSSetISyntheticFrontEnd` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 55-70
```cpp
private:
  struct DataDescriptor_32 {
    uint32_t _used : 26;
    uint32_t _szidx : 6;
  };

  struct DataDescriptor_64 {
    uint64_t _used : 58;
    uint32_t _szidx : 6;
  };

  struct SetItemDescriptor {
    lldb::addr_t item_ptr;
    lldb::ValueObjectSP valobj_sp;
  };

```
- **EN**: Introduces declarations for `DataDescriptor_32`, `DataDescriptor_64`, `SetItemDescriptor`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `DataDescriptor_32`, `DataDescriptor_64`, `SetItemDescriptor` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 71-84
```cpp
  ExecutionContextRef m_exe_ctx_ref;
  uint8_t m_ptr_size = 8;
  DataDescriptor_32 *m_data_32 = nullptr;
  DataDescriptor_64 *m_data_64 = nullptr;
  lldb::addr_t m_data_ptr = LLDB_INVALID_ADDRESS;
  std::vector<SetItemDescriptor> m_children;
};

class NSCFSetSyntheticFrontEnd : public SyntheticChildrenFrontEnd {
public:
  NSCFSetSyntheticFrontEnd(lldb::ValueObjectSP valobj_sp);

  llvm::Expected<uint32_t> CalculateNumChildren() override;

```
- **EN**: Introduces declarations for `NSCFSetSyntheticFrontEnd`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `NSCFSetSyntheticFrontEnd` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 85-98
```cpp
  lldb::ValueObjectSP GetChildAtIndex(uint32_t idx) override;

  lldb::ChildCacheState Update() override;

private:
  struct SetItemDescriptor {
    lldb::addr_t item_ptr;
    lldb::ValueObjectSP valobj_sp;
  };

  ExecutionContextRef m_exe_ctx_ref;
  uint8_t m_ptr_size = 8;
  lldb::ByteOrder m_order = lldb::eByteOrderInvalid;

```
- **EN**: Introduces declarations for `SetItemDescriptor`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `SetItemDescriptor` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 99-113
```cpp
  CFBasicHash m_hashtable;

  CompilerType m_pair_type;
  std::vector<SetItemDescriptor> m_children;
};

template <typename D32, typename D64>
class GenericNSSetMSyntheticFrontEnd : public SyntheticChildrenFrontEnd {
public:
  GenericNSSetMSyntheticFrontEnd(lldb::ValueObjectSP valobj_sp);

  ~GenericNSSetMSyntheticFrontEnd() override;

  llvm::Expected<uint32_t> CalculateNumChildren() override;

```
- **EN**: Introduces declarations for `GenericNSSetMSyntheticFrontEnd`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `GenericNSSetMSyntheticFrontEnd` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 114-131
```cpp
  lldb::ValueObjectSP GetChildAtIndex(uint32_t idx) override;

  lldb::ChildCacheState Update() override;

private:

  struct SetItemDescriptor {
    lldb::addr_t item_ptr;
    lldb::ValueObjectSP valobj_sp;
  };

  ExecutionContextRef m_exe_ctx_ref;
  uint8_t m_ptr_size = 8;
  D32 *m_data_32;
  D64 *m_data_64;
  std::vector<SetItemDescriptor> m_children;
};
  
```
- **EN**: Introduces declarations for `SetItemDescriptor`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `SetItemDescriptor` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 132-146
```cpp
namespace Foundation1300 {
  struct DataDescriptor_32 {
    uint32_t _used : 26;
    uint32_t _size;
    uint32_t _mutations;
    uint32_t _objs_addr;
  };
  
  struct DataDescriptor_64 {
    uint64_t _used : 58;
    uint64_t _size;
    uint64_t _mutations;
    uint64_t _objs_addr;
  };
  
```
- **EN**: Introduces declarations for `Foundation1300`, `DataDescriptor_32`, `DataDescriptor_64`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `Foundation1300`, `DataDescriptor_32`, `DataDescriptor_64` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 147-165
```cpp
  using NSSetMSyntheticFrontEnd =
      GenericNSSetMSyntheticFrontEnd<DataDescriptor_32, DataDescriptor_64>;
}
  
namespace Foundation1428 {
  struct DataDescriptor_32 {
    uint32_t _used : 26;
    uint32_t _size;
    uint32_t _objs_addr;
    uint32_t _mutations;
  };
  
  struct DataDescriptor_64 {
    uint64_t _used : 58;
    uint64_t _size;
    uint64_t _objs_addr;
    uint64_t _mutations;
  };
  
```
- **EN**: Introduces declarations for `Foundation1428`, `DataDescriptor_32`, `DataDescriptor_64`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `Foundation1428`, `DataDescriptor_32`, `DataDescriptor_64` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 166-179
```cpp
  using NSSetMSyntheticFrontEnd =
      GenericNSSetMSyntheticFrontEnd<DataDescriptor_32, DataDescriptor_64>;
}
  
namespace Foundation1437 {
  struct DataDescriptor_32 {
    uint32_t _cow;
    // __table storage
    uint32_t _objs_addr;
    uint32_t _muts;
    uint32_t _used : 26;
    uint32_t _szidx : 6;
  };
  
```
- **EN**: Introduces declarations for `Foundation1437`, `DataDescriptor_32`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `Foundation1437`, `DataDescriptor_32` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 180-206
```cpp
  struct DataDescriptor_64 {
    uint64_t _cow;
    // __Table storage
    uint64_t _objs_addr;
    uint32_t _muts;
    uint32_t _used : 26;
    uint32_t _szidx : 6;
  };
  
  using NSSetMSyntheticFrontEnd =
      GenericNSSetMSyntheticFrontEnd<DataDescriptor_32, DataDescriptor_64>;
  
  template <typename DD>
  uint64_t
  __NSSetMSize_Impl(lldb_private::Process &process, lldb::addr_t valobj_addr,
                    Status &error) {
    const lldb::addr_t start_of_descriptor =
        valobj_addr + process.GetAddressByteSize();
    DD descriptor = DD();
    process.ReadMemory(start_of_descriptor, &descriptor, sizeof(descriptor),
                       error);
    if (error.Fail()) {
      return 0;
    }
    return descriptor._used;
  }
  
```
- **EN**: Introduces declarations for `DataDescriptor_64`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `DataDescriptor_64` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 207-224
```cpp
  uint64_t
  __NSSetMSize(lldb_private::Process &process, lldb::addr_t valobj_addr,
               Status &error) {
    if (process.GetAddressByteSize() == 4) {
      return __NSSetMSize_Impl<DataDescriptor_32>(process, valobj_addr, error);
    } else {
      return __NSSetMSize_Impl<DataDescriptor_64>(process, valobj_addr, error);
    }
  }
  } // namespace Foundation1437
} // namespace formatters
} // namespace lldb_private

template <bool cf_style>
bool lldb_private::formatters::NSSetSummaryProvider(
    ValueObject &valobj, Stream &stream, const TypeSummaryOptions &options) {
  static constexpr llvm::StringLiteral g_TypeHint("NSSet");

```
- **EN**: Implements logic around `__NSSetMSize`, `GetAddressByteSize`, `__NSSetMSize_Impl`, `NSSetSummaryProvider`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `__NSSetMSize`, `GetAddressByteSize`, `__NSSetMSize_Impl`, `NSSetSummaryProvider`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 225-239
```cpp
  ProcessSP process_sp = valobj.GetProcessSP();
  if (!process_sp)
    return false;

  ObjCLanguageRuntime *runtime = ObjCLanguageRuntime::Get(*process_sp);

  if (!runtime)
    return false;

  ObjCLanguageRuntime::ClassDescriptorSP descriptor(
      runtime->GetClassDescriptor(valobj));

  if (!descriptor || !descriptor->IsValid())
    return false;

```
- **EN**: Implements logic around `GetProcessSP`, `Get`, `descriptor`, `GetClassDescriptor`, and 1 more symbols; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetProcessSP`, `Get`, `descriptor`, `GetClassDescriptor`, and 1 more symbols 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 240-257
```cpp
  uint32_t ptr_size = process_sp->GetAddressByteSize();
  bool is_64bit = (ptr_size == 8);

  lldb::addr_t valobj_addr = valobj.GetValueAsUnsigned(0);

  if (!valobj_addr)
    return false;

  uint64_t value = 0;

  ConstString class_name(descriptor->GetClassName());

  static const ConstString g_SetI("__NSSetI");
  static const ConstString g_OrderedSetI("__NSOrderedSetI");
  static const ConstString g_SetM("__NSSetM");
  static const ConstString g_SetCF("__NSCFSet");
  static const ConstString g_SetCFRef("CFSetRef");

```
- **EN**: Implements logic around `GetAddressByteSize`, `GetValueAsUnsigned`, `class_name`, `g_SetI`, and 4 more symbols; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetAddressByteSize`, `GetValueAsUnsigned`, `class_name`, `g_SetI`, and 4 more symbols 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 258-285
```cpp
  if (class_name.IsEmpty())
    return false;

  if (class_name == g_SetI || class_name == g_OrderedSetI) {
    Status error;
    value = process_sp->ReadUnsignedIntegerFromMemory(valobj_addr + ptr_size,
                                                      ptr_size, 0, error);
    if (error.Fail())
      return false;
    value &= (is_64bit ? ~0xFC00000000000000UL : ~0xFC000000U);
  } else if (class_name == g_SetM) {
    AppleObjCRuntime *apple_runtime =
        llvm::dyn_cast_or_null<AppleObjCRuntime>(runtime);
    Status error;
    if (apple_runtime && apple_runtime->GetFoundationVersion() >= 1437) {
      value = Foundation1437::__NSSetMSize(*process_sp, valobj_addr, error);
    } else {
      value = process_sp->ReadUnsignedIntegerFromMemory(valobj_addr + ptr_size,
                                                        ptr_size, 0, error);
      value &= (is_64bit ? ~0xFC00000000000000UL : ~0xFC000000U);
    }
    if (error.Fail())
      return false;
  } else if (class_name == g_SetCF || class_name == g_SetCFRef) {
    ExecutionContext exe_ctx(process_sp);
    CFBasicHash cfbh;
    if (!cfbh.Update(valobj_addr, exe_ctx))
      return false;
```
- **EN**: Implements logic around `IsEmpty`, `ReadUnsignedIntegerFromMemory`, `Fail`, `dyn_cast_or_null`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `IsEmpty`, `ReadUnsignedIntegerFromMemory`, `Fail`, `dyn_cast_or_null`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 286-299
```cpp
    value = cfbh.GetCount();
  } else {
    auto &map(NSSet_Additionals::GetAdditionalSummaries());
    auto iter = map.find(class_name), end = map.end();
    if (iter != end)
      return iter->second(valobj, stream, options);
    else
      return false;
  }

  llvm::StringRef prefix, suffix;
  if (Language *language = Language::FindPlugin(options.GetLanguage()))
    std::tie(prefix, suffix) = language->GetFormatterPrefixSuffix(g_TypeHint);

```
- **EN**: Implements logic around `GetCount`, `map`, `find`, `second`, and 2 more symbols; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetCount`, `map`, `find`, `second`, and 2 more symbols 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 300-315
```cpp
  stream << prefix;
  stream.Printf("%" PRIu64 " %s%s", value, "element", value == 1 ? "" : "s");
  stream << suffix;
  return true;
}

SyntheticChildrenFrontEnd *
lldb_private::formatters::NSSetSyntheticFrontEndCreator(
    CXXSyntheticChildren *synth, lldb::ValueObjectSP valobj_sp) {
  lldb::ProcessSP process_sp(valobj_sp->GetProcessSP());
  if (!process_sp)
    return nullptr;
  ObjCLanguageRuntime *runtime = ObjCLanguageRuntime::Get(*process_sp);
  if (!runtime)
    return nullptr;

```
- **EN**: Implements logic around `Printf`, `NSSetSyntheticFrontEndCreator`, `process_sp`, `Get`.
- **CN**: 围绕 `Printf`, `NSSetSyntheticFrontEndCreator`, `process_sp`, `Get` 实现具体逻辑。

### Lines 316-331
```cpp
  CompilerType valobj_type(valobj_sp->GetCompilerType());
  Flags flags(valobj_type.GetTypeInfo());

  if (flags.IsClear(eTypeIsPointer)) {
    Status error;
    valobj_sp = valobj_sp->AddressOf(error);
    if (error.Fail() || !valobj_sp)
      return nullptr;
  }

  ObjCLanguageRuntime::ClassDescriptorSP descriptor(
      runtime->GetClassDescriptor(*valobj_sp));

  if (!descriptor || !descriptor->IsValid())
    return nullptr;

```
- **EN**: Implements logic around `valobj_type`, `flags`, `IsClear`, `AddressOf`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `valobj_type`, `flags`, `IsClear`, `AddressOf`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 332-359
```cpp
  ConstString class_name = descriptor->GetClassName();

  static const ConstString g_SetI("__NSSetI");
  static const ConstString g_OrderedSetI("__NSOrderedSetI");
  static const ConstString g_SetM("__NSSetM");
  static const ConstString g_SetCF("__NSCFSet");
  static const ConstString g_SetCFRef("CFSetRef");

  if (class_name.IsEmpty())
    return nullptr;

  if (class_name == g_SetI || class_name == g_OrderedSetI) {
    return (new NSSetISyntheticFrontEnd(valobj_sp));
  } else if (class_name == g_SetM) {
    AppleObjCRuntime *apple_runtime =
        llvm::dyn_cast_or_null<AppleObjCRuntime>(runtime);
    if (apple_runtime) {
      if (apple_runtime->GetFoundationVersion() >= 1437)
        return (new Foundation1437::NSSetMSyntheticFrontEnd(valobj_sp));
      else if (apple_runtime->GetFoundationVersion() >= 1428)
        return (new Foundation1428::NSSetMSyntheticFrontEnd(valobj_sp));
      else
        return (new Foundation1300::NSSetMSyntheticFrontEnd(valobj_sp));
    } else {
      return (new Foundation1300::NSSetMSyntheticFrontEnd(valobj_sp));
    }
  } else if (class_name == g_SetCF || class_name == g_SetCFRef) {
    return (new NSCFSetSyntheticFrontEnd(valobj_sp));
```
- **EN**: Implements logic around `GetClassName`, `g_SetI`, `g_OrderedSetI`, `g_SetM`, and 8 more symbols.
- **CN**: 围绕 `GetClassName`, `g_SetI`, `g_OrderedSetI`, `g_SetM`, and 8 more symbols 实现具体逻辑。

### Lines 360-375
```cpp
  } else {
    auto &map(NSSet_Additionals::GetAdditionalSynthetics());
    auto iter = map.find(class_name), end = map.end();
    if (iter != end)
      return iter->second(synth, valobj_sp);
    return nullptr;
  }
}

lldb_private::formatters::NSSetISyntheticFrontEnd::NSSetISyntheticFrontEnd(
    lldb::ValueObjectSP valobj_sp)
    : SyntheticChildrenFrontEnd(*valobj_sp), m_exe_ctx_ref() {
  if (valobj_sp)
    Update();
}

```
- **EN**: Implements logic around `map`, `find`, `second`, `NSSetISyntheticFrontEnd`, and 2 more symbols.
- **CN**: 围绕 `map`, `find`, `second`, `NSSetISyntheticFrontEnd`, and 2 more symbols 实现具体逻辑。

### Lines 376-389
```cpp
lldb_private::formatters::NSSetISyntheticFrontEnd::~NSSetISyntheticFrontEnd() {
  delete m_data_32;
  m_data_32 = nullptr;
  delete m_data_64;
  m_data_64 = nullptr;
}

llvm::Expected<uint32_t>
lldb_private::formatters::NSSetISyntheticFrontEnd::CalculateNumChildren() {
  if (!m_data_32 && !m_data_64)
    return 0;
  return (m_data_32 ? m_data_32->_used : m_data_64->_used);
}

```
- **EN**: Implements logic around `~NSSetISyntheticFrontEnd`, `CalculateNumChildren`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `~NSSetISyntheticFrontEnd`, `CalculateNumChildren` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 390-417
```cpp
lldb::ChildCacheState
lldb_private::formatters::NSSetISyntheticFrontEnd::Update() {
  m_children.clear();
  delete m_data_32;
  m_data_32 = nullptr;
  delete m_data_64;
  m_data_64 = nullptr;
  m_ptr_size = 0;
  ValueObjectSP valobj_sp = m_backend.GetSP();
  if (!valobj_sp)
    return lldb::ChildCacheState::eRefetch;
  m_exe_ctx_ref = valobj_sp->GetExecutionContextRef();
  lldb::ProcessSP process_sp(valobj_sp->GetProcessSP());
  if (!process_sp)
    return lldb::ChildCacheState::eRefetch;
  m_ptr_size = process_sp->GetAddressByteSize();
  uint64_t data_location = valobj_sp->GetValueAsUnsigned(0) + m_ptr_size;
  Status error;
  if (m_ptr_size == 4) {
    m_data_32 = new DataDescriptor_32();
    process_sp->ReadMemory(data_location, m_data_32, sizeof(DataDescriptor_32),
                           error);
  } else {
    m_data_64 = new DataDescriptor_64();
    process_sp->ReadMemory(data_location, m_data_64, sizeof(DataDescriptor_64),
                           error);
  }
  if (error.Fail())
```
- **EN**: Implements logic around `Update`, `clear`, `GetSP`, `GetExecutionContextRef`, and 7 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `Update`, `clear`, `GetSP`, `GetExecutionContextRef`, and 7 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 418-434
```cpp
    return lldb::ChildCacheState::eRefetch;
  m_data_ptr = data_location + m_ptr_size;
  return lldb::ChildCacheState::eReuse;
}

lldb::ValueObjectSP
lldb_private::formatters::NSSetISyntheticFrontEnd::GetChildAtIndex(
    uint32_t idx) {
  uint32_t num_children = CalculateNumChildrenIgnoringErrors();

  if (idx >= num_children)
    return lldb::ValueObjectSP();

  ProcessSP process_sp = m_exe_ctx_ref.GetProcessSP();
  if (!process_sp)
    return lldb::ValueObjectSP();

```
- **EN**: Implements logic around `GetChildAtIndex`, `CalculateNumChildrenIgnoringErrors`, `ValueObjectSP`, `GetProcessSP`.
- **CN**: 围绕 `GetChildAtIndex`, `CalculateNumChildrenIgnoringErrors`, `ValueObjectSP`, `GetProcessSP` 实现具体逻辑。

### Lines 435-450
```cpp
  if (m_children.empty()) {
    // do the scan phase
    lldb::addr_t obj_at_idx = 0;

    uint32_t tries = 0;
    uint32_t test_idx = 0;

    while (tries < num_children) {
      obj_at_idx = m_data_ptr + (test_idx * m_ptr_size);
      if (!process_sp)
        return lldb::ValueObjectSP();
      Status error;
      obj_at_idx = process_sp->ReadPointerFromMemory(obj_at_idx, error);
      if (error.Fail())
        return lldb::ValueObjectSP();

```
- **EN**: Implements logic around `empty`, `ValueObjectSP`, `ReadPointerFromMemory`, `Fail`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `empty`, `ValueObjectSP`, `ReadPointerFromMemory`, `Fail` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 451-465
```cpp
      test_idx++;

      if (!obj_at_idx)
        continue;
      tries++;

      SetItemDescriptor descriptor = {obj_at_idx, lldb::ValueObjectSP()};

      m_children.push_back(descriptor);
    }
  }

  if (idx >= m_children.size()) // should never happen
    return lldb::ValueObjectSP();

```
- **EN**: Implements logic around `ValueObjectSP`, `push_back`, `size`.
- **CN**: 围绕 `ValueObjectSP`, `push_back`, `size` 实现具体逻辑。

### Lines 466-486
```cpp
  SetItemDescriptor &set_item = m_children[idx];
  if (!set_item.valobj_sp) {
    auto ptr_size = process_sp->GetAddressByteSize();
    DataBufferHeap buffer(ptr_size, 0);
    switch (ptr_size) {
    case 0: // architecture has no clue - fail
      return lldb::ValueObjectSP();
    case 4:
      *reinterpret_cast<uint32_t *>(buffer.GetBytes()) =
          static_cast<uint32_t>(set_item.item_ptr);
      break;
    case 8:
      *reinterpret_cast<uint64_t *>(buffer.GetBytes()) =
          static_cast<uint64_t>(set_item.item_ptr);
      break;
    default:
      lldbassert(false && "pointer size is not 4 nor 8");
    }
    StreamString idx_name;
    idx_name.Printf("[%" PRIu64 "]", (uint64_t)idx);

```
- **EN**: Implements logic around `GetAddressByteSize`, `buffer`, `ValueObjectSP`, `static_cast`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetAddressByteSize`, `buffer`, `ValueObjectSP`, `static_cast`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 487-503
```cpp
    DataExtractor data(buffer.GetBytes(), buffer.GetByteSize(),
                       process_sp->GetByteOrder(),
                       process_sp->GetAddressByteSize());

    set_item.valobj_sp = CreateChildValueObjectFromData(
        idx_name.GetString(), data, m_exe_ctx_ref,
        m_backend.GetCompilerType().GetBasicTypeFromAST(
            lldb::eBasicTypeObjCID));
  }
  return set_item.valobj_sp;
}

lldb_private::formatters::NSCFSetSyntheticFrontEnd::NSCFSetSyntheticFrontEnd(
    lldb::ValueObjectSP valobj_sp)
    : SyntheticChildrenFrontEnd(*valobj_sp), m_exe_ctx_ref(), m_hashtable(),
      m_pair_type() {}

```
- **EN**: Implements logic around `data`, `GetByteOrder`, `GetAddressByteSize`, `CreateChildValueObjectFromData`, and 5 more symbols.
- **CN**: 围绕 `data`, `GetByteOrder`, `GetAddressByteSize`, `CreateChildValueObjectFromData`, and 5 more symbols 实现具体逻辑。

### Lines 504-519
```cpp
llvm::Expected<uint32_t>
lldb_private::formatters::NSCFSetSyntheticFrontEnd::CalculateNumChildren() {
  if (!m_hashtable.IsValid())
    return 0;
  return m_hashtable.GetCount();
}

lldb::ChildCacheState
lldb_private::formatters::NSCFSetSyntheticFrontEnd::Update() {
  m_children.clear();
  ValueObjectSP valobj_sp = m_backend.GetSP();
  m_ptr_size = 0;
  if (!valobj_sp)
    return lldb::ChildCacheState::eRefetch;
  m_exe_ctx_ref = valobj_sp->GetExecutionContextRef();

```
- **EN**: Implements logic around `CalculateNumChildren`, `IsValid`, `GetCount`, `Update`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `CalculateNumChildren`, `IsValid`, `GetCount`, `Update`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 520-534
```cpp
  lldb::ProcessSP process_sp(valobj_sp->GetProcessSP());
  if (!process_sp)
    return lldb::ChildCacheState::eRefetch;
  m_ptr_size = process_sp->GetAddressByteSize();
  m_order = process_sp->GetByteOrder();
  return m_hashtable.Update(valobj_sp->GetValueAsUnsigned(0), m_exe_ctx_ref)
             ? lldb::ChildCacheState::eReuse
             : lldb::ChildCacheState::eRefetch;
}

lldb::ValueObjectSP
lldb_private::formatters::NSCFSetSyntheticFrontEnd::GetChildAtIndex(
    uint32_t idx) {
  lldb::addr_t m_values_ptr = m_hashtable.GetValuePointer();

```
- **EN**: Implements logic around `process_sp`, `GetAddressByteSize`, `GetByteOrder`, `Update`, and 2 more symbols.
- **CN**: 围绕 `process_sp`, `GetAddressByteSize`, `GetByteOrder`, `Update`, and 2 more symbols 实现具体逻辑。

### Lines 535-550
```cpp
  const uint32_t num_children = CalculateNumChildrenIgnoringErrors();

  if (idx >= num_children)
    return lldb::ValueObjectSP();

  if (m_children.empty()) {
    ProcessSP process_sp = m_exe_ctx_ref.GetProcessSP();
    if (!process_sp)
      return lldb::ValueObjectSP();

    Status error;
    lldb::addr_t val_at_idx = 0;

    uint32_t tries = 0;
    uint32_t test_idx = 0;

```
- **EN**: Implements logic around `CalculateNumChildrenIgnoringErrors`, `ValueObjectSP`, `empty`, `GetProcessSP`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `CalculateNumChildrenIgnoringErrors`, `ValueObjectSP`, `empty`, `GetProcessSP` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 551-567
```cpp
    // Iterate over inferior memory, reading value pointers by shifting the
    // cursor by test_index * m_ptr_size. Returns an empty ValueObject if a read
    // fails, otherwise, continue until the number of tries matches the number
    // of childen.
    while (tries < num_children) {
      val_at_idx = m_values_ptr + (test_idx * m_ptr_size);

      val_at_idx = process_sp->ReadPointerFromMemory(val_at_idx, error);
      if (error.Fail())
        return lldb::ValueObjectSP();

      test_idx++;

      if (!val_at_idx)
        continue;
      tries++;

```
- **EN**: Implements logic around `ReadPointerFromMemory`, `Fail`, `ValueObjectSP`; this block propagates recoverable errors, status objects, or diagnostics; controls debugger-side formatting or synthetic presentation of values; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ReadPointerFromMemory`, `Fail`, `ValueObjectSP` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并控制调试器侧的值格式化或 synthetic 展示，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 568-581
```cpp
      SetItemDescriptor descriptor = {val_at_idx, lldb::ValueObjectSP()};

      m_children.push_back(descriptor);
    }
  }

  if (idx >= m_children.size()) // should never happen
    return lldb::ValueObjectSP();

  SetItemDescriptor &set_item = m_children[idx];
  if (!set_item.valobj_sp) {

    WritableDataBufferSP buffer_sp(new DataBufferHeap(m_ptr_size, 0));

```
- **EN**: Implements logic around `ValueObjectSP`, `push_back`, `size`, `buffer_sp`.
- **CN**: 围绕 `ValueObjectSP`, `push_back`, `size`, `buffer_sp` 实现具体逻辑。

### Lines 582-598
```cpp
    switch (m_ptr_size) {
    case 0: // architecture has no clue - fail
      return lldb::ValueObjectSP();
    case 4:
      *reinterpret_cast<uint32_t *>(buffer_sp->GetBytes()) =
          static_cast<uint32_t>(set_item.item_ptr);
      break;
    case 8:
      *reinterpret_cast<uint64_t *>(buffer_sp->GetBytes()) =
          static_cast<uint64_t>(set_item.item_ptr);
      break;
    default:
      lldbassert(false && "pointer size is not 4 nor 8");
    }
    StreamString idx_name;
    idx_name.Printf("[%" PRIu64 "]", (uint64_t)idx);

```
- **EN**: Implements logic around `ValueObjectSP`, `static_cast`, `lldbassert`, `Printf`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `ValueObjectSP`, `static_cast`, `lldbassert`, `Printf` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 599-618
```cpp
    DataExtractor data(buffer_sp, m_order, m_ptr_size);

    set_item.valobj_sp = CreateChildValueObjectFromData(
        idx_name.GetString(), data, m_exe_ctx_ref,
        m_backend.GetCompilerType().GetBasicTypeFromAST(
            lldb::eBasicTypeObjCID));
  }

  return set_item.valobj_sp;
}

template <typename D32, typename D64>
lldb_private::formatters::GenericNSSetMSyntheticFrontEnd<
    D32, D64>::GenericNSSetMSyntheticFrontEnd(lldb::ValueObjectSP valobj_sp)
    : SyntheticChildrenFrontEnd(*valobj_sp), m_exe_ctx_ref(),
      m_data_32(nullptr), m_data_64(nullptr) {
  if (valobj_sp)
    Update();
}

```
- **EN**: Implements logic around `data`, `CreateChildValueObjectFromData`, `GetString`, `GetCompilerType`, and 4 more symbols.
- **CN**: 围绕 `data`, `CreateChildValueObjectFromData`, `GetString`, `GetCompilerType`, and 4 more symbols 实现具体逻辑。

### Lines 619-636
```cpp
template <typename D32, typename D64>
lldb_private::formatters::GenericNSSetMSyntheticFrontEnd<D32, D64>::
    GenericNSSetMSyntheticFrontEnd::~GenericNSSetMSyntheticFrontEnd() {
  delete m_data_32;
  m_data_32 = nullptr;
  delete m_data_64;
  m_data_64 = nullptr;
}

template <typename D32, typename D64>
llvm::Expected<uint32_t>
lldb_private::formatters::GenericNSSetMSyntheticFrontEnd<
    D32, D64>::CalculateNumChildren() {
  if (!m_data_32 && !m_data_64)
    return 0;
  return (m_data_32 ? (uint32_t)m_data_32->_used : (uint32_t)m_data_64->_used);
}

```
- **EN**: Implements logic around `~GenericNSSetMSyntheticFrontEnd`, `CalculateNumChildren`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `~GenericNSSetMSyntheticFrontEnd`, `CalculateNumChildren` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 637-664
```cpp
template <typename D32, typename D64>
lldb::ChildCacheState
lldb_private::formatters::GenericNSSetMSyntheticFrontEnd<D32, D64>::Update() {
  m_children.clear();
  ValueObjectSP valobj_sp = m_backend.GetSP();
  m_ptr_size = 0;
  delete m_data_32;
  m_data_32 = nullptr;
  delete m_data_64;
  m_data_64 = nullptr;
  if (!valobj_sp)
    return lldb::ChildCacheState::eRefetch;
  if (!valobj_sp)
    return lldb::ChildCacheState::eRefetch;
  m_exe_ctx_ref = valobj_sp->GetExecutionContextRef();
  lldb::ProcessSP process_sp(valobj_sp->GetProcessSP());
  if (!process_sp)
    return lldb::ChildCacheState::eRefetch;
  m_ptr_size = process_sp->GetAddressByteSize();
  uint64_t data_location = valobj_sp->GetValueAsUnsigned(0) + m_ptr_size;
  Status error;
  if (m_ptr_size == 4) {
    m_data_32 = new D32();
    process_sp->ReadMemory(data_location, m_data_32, sizeof(D32),
                           error);
  } else {
    m_data_64 = new D64();
    process_sp->ReadMemory(data_location, m_data_64, sizeof(D64),
```
- **EN**: Implements logic around `Update`, `clear`, `GetSP`, `GetExecutionContextRef`, and 6 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `Update`, `clear`, `GetSP`, `GetExecutionContextRef`, and 6 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 665-679
```cpp
                           error);
  }
  return error.Success() ? lldb::ChildCacheState::eReuse
                         : lldb::ChildCacheState::eRefetch;
}

template <typename D32, typename D64>
lldb::ValueObjectSP
lldb_private::formatters::
  GenericNSSetMSyntheticFrontEnd<D32, D64>::GetChildAtIndex(uint32_t idx) {
  lldb::addr_t m_objs_addr =
      (m_data_32 ? m_data_32->_objs_addr : m_data_64->_objs_addr);

  uint32_t num_children = CalculateNumChildrenIgnoringErrors();

```
- **EN**: Implements logic around `Success`, `GetChildAtIndex`, `CalculateNumChildrenIgnoringErrors`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `Success`, `GetChildAtIndex`, `CalculateNumChildrenIgnoringErrors` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 680-693
```cpp
  if (idx >= num_children)
    return lldb::ValueObjectSP();

  ProcessSP process_sp = m_exe_ctx_ref.GetProcessSP();
  if (!process_sp)
    return lldb::ValueObjectSP();

  if (m_children.empty()) {
    // do the scan phase
    lldb::addr_t obj_at_idx = 0;

    uint32_t tries = 0;
    uint32_t test_idx = 0;

```
- **EN**: Implements logic around `ValueObjectSP`, `GetProcessSP`, `empty`.
- **CN**: 围绕 `ValueObjectSP`, `GetProcessSP`, `empty` 实现具体逻辑。

### Lines 694-708
```cpp
    while (tries < num_children) {
      obj_at_idx = m_objs_addr + (test_idx * m_ptr_size);
      if (!process_sp)
        return lldb::ValueObjectSP();
      Status error;
      obj_at_idx = process_sp->ReadPointerFromMemory(obj_at_idx, error);
      if (error.Fail())
        return lldb::ValueObjectSP();

      test_idx++;

      if (!obj_at_idx)
        continue;
      tries++;

```
- **EN**: Implements logic around `ValueObjectSP`, `ReadPointerFromMemory`, `Fail`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `ValueObjectSP`, `ReadPointerFromMemory`, `Fail` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 709-736
```cpp
      SetItemDescriptor descriptor = {obj_at_idx, lldb::ValueObjectSP()};

      m_children.push_back(descriptor);
    }
  }

  if (idx >= m_children.size()) // should never happen
    return lldb::ValueObjectSP();

  SetItemDescriptor &set_item = m_children[idx];
  if (!set_item.valobj_sp) {
    auto ptr_size = process_sp->GetAddressByteSize();
    DataBufferHeap buffer(ptr_size, 0);
    switch (ptr_size) {
    case 0: // architecture has no clue?? - fail
      return lldb::ValueObjectSP();
    case 4:
      *((uint32_t *)buffer.GetBytes()) = (uint32_t)set_item.item_ptr;
      break;
    case 8:
      *((uint64_t *)buffer.GetBytes()) = (uint64_t)set_item.item_ptr;
      break;
    default:
      assert(false && "pointer size is not 4 nor 8 - get out of here ASAP");
    }
    StreamString idx_name;
    idx_name.Printf("[%" PRIu64 "]", (uint64_t)idx);

```
- **EN**: Implements logic around `ValueObjectSP`, `push_back`, `size`, `GetAddressByteSize`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `ValueObjectSP`, `push_back`, `size`, `GetAddressByteSize`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 737-751
```cpp
    DataExtractor data(buffer.GetBytes(), buffer.GetByteSize(),
                       process_sp->GetByteOrder(),
                       process_sp->GetAddressByteSize());

    set_item.valobj_sp = CreateChildValueObjectFromData(
        idx_name.GetString(), data, m_exe_ctx_ref,
        m_backend.GetCompilerType().GetBasicTypeFromAST(
            lldb::eBasicTypeObjCID));
  }
  return set_item.valobj_sp;
}

template bool lldb_private::formatters::NSSetSummaryProvider<true>(
    ValueObject &valobj, Stream &stream, const TypeSummaryOptions &options);

```
- **EN**: Implements logic around `data`, `GetByteOrder`, `GetAddressByteSize`, `CreateChildValueObjectFromData`, and 3 more symbols; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `data`, `GetByteOrder`, `GetAddressByteSize`, `CreateChildValueObjectFromData`, and 3 more symbols 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 752-753
```cpp
template bool lldb_private::formatters::NSSetSummaryProvider<false>(
    ValueObject &valobj, Stream &stream, const TypeSummaryOptions &options);
```
- **EN**: Implements logic around `NSSetSummaryProvider`; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `NSSetSummaryProvider` 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
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

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `NSSet.h`, `CFBasicHash.h`, `Plugins/LanguageRuntime/ObjC/AppleObjCRuntime/AppleObjCRuntime.h`, `Plugins/TypeSystem/Clang/TypeSystemClang.h`, `lldb/DataFormatters/FormattersHelpers.h`, `lldb/Target/Language.h`, `lldb/Target/Target.h`, `lldb/Utility/DataBufferHeap.h`, `lldb/Utility/Endian.h`, `lldb/Utility/Status.h` ... (+3 more)
- **Subsystem categories / 子系统类别**: shared LLDB utility classes / 共享 LLDB 工具类 (4), target, process, and thread control / 目标、进程与线程控制 (2), value inspection and presentation helpers / 值检查与展示辅助逻辑 (2), LLDB data formatter components / LLDB 数据格式化组件 (1)
