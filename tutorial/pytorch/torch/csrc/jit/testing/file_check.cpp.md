# file_check.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/testing/file_check.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Contains testing-only hooks or helpers for exercising JIT behavior. This specific file centers on `file_check.cpp`. The file header states: "==-- llvm/Support/FileCheck.h ---------------------------*- C++ -*-==// The LLVM Compiler Infrastructure This file is distributed under the University of Illinois Open Source License. See LICENSE.TXT for details. ===----------------------------------------------------------------------===//."
- **Purpose (CN)**: 包含用于验证 JIT 行为的测试专用钩子或辅助逻辑。 该文件具体围绕 `file_check.cpp` 展开。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40 / 第 1-40 行

```cpp
//==-- llvm/Support/FileCheck.h ---------------------------*- C++ -*-==//
//
//                     The LLVM Compiler Infrastructure
//
// This file is distributed under the University of Illinois Open Source
// License. See LICENSE.TXT for details.
//
//===----------------------------------------------------------------------===//

// API modified from llvm::FileCheck

#include <c10/util/Exception.h>
#include <c10/util/StringUtil.h>
#include <c10/util/irange.h>
#include <torch/csrc/jit/frontend/source_range.h>
#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/testing/file_check.h>
#include <optional>

#include <algorithm>
#include <iostream>
#include <sstream>
#include <string>

namespace torch::jit::testing {

enum CheckType {
  CHECK,
  CHECK_NEXT,
  CHECK_SAME,
  CHECK_NOT,
  CHECK_COUNT,
  CHECK_DAG,
  CHECK_SOURCE_HIGHLIGHTED,
  CHECK_REGEX,
};

struct Check {
  Check(
      CheckType type,
```

- **EN:** It enters or references namespace scopes such as torch::jit::testing, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit::testing 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** The block declares or refines core types including Check, CheckType.
- **CN:** 该代码块声明或细化了 Check, CheckType 等核心类型。
- **EN:** Concepts touched here: Type system / 类型系统, Backend integration / 后端集成, Declared symbols / 声明的符号, Header composition / 头文件组织, Namespace scoping / 命名空间作用域, Type definition / 类型定义.
- **CN:** 这里涉及的概念包括：Type system / 类型系统, Backend integration / 后端集成, Declared symbols / 声明的符号, Header composition / 头文件组织, Namespace scoping / 命名空间作用域, Type definition / 类型定义。

### Lines 41-80 / 第 41-80 行

```cpp
      std::string str,
      std::optional<size_t> count = std::nullopt)
      : type_(type), count_(count), search_str_(std::move(str)) {}

  Check(
      CheckType type,
      std::string_view str,
      std::optional<size_t> count = std::nullopt)
      : Check(type, std::string(str.begin(), str.end()), count) {}

  CheckType type_;
  std::optional<size_t> count_;
  const std::string search_str_;

  friend std::ostream& operator<<(std::ostream& out, const Check& c);
};

std::ostream& operator<<(std::ostream& out, const Check& c) {
  switch (c.type_) {
    case CHECK:
      out << "CHECK";
      break;
    case CHECK_NEXT:
      out << "CHECK-NEXT";
      break;
    case CHECK_SAME:
      out << "CHECK-SAME";
      break;
    case CHECK_NOT:
      out << "CHECK-NOT";
      break;
    case CHECK_DAG:
      out << "CHECK-DAG";
      break;
    case CHECK_COUNT:
      out << "CHECK-COUNT-" << *c.count_;
      break;
    case CHECK_SOURCE_HIGHLIGHTED:
      out << "CHECK-SOURCE-HIGHLIGHTED";
      break;
```

- **EN:** Important callable entry points in this range include Check.
- **CN:** 这一段的重要可调用入口包括 Check。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Type system / 类型系统, Registration / 注册机制, Declared symbols / 声明的符号, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Type system / 类型系统, Registration / 注册机制, Declared symbols / 声明的符号, Branching logic / 分支逻辑。

### Lines 81-120 / 第 81-120 行

```cpp
    case CHECK_REGEX:
      out << "CHECK-REGEX";
      break;
  }
  out << ": " << c.search_str_;
  return out;
}

namespace {

size_t assertFind(
    const SourceRange& search_range,
    const std::string& sub,
    const std::function<void(std::ostream& out)>& extra_msg = nullptr) {
  auto pos = search_range.source()->text_str().find(sub, search_range.start());
  if (pos == std::string::npos || (pos + sub.size()) > search_range.end()) {
    auto found_range =
        SourceRange(search_range.source(), search_range.start(), sub.size());
    std::stringstream ss;
    ss << "Expected to find ";
    c10::printQuotedString(ss, sub);
    ss << " but did not find it" << '\n';
    ss << "Searched string:" << '\n';
    found_range.highlight(ss);
    if (extra_msg) {
      extra_msg(ss);
    }
    throw std::runtime_error(ss.str());
  }
  return pos;
}

size_t assertFind(
    const SourceRange& search_range,
    const std::string& sub,
    const Check& check) {
  return assertFind(search_range, sub, [&](std::ostream& out) {
    out << "From " << check << '\n';
  });
}
```

- **EN:** Important callable entry points in this range include assertFind, SourceRange, printQuotedString, extra_msg, runtime_error.
- **CN:** 这一段的重要可调用入口包括 assertFind, SourceRange, printQuotedString, extra_msg, runtime_error。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Operator schema / 算子模式, Declared symbols / 声明的符号, Namespace scoping / 命名空间作用域, Result propagation / 结果传递, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Operator schema / 算子模式, Declared symbols / 声明的符号, Namespace scoping / 命名空间作用域, Result propagation / 结果传递, Branching logic / 分支逻辑。

### Lines 121-160 / 第 121-160 行

```cpp

size_t assertFind(
    const std::shared_ptr<Source>& source,
    const std::string& sub,
    size_t start,
    const Check& check) {
  return assertFind(SourceRange(source, start, source->size()), sub, check);
}

size_t assertFindRegex(
    const SourceRange& search_range,
    const std::string& sub,
    const std::function<void(std::ostream& out)>& extra_msg = nullptr) {
  auto pos =
      search_range.source()->text_str().find_regex(sub, search_range.start());

  if (pos == std::string::npos) {
    std::stringstream ss;
    ss << "Expected to find regex ";
    c10::printQuotedString(ss, sub);
    ss << " but did not find it" << '\n';
    ss << "Searched string:" << '\n';
    if (extra_msg) {
      extra_msg(ss);
    }
    throw std::runtime_error(ss.str());
  }
  return pos;
}

size_t assertFindRegex(
    const SourceRange& search_range,
    const std::string& sub,
    const Check& check) {
  return assertFindRegex(search_range, sub, [&](std::ostream& out) {
    out << "From " << check << '\n';
  });
}

size_t assertFindRegex(
```

- **EN:** Important callable entry points in this range include assertFind, assertFindRegex, printQuotedString, extra_msg, runtime_error.
- **CN:** 这一段的重要可调用入口包括 assertFind, assertFindRegex, printQuotedString, extra_msg, runtime_error。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Operator schema / 算子模式, Result propagation / 结果传递, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Operator schema / 算子模式, Result propagation / 结果传递, Branching logic / 分支逻辑。

### Lines 161-200 / 第 161-200 行

```cpp
    const std::shared_ptr<Source>& source,
    const std::string& sub,
    size_t start,
    const Check& check) {
  return assertFindRegex(
      SourceRange(source, start, source->size()), sub, check);
}

void assertNotFind(
    const SourceRange& search_range,
    const std::string& sub,
    const Check& check) {
  auto pos = search_range.source()->text_str().find(sub, search_range.start());
  if (pos != std::string::npos && (pos + sub.size()) <= search_range.end()) {
    auto found_range =
        SourceRange(search_range.source(), pos, sub.size() + pos);
    std::stringstream ss;
    ss << "Expected to not find ";
    c10::printQuotedString(ss, sub);
    ss << " but found it\n";
    found_range.highlight(ss);
    ss << "From " << check << '\n';
    throw std::runtime_error(ss.str());
  }
}

} // namespace

struct FileCheckImpl {
  TORCH_API explicit FileCheckImpl() = default;

  TORCH_API void run(const std::string& test_file) {
    has_run = true;

    if (groups.empty() || groups[0].empty()) {
      throw std::runtime_error(
          "No checks have been added to this instance of"
          "Filecheck! Check for bad input.");
    }

```

- **EN:** The block declares or refines core types including FileCheckImpl.
- **CN:** 该代码块声明或细化了 FileCheckImpl 等核心类型。
- **EN:** Important callable entry points in this range include assertFindRegex, assertNotFind, SourceRange, printQuotedString, runtime_error, run.
- **CN:** 这一段的重要可调用入口包括 assertFindRegex, assertNotFind, SourceRange, printQuotedString, runtime_error, run。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 201-240 / 第 201-240 行

```cpp
    doChecks(std::make_shared<Source>(test_file));
  }

  TORCH_API void run(
      const std::string& checks_file,
      const std::string& test_file) {
    auto source = std::make_shared<Source>(checks_file);
    parseStrings(source);
    run(test_file);
  }

  TORCH_API void addCheck(const Check& check) {
    // consecutive CHECK_DAGs & CHECK_NOTs need to be evaluated as a group
    if (groups.empty() ||
        (check.type_ != CHECK_NOT && check.type_ != CHECK_DAG)) {
      groups.push_back({check});
    } else {
      auto& last_group = groups.back();
      if (last_group.at(0).type_ == check.type_) {
        last_group.push_back(check);
      } else {
        groups.push_back({check});
      }
    }
    checks.push_back(check);
    has_run = false;
  }

  TORCH_API void addCheck(
      CheckType type,
      const std::string& s,
      std::optional<size_t> count = std::nullopt) {
    addCheck(Check(type, s, count));
  }

  // NOLINTNEXTLINE(cppcoreguidelines-non-private-member-variables-in-classes)
  bool has_run = false;

  friend std::ostream& operator<<(std::ostream& out, const FileCheckImpl& fc);

```

- **EN:** Important callable entry points in this range include doChecks, run, parseStrings, addCheck.
- **CN:** 这一段的重要可调用入口包括 doChecks, run, parseStrings, addCheck。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Type system / 类型系统, Registration / 注册机制, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Type system / 类型系统, Registration / 注册机制, Branching logic / 分支逻辑。

### Lines 241-280 / 第 241-280 行

```cpp
 private:
  bool parseSingleCheck(const std::shared_ptr<Source>& source, size_t* start) {
    const static std::vector<std::pair<CheckType, std::string>> check_pairs = {
        {CHECK, ": "},
        {CHECK_NEXT, "-NEXT: "},
        {CHECK_SAME, "-SAME: "},
        {CHECK_NOT, "-NOT: "},
        {CHECK_DAG, "-DAG: "},
        {CHECK_COUNT, "-COUNT-"}, // needs special parsing
        {CHECK_SOURCE_HIGHLIGHTED, "-SOURCE-HIGHLIGHTED: "},
        {CHECK_REGEX, "-REGEX: "},
    };

    for (const auto& check_pair : check_pairs) {
      const std::string& check_suffix = check_pair.second;
      auto suffix_pos = source->text_str().find(check_suffix, *start);
      if (suffix_pos != *start) {
        continue;
      }
      size_t end_check_string = suffix_pos + check_suffix.size();
      CheckType type = check_pair.first;
      std::optional<size_t> count = std::nullopt;
      auto end_line = source->text_str().find("\n", end_check_string);
      bool exactly = false;
      if (type == CHECK_COUNT) {
        const std::string exact = "EXACTLY-";
        if (source->text_str().find(exact, end_check_string) ==
            end_check_string) {
          exactly = true;
          end_check_string += exact.size();
        }
        size_t end =
            assertFind(SourceRange(source, end_check_string, end_line), ":");
        auto count_view = source->text_str()
                              .substr(end_check_string, end - end_check_string)
                              .str();
        count = std::stoll(std::string(count_view.begin(), count_view.end()));
        end_check_string = end + 2; // add ':' and the space
      }
      auto check = Check(
```

- **EN:** Important callable entry points in this range include parseSingleCheck, assertFind.
- **CN:** 这一段的重要可调用入口包括 parseSingleCheck, assertFind。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Concepts touched here: Type system / 类型系统, Branching logic / 分支逻辑, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Type system / 类型系统, Branching logic / 分支逻辑, Iteration / 迭代处理。

### Lines 281-320 / 第 281-320 行

```cpp
          type,
          source->text_str()
              .substr(end_check_string, end_line - end_check_string)
              .str(),
          count);
      addCheck(check);
      if (exactly) {
        addCheck(CHECK_NOT, check.search_str_);
      }
      *start = end_line;
      return true;
    }
    return false;
  }

  size_t findNextStart(const std::shared_ptr<Source>& source, size_t prev_end) {
    size_t start = source->text_str().find("#", prev_end);
    if (start == std::string::npos) {
      return start;
    }
    start += 1;
    static constexpr size_t max_whitespace = 6;
    size_t i = 0;
    while (start + i < source->size() && i < max_whitespace) {
      auto c = source->char_at(start + i);
      if (c != ' ' && c != '\t') {
        break;
      }
      i++;
    }
    static const std::string check = "CHECK";
    if (source->text_str().substr(start + i, check.size()) == check) {
      return start + i + check.size();
    } else {
      return findNextStart(source, start + i + 1);
    }
  }

  void parseStrings(const std::shared_ptr<Source>& source) {
    size_t start = 0;
```

- **EN:** Important callable entry points in this range include addCheck, findNextStart, parseStrings.
- **CN:** 这一段的重要可调用入口包括 addCheck, findNextStart, parseStrings。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 321-360 / 第 321-360 行

```cpp
    start = findNextStart(source, 0);
    while (start != std::string::npos) {
      bool found_match = parseSingleCheck(source, &start);
      if (!found_match) {
        std::ostringstream ss;
        ss << "Could not parse check at:\n";
        SourceRange(source, start, start + 1).highlight(ss);
        ss << "Check for bad input.";
        has_run = true;
        throw std::runtime_error(ss.str());
      }
      start = findNextStart(source, start);
    }
  }

  void doCheckNot(
      const std::vector<Check>& nots,
      const std::shared_ptr<Source>& source,
      const SourceRange& prev,
      const SourceRange& next) {
    auto start = prev.end(); // inclusive
    auto end = next.start(); // exclusive
    if (end < start) {
      return;
    }
    for (const auto& check : nots) {
      AT_ASSERT(check.type_ == CHECK_NOT);
      assertNotFind(SourceRange(source, start, end), check.search_str_, check);
    }
  }

  // Checks that source token is highlighted, does not advance search range.
  void doCheckSourceHighlighted(
      const Check& check,
      const std::shared_ptr<Source>& source,
      size_t start_offset) {
    auto construct_error_and_throw = [&](size_t error_start_pos) {
      SourceRange error_range(
          source, error_start_pos, check.search_str_.size());
      std::stringstream ss;
```

- **EN:** Important callable entry points in this range include SourceRange, runtime_error, doCheckNot, AT_ASSERT, assertNotFind, doCheckSourceHighlighted.
- **CN:** 这一段的重要可调用入口包括 SourceRange, runtime_error, doCheckNot, AT_ASSERT, assertNotFind, doCheckSourceHighlighted。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 361-400 / 第 361-400 行

```cpp
      ss << "Expected to find ";
      c10::printQuotedString(ss, check.search_str_);
      ss << "highlighted but it is not." << '\n';
      error_range.highlight(ss);
      throw std::runtime_error(ss.str());
    };

    size_t search_start_offset = start_offset;
    bool found_token_at_least_once = false;
    size_t pos = search_start_offset;
    while (pos < source->size()) {
      pos = source->text_str().find(check.search_str_, search_start_offset);
      if (pos == std::string::npos) {
        break;
      }

      found_token_at_least_once = true;

      auto lineno = source->lineno_for_offset(pos);
      auto col = pos - source->offset_for_line(lineno);
      auto highlight_lineno = lineno + 1;

      if (highlight_lineno >= source->num_lines()) {
        construct_error_and_throw(pos);
      }

      auto highlight_start_offset =
          source->offset_for_line(highlight_lineno) + col;
      auto highlight_end_offset = std::min(
          highlight_start_offset + check.search_str_.size(), source->size());

      if (highlight_end_offset >= source->size()) {
        construct_error_and_throw(pos);
      }

      bool found_highlight = true;
      for (const auto posi :
           c10::irange(highlight_start_offset, highlight_end_offset)) {
        if (source->char_at(posi) != '~') {
          found_highlight = false;
```

- **EN:** Important callable entry points in this range include printQuotedString, runtime_error, construct_error_and_throw.
- **CN:** 这一段的重要可调用入口包括 printQuotedString, runtime_error, construct_error_and_throw。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Concepts touched here: Branching logic / 分支逻辑, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Branching logic / 分支逻辑, Iteration / 迭代处理。

### Lines 401-440 / 第 401-440 行

```cpp
        }
      }

      if (found_highlight) {
        assertNotFind(
            SourceRange(
                source, highlight_start_offset - 1, highlight_start_offset),
            "~",
            check);
        assertNotFind(
            SourceRange(source, highlight_end_offset, highlight_end_offset + 1),
            "~",
            check);
        return;
      }

      search_start_offset = pos + 1;
    }

    if (!found_token_at_least_once) {
      // Guaranteed to fail to generate error message.
      assertFind(source, check.search_str_, start_offset, check);
    }

    construct_error_and_throw(start_offset);
  }

  SourceRange matchDagGroup(
      const std::vector<Check>& group,
      const std::shared_ptr<Source>& source,
      const SourceRange& prev) {
    size_t group_beg = std::string::npos;
    size_t group_end = 0;

    AT_ASSERT(!groups.empty());
    for (const auto& check : group) {
      AT_ASSERT(check.type_ == group[0].type_);
      auto pos = assertFind(source, check.search_str_, prev.end(), check);
      group_beg = std::min(pos, group_beg);
      group_end = std::max(pos + check.search_str_.size(), group_end);
```

- **EN:** Important callable entry points in this range include assertNotFind, assertFind, construct_error_and_throw, matchDagGroup, AT_ASSERT.
- **CN:** 这一段的重要可调用入口包括 assertNotFind, assertFind, construct_error_and_throw, matchDagGroup, AT_ASSERT。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 441-480 / 第 441-480 行

```cpp
    }

    return SourceRange(source, group_beg, group_end);
  }

  SourceRange matchGroup(
      const std::vector<Check>& group,
      const std::shared_ptr<Source>& source,
      const SourceRange& prev) {
    AT_ASSERT(!group.empty());
    CheckType type = group[0].type_;

    if (type == CHECK_DAG) {
      return matchDagGroup(group, source, prev);
    }
    AT_ASSERT(type != CHECK_NOT);
    AT_ASSERT(group.size() == 1);

    const auto& check = group[0];
    size_t start_range = prev.end();
    size_t end_range = start_range;

    switch (check.type_) {
      case CHECK: {
        start_range = assertFind(source, check.search_str_, start_range, check);
        end_range = start_range + check.search_str_.size();
      } break;
      case CHECK_SAME: {
        auto pos = assertFind(source, check.search_str_, start_range, check);
        assertNotFind(SourceRange(source, prev.end(), pos), "\n", check);
        start_range = pos;
        end_range = pos + check.search_str_.size();
      } break;
      case CHECK_NEXT: {
        auto line_end = assertFind(source, "\n", start_range, check);
        auto pos = assertFind(source, check.search_str_, line_end + 1, check);
        assertNotFind(SourceRange(source, line_end + 1, pos), "\n", check);
        start_range = pos;
        end_range = pos + check.search_str_.size();
      } break;
```

- **EN:** Important callable entry points in this range include SourceRange, matchGroup, AT_ASSERT, matchDagGroup, assertNotFind.
- **CN:** 这一段的重要可调用入口包括 SourceRange, matchGroup, AT_ASSERT, matchDagGroup, assertNotFind。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Type system / 类型系统, Operator schema / 算子模式, Declared symbols / 声明的符号, Result propagation / 结果传递, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Type system / 类型系统, Operator schema / 算子模式, Declared symbols / 声明的符号, Result propagation / 结果传递, Branching logic / 分支逻辑。

### Lines 481-520 / 第 481-520 行

```cpp
      case CHECK_COUNT: {
        auto group_start_range = std::string::npos;
        AT_ASSERT(check.count_ && *check.count_ != 0);
        for (size_t i = 0; i < *check.count_; ++i) {
          start_range =
              assertFind(source, check.search_str_, start_range, check);
          group_start_range = std::min(start_range, group_start_range);
          end_range = start_range + check.search_str_.size();
          start_range = end_range;
        }
        start_range = group_start_range;
      } break;
      case CHECK_SOURCE_HIGHLIGHTED: {
        doCheckSourceHighlighted(check, source, start_range);
        break;
      }
      case CHECK_REGEX: {
        start_range =
            assertFindRegex(source, check.search_str_, start_range, check);
        end_range = start_range + check.search_str_.size();
        break;
      }
      default:
        TORCH_CHECK(false);
    }

    return SourceRange(source, start_range, end_range);
  }

  void doChecks(const std::shared_ptr<Source>& source) {
    SourceRange prev(source, 0, 0);
    for (size_t i = 0; i < groups.size(); i++) {
      const auto& curr_group = groups[i];
      CheckType type = curr_group.at(0).type_;
      if (type != CHECK_NOT) {
        prev = matchGroup(curr_group, source, prev);
      } else {
        if (i + 1 < groups.size()) {
          const auto& next_group = groups[i + 1];
          AT_ASSERT(next_group.at(0).type_ != CHECK_NOT);
```

- **EN:** Important callable entry points in this range include AT_ASSERT, assertFind, doCheckSourceHighlighted, assertFindRegex, TORCH_CHECK, SourceRange.
- **CN:** 这一段的重要可调用入口包括 AT_ASSERT, assertFind, doCheckSourceHighlighted, assertFindRegex, TORCH_CHECK, SourceRange。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 521-560 / 第 521-560 行

```cpp
          SourceRange after_not = matchGroup(next_group, source, prev);
          doCheckNot(curr_group, source, prev, after_not);
          prev = after_not;
          ++i; // already checked the group after
        } else {
          SourceRange end_of_file(
              source, source->size() + 1, source->size() + 1);
          doCheckNot(curr_group, source, prev, end_of_file);
        }
      }
    }
  }

  std::vector<Check> checks;
  std::vector<std::vector<Check>> groups;
};

FileCheck::FileCheck() : fcImpl(new FileCheckImpl()) {}

std::ostream& operator<<(std::ostream& out, const FileCheckImpl& fc) {
  out << "FileCheck checks:\n";
  for (const Check& c : fc.checks) {
    out << '\t' << c << '\n';
  }
  return out;
}

FileCheck::~FileCheck() {
  if (!fcImpl->has_run) {
    std::cout << "You have not run this instance of FileCheck!\n";
    std::cout << *fcImpl;
  }
  fcImpl.reset();
}

void FileCheck::run(const std::string& test_file) {
  fcImpl->run(test_file);
}

void FileCheck::run(const Graph& graph) {
```

- **EN:** Important callable entry points in this range include doCheckNot, end_of_file, FileCheck, run.
- **CN:** 这一段的重要可调用入口包括 doCheckNot, end_of_file, FileCheck, run。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 561-600 / 第 561-600 行

```cpp
  std::stringstream graph_str;
  graph_str << graph;
  fcImpl->run(graph_str.str());
}

void FileCheck::run(
    const std::string& input_checks_string,
    const std::string& test_string) {
  fcImpl->run(input_checks_string, test_string);
}

void FileCheck::run(
    const std::string& input_checks_string,
    const Graph& graph) {
  std::stringstream graph_str;
  graph_str << graph;
  fcImpl->run(input_checks_string, graph_str.str());
}

FileCheck* FileCheck::check(const std::string& str) {
  fcImpl->addCheck(CHECK, str);
  return this;
}

FileCheck* FileCheck::check_not(const std::string& str) {
  fcImpl->addCheck(CHECK_NOT, str);
  return this;
}

FileCheck* FileCheck::check_same(const std::string& str) {
  fcImpl->addCheck(CHECK_SAME, str);
  return this;
}

FileCheck* FileCheck::check_next(const std::string& str) {
  fcImpl->addCheck(CHECK_NEXT, str);
  return this;
}

FileCheck* FileCheck::check_count(
```

- **EN:** Important callable entry points in this range include run, check, check_not, check_same, check_next.
- **CN:** 这一段的重要可调用入口包括 run, check, check_not, check_same, check_next。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Operator schema / 算子模式, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Operator schema / 算子模式, Result propagation / 结果传递。

### Lines 601-630 / 第 601-630 行

```cpp
    const std::string& str,
    size_t count,
    bool exactly) {
  TORCH_INTERNAL_ASSERT(
      count != 0 || exactly, "Count == 0 && !exactly doesn't do anything");
  if (count) {
    fcImpl->addCheck(CHECK_COUNT, str, count);
  }
  if (exactly) {
    fcImpl->addCheck(CHECK_NOT, str);
  }
  return this;
}

FileCheck* FileCheck::check_dag(const std::string& str) {
  fcImpl->addCheck(CHECK_DAG, str);
  return this;
}

FileCheck* FileCheck::check_source_highlighted(const std::string& str) {
  fcImpl->addCheck(CHECK_SOURCE_HIGHLIGHTED, str);
  return this;
}

FileCheck* FileCheck::check_regex(const std::string& str) {
  fcImpl->addCheck(CHECK_REGEX, str);
  return this;
}

} // namespace torch::jit::testing
```

- **EN:** Important callable entry points in this range include TORCH_INTERNAL_ASSERT, check_dag, check_source_highlighted, check_regex.
- **CN:** 这一段的重要可调用入口包括 TORCH_INTERNAL_ASSERT, check_dag, check_source_highlighted, check_regex。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Operator schema / 算子模式, Declared symbols / 声明的符号, Namespace scoping / 命名空间作用域, Result propagation / 结果传递, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Operator schema / 算子模式, Declared symbols / 声明的符号, Namespace scoping / 命名空间作用域, Result propagation / 结果传递, Branching logic / 分支逻辑。

## Key Concepts / 关键概念

- **Testing hooks** — 测试钩子
- **Graph IR** — 图中间表示
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Backend integration** — 后端集成
- **Registration** — 注册机制
- **Core symbols: Check, FileCheckImpl, CheckType, assertFind, SourceRange, printQuotedString, extra_msg, runtime_error** — 核心符号：Check、FileCheckImpl、CheckType、assertFind、SourceRange、printQuotedString、extra_msg、runtime_error

## Dependencies / 依赖关系

- `c10/util/Exception.h`
- `c10/util/StringUtil.h`
- `c10/util/irange.h`
- `torch/csrc/jit/frontend/source_range.h`
- `torch/csrc/jit/ir/ir.h`
- `torch/csrc/jit/testing/file_check.h`
