# TreeSitterHighlighter.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Highlighter/TreeSitter/TreeSitterHighlighter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `TreeSitterHighlighter`.
  - **CN**: 实现与 `TreeSitterHighlighter` 相关的 LLDB 支持逻辑。

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

### Lines 8-15
```cpp

#include "TreeSitterHighlighter.h"
#include "lldb/Utility/LLDBLog.h"
#include "lldb/Utility/Log.h"
#include "lldb/Utility/StreamString.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/StringSwitch.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `TreeSitterHighlighter.h`, `lldb/Utility/LLDBLog.h`, `lldb/Utility/Log.h`, `lldb/Utility/StreamString.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `TreeSitterHighlighter.h`, `lldb/Utility/LLDBLog.h`, `lldb/Utility/Log.h`, `lldb/Utility/StreamString.h`。

### Lines 16-24
```cpp
using namespace lldb_private;

TreeSitterHighlighter::TSState::~TSState() {
  if (query)
    ts_query_delete(query);
  if (parser)
    ts_parser_delete(parser);
}

```
- **EN**: Implements logic around `~TSState`, `ts_query_delete`, `ts_parser_delete`.
- **CN**: 围绕 `~TSState`, `ts_query_delete`, `ts_parser_delete` 实现具体逻辑。

### Lines 25-32
```cpp
TreeSitterHighlighter::TSState::operator bool() const {
  return parser && query;
}

TreeSitterHighlighter::TSState &TreeSitterHighlighter::GetTSState() const {
  if (m_ts_state)
    return *m_ts_state;

```
- **EN**: Implements logic around `bool`, `GetTSState`.
- **CN**: 围绕 `bool`, `GetTSState` 实现具体逻辑。

### Lines 33-41
```cpp
  Log *log = GetLog(LLDBLog::Source);

  m_ts_state.emplace();
  m_ts_state->parser = ts_parser_new();
  if (!m_ts_state->parser) {
    LLDB_LOG(log, "Creating tree-sitter parser failed for {0}", GetName());
    return *m_ts_state;
  }

```
- **EN**: Implements logic around `GetLog`, `emplace`, `ts_parser_new`, `LLDB_LOG`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetLog`, `emplace`, `ts_parser_new`, `LLDB_LOG` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 42-55
```cpp
  const TSLanguage *language = GetLanguage();
  if (!language || !ts_parser_set_language(m_ts_state->parser, language)) {
    LLDB_LOG(log, "Creating tree-sitter language failed for {0}", GetName());
    return *m_ts_state;
  }

  llvm::StringRef query_source = GetHighlightQuery();
  uint32_t error_offset = 0;
  TSQueryError error_type = TSQueryErrorNone;
  m_ts_state->query = ts_query_new(language, query_source.data(),
                                   static_cast<uint32_t>(query_source.size()),
                                   &error_offset, &error_type);
  if (!m_ts_state->query || error_type != TSQueryErrorNone) {
    LLDB_LOG(log,
```
- **EN**: Implements logic around `GetLanguage`, `ts_parser_set_language`, `LLDB_LOG`, `GetHighlightQuery`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetLanguage`, `ts_parser_set_language`, `LLDB_LOG`, `GetHighlightQuery`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 56-62
```cpp
             "Creating tree-sitter query failed for {0} with error {1}: {2}",
             GetName(), error_type, query_source.substr(error_offset, 64));
    // If we have an error but a valid query, we need to reset the object to
    // (1) avoid it looking valid and (2) release the parser.
    m_ts_state.emplace();
  }

```
- **EN**: Implements logic around `GetName`, `emplace`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetName`, `emplace` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 63-76
```cpp
  return *m_ts_state;
}

HighlightStyle::ColorStyle
TreeSitterHighlighter::GetStyleForCapture(llvm::StringRef capture_name,
                                          const HighlightStyle &options) const {
  return llvm::StringSwitch<HighlightStyle::ColorStyle>(capture_name)
      .Case("comment", options.comment)
      .Case("keyword", options.keyword)
      .Case("operator", options.operators)
      .Case("type", options.keyword)
      .Case("punctuation.delimiter.comma", options.comma)
      .Case("punctuation.delimiter.colon", options.colon)
      .Case("punctuation.delimiter.semicolon", options.semicolons)
```
- **EN**: Implements logic around `GetStyleForCapture`, `ColorStyle>`, `Case`.
- **CN**: 围绕 `GetStyleForCapture`, `ColorStyle>`, `Case` 实现具体逻辑。

### Lines 77-89
```cpp
      .Case("punctuation.bracket.square", options.square_brackets)
      .Cases({"keyword.directive", "preproc"}, options.pp_directive)
      .Cases({"string", "string.literal"}, options.string_literal)
      .Cases({"number", "number.literal", "constant.numeric"},
             options.scalar_literal)
      .Cases({"identifier", "variable", "function"}, options.identifier)
      .Cases({"punctuation.bracket.curly", "punctuation.brace"}, options.braces)
      .Cases({"punctuation.bracket.round", "punctuation.bracket",
              "punctuation.paren"},
             options.parentheses)
      .Default({});
}

```
- **EN**: Implements logic around `Case`, `Cases`, `Default`; this block supports documentation parsing, directives, or build-time rendering.
- **CN**: 围绕 `Case`, `Cases`, `Default` 实现具体逻辑；该代码块支持文档解析、指令或构建期渲染。

### Lines 90-98
```cpp
void TreeSitterHighlighter::HighlightRange(
    const HighlightStyle &options, llvm::StringRef text, uint32_t start_byte,
    uint32_t end_byte, const HighlightStyle::ColorStyle &style,
    std::optional<size_t> cursor_pos, bool &highlighted_cursor,
    Stream &s) const {

  if (start_byte >= end_byte || start_byte >= text.size())
    return;

```
- **EN**: Implements logic around `HighlightRange`, `size`.
- **CN**: 围绕 `HighlightRange`, `size` 实现具体逻辑。

### Lines 99-109
```cpp
  end_byte = std::min(end_byte, static_cast<uint32_t>(text.size()));

  llvm::StringRef range = text.substr(start_byte, end_byte - start_byte);

  auto print = [&](llvm::StringRef str) {
    if (style)
      style.Apply(s, str);
    else
      s << str;
  };

```
- **EN**: Implements logic around `min`, `substr`, `Apply`.
- **CN**: 围绕 `min`, `substr`, `Apply` 实现具体逻辑。

### Lines 110-117
```cpp
  // Check if cursor is within this range.
  if (cursor_pos && *cursor_pos >= start_byte && *cursor_pos < end_byte &&
      !highlighted_cursor) {
    highlighted_cursor = true;

    // Split range around cursor position.
    const size_t cursor_in_range = *cursor_pos - start_byte;

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 118-131
```cpp
    // Print everything before the cursor.
    if (cursor_in_range > 0) {
      llvm::StringRef before = range.substr(0, cursor_in_range);
      print(before);
    }

    // Print the cursor itself.
    if (cursor_in_range < range.size()) {
      StreamString cursor_str;
      llvm::StringRef cursor_char = range.substr(cursor_in_range, 1);
      if (style)
        style.Apply(cursor_str, cursor_char);
      else
        cursor_str << cursor_char;
```
- **EN**: Implements logic around `substr`, `size`, `Apply`.
- **CN**: 围绕 `substr`, `size`, `Apply` 实现具体逻辑。

### Lines 132-145
```cpp
      options.selected.Apply(s, cursor_str.GetString());
    }

    // Print everything after the cursor.
    if (cursor_in_range + 1 < range.size()) {
      llvm::StringRef after = range.substr(cursor_in_range + 1);
      print(after);
    }
  } else {
    // No cursor in this range, apply style directly.
    print(range);
  }
}

```
- **EN**: Implements logic around `Apply`, `size`, `substr`.
- **CN**: 围绕 `Apply`, `size`, `substr` 实现具体逻辑。

### Lines 146-152
```cpp
void TreeSitterHighlighter::Highlight(const HighlightStyle &options,
                                      llvm::StringRef line,
                                      std::optional<size_t> cursor_pos,
                                      llvm::StringRef previous_lines,
                                      Stream &s) const {
  auto unformatted = [&]() -> void { s << line; };

```
- **EN**: Implements logic around `Highlight`.
- **CN**: 围绕 `Highlight` 实现具体逻辑。

### Lines 153-163
```cpp
  TSState &ts_state = GetTSState();
  if (!ts_state)
    return unformatted();

  std::string source = previous_lines.str() + line.str();
  TSTree *tree =
      ts_parser_parse_string(ts_state.parser, nullptr, source.c_str(),
                             static_cast<uint32_t>(source.size()));
  if (!tree)
    return unformatted();

```
- **EN**: Implements logic around `GetTSState`, `unformatted`, `str`, `ts_parser_parse_string`, and 1 more symbols.
- **CN**: 围绕 `GetTSState`, `unformatted`, `str`, `ts_parser_parse_string`, and 1 more symbols 实现具体逻辑。

### Lines 164-171
```cpp
  TSQueryCursor *cursor = ts_query_cursor_new();
  assert(cursor);

  llvm::scope_exit delete_cusor([&] { ts_query_cursor_delete(cursor); });

  TSNode root_node = ts_tree_root_node(tree);
  ts_query_cursor_exec(cursor, ts_state.query, root_node);

```
- **EN**: Implements logic around `ts_query_cursor_new`, `assert`, `delete_cusor`, `ts_tree_root_node`, and 1 more symbols.
- **CN**: 围绕 `ts_query_cursor_new`, `assert`, `delete_cusor`, `ts_tree_root_node`, and 1 more symbols 实现具体逻辑。

### Lines 172-178
```cpp
  // Collect all matches and their byte ranges.
  std::vector<HLRange> highlights;
  TSQueryMatch match;
  uint32_t capture_index;
  while (ts_query_cursor_next_capture(cursor, &match, &capture_index)) {
    TSQueryCapture capture = match.captures[capture_index];

```
- **EN**: Implements logic around `ts_query_cursor_next_capture`.
- **CN**: 围绕 `ts_query_cursor_next_capture` 实现具体逻辑。

### Lines 179-185
```cpp
    uint32_t capture_name_len = 0;
    const char *capture_name = ts_query_capture_name_for_id(
        ts_state.query, capture.index, &capture_name_len);

    HighlightStyle::ColorStyle style = GetStyleForCapture(
        llvm::StringRef(capture_name, capture_name_len), options);

```
- **EN**: Implements logic around `ts_query_capture_name_for_id`, `GetStyleForCapture`, `StringRef`.
- **CN**: 围绕 `ts_query_capture_name_for_id`, `GetStyleForCapture`, `StringRef` 实现具体逻辑。

### Lines 186-193
```cpp
    TSNode node = capture.node;
    uint32_t start = ts_node_start_byte(node);
    uint32_t end = ts_node_end_byte(node);

    if (style && start < end)
      highlights.push_back({start, end, style});
  }

```
- **EN**: Implements logic around `ts_node_start_byte`, `ts_node_end_byte`, `push_back`.
- **CN**: 围绕 `ts_node_start_byte`, `ts_node_end_byte`, `push_back` 实现具体逻辑。

### Lines 194-203
```cpp
  std::sort(highlights.begin(), highlights.end(),
            [](const HLRange &a, const HLRange &b) {
              if (a.start_byte != b.start_byte)
                return a.start_byte < b.start_byte;
              // Prefer shorter matches. For example, if we have an expression
              // consisting of a variable and a property, we want to highlight
              // them as individual components.
              return (b.end_byte - b.start_byte) > (a.end_byte - a.start_byte);
            });

```
- **EN**: Implements logic around `sort`; this block supports expression parsing, wrapping, or debug-time code generation.
- **CN**: 围绕 `sort` 实现具体逻辑；该代码块支持表达式解析、包装或调试期代码生成。

### Lines 204-212
```cpp
  uint32_t current_pos = 0;
  bool highlighted_cursor = false;

  for (const auto &h : highlights) {
    // Skip over highlights that start before our current position, which means
    // there's overlap.
    if (h.start_byte < current_pos)
      continue;

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 213-219
```cpp
    // Output any unhighlighted text before this highlight.
    if (current_pos < h.start_byte) {
      HighlightRange(options, line, current_pos, h.start_byte, {}, cursor_pos,
                     highlighted_cursor, s);
      current_pos = h.start_byte;
    }

```
- **EN**: Implements logic around `HighlightRange`.
- **CN**: 围绕 `HighlightRange` 实现具体逻辑。

### Lines 220-232
```cpp
    // Output the highlighted range.
    HighlightRange(options, line, h.start_byte, h.end_byte, h.style, cursor_pos,
                   highlighted_cursor, s);
    current_pos = h.end_byte;
  }

  // Output any remaining unhighlighted text.
  if (current_pos < line.size()) {
    HighlightRange(options, line, current_pos,
                   static_cast<uint32_t>(line.size()), {}, cursor_pos,
                   highlighted_cursor, s);
  }
}
```
- **EN**: Implements logic around `HighlightRange`, `size`, `static_cast`.
- **CN**: 围绕 `HighlightRange`, `size`, `static_cast` 实现具体逻辑。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **User expressions / 用户表达式**:
  - **EN**: Wraps and evaluates source snippets inside the debuggee context.
  - **CN**: 在被调试进程上下文中包装并求值源码片段。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `TreeSitterHighlighter.h`, `lldb/Utility/LLDBLog.h`, `lldb/Utility/Log.h`, `lldb/Utility/StreamString.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`
- **Subsystem categories / 子系统类别**: shared LLDB utility classes / 共享 LLDB 工具类 (3), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (2)
