# FormattersContainer.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/include/lldb/DataFormatters/FormattersContainer.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Declares LLDB data formatter interfaces for summaries, synthetic children, and value presentation.
  - **CN**: 声明 LLDB 数据格式化接口，用于摘要、synthetic children 与数值展示。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- FormattersContainer.h -----------------------------------*- C++ -*-===//
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

#ifndef LLDB_DATAFORMATTERS_FORMATTERSCONTAINER_H
#define LLDB_DATAFORMATTERS_FORMATTERSCONTAINER_H

#include <functional>
#include <map>
#include <memory>
#include <mutex>
#include <string>

```
- **EN**: Pulls in the headers needed by this translation unit, including `functional`, `map`, `memory`, `mutex`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `functional`, `map`, `memory`, `mutex`。

### Lines 18-27
```cpp
#include "lldb/lldb-public.h"

#include "lldb/DataFormatters/FormatClasses.h"
#include "lldb/DataFormatters/TypeFormat.h"
#include "lldb/DataFormatters/TypeSummary.h"
#include "lldb/DataFormatters/TypeSynthetic.h"
#include "lldb/Symbol/CompilerType.h"
#include "lldb/Utility/RegularExpression.h"
#include "lldb/ValueObject/ValueObject.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `lldb/lldb-public.h`, `lldb/DataFormatters/FormatClasses.h`, `lldb/DataFormatters/TypeFormat.h`, `lldb/DataFormatters/TypeSummary.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `lldb/lldb-public.h`, `lldb/DataFormatters/FormatClasses.h`, `lldb/DataFormatters/TypeFormat.h`, `lldb/DataFormatters/TypeSummary.h`。

### Lines 28-35
```cpp
namespace lldb_private {

class IFormatChangeListener {
public:
  virtual ~IFormatChangeListener() = default;

  virtual void Changed() = 0;

```
- **EN**: Introduces declarations for `lldb_private`, `IFormatChangeListener`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `lldb_private`, `IFormatChangeListener` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 36-42
```cpp
  virtual uint32_t GetCurrentRevision() = 0;
};

/// Class for matching type names.
class TypeMatcher {
  /// Type name for exact match, or name of the python callback if m_match_type
  /// is `eFormatterMatchCallback`.
```
- **EN**: Introduces declarations for `TypeMatcher`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `TypeMatcher` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 43-49
```cpp
  ConstString m_name;
  RegularExpression m_type_name_regex;
  /// Indicates what kind of matching strategy should be used:
  /// - eFormatterMatchExact: match the exact type name in m_name.
  /// - eFormatterMatchRegex: match using the RegularExpression object
  ///   `m_type_name_regex` instead.
  /// - eFormatterMatchCallback: run the function in m_name to decide if a type
```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 50-60
```cpp
  ///   matches or not.
  lldb::FormatterMatchType m_match_type;

  // if the user tries to add formatters for, say, "struct Foo" those will not
  // match any type because of the way we strip qualifiers from typenames this
  // method looks for the case where the user is adding a
  // "class","struct","enum" or "union" Foo and strips the unnecessary qualifier
  static ConstString StripTypeName(ConstString type) {
    if (type.IsEmpty())
      return type;

```
- **EN**: Implements logic around `StripTypeName`, `IsEmpty`.
- **CN**: 围绕 `StripTypeName`, `IsEmpty` 实现具体逻辑。

### Lines 61-68
```cpp
    llvm::StringRef type_lexer(type);

    type_lexer.consume_front("class ");
    type_lexer.consume_front("enum ");
    type_lexer.consume_front("struct ");
    type_lexer.consume_front("union ");
    type_lexer = type_lexer.ltrim();

```
- **EN**: Declares APIs around `type_lexer`, `consume_front`, `ltrim`.
- **CN**: 声明与 `type_lexer`, `consume_front`, `ltrim` 相关的 API。

### Lines 69-77
```cpp
    return ConstString(type_lexer);
  }

public:
  TypeMatcher() = delete;
  /// Creates a matcher that accepts any type with exactly the given type name.
  TypeMatcher(ConstString type_name)
      : m_name(type_name), m_match_type(lldb::eFormatterMatchExact) {}
  /// Creates a matcher that accepts any type matching the given regex.
```
- **EN**: Implements logic around `ConstString`, `TypeMatcher`, `m_name`.
- **CN**: 围绕 `ConstString`, `TypeMatcher`, `m_name` 实现具体逻辑。

### Lines 78-89
```cpp
  TypeMatcher(RegularExpression regex)
      : m_type_name_regex(std::move(regex)),
        m_match_type(lldb::eFormatterMatchRegex) {}
  /// Creates a matcher using the matching type and string from the given type
  /// name specifier.
  TypeMatcher(lldb::TypeNameSpecifierImplSP type_specifier)
      : m_name(type_specifier->GetName()),
        m_match_type(type_specifier->GetMatchType()) {
    if (m_match_type == lldb::eFormatterMatchRegex)
      m_type_name_regex = RegularExpression(type_specifier->GetName());
  }

```
- **EN**: Implements logic around `TypeMatcher`, `m_type_name_regex`, `m_match_type`, `m_name`, and 1 more symbols.
- **CN**: 围绕 `TypeMatcher`, `m_type_name_regex`, `m_match_type`, `m_name`, and 1 more symbols 实现具体逻辑。

### Lines 90-103
```cpp
  /// True iff this matches the given type.
  bool Matches(FormattersMatchCandidate candidate_type) const {
    ConstString type_name = candidate_type.GetTypeName();
    switch (m_match_type) {
    case lldb::eFormatterMatchExact:
      return m_name == type_name ||
             StripTypeName(m_name) == StripTypeName(type_name);
    case lldb::eFormatterMatchRegex:
      return m_type_name_regex.Execute(type_name.GetStringRef());
    case lldb::eFormatterMatchCallback:
      // CommandObjectType{Synth,Filter}Add tries to prevent the user from
      // creating both a synthetic child provider and a filter for the same type
      // in the same category, but we don't have a type object at that point, so
      // it creates a dummy candidate without type or script interpreter.
```
- **EN**: Implements logic around `Matches`, `GetTypeName`, `StripTypeName`, `Execute`; this block controls debugger-side formatting or synthetic presentation of values; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `Matches`, `GetTypeName`, `StripTypeName`, `Execute` 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示，并注册命令、插件或面向解释器的扩展点。

### Lines 104-112
```cpp
      // Skip callback matching in these cases.
      if (candidate_type.GetScriptInterpreter())
        return candidate_type.GetScriptInterpreter()->FormatterCallbackFunction(
            m_name.AsCString(nullptr),
            std::make_shared<TypeImpl>(candidate_type.GetType()));
    }
    return false;
  }

```
- **EN**: Declares APIs around `GetScriptInterpreter`, `AsCString`, `make_shared`.
- **CN**: 声明与 `GetScriptInterpreter`, `AsCString`, `make_shared` 相关的 API。

### Lines 113-123
```cpp
  lldb::FormatterMatchType GetMatchType() const { return m_match_type; }

  /// Returns the underlying match string for this TypeMatcher.
  ConstString GetMatchString() const {
    if (m_match_type == lldb::eFormatterMatchExact)
        return StripTypeName(m_name);
    if (m_match_type == lldb::eFormatterMatchRegex)
        return ConstString(m_type_name_regex.GetText());
    return m_name;
  }

```
- **EN**: Implements logic around `GetMatchType`, `GetMatchString`, `StripTypeName`, `ConstString`.
- **CN**: 围绕 `GetMatchType`, `GetMatchString`, `StripTypeName`, `ConstString` 实现具体逻辑。

### Lines 124-130
```cpp
  /// Returns true if this TypeMatcher and the given one were most created by
  /// the same match string.
  /// The main purpose of this function is to find existing TypeMatcher
  /// instances by the user input that created them. This is necessary as LLDB
  /// allows referencing existing TypeMatchers in commands by the user input
  /// that originally created them:
  /// (lldb) type summary add --summary-string \"A\" -x TypeName
```
- **EN**: Documents the surrounding declarations, example workflow, or design constraints for this LLDB component.
- **CN**: 说明周边声明、示例流程，或记录该 LLDB 组件的设计约束。

### Lines 131-138
```cpp
  /// (lldb) type summary delete TypeName
  bool CreatedBySameMatchString(TypeMatcher other) const {
    return GetMatchString() == other.GetMatchString();
  }
};

template <typename ValueType> class FormattersContainer {
public:
```
- **EN**: Implements logic around `CreatedBySameMatchString`, `GetMatchString`; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `CreatedBySameMatchString`, `GetMatchString` 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 139-145
```cpp
  typedef typename std::shared_ptr<ValueType> ValueSP;
  typedef std::vector<std::pair<TypeMatcher, ValueSP>> MapType;
  typedef std::function<bool(const TypeMatcher &, const ValueSP &)>
      ForEachCallback;
  typedef typename std::shared_ptr<FormattersContainer<ValueType>>
      SharedPointer;

```
- **EN**: Declares APIs around `function`.
- **CN**: 声明与 `function` 相关的 API。

### Lines 146-155
```cpp
  friend class TypeCategoryImpl;

  FormattersContainer(IFormatChangeListener *lst) : listener(lst) {}

  void Add(TypeMatcher matcher, const ValueSP &entry) {
    if (listener)
      entry->GetRevision() = listener->GetCurrentRevision();
    else
      entry->GetRevision() = 0;

```
- **EN**: Implements logic around `FormattersContainer`, `Add`, `GetRevision`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `FormattersContainer`, `Add`, `GetRevision` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 156-162
```cpp
    std::lock_guard<std::recursive_mutex> guard(m_map_mutex);
    Delete(matcher);
    m_map.emplace_back(std::move(matcher), std::move(entry));
    if (listener)
      listener->Changed();
  }

```
- **EN**: Declares APIs around `guard`, `Delete`, `emplace_back`, `Changed`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 声明与 `guard`, `Delete`, `emplace_back`, `Changed` 相关的 API；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 163-174
```cpp
  bool Delete(TypeMatcher matcher) {
    std::lock_guard<std::recursive_mutex> guard(m_map_mutex);
    for (auto iter = m_map.begin(); iter != m_map.end(); ++iter)
      if (iter->first.CreatedBySameMatchString(matcher)) {
        m_map.erase(iter);
        if (listener)
          listener->Changed();
        return true;
      }
    return false;
  }

```
- **EN**: Implements logic around `Delete`, `guard`, `begin`, `CreatedBySameMatchString`, and 2 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Delete`, `guard`, `begin`, `CreatedBySameMatchString`, and 2 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 175-186
```cpp
  // Finds the first formatter in the container that matches `candidate`.
  bool Get(FormattersMatchCandidate candidate, ValueSP &entry) {
    std::lock_guard<std::recursive_mutex> guard(m_map_mutex);
    for (auto &formatter : llvm::reverse(m_map)) {
      if (formatter.first.Matches(candidate)) {
        entry = formatter.second;
        return true;
      }
    }
    return false;
  }

```
- **EN**: Implements logic around `Get`, `guard`, `reverse`, `Matches`; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `Get`, `guard`, `reverse`, `Matches` 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 187-200
```cpp
  // Finds the first match between candidate types in `candidates` and
  // formatters in this container.
  bool Get(const FormattersMatchVector &candidates, ValueSP &entry) {
    for (const FormattersMatchCandidate &candidate : candidates) {
      if (Get(candidate, entry)) {
        if (candidate.IsMatch(entry))
          return true;
        entry.reset();
        continue;
      }
    }
    return false;
  }

```
- **EN**: Implements logic around `Get`, `IsMatch`, `reset`.
- **CN**: 围绕 `Get`, `IsMatch`, `reset` 实现具体逻辑。

### Lines 201-210
```cpp
  bool GetExact(TypeMatcher matcher, ValueSP &entry) {
    std::lock_guard<std::recursive_mutex> guard(m_map_mutex);
    for (const auto &pos : m_map)
      if (pos.first.CreatedBySameMatchString(matcher)) {
        entry = pos.second;
        return true;
      }
    return false;
  }

```
- **EN**: Implements logic around `GetExact`, `guard`, `CreatedBySameMatchString`.
- **CN**: 围绕 `GetExact`, `guard`, `CreatedBySameMatchString` 实现具体逻辑。

### Lines 211-217
```cpp
  ValueSP GetAtIndex(size_t index) {
    std::lock_guard<std::recursive_mutex> guard(m_map_mutex);
    if (index >= m_map.size())
      return ValueSP();
    return m_map[index].second;
  }

```
- **EN**: Implements logic around `GetAtIndex`, `guard`, `size`, `ValueSP`.
- **CN**: 围绕 `GetAtIndex`, `guard`, `size`, `ValueSP` 实现具体逻辑。

### Lines 218-227
```cpp
  lldb::TypeNameSpecifierImplSP GetTypeNameSpecifierAtIndex(size_t index) {
    std::lock_guard<std::recursive_mutex> guard(m_map_mutex);
    if (index >= m_map.size())
      return lldb::TypeNameSpecifierImplSP();
    TypeMatcher type_matcher = m_map[index].first;
    return std::make_shared<TypeNameSpecifierImpl>(
        type_matcher.GetMatchString().GetStringRef(),
        type_matcher.GetMatchType());
  }

```
- **EN**: Implements logic around `GetTypeNameSpecifierAtIndex`, `guard`, `size`, `TypeNameSpecifierImplSP`, and 3 more symbols.
- **CN**: 围绕 `GetTypeNameSpecifierAtIndex`, `guard`, `size`, `TypeNameSpecifierImplSP`, and 3 more symbols 实现具体逻辑。

### Lines 228-234
```cpp
  void Clear() {
    std::lock_guard<std::recursive_mutex> guard(m_map_mutex);
    m_map.clear();
    if (listener)
      listener->Changed();
  }

```
- **EN**: Implements logic around `Clear`, `guard`, `clear`, `Changed`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Clear`, `guard`, `clear`, `Changed` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 235-245
```cpp
  void ForEach(ForEachCallback callback) {
    if (callback) {
      std::lock_guard<std::recursive_mutex> guard(m_map_mutex);
      for (const auto &pos : m_map) {
        const TypeMatcher &type = pos.first;
        if (!callback(type, pos.second))
          break;
      }
    }
  }

```
- **EN**: Implements logic around `ForEach`, `guard`, `callback`.
- **CN**: 围绕 `ForEach`, `guard`, `callback` 实现具体逻辑。

### Lines 246-257
```cpp
  uint32_t GetCount() {
    std::lock_guard<std::recursive_mutex> guard(m_map_mutex);
    return m_map.size();
  }

  void AutoComplete(CompletionRequest &request) {
    ForEach([&request](const TypeMatcher &matcher, const ValueSP &value) {
      request.TryCompleteCurrentArg(matcher.GetMatchString().GetStringRef());
      return true;
    });
  }

```
- **EN**: Implements logic around `GetCount`, `guard`, `size`, `AutoComplete`, and 2 more symbols.
- **CN**: 围绕 `GetCount`, `guard`, `size`, `AutoComplete`, and 2 more symbols 实现具体逻辑。

### Lines 258-266
```cpp
protected:
  FormattersContainer(const FormattersContainer &) = delete;
  const FormattersContainer &operator=(const FormattersContainer &) = delete;

  MapType m_map;
  std::recursive_mutex m_map_mutex;
  IFormatChangeListener *listener;
};

```
- **EN**: Declares APIs around `FormattersContainer`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 声明与 `FormattersContainer` 相关的 API；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 267-269
```cpp
} // namespace lldb_private

#endif // LLDB_DATAFORMATTERS_FORMATTERSCONTAINER_H
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
- **Symbolication / 符号化**:
  - **EN**: Relates raw addresses back to modules, sections, symbols, and source locations.
  - **CN**: 将原始地址关联回模块、节区、符号与源码位置。
- **Value presentation / 值展示**:
  - **EN**: Controls how variables are rendered, summarized, and expanded in debugger views.
  - **CN**: 控制变量在调试器视图中的渲染、摘要与展开方式。
- **User expressions / 用户表达式**:
  - **EN**: Wraps and evaluates source snippets inside the debuggee context.
  - **CN**: 在被调试进程上下文中包装并求值源码片段。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `lldb/lldb-public.h`, `lldb/DataFormatters/FormatClasses.h`, `lldb/DataFormatters/TypeFormat.h`, `lldb/DataFormatters/TypeSummary.h`, `lldb/DataFormatters/TypeSynthetic.h`, `lldb/Symbol/CompilerType.h`, `lldb/Utility/RegularExpression.h`, `lldb/ValueObject/ValueObject.h`
- **Standard-library headers / 标准库头文件**: `<functional>`, `<map>`, `<memory>`, `<mutex>`, `<string>`
- **Subsystem categories / 子系统类别**: LLDB data formatter components / LLDB 数据格式化组件 (4), shared LLDB public/private type declarations / 共享 LLDB 公共/私有类型声明 (1), symbol and debug-info abstractions / 符号与调试信息抽象 (1), shared LLDB utility classes / 共享 LLDB 工具类 (1), value inspection and presentation helpers / 值检查与展示辅助逻辑 (1)
