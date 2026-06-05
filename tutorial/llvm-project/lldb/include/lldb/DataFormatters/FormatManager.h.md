# FormatManager.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/include/lldb/DataFormatters/FormatManager.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Declares LLDB data formatter interfaces for summaries, synthetic children, and value presentation.
  - **CN**: 声明 LLDB 数据格式化接口，用于摘要、synthetic children 与数值展示。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- FormatManager.h -----------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/LLDB file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/LLDB 标准文件头、许可证声明以及文件的高层说明。

### Lines 8-17
```cpp

#ifndef LLDB_DATAFORMATTERS_FORMATMANAGER_H
#define LLDB_DATAFORMATTERS_FORMATMANAGER_H

#include <atomic>
#include <initializer_list>
#include <map>
#include <mutex>
#include <vector>

```
- **EN**: Pulls in the headers needed by this translation unit, including `atomic`, `initializer_list`, `map`, `mutex`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `atomic`, `initializer_list`, `map`, `mutex`。

### Lines 18-27
```cpp
#include "lldb/lldb-enumerations.h"
#include "lldb/lldb-public.h"

#include "lldb/DataFormatters/FormatCache.h"
#include "lldb/DataFormatters/FormatClasses.h"
#include "lldb/DataFormatters/FormattersContainer.h"
#include "lldb/DataFormatters/LanguageCategory.h"
#include "lldb/DataFormatters/TypeCategory.h"
#include "lldb/DataFormatters/TypeCategoryMap.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `lldb/lldb-enumerations.h`, `lldb/lldb-public.h`, `lldb/DataFormatters/FormatCache.h`, `lldb/DataFormatters/FormatClasses.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `lldb/lldb-enumerations.h`, `lldb/lldb-public.h`, `lldb/DataFormatters/FormatCache.h`, `lldb/DataFormatters/FormatClasses.h`。

### Lines 28-35
```cpp
namespace lldb_private {

// this file (and its. cpp) contain the low-level implementation of LLDB Data
// Visualization class DataVisualization is the high-level front-end of this
// feature clients should refer to that class as the entry-point into the data
// formatters unless they have a good reason to bypass it and prefer to use
// this file's objects directly

```
- **EN**: Introduces declarations for `lldb_private`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `lldb_private` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 36-43
```cpp
class FormatManager : public IFormatChangeListener {
  typedef FormattersContainer<TypeSummaryImpl> NamedSummariesMap;
  typedef TypeCategoryMap::MapType::iterator CategoryMapIterator;

public:
  typedef std::map<lldb::LanguageType, LanguageCategory::UniquePointer>
      LanguageCategories;

```
- **EN**: Introduces declarations for `FormatManager`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `FormatManager` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 44-51
```cpp
  FormatManager();

  ~FormatManager() override = default;

  NamedSummariesMap &GetNamedSummaryContainer() {
    return m_named_summaries_map;
  }

```
- **EN**: Implements logic around `FormatManager`, `~FormatManager`, `GetNamedSummaryContainer`.
- **CN**: 围绕 `FormatManager`, `~FormatManager`, `GetNamedSummaryContainer` 实现具体逻辑。

### Lines 52-65
```cpp
  void
  EnableCategory(ConstString category_name,
                 TypeCategoryMap::Position pos = TypeCategoryMap::Default) {
    EnableCategory(category_name, pos, {});
  }

  void EnableCategory(ConstString category_name,
                      TypeCategoryMap::Position pos, lldb::LanguageType lang) {
    lldb::TypeCategoryImplSP category_sp;
    if (m_categories_map.Get(category_name, category_sp) && category_sp) {
      m_categories_map.Enable(category_sp, pos);
      category_sp->AddLanguage(lang);
    }
  }
```
- **EN**: Implements logic around `EnableCategory`, `Get`, `Enable`, `AddLanguage`.
- **CN**: 围绕 `EnableCategory`, `Get`, `Enable`, `AddLanguage` 实现具体逻辑。

### Lines 66-76
```cpp

  void DisableCategory(ConstString category_name) {
    m_categories_map.Disable(category_name);
  }

  void
  EnableCategory(const lldb::TypeCategoryImplSP &category,
                 TypeCategoryMap::Position pos = TypeCategoryMap::Default) {
    m_categories_map.Enable(category, pos);
  }

```
- **EN**: Implements logic around `DisableCategory`, `Disable`, `EnableCategory`, `Enable`.
- **CN**: 围绕 `DisableCategory`, `Disable`, `EnableCategory`, `Enable` 实现具体逻辑。

### Lines 77-84
```cpp
  void DisableCategory(const lldb::TypeCategoryImplSP &category) {
    m_categories_map.Disable(category);
  }

  void EnableAllCategories();

  void DisableAllCategories();

```
- **EN**: Implements logic around `DisableCategory`, `Disable`, `EnableAllCategories`, `DisableAllCategories`.
- **CN**: 围绕 `DisableCategory`, `Disable`, `EnableAllCategories`, `DisableAllCategories` 实现具体逻辑。

### Lines 85-92
```cpp
  bool DeleteCategory(ConstString category_name) {
    return m_categories_map.Delete(category_name);
  }

  void ClearCategories() { return m_categories_map.Clear(); }

  uint32_t GetCategoriesCount() { return m_categories_map.GetCount(); }

```
- **EN**: Implements logic around `DeleteCategory`, `Delete`, `ClearCategories`, `GetCategoriesCount`.
- **CN**: 围绕 `DeleteCategory`, `Delete`, `ClearCategories`, `GetCategoriesCount` 实现具体逻辑。

### Lines 93-105
```cpp
  lldb::TypeCategoryImplSP GetCategoryAtIndex(size_t index) {
    return m_categories_map.GetAtIndex(index);
  }

  void ForEachCategory(TypeCategoryMap::ForEachCallback callback);

  lldb::TypeCategoryImplSP GetCategory(const char *category_name = nullptr,
                                       bool can_create = true) {
    if (!category_name)
      return GetCategory(m_default_category_name);
    return GetCategory(ConstString(category_name));
  }

```
- **EN**: Implements logic around `GetCategoryAtIndex`, `GetAtIndex`, `ForEachCategory`, `GetCategory`.
- **CN**: 围绕 `GetCategoryAtIndex`, `GetAtIndex`, `ForEachCategory`, `GetCategory` 实现具体逻辑。

### Lines 106-114
```cpp
  lldb::TypeCategoryImplSP GetCategory(ConstString category_name,
                                       bool can_create = true);

  lldb::TypeFormatImplSP
  GetFormatForType(lldb::TypeNameSpecifierImplSP type_sp);

  lldb::TypeSummaryImplSP
  GetSummaryForType(lldb::TypeNameSpecifierImplSP type_sp);

```
- **EN**: Declares APIs around `GetCategory`, `GetFormatForType`, `GetSummaryForType`.
- **CN**: 声明与 `GetCategory`, `GetFormatForType`, `GetSummaryForType` 相关的 API。

### Lines 115-123
```cpp
  lldb::TypeFilterImplSP
  GetFilterForType(lldb::TypeNameSpecifierImplSP type_sp);

  lldb::ScriptedSyntheticChildrenSP
  GetSyntheticForType(lldb::TypeNameSpecifierImplSP type_sp);

  lldb::TypeFormatImplSP GetFormat(ValueObject &valobj,
                                   lldb::DynamicValueType use_dynamic);

```
- **EN**: Declares APIs around `GetFilterForType`, `GetSyntheticForType`, `GetFormat`; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 声明与 `GetFilterForType`, `GetSyntheticForType`, `GetFormat` 相关的 API；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 124-137
```cpp
  lldb::TypeSummaryImplSP GetSummaryFormat(ValueObject &valobj,
                                           lldb::DynamicValueType use_dynamic);

  lldb::SyntheticChildrenSP
  GetSyntheticChildren(ValueObject &valobj, lldb::DynamicValueType use_dynamic);

  bool
  AnyMatches(const FormattersMatchCandidate &candidate_type,
             TypeCategoryImpl::FormatCategoryItems items =
                 TypeCategoryImpl::ALL_ITEM_TYPES,
             bool only_enabled = true, const char **matching_category = nullptr,
             TypeCategoryImpl::FormatCategoryItems *matching_type = nullptr) {
    return m_categories_map.AnyMatches(candidate_type, items, only_enabled,
                                       matching_category, matching_type);
```
- **EN**: Implements logic around `GetSummaryFormat`, `GetSyntheticChildren`, `AnyMatches`; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetSummaryFormat`, `GetSyntheticChildren`, `AnyMatches` 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 138-144
```cpp
  }

  static bool GetFormatFromCString(const char *format_cstr,
                                   lldb::Format &format);

  static char GetFormatAsFormatChar(lldb::Format format);

```
- **EN**: Declares APIs around `GetFormatFromCString`, `GetFormatAsFormatChar`; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 声明与 `GetFormatFromCString`, `GetFormatAsFormatChar` 相关的 API；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 145-151
```cpp
  static const char *GetFormatAsCString(lldb::Format format);

  // when DataExtractor dumps a vectorOfT, it uses a predefined format for each
  // item this method returns it, or eFormatInvalid if vector_format is not a
  // vectorOf
  static lldb::Format GetSingleItemFormat(lldb::Format vector_format);

```
- **EN**: Declares APIs around `GetFormatAsCString`, `GetSingleItemFormat`; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 声明与 `GetFormatAsCString`, `GetSingleItemFormat` 相关的 API；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 152-159
```cpp
  // this returns true if the ValueObjectPrinter is *highly encouraged* to
  // actually represent this ValueObject in one-liner format If this object has
  // a summary formatter, however, we should not try and do one-lining, just
  // let the summary do the right thing
  bool ShouldPrintAsOneLiner(ValueObject &valobj);

  void Changed() override;

```
- **EN**: Declares APIs around `ShouldPrintAsOneLiner`, `Changed`; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 声明与 `ShouldPrintAsOneLiner`, `Changed` 相关的 API；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 160-169
```cpp
  uint32_t GetCurrentRevision() override { return m_last_revision; }

  static FormattersMatchVector
  GetPossibleMatches(ValueObject &valobj, lldb::DynamicValueType use_dynamic) {
    FormattersMatchVector matches;
    GetPossibleMatches(valobj, valobj.GetCompilerType(), use_dynamic, matches,
                       FormattersMatchCandidate::Flags(), true);
    return matches;
  }

```
- **EN**: Implements logic around `GetCurrentRevision`, `GetPossibleMatches`, `Flags`; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetCurrentRevision`, `GetPossibleMatches`, `Flags` 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 170-176
```cpp
  static ConstString GetTypeForCache(ValueObject &, lldb::DynamicValueType);

  LanguageCategory *GetCategoryForLanguage(lldb::LanguageType lang_type);

  static std::vector<lldb::LanguageType>
  GetCandidateLanguages(lldb::LanguageType lang_type);

```
- **EN**: Declares APIs around `GetTypeForCache`, `GetCategoryForLanguage`, `GetCandidateLanguages`; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 声明与 `GetTypeForCache`, `GetCategoryForLanguage`, `GetCandidateLanguages` 相关的 API；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 177-185
```cpp
private:
  static void GetPossibleMatches(ValueObject &valobj,
                                 CompilerType compiler_type,
                                 lldb::DynamicValueType use_dynamic,
                                 FormattersMatchVector &entries,
                                 FormattersMatchCandidate::Flags current_flags,
                                 bool root_level = false,
                                 uint32_t ptr_stripped_depth = 0);

```
- **EN**: Declares APIs around `GetPossibleMatches`; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 声明与 `GetPossibleMatches` 相关的 API；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 186-192
```cpp
  std::atomic<uint32_t> m_last_revision;
  FormatCache m_format_cache;
  std::recursive_mutex m_language_categories_mutex;
  LanguageCategories m_language_categories_map;
  NamedSummariesMap m_named_summaries_map;
  TypeCategoryMap m_categories_map;

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 193-201
```cpp
  ConstString m_default_category_name;
  ConstString m_system_category_name;
  ConstString m_vectortypes_category_name;

  template <typename ImplSP>
  ImplSP Get(ValueObject &valobj, lldb::DynamicValueType use_dynamic);
  template <typename ImplSP> ImplSP GetCached(FormattersMatchData &match_data);
  template <typename ImplSP> ImplSP GetHardcoded(FormattersMatchData &);

```
- **EN**: Declares APIs around `Get`, `GetCached`, `GetHardcoded`; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 声明与 `Get`, `GetCached`, `GetHardcoded` 相关的 API；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 202-211
```cpp
  TypeCategoryMap &GetCategories() { return m_categories_map; }

  // These functions are meant to initialize formatters that are very low-
  // level/global in nature and do not naturally belong in any language. The
  // intent is that most formatters go in language-specific categories.
  // Eventually, the runtimes should also be allowed to vend their own
  // formatters, and then one could put formatters that depend on specific
  // library load events in the language runtimes, on an as-needed basis
  void LoadSystemFormatters();

```
- **EN**: Implements logic around `GetCategories`, `LoadSystemFormatters`.
- **CN**: 围绕 `GetCategories`, `LoadSystemFormatters` 实现具体逻辑。

### Lines 212-218
```cpp
  void LoadVectorFormatters();

  friend class FormattersMatchData;
};

} // namespace lldb_private

```
- **EN**: Declares APIs around `LoadVectorFormatters`.
- **CN**: 声明与 `LoadVectorFormatters` 相关的 API。

### Lines 219-219
```cpp
#endif // LLDB_DATAFORMATTERS_FORMATMANAGER_H
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Data formatting / 数据格式化**:
  - **EN**: Controls how LLDB prints values, synthesizes children, and formats user-visible summaries.
  - **CN**: 控制 LLDB 如何打印值、合成子节点以及格式化用户可见摘要。
- **Value presentation / 值展示**:
  - **EN**: Controls how variables are rendered, summarized, and expanded in debugger views.
  - **CN**: 控制变量在调试器视图中的渲染、摘要与展开方式。
- **OS integration / 操作系统集成**:
  - **EN**: Bridges LLDB abstractions to host files, sockets, terminals, and native process facilities.
  - **CN**: 将 LLDB 抽象连接到主机文件、套接字、终端与本地进程设施。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `lldb/lldb-enumerations.h`, `lldb/lldb-public.h`, `lldb/DataFormatters/FormatCache.h`, `lldb/DataFormatters/FormatClasses.h`, `lldb/DataFormatters/FormattersContainer.h`, `lldb/DataFormatters/LanguageCategory.h`, `lldb/DataFormatters/TypeCategory.h`, `lldb/DataFormatters/TypeCategoryMap.h`
- **Standard-library headers / 标准库头文件**: `<atomic>`, `<initializer_list>`, `<map>`, `<mutex>`, `<vector>`
- **Subsystem categories / 子系统类别**: LLDB data formatter components / LLDB 数据格式化组件 (6), shared LLDB public/private type declarations / 共享 LLDB 公共/私有类型声明 (2)
