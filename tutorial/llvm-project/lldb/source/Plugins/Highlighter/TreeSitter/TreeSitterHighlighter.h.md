# TreeSitterHighlighter.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Highlighter/TreeSitter/TreeSitterHighlighter.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Declares the LLDB interfaces, data structures, and helper APIs associated with `TreeSitterHighlighter`.
  - **CN**: 声明与 `TreeSitterHighlighter` 相关的 LLDB 接口、数据结构以及辅助 API。

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
- **EN**: Contains the standard LLVM/LLDB file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/LLDB 标准文件头、许可证声明以及文件的高层说明。

### Lines 8-11
```cpp

#ifndef LLDB_SOURCE_PLUGINS_HIGHLIGHTER_TREESITTER_TREESITTERHIGHLIGHTER_H
#define LLDB_SOURCE_PLUGINS_HIGHLIGHTER_TREESITTER_TREESITTERHIGHLIGHTER_H

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 12-15
```cpp
#include "lldb/Core/Highlighter.h"
#include "lldb/Utility/Stream.h"
#include "llvm/ADT/StringRef.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `lldb/Core/Highlighter.h`, `lldb/Utility/Stream.h`, `llvm/ADT/StringRef.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `lldb/Core/Highlighter.h`, `lldb/Utility/Stream.h`, `llvm/ADT/StringRef.h`。

### Lines 16-19
```cpp
#include <optional>
#include <tree_sitter/api.h>

namespace lldb_private {
```
- **EN**: Pulls in the headers needed by this translation unit, including `optional`, `tree_sitter/api.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `optional`, `tree_sitter/api.h`。

### Lines 20-23
```cpp

struct TSState;

class TreeSitterHighlighter : public Highlighter {
```
- **EN**: Introduces declarations for `TSState`, `TreeSitterHighlighter`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `TSState`, `TreeSitterHighlighter` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 24-27
```cpp
public:
  TreeSitterHighlighter() = default;
  ~TreeSitterHighlighter() override = default;

```
- **EN**: Declares APIs around `TreeSitterHighlighter`, `~TreeSitterHighlighter`.
- **CN**: 声明与 `TreeSitterHighlighter`, `~TreeSitterHighlighter` 相关的 API。

### Lines 28-32
```cpp
  /// Highlights a single line of code using tree-sitter parsing.
  void Highlight(const HighlightStyle &options, llvm::StringRef line,
                 std::optional<size_t> cursor_pos,
                 llvm::StringRef previous_lines, Stream &s) const override;

```
- **EN**: Declares APIs around `Highlight`.
- **CN**: 声明与 `Highlight` 相关的 API。

### Lines 33-36
```cpp
protected:
  /// Returns the tree-sitter language for this highlighter.
  virtual const TSLanguage *GetLanguage() const = 0;

```
- **EN**: Declares APIs around `GetLanguage`.
- **CN**: 声明与 `GetLanguage` 相关的 API。

### Lines 37-40
```cpp
  /// Returns the tree-sitter highlight query for this language.
  virtual llvm::StringRef GetHighlightQuery() const = 0;

private:
```
- **EN**: Declares APIs around `GetHighlightQuery`.
- **CN**: 声明与 `GetHighlightQuery` 相关的 API。

### Lines 41-45
```cpp
  /// Maps a tree-sitter capture name to a HighlightStyle color.
  HighlightStyle::ColorStyle
  GetStyleForCapture(llvm::StringRef capture_name,
                     const HighlightStyle &options) const;

```
- **EN**: Declares APIs around `GetStyleForCapture`.
- **CN**: 声明与 `GetStyleForCapture` 相关的 API。

### Lines 46-52
```cpp
  /// Applies syntax highlighting to a range of text.
  void HighlightRange(const HighlightStyle &options, llvm::StringRef text,
                      uint32_t start_byte, uint32_t end_byte,
                      const HighlightStyle::ColorStyle &style,
                      std::optional<size_t> cursor_pos,
                      bool &highlighted_cursor, Stream &s) const;

```
- **EN**: Declares APIs around `HighlightRange`.
- **CN**: 声明与 `HighlightRange` 相关的 API。

### Lines 53-58
```cpp
  struct HLRange {
    uint32_t start_byte;
    uint32_t end_byte;
    HighlightStyle::ColorStyle style;
  };

```
- **EN**: Introduces declarations for `HLRange`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `HLRange` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 59-64
```cpp
  struct TSState {
    TSState() = default;
    TSState &operator=(const TSState &) = delete;
    TSState(const TSState &) = delete;
    ~TSState();

```
- **EN**: Introduces declarations for `TSState`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `TSState` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 65-69
```cpp
    explicit operator bool() const;
    TSParser *parser = nullptr;
    TSQuery *query = nullptr;
  };

```
- **EN**: Declares APIs around `bool`.
- **CN**: 声明与 `bool` 相关的 API。

### Lines 70-74
```cpp
  /// Lazily creates a tree-sitter state (TSState).
  TSState &GetTSState() const;
  mutable std::optional<TSState> m_ts_state;
};

```
- **EN**: Declares APIs around `GetTSState`.
- **CN**: 声明与 `GetTSState` 相关的 API。

### Lines 75-77
```cpp
} // namespace lldb_private

#endif // LLDB_SOURCE_PLUGINS_LANGUAGE_TREESITTERCOMMON_TREESITTERHIGHLIGHTER_H
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `lldb/Core/Highlighter.h`, `lldb/Utility/Stream.h`, `llvm/ADT/StringRef.h`
- **Standard-library headers / 标准库头文件**: `<optional>`, `<tree_sitter/api.h>`
- **Subsystem categories / 子系统类别**: LLDB core debugger abstractions / LLDB 核心调试抽象 (1), shared LLDB utility classes / 共享 LLDB 工具类 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
