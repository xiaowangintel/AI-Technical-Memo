# check-io.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Semantics/check-io.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements semantic checking, symbol reasoning, and language-rule enforcement for check io.
- **Purpose (CN)**: 实现 check io 相关的语义检查、符号推理与语言规则约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- lib/Semantics/check-io.cpp ----------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "check-io.h"
#include "definable.h"
#include "flang/Common/format.h"
#include "flang/Common/indirection.h"
#include "flang/Evaluate/tools.h"
#include "flang/Parser/characters.h"
#include "flang/Parser/tools.h"
#include "flang/Semantics/expression.h"
#include "flang/Semantics/tools.h"
#include <unordered_map>
#include <unordered_set>

namespace Fortran::semantics {

// TODO: C1234, C1235 -- defined I/O constraints

````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, intent, or metadata: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明附近代码的逻辑、意图或元数据：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, intent, or metadata: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明附近代码的逻辑、意图或元数据：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, intent, or metadata: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明附近代码的逻辑、意图或元数据：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L9 EN**: Includes "check-io.h" to access local declarations paired with this implementation.
  **L9 CN**: 引入 "check-io.h" 以使用与该实现配套的本地声明。
- **L10 EN**: Includes "definable.h" to access local declarations paired with this implementation.
  **L10 CN**: 引入 "definable.h" 以使用与该实现配套的本地声明。
- **L11 EN**: Includes "flang/Common/format.h" to access shared Flang utility infrastructure.
  **L11 CN**: 引入 "flang/Common/format.h" 以使用Flang 共享工具基础设施。
- **L12 EN**: Includes "flang/Common/indirection.h" to access shared Flang utility infrastructure.
  **L12 CN**: 引入 "flang/Common/indirection.h" 以使用Flang 共享工具基础设施。
- **L13 EN**: Includes "flang/Evaluate/tools.h" to access Fortran constant-folding and evaluation facilities.
  **L13 CN**: 引入 "flang/Evaluate/tools.h" 以使用Fortran 常量折叠与求值能力。
- **L14 EN**: Includes "flang/Parser/characters.h" to access parse-tree, token, or source representation support.
  **L14 CN**: 引入 "flang/Parser/characters.h" 以使用语法树、词法单元或源码表示支持。
- **L15 EN**: Includes "flang/Parser/tools.h" to access parse-tree, token, or source representation support.
  **L15 CN**: 引入 "flang/Parser/tools.h" 以使用语法树、词法单元或源码表示支持。
- **L16 EN**: Includes "flang/Semantics/expression.h" to access Fortran semantic analysis, symbol, and type information.
  **L16 CN**: 引入 "flang/Semantics/expression.h" 以使用Fortran 语义分析、符号与类型信息。
- **L17 EN**: Includes "flang/Semantics/tools.h" to access Fortran semantic analysis, symbol, and type information.
  **L17 CN**: 引入 "flang/Semantics/tools.h" 以使用Fortran 语义分析、符号与类型信息。
- **L18 EN**: Includes <unordered_map> to access supporting declarations used by this translation unit.
  **L18 CN**: 引入 <unordered_map> 以使用当前编译单元使用的辅助声明。
- **L19 EN**: Includes <unordered_set> to access supporting declarations used by this translation unit.
  **L19 CN**: 引入 <unordered_set> 以使用当前编译单元使用的辅助声明。
- **L20 EN**: Blank line separating nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L21 EN**: Opens namespace scope `Fortran::semantics`.
  **L21 CN**: 打开命名空间作用域 `Fortran::semantics`。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L23 EN**: Comment records a pending task or caution: `TODO: C1234, C1235 -- defined I/O constraints`.
  **L23 CN**: 注释记录待办事项或注意点：`TODO: C1234, C1235 -- defined I/O constraints`。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 25-48

````cpp
class FormatErrorReporter {
public:
  FormatErrorReporter(SemanticsContext &context,
      const parser::CharBlock &formatCharBlock, int errorAllowance = 3)
      : context_{context}, formatCharBlock_{formatCharBlock},
        errorAllowance_{errorAllowance} {}

  bool Say(const common::FormatMessage &);

private:
  SemanticsContext &context_;
  const parser::CharBlock &formatCharBlock_;
  int errorAllowance_; // initialized to maximum number of errors to report
};

bool FormatErrorReporter::Say(const common::FormatMessage &msg) {
  if (!msg.isError &&
      !context_.ShouldWarn(common::LanguageFeature::AdditionalFormats)) {
    return false;
  }
  parser::MessageFormattedText text{
      parser::MessageFixedText{msg.text, strlen(msg.text),
          msg.isError ? parser::Severity::Error : parser::Severity::Warning},
      msg.arg};
````
- **L25 EN**: Declares class `FormatErrorReporter`.
  **L25 CN**: 声明 class `FormatErrorReporter`。
- **L26 EN**: Sets the following members to `public` access.
  **L26 CN**: 将后续成员的访问级别设为 `public`。
- **L27 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `FormatErrorReporter(SemanticsContext &context,`.
  **L27 CN**: 继续一个多行参数列表、初始化器或聚合项：`FormatErrorReporter(SemanticsContext &context,`。
- **L28 EN**: Continues the surrounding expression or declaration: `const parser::CharBlock &formatCharBlock, int errorAllowance = 3)`.
  **L28 CN**: 继续构造周围的表达式或声明：`const parser::CharBlock &formatCharBlock, int errorAllowance = 3)`。
- **L29 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: context_{context}, formatCharBlock_{formatCharBlock},`.
  **L29 CN**: 继续一个多行参数列表、初始化器或聚合项：`: context_{context}, formatCharBlock_{formatCharBlock},`。
- **L30 EN**: Continues the surrounding expression or declaration: `errorAllowance_{errorAllowance} {}`.
  **L30 CN**: 继续构造周围的表达式或声明：`errorAllowance_{errorAllowance} {}`。
- **L31 EN**: Blank line separating nearby declarations or logic blocks.
  **L31 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L32 EN**: Executes a call or declaration centered on `Say`.
  **L32 CN**: 执行以 `Say` 为核心的调用或声明。
- **L33 EN**: Blank line separating nearby declarations or logic blocks.
  **L33 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L34 EN**: Sets the following members to `private` access.
  **L34 CN**: 将后续成员的访问级别设为 `private`。
- **L35 EN**: Executes a standalone statement or declaration: `SemanticsContext &context_;`.
  **L35 CN**: 执行一条独立语句或声明：`SemanticsContext &context_;`。
- **L36 EN**: Executes a standalone statement or declaration: `const parser::CharBlock &formatCharBlock_;`.
  **L36 CN**: 执行一条独立语句或声明：`const parser::CharBlock &formatCharBlock_;`。
- **L37 EN**: Continues the surrounding expression or declaration: `int errorAllowance_; // initialized to maximum number of errors to report`.
  **L37 CN**: 继续构造周围的表达式或声明：`int errorAllowance_; // initialized to maximum number of errors to report`。
- **L38 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L38 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L39 EN**: Blank line separating nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L40 EN**: Starts a function, method, lambda, or structured scope: `bool FormatErrorReporter::Say(const common::FormatMessage &msg) {`.
  **L40 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool FormatErrorReporter::Say(const common::FormatMessage &msg) {`。
- **L41 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L41 CN**: 开始 `if` 控制流语句并计算其条件。
- **L42 EN**: Starts a function, method, lambda, or structured scope: `!context_.ShouldWarn(common::LanguageFeature::AdditionalFormats)) {`.
  **L42 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!context_.ShouldWarn(common::LanguageFeature::AdditionalFormats)) {`。
- **L43 EN**: Returns from the current function with `false`.
  **L43 CN**: 以 `false` 从当前函数返回。
- **L44 EN**: Closes the current lexical scope or compound statement.
  **L44 CN**: 结束当前词法作用域或复合语句块。
- **L45 EN**: Continues the surrounding expression or declaration: `parser::MessageFormattedText text{`.
  **L45 CN**: 继续构造周围的表达式或声明：`parser::MessageFormattedText text{`。
- **L46 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::MessageFixedText{msg.text, strlen(msg.text),`.
  **L46 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::MessageFixedText{msg.text, strlen(msg.text),`。
- **L47 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `msg.isError ? parser::Severity::Error : parser::Severity::Warning},`.
  **L47 CN**: 继续一个多行参数列表、初始化器或聚合项：`msg.isError ? parser::Severity::Error : parser::Severity::Warning},`。
- **L48 EN**: Executes a standalone statement or declaration: `msg.arg};`.
  **L48 CN**: 执行一条独立语句或声明：`msg.arg};`。

### Lines 49-72

````cpp
  if (formatCharBlock_.size()) {
    // The input format is a folded expression.  Error markers span the full
    // original unfolded expression in formatCharBlock_.
    context_.Say(formatCharBlock_, text);
  } else {
    // The input format is a source expression.  Error markers have an offset
    // and length relative to the beginning of formatCharBlock_.
    parser::CharBlock messageCharBlock{
        parser::CharBlock(formatCharBlock_.begin() + msg.offset, msg.length)};
    context_.Say(messageCharBlock, text);
  }
  return msg.isError && --errorAllowance_ <= 0;
}

void IoChecker::Enter(
    const parser::Statement<common::Indirection<parser::FormatStmt>> &stmt) {
  if (!stmt.label) {
    context_.Say("Format statement must be labeled"_err_en_US); // C1301
  }
  const char *formatStart{static_cast<const char *>(
      std::memchr(stmt.source.begin(), '(', stmt.source.size()))};
  parser::CharBlock reporterCharBlock{formatStart, static_cast<std::size_t>(0)};
  FormatErrorReporter reporter{context_, reporterCharBlock};
  auto reporterWrapper{[&](const auto &msg) { return reporter.Say(msg); }};
````
- **L49 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L49 CN**: 开始 `if` 控制流语句并计算其条件。
- **L50 EN**: Comment explains nearby logic, intent, or metadata: `The input format is a folded expression.  Error markers span the full`.
  **L50 CN**: 注释说明附近代码的逻辑、意图或元数据：`The input format is a folded expression.  Error markers span the full`。
- **L51 EN**: Comment explains nearby logic, intent, or metadata: `original unfolded expression in formatCharBlock_.`.
  **L51 CN**: 注释说明附近代码的逻辑、意图或元数据：`original unfolded expression in formatCharBlock_.`。
- **L52 EN**: Executes a call or declaration centered on `context_.Say`.
  **L52 CN**: 执行以 `context_.Say` 为核心的调用或声明。
- **L53 EN**: Transitions from the previous branch into the alternative path.
  **L53 CN**: 从前一个分支过渡到备选路径。
- **L54 EN**: Comment explains nearby logic, intent, or metadata: `The input format is a source expression.  Error markers have an offset`.
  **L54 CN**: 注释说明附近代码的逻辑、意图或元数据：`The input format is a source expression.  Error markers have an offset`。
- **L55 EN**: Comment explains nearby logic, intent, or metadata: `and length relative to the beginning of formatCharBlock_.`.
  **L55 CN**: 注释说明附近代码的逻辑、意图或元数据：`and length relative to the beginning of formatCharBlock_.`。
- **L56 EN**: Continues the surrounding expression or declaration: `parser::CharBlock messageCharBlock{`.
  **L56 CN**: 继续构造周围的表达式或声明：`parser::CharBlock messageCharBlock{`。
- **L57 EN**: Executes a call or declaration centered on `parser::CharBlock`.
  **L57 CN**: 执行以 `parser::CharBlock` 为核心的调用或声明。
- **L58 EN**: Executes a call or declaration centered on `context_.Say`.
  **L58 CN**: 执行以 `context_.Say` 为核心的调用或声明。
- **L59 EN**: Closes the current lexical scope or compound statement.
  **L59 CN**: 结束当前词法作用域或复合语句块。
- **L60 EN**: Returns from the current function with `msg.isError && --errorAllowance_ <= 0`.
  **L60 CN**: 以 `msg.isError && --errorAllowance_ <= 0` 从当前函数返回。
- **L61 EN**: Closes the current lexical scope or compound statement.
  **L61 CN**: 结束当前词法作用域或复合语句块。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L63 EN**: Continues logic associated with callable symbol `Enter`.
  **L63 CN**: 继续与可调用符号 `Enter` 相关的逻辑。
- **L64 EN**: Continues the surrounding expression or declaration: `const parser::Statement<common::Indirection<parser::FormatStmt>> &stmt) {`.
  **L64 CN**: 继续构造周围的表达式或声明：`const parser::Statement<common::Indirection<parser::FormatStmt>> &stmt) {`。
- **L65 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L65 CN**: 开始 `if` 控制流语句并计算其条件。
- **L66 EN**: Continues logic associated with callable symbol `Say`.
  **L66 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L67 EN**: Closes the current lexical scope or compound statement.
  **L67 CN**: 结束当前词法作用域或复合语句块。
- **L68 EN**: Continues the surrounding expression or declaration: `const char *formatStart{static_cast<const char *>(`.
  **L68 CN**: 继续构造周围的表达式或声明：`const char *formatStart{static_cast<const char *>(`。
- **L69 EN**: Executes a call or declaration centered on `std::memchr`.
  **L69 CN**: 执行以 `std::memchr` 为核心的调用或声明。
- **L70 EN**: Executes a call or declaration centered on `static_cast<std::size_t>`.
  **L70 CN**: 执行以 `static_cast<std::size_t>` 为核心的调用或声明。
- **L71 EN**: Executes a standalone statement or declaration: `FormatErrorReporter reporter{context_, reporterCharBlock};`.
  **L71 CN**: 执行一条独立语句或声明：`FormatErrorReporter reporter{context_, reporterCharBlock};`。
- **L72 EN**: Executes a call or declaration centered on `reporterWrapper{[&]`.
  **L72 CN**: 执行以 `reporterWrapper{[&]` 为核心的调用或声明。

### Lines 73-96

````cpp
  switch (context_.GetDefaultKind(TypeCategory::Character)) {
  case 1: {
    common::FormatValidator<char> validator{formatStart,
        stmt.source.size() - (formatStart - stmt.source.begin()),
        reporterWrapper};
    validator.Check();
    break;
  }
  case 2: { // TODO: Get this to work.
    common::FormatValidator<char16_t> validator{
        /*???*/ nullptr, /*???*/ 0, reporterWrapper};
    validator.Check();
    break;
  }
  case 4: { // TODO: Get this to work.
    common::FormatValidator<char32_t> validator{
        /*???*/ nullptr, /*???*/ 0, reporterWrapper};
    validator.Check();
    break;
  }
  default:
    CRASH_NO_CASE;
  }
}
````
- **L73 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L73 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L74 EN**: Introduces a switch dispatch label: `case 1: {`.
  **L74 CN**: 引入一个 switch 分发标签：`case 1: {`。
- **L75 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::FormatValidator<char> validator{formatStart,`.
  **L75 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::FormatValidator<char> validator{formatStart,`。
- **L76 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `stmt.source.size() - (formatStart - stmt.source.begin()),`.
  **L76 CN**: 继续一个多行参数列表、初始化器或聚合项：`stmt.source.size() - (formatStart - stmt.source.begin()),`。
- **L77 EN**: Executes a standalone statement or declaration: `reporterWrapper};`.
  **L77 CN**: 执行一条独立语句或声明：`reporterWrapper};`。
- **L78 EN**: Executes a call or declaration centered on `validator.Check`.
  **L78 CN**: 执行以 `validator.Check` 为核心的调用或声明。
- **L79 EN**: Exits the nearest loop or switch statement.
  **L79 CN**: 退出最近的循环或 switch 语句。
- **L80 EN**: Closes the current lexical scope or compound statement.
  **L80 CN**: 结束当前词法作用域或复合语句块。
- **L81 EN**: Introduces a switch dispatch label: `case 2: { // TODO: Get this to work.`.
  **L81 CN**: 引入一个 switch 分发标签：`case 2: { // TODO: Get this to work.`。
- **L82 EN**: Continues the surrounding expression or declaration: `common::FormatValidator<char16_t> validator{`.
  **L82 CN**: 继续构造周围的表达式或声明：`common::FormatValidator<char16_t> validator{`。
- **L83 EN**: Comment explains nearby logic, intent, or metadata: `???*/ nullptr, /*???*/ 0, reporterWrapper};`.
  **L83 CN**: 注释说明附近代码的逻辑、意图或元数据：`???*/ nullptr, /*???*/ 0, reporterWrapper};`。
- **L84 EN**: Executes a call or declaration centered on `validator.Check`.
  **L84 CN**: 执行以 `validator.Check` 为核心的调用或声明。
- **L85 EN**: Exits the nearest loop or switch statement.
  **L85 CN**: 退出最近的循环或 switch 语句。
- **L86 EN**: Closes the current lexical scope or compound statement.
  **L86 CN**: 结束当前词法作用域或复合语句块。
- **L87 EN**: Introduces a switch dispatch label: `case 4: { // TODO: Get this to work.`.
  **L87 CN**: 引入一个 switch 分发标签：`case 4: { // TODO: Get this to work.`。
- **L88 EN**: Continues the surrounding expression or declaration: `common::FormatValidator<char32_t> validator{`.
  **L88 CN**: 继续构造周围的表达式或声明：`common::FormatValidator<char32_t> validator{`。
- **L89 EN**: Comment explains nearby logic, intent, or metadata: `???*/ nullptr, /*???*/ 0, reporterWrapper};`.
  **L89 CN**: 注释说明附近代码的逻辑、意图或元数据：`???*/ nullptr, /*???*/ 0, reporterWrapper};`。
- **L90 EN**: Executes a call or declaration centered on `validator.Check`.
  **L90 CN**: 执行以 `validator.Check` 为核心的调用或声明。
- **L91 EN**: Exits the nearest loop or switch statement.
  **L91 CN**: 退出最近的循环或 switch 语句。
- **L92 EN**: Closes the current lexical scope or compound statement.
  **L92 CN**: 结束当前词法作用域或复合语句块。
- **L93 EN**: Introduces a switch dispatch label: `default:`.
  **L93 CN**: 引入一个 switch 分发标签：`default:`。
- **L94 EN**: Executes a standalone statement or declaration: `CRASH_NO_CASE;`.
  **L94 CN**: 执行一条独立语句或声明：`CRASH_NO_CASE;`。
- **L95 EN**: Closes the current lexical scope or compound statement.
  **L95 CN**: 结束当前词法作用域或复合语句块。
- **L96 EN**: Closes the current lexical scope or compound statement.
  **L96 CN**: 结束当前词法作用域或复合语句块。

### Lines 97-120

````cpp

void IoChecker::Enter(const parser::ConnectSpec &spec) {
  // ConnectSpec context FileNameExpr
  if (std::get_if<parser::FileNameExpr>(&spec.u)) {
    SetSpecifier(IoSpecKind::File);
  }
}

// Ignore trailing spaces (12.5.6.2 p1) and convert to upper case
static std::string Normalize(const std::string &value) {
  auto upper{parser::ToUpperCaseLetters(value)};
  std::size_t lastNonBlank{upper.find_last_not_of(' ')};
  upper.resize(lastNonBlank == std::string::npos ? 0 : lastNonBlank + 1);
  return upper;
}

void IoChecker::Enter(const parser::ConnectSpec::CharExpr &spec) {
  IoSpecKind specKind{};
  using ParseKind = parser::ConnectSpec::CharExpr::Kind;
  switch (std::get<ParseKind>(spec.t)) {
  case ParseKind::Access:
    specKind = IoSpecKind::Access;
    break;
  case ParseKind::Action:
````
- **L97 EN**: Blank line separating nearby declarations or logic blocks.
  **L97 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L98 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::ConnectSpec &spec) {`.
  **L98 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::ConnectSpec &spec) {`。
- **L99 EN**: Comment explains nearby logic, intent, or metadata: `ConnectSpec context FileNameExpr`.
  **L99 CN**: 注释说明附近代码的逻辑、意图或元数据：`ConnectSpec context FileNameExpr`。
- **L100 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L100 CN**: 开始 `if` 控制流语句并计算其条件。
- **L101 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L101 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L102 EN**: Closes the current lexical scope or compound statement.
  **L102 CN**: 结束当前词法作用域或复合语句块。
- **L103 EN**: Closes the current lexical scope or compound statement.
  **L103 CN**: 结束当前词法作用域或复合语句块。
- **L104 EN**: Blank line separating nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L105 EN**: Comment explains nearby logic, intent, or metadata: `Ignore trailing spaces (12.5.6.2 p1) and convert to upper case`.
  **L105 CN**: 注释说明附近代码的逻辑、意图或元数据：`Ignore trailing spaces (12.5.6.2 p1) and convert to upper case`。
- **L106 EN**: Starts a function, method, lambda, or structured scope: `static std::string Normalize(const std::string &value) {`.
  **L106 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static std::string Normalize(const std::string &value) {`。
- **L107 EN**: Executes a call or declaration centered on `upper{parser::ToUpperCaseLetters`.
  **L107 CN**: 执行以 `upper{parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L108 EN**: Executes a call or declaration centered on `lastNonBlank{upper.find_last_not_of`.
  **L108 CN**: 执行以 `lastNonBlank{upper.find_last_not_of` 为核心的调用或声明。
- **L109 EN**: Executes a call or declaration centered on `upper.resize`.
  **L109 CN**: 执行以 `upper.resize` 为核心的调用或声明。
- **L110 EN**: Returns from the current function with `upper`.
  **L110 CN**: 以 `upper` 从当前函数返回。
- **L111 EN**: Closes the current lexical scope or compound statement.
  **L111 CN**: 结束当前词法作用域或复合语句块。
- **L112 EN**: Blank line separating nearby declarations or logic blocks.
  **L112 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L113 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::ConnectSpec::CharExpr &spec) {`.
  **L113 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::ConnectSpec::CharExpr &spec) {`。
- **L114 EN**: Executes a standalone statement or declaration: `IoSpecKind specKind{};`.
  **L114 CN**: 执行一条独立语句或声明：`IoSpecKind specKind{};`。
- **L115 EN**: Defines alias `ParseKind` to simplify later code.
  **L115 CN**: 定义别名 `ParseKind` 以简化后续代码。
- **L116 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L116 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L117 EN**: Introduces a switch dispatch label: `case ParseKind::Access:`.
  **L117 CN**: 引入一个 switch 分发标签：`case ParseKind::Access:`。
- **L118 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Access;`.
  **L118 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Access;`。
- **L119 EN**: Exits the nearest loop or switch statement.
  **L119 CN**: 退出最近的循环或 switch 语句。
- **L120 EN**: Introduces a switch dispatch label: `case ParseKind::Action:`.
  **L120 CN**: 引入一个 switch 分发标签：`case ParseKind::Action:`。

### Lines 121-144

````cpp
    specKind = IoSpecKind::Action;
    break;
  case ParseKind::Asynchronous:
    specKind = IoSpecKind::Asynchronous;
    break;
  case ParseKind::Blank:
    specKind = IoSpecKind::Blank;
    break;
  case ParseKind::Decimal:
    specKind = IoSpecKind::Decimal;
    break;
  case ParseKind::Delim:
    specKind = IoSpecKind::Delim;
    break;
  case ParseKind::Encoding:
    specKind = IoSpecKind::Encoding;
    break;
  case ParseKind::Form:
    specKind = IoSpecKind::Form;
    break;
  case ParseKind::Leading_Zero:
    specKind = IoSpecKind::Leading_Zero;
    break;
  case ParseKind::Pad:
````
- **L121 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Action;`.
  **L121 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Action;`。
- **L122 EN**: Exits the nearest loop or switch statement.
  **L122 CN**: 退出最近的循环或 switch 语句。
- **L123 EN**: Introduces a switch dispatch label: `case ParseKind::Asynchronous:`.
  **L123 CN**: 引入一个 switch 分发标签：`case ParseKind::Asynchronous:`。
- **L124 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Asynchronous;`.
  **L124 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Asynchronous;`。
- **L125 EN**: Exits the nearest loop or switch statement.
  **L125 CN**: 退出最近的循环或 switch 语句。
- **L126 EN**: Introduces a switch dispatch label: `case ParseKind::Blank:`.
  **L126 CN**: 引入一个 switch 分发标签：`case ParseKind::Blank:`。
- **L127 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Blank;`.
  **L127 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Blank;`。
- **L128 EN**: Exits the nearest loop or switch statement.
  **L128 CN**: 退出最近的循环或 switch 语句。
- **L129 EN**: Introduces a switch dispatch label: `case ParseKind::Decimal:`.
  **L129 CN**: 引入一个 switch 分发标签：`case ParseKind::Decimal:`。
- **L130 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Decimal;`.
  **L130 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Decimal;`。
- **L131 EN**: Exits the nearest loop or switch statement.
  **L131 CN**: 退出最近的循环或 switch 语句。
- **L132 EN**: Introduces a switch dispatch label: `case ParseKind::Delim:`.
  **L132 CN**: 引入一个 switch 分发标签：`case ParseKind::Delim:`。
- **L133 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Delim;`.
  **L133 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Delim;`。
- **L134 EN**: Exits the nearest loop or switch statement.
  **L134 CN**: 退出最近的循环或 switch 语句。
- **L135 EN**: Introduces a switch dispatch label: `case ParseKind::Encoding:`.
  **L135 CN**: 引入一个 switch 分发标签：`case ParseKind::Encoding:`。
- **L136 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Encoding;`.
  **L136 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Encoding;`。
- **L137 EN**: Exits the nearest loop or switch statement.
  **L137 CN**: 退出最近的循环或 switch 语句。
- **L138 EN**: Introduces a switch dispatch label: `case ParseKind::Form:`.
  **L138 CN**: 引入一个 switch 分发标签：`case ParseKind::Form:`。
- **L139 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Form;`.
  **L139 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Form;`。
- **L140 EN**: Exits the nearest loop or switch statement.
  **L140 CN**: 退出最近的循环或 switch 语句。
- **L141 EN**: Introduces a switch dispatch label: `case ParseKind::Leading_Zero:`.
  **L141 CN**: 引入一个 switch 分发标签：`case ParseKind::Leading_Zero:`。
- **L142 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Leading_Zero;`.
  **L142 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Leading_Zero;`。
- **L143 EN**: Exits the nearest loop or switch statement.
  **L143 CN**: 退出最近的循环或 switch 语句。
- **L144 EN**: Introduces a switch dispatch label: `case ParseKind::Pad:`.
  **L144 CN**: 引入一个 switch 分发标签：`case ParseKind::Pad:`。

### Lines 145-168

````cpp
    specKind = IoSpecKind::Pad;
    break;
  case ParseKind::Position:
    specKind = IoSpecKind::Position;
    break;
  case ParseKind::Round:
    specKind = IoSpecKind::Round;
    break;
  case ParseKind::Sign:
    specKind = IoSpecKind::Sign;
    break;
  case ParseKind::Carriagecontrol:
    specKind = IoSpecKind::Carriagecontrol;
    break;
  case ParseKind::Convert:
    specKind = IoSpecKind::Convert;
    break;
  case ParseKind::Dispose:
    specKind = IoSpecKind::Dispose;
    break;
  }
  SetSpecifier(specKind);
  if (const std::optional<std::string> charConst{GetConstExpr<std::string>(
          std::get<parser::ScalarDefaultCharExpr>(spec.t))}) {
````
- **L145 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Pad;`.
  **L145 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Pad;`。
- **L146 EN**: Exits the nearest loop or switch statement.
  **L146 CN**: 退出最近的循环或 switch 语句。
- **L147 EN**: Introduces a switch dispatch label: `case ParseKind::Position:`.
  **L147 CN**: 引入一个 switch 分发标签：`case ParseKind::Position:`。
- **L148 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Position;`.
  **L148 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Position;`。
- **L149 EN**: Exits the nearest loop or switch statement.
  **L149 CN**: 退出最近的循环或 switch 语句。
- **L150 EN**: Introduces a switch dispatch label: `case ParseKind::Round:`.
  **L150 CN**: 引入一个 switch 分发标签：`case ParseKind::Round:`。
- **L151 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Round;`.
  **L151 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Round;`。
- **L152 EN**: Exits the nearest loop or switch statement.
  **L152 CN**: 退出最近的循环或 switch 语句。
- **L153 EN**: Introduces a switch dispatch label: `case ParseKind::Sign:`.
  **L153 CN**: 引入一个 switch 分发标签：`case ParseKind::Sign:`。
- **L154 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Sign;`.
  **L154 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Sign;`。
- **L155 EN**: Exits the nearest loop or switch statement.
  **L155 CN**: 退出最近的循环或 switch 语句。
- **L156 EN**: Introduces a switch dispatch label: `case ParseKind::Carriagecontrol:`.
  **L156 CN**: 引入一个 switch 分发标签：`case ParseKind::Carriagecontrol:`。
- **L157 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Carriagecontrol;`.
  **L157 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Carriagecontrol;`。
- **L158 EN**: Exits the nearest loop or switch statement.
  **L158 CN**: 退出最近的循环或 switch 语句。
- **L159 EN**: Introduces a switch dispatch label: `case ParseKind::Convert:`.
  **L159 CN**: 引入一个 switch 分发标签：`case ParseKind::Convert:`。
- **L160 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Convert;`.
  **L160 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Convert;`。
- **L161 EN**: Exits the nearest loop or switch statement.
  **L161 CN**: 退出最近的循环或 switch 语句。
- **L162 EN**: Introduces a switch dispatch label: `case ParseKind::Dispose:`.
  **L162 CN**: 引入一个 switch 分发标签：`case ParseKind::Dispose:`。
- **L163 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Dispose;`.
  **L163 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Dispose;`。
- **L164 EN**: Exits the nearest loop or switch statement.
  **L164 CN**: 退出最近的循环或 switch 语句。
- **L165 EN**: Closes the current lexical scope or compound statement.
  **L165 CN**: 结束当前词法作用域或复合语句块。
- **L166 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L166 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L167 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L167 CN**: 开始 `if` 控制流语句并计算其条件。
- **L168 EN**: Starts a function, method, lambda, or structured scope: `std::get<parser::ScalarDefaultCharExpr>(spec.t))}) {`.
  **L168 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<parser::ScalarDefaultCharExpr>(spec.t))}) {`。

### Lines 169-192

````cpp
    std::string s{Normalize(*charConst)};
    if (specKind == IoSpecKind::Access) {
      flags_.set(Flag::KnownAccess);
      flags_.set(Flag::AccessDirect, s == "DIRECT");
      flags_.set(Flag::AccessStream, s == "STREAM");
    }
    CheckStringValue(specKind, *charConst, parser::FindSourceLocation(spec));
    if (specKind == IoSpecKind::Carriagecontrol &&
        (s == "FORTRAN" || s == "NONE")) {
      context_.Say(parser::FindSourceLocation(spec),
          "Unimplemented %s value '%s'"_err_en_US,
          parser::ToUpperCaseLetters(common::EnumToString(specKind)),
          *charConst);
    }
  }
}

void IoChecker::Enter(const parser::ConnectSpec::Newunit &var) {
  CheckForDefinableVariable(var, "NEWUNIT");
  SetSpecifier(IoSpecKind::Newunit);
}

void IoChecker::Enter(const parser::ConnectSpec::Recl &spec) {
  SetSpecifier(IoSpecKind::Recl);
````
- **L169 EN**: Executes a call or declaration centered on `s{Normalize`.
  **L169 CN**: 执行以 `s{Normalize` 为核心的调用或声明。
- **L170 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L170 CN**: 开始 `if` 控制流语句并计算其条件。
- **L171 EN**: Executes a call or declaration centered on `flags_.set`.
  **L171 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L172 EN**: Executes a call or declaration centered on `flags_.set`.
  **L172 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L173 EN**: Executes a call or declaration centered on `flags_.set`.
  **L173 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L174 EN**: Closes the current lexical scope or compound statement.
  **L174 CN**: 结束当前词法作用域或复合语句块。
- **L175 EN**: Executes a call or declaration centered on `CheckStringValue`.
  **L175 CN**: 执行以 `CheckStringValue` 为核心的调用或声明。
- **L176 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L176 CN**: 开始 `if` 控制流语句并计算其条件。
- **L177 EN**: Starts a function, method, lambda, or structured scope: `(s == "FORTRAN" || s == "NONE")) {`.
  **L177 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(s == "FORTRAN" || s == "NONE")) {`。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(parser::FindSourceLocation(spec),`.
  **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(parser::FindSourceLocation(spec),`。
- **L179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Unimplemented %s value '%s'"_err_en_US,`.
  **L179 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Unimplemented %s value '%s'"_err_en_US,`。
- **L180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(common::EnumToString(specKind)),`.
  **L180 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(common::EnumToString(specKind)),`。
- **L181 EN**: Comment explains nearby logic, intent, or metadata: `charConst);`.
  **L181 CN**: 注释说明附近代码的逻辑、意图或元数据：`charConst);`。
- **L182 EN**: Closes the current lexical scope or compound statement.
  **L182 CN**: 结束当前词法作用域或复合语句块。
- **L183 EN**: Closes the current lexical scope or compound statement.
  **L183 CN**: 结束当前词法作用域或复合语句块。
- **L184 EN**: Closes the current lexical scope or compound statement.
  **L184 CN**: 结束当前词法作用域或复合语句块。
- **L185 EN**: Blank line separating nearby declarations or logic blocks.
  **L185 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L186 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::ConnectSpec::Newunit &var) {`.
  **L186 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::ConnectSpec::Newunit &var) {`。
- **L187 EN**: Executes a call or declaration centered on `CheckForDefinableVariable`.
  **L187 CN**: 执行以 `CheckForDefinableVariable` 为核心的调用或声明。
- **L188 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L188 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L189 EN**: Closes the current lexical scope or compound statement.
  **L189 CN**: 结束当前词法作用域或复合语句块。
- **L190 EN**: Blank line separating nearby declarations or logic blocks.
  **L190 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L191 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::ConnectSpec::Recl &spec) {`.
  **L191 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::ConnectSpec::Recl &spec) {`。
- **L192 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L192 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。

### Lines 193-216

````cpp
  if (const std::optional<std::int64_t> recl{
          GetConstExpr<std::int64_t>(spec)}) {
    if (*recl <= 0) {
      context_.Say(parser::FindSourceLocation(spec),
          "RECL value (%jd) must be positive"_err_en_US,
          *recl); // 12.5.6.15
    }
  }
}

void IoChecker::Enter(const parser::EndLabel &) {
  SetSpecifier(IoSpecKind::End);
}

void IoChecker::Enter(const parser::EorLabel &) {
  SetSpecifier(IoSpecKind::Eor);
}

void IoChecker::Enter(const parser::ErrLabel &) {
  SetSpecifier(IoSpecKind::Err);
}

void IoChecker::Enter(const parser::FileUnitNumber &) {
  SetSpecifier(IoSpecKind::Unit);
````
- **L193 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L193 CN**: 开始 `if` 控制流语句并计算其条件。
- **L194 EN**: Starts a function, method, lambda, or structured scope: `GetConstExpr<std::int64_t>(spec)}) {`.
  **L194 CN**: 开始一个函数、方法、lambda 或结构化作用域：`GetConstExpr<std::int64_t>(spec)}) {`。
- **L195 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L195 CN**: 开始 `if` 控制流语句并计算其条件。
- **L196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(parser::FindSourceLocation(spec),`.
  **L196 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(parser::FindSourceLocation(spec),`。
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"RECL value (%jd) must be positive"_err_en_US,`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`"RECL value (%jd) must be positive"_err_en_US,`。
- **L198 EN**: Comment explains nearby logic, intent, or metadata: `recl); // 12.5.6.15`.
  **L198 CN**: 注释说明附近代码的逻辑、意图或元数据：`recl); // 12.5.6.15`。
- **L199 EN**: Closes the current lexical scope or compound statement.
  **L199 CN**: 结束当前词法作用域或复合语句块。
- **L200 EN**: Closes the current lexical scope or compound statement.
  **L200 CN**: 结束当前词法作用域或复合语句块。
- **L201 EN**: Closes the current lexical scope or compound statement.
  **L201 CN**: 结束当前词法作用域或复合语句块。
- **L202 EN**: Blank line separating nearby declarations or logic blocks.
  **L202 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L203 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::EndLabel &) {`.
  **L203 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::EndLabel &) {`。
- **L204 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L204 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L205 EN**: Closes the current lexical scope or compound statement.
  **L205 CN**: 结束当前词法作用域或复合语句块。
- **L206 EN**: Blank line separating nearby declarations or logic blocks.
  **L206 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L207 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::EorLabel &) {`.
  **L207 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::EorLabel &) {`。
- **L208 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L208 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L209 EN**: Closes the current lexical scope or compound statement.
  **L209 CN**: 结束当前词法作用域或复合语句块。
- **L210 EN**: Blank line separating nearby declarations or logic blocks.
  **L210 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L211 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::ErrLabel &) {`.
  **L211 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::ErrLabel &) {`。
- **L212 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L212 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L213 EN**: Closes the current lexical scope or compound statement.
  **L213 CN**: 结束当前词法作用域或复合语句块。
- **L214 EN**: Blank line separating nearby declarations or logic blocks.
  **L214 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L215 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::FileUnitNumber &) {`.
  **L215 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::FileUnitNumber &) {`。
- **L216 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L216 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。

### Lines 217-240

````cpp
  flags_.set(Flag::NumberUnit);
}

void IoChecker::Enter(const parser::Format &spec) {
  SetSpecifier(IoSpecKind::Fmt);
  flags_.set(Flag::FmtOrNml);
  common::visit(
      common::visitors{
          [&](const parser::Label &) { flags_.set(Flag::LabelFmt); },
          [&](const parser::Star &) { flags_.set(Flag::StarFmt); },
          [&](const parser::Expr &format) {
            const SomeExpr *expr{GetExpr(context_, format)};
            if (!expr) {
              return;
            }
            auto type{expr->GetType()};
            if (type && type->category() == TypeCategory::Integer &&
                type->kind() ==
                    context_.defaultKinds().GetDefaultKind(type->category()) &&
                expr->Rank() == 0) {
              flags_.set(Flag::AssignFmt);
              if (!IsVariable(*expr)) {
                context_.Say(format.source,
                    "Assigned format label must be a scalar variable"_err_en_US);
````
- **L217 EN**: Executes a call or declaration centered on `flags_.set`.
  **L217 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L218 EN**: Closes the current lexical scope or compound statement.
  **L218 CN**: 结束当前词法作用域或复合语句块。
- **L219 EN**: Blank line separating nearby declarations or logic blocks.
  **L219 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L220 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::Format &spec) {`.
  **L220 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::Format &spec) {`。
- **L221 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L221 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L222 EN**: Executes a call or declaration centered on `flags_.set`.
  **L222 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L223 EN**: Continues logic associated with callable symbol `visit`.
  **L223 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L224 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L224 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](const parser::Label &) { flags_.set(Flag::LabelFmt); },`.
  **L225 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](const parser::Label &) { flags_.set(Flag::LabelFmt); },`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](const parser::Star &) { flags_.set(Flag::StarFmt); },`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](const parser::Star &) { flags_.set(Flag::StarFmt); },`。
- **L227 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::Expr &format) {`.
  **L227 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::Expr &format) {`。
- **L228 EN**: Executes a call or declaration centered on `*expr{GetExpr`.
  **L228 CN**: 执行以 `*expr{GetExpr` 为核心的调用或声明。
- **L229 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L229 CN**: 开始 `if` 控制流语句并计算其条件。
- **L230 EN**: Returns from the current function with `void`.
  **L230 CN**: 以 `void` 从当前函数返回。
- **L231 EN**: Closes the current lexical scope or compound statement.
  **L231 CN**: 结束当前词法作用域或复合语句块。
- **L232 EN**: Executes a call or declaration centered on `type{expr->GetType`.
  **L232 CN**: 执行以 `type{expr->GetType` 为核心的调用或声明。
- **L233 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L233 CN**: 开始 `if` 控制流语句并计算其条件。
- **L234 EN**: Continues logic associated with callable symbol `kind`.
  **L234 CN**: 继续与可调用符号 `kind` 相关的逻辑。
- **L235 EN**: Continues logic associated with callable symbol `defaultKinds`.
  **L235 CN**: 继续与可调用符号 `defaultKinds` 相关的逻辑。
- **L236 EN**: Starts a function, method, lambda, or structured scope: `expr->Rank() == 0) {`.
  **L236 CN**: 开始一个函数、方法、lambda 或结构化作用域：`expr->Rank() == 0) {`。
- **L237 EN**: Executes a call or declaration centered on `flags_.set`.
  **L237 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L238 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L238 CN**: 开始 `if` 控制流语句并计算其条件。
- **L239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(format.source,`.
  **L239 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(format.source,`。
- **L240 EN**: Executes a standalone statement or declaration: `"Assigned format label must be a scalar variable"_err_en_US);`.
  **L240 CN**: 执行一条独立语句或声明：`"Assigned format label must be a scalar variable"_err_en_US);`。

### Lines 241-264

````cpp
              } else {
                context_.Warn(common::LanguageFeature::Assign, format.source,
                    "Assigned format labels are deprecated"_port_en_US);
              }
              return;
            }
            if (type && type->category() != TypeCategory::Character &&
                (type->category() != TypeCategory::Integer ||
                    expr->Rank() > 0) &&
                context_.IsEnabled(
                    common::LanguageFeature::NonCharacterFormat)) {
              // Legacy extension: using non-character variables, typically
              // DATA-initialized with Hollerith, as format expressions.
              context_.Warn(common::LanguageFeature::NonCharacterFormat,
                  format.source,
                  "Non-character format expression is not standard"_port_en_US);
            } else if (!type ||
                type->kind() !=
                    context_.defaultKinds().GetDefaultKind(type->category())) {
              context_.Say(format.source,
                  "Format expression must be default character or default scalar integer"_err_en_US);
              return;
            }
            flags_.set(Flag::CharFmt);
````
- **L241 EN**: Transitions from the previous branch into the alternative path.
  **L241 CN**: 从前一个分支过渡到备选路径。
- **L242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Warn(common::LanguageFeature::Assign, format.source,`.
  **L242 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Warn(common::LanguageFeature::Assign, format.source,`。
- **L243 EN**: Executes a standalone statement or declaration: `"Assigned format labels are deprecated"_port_en_US);`.
  **L243 CN**: 执行一条独立语句或声明：`"Assigned format labels are deprecated"_port_en_US);`。
- **L244 EN**: Closes the current lexical scope or compound statement.
  **L244 CN**: 结束当前词法作用域或复合语句块。
- **L245 EN**: Returns from the current function with `void`.
  **L245 CN**: 以 `void` 从当前函数返回。
- **L246 EN**: Closes the current lexical scope or compound statement.
  **L246 CN**: 结束当前词法作用域或复合语句块。
- **L247 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L247 CN**: 开始 `if` 控制流语句并计算其条件。
- **L248 EN**: Continues logic associated with callable symbol `category`.
  **L248 CN**: 继续与可调用符号 `category` 相关的逻辑。
- **L249 EN**: Continues logic associated with callable symbol `Rank`.
  **L249 CN**: 继续与可调用符号 `Rank` 相关的逻辑。
- **L250 EN**: Continues logic associated with callable symbol `IsEnabled`.
  **L250 CN**: 继续与可调用符号 `IsEnabled` 相关的逻辑。
- **L251 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::NonCharacterFormat)) {`.
  **L251 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::NonCharacterFormat)) {`。
- **L252 EN**: Comment explains nearby logic, intent, or metadata: `Legacy extension: using non-character variables, typically`.
  **L252 CN**: 注释说明附近代码的逻辑、意图或元数据：`Legacy extension: using non-character variables, typically`。
- **L253 EN**: Comment explains nearby logic, intent, or metadata: `DATA-initialized with Hollerith, as format expressions.`.
  **L253 CN**: 注释说明附近代码的逻辑、意图或元数据：`DATA-initialized with Hollerith, as format expressions.`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Warn(common::LanguageFeature::NonCharacterFormat,`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Warn(common::LanguageFeature::NonCharacterFormat,`。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `format.source,`.
  **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`format.source,`。
- **L256 EN**: Executes a standalone statement or declaration: `"Non-character format expression is not standard"_port_en_US);`.
  **L256 CN**: 执行一条独立语句或声明：`"Non-character format expression is not standard"_port_en_US);`。
- **L257 EN**: Transitions from the previous branch into an `else if` condition.
  **L257 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L258 EN**: Continues logic associated with callable symbol `kind`.
  **L258 CN**: 继续与可调用符号 `kind` 相关的逻辑。
- **L259 EN**: Starts a function, method, lambda, or structured scope: `context_.defaultKinds().GetDefaultKind(type->category())) {`.
  **L259 CN**: 开始一个函数、方法、lambda 或结构化作用域：`context_.defaultKinds().GetDefaultKind(type->category())) {`。
- **L260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(format.source,`.
  **L260 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(format.source,`。
- **L261 EN**: Executes a standalone statement or declaration: `"Format expression must be default character or default scalar integer"_err_en_US);`.
  **L261 CN**: 执行一条独立语句或声明：`"Format expression must be default character or default scalar integer"_err_en_US);`。
- **L262 EN**: Returns from the current function with `void`.
  **L262 CN**: 以 `void` 从当前函数返回。
- **L263 EN**: Closes the current lexical scope or compound statement.
  **L263 CN**: 结束当前词法作用域或复合语句块。
- **L264 EN**: Executes a call or declaration centered on `flags_.set`.
  **L264 CN**: 执行以 `flags_.set` 为核心的调用或声明。

### Lines 265-288

````cpp
            const std::optional<std::string> constantFormat{
                GetConstExpr<std::string>(format)};
            if (!constantFormat) {
              return;
            }
            // validate constant format -- 12.6.2.2
            bool isFolded{constantFormat->size() != format.source.size() - 2};
            parser::CharBlock reporterCharBlock{isFolded
                    ? parser::CharBlock{format.source}
                    : parser::CharBlock{format.source.begin() + 1,
                          static_cast<std::size_t>(0)}};
            FormatErrorReporter reporter{context_, reporterCharBlock};
            auto reporterWrapper{
                [&](const auto &msg) { return reporter.Say(msg); }};
            switch (context_.GetDefaultKind(TypeCategory::Character)) {
            case 1: {
              common::FormatValidator<char> validator{constantFormat->c_str(),
                  constantFormat->length(), reporterWrapper, stmt_};
              validator.Check();
              break;
            }
            case 2: {
              // TODO: Get this to work.  (Maybe combine with earlier instance?)
              common::FormatValidator<char16_t> validator{
````
- **L265 EN**: Continues the surrounding expression or declaration: `const std::optional<std::string> constantFormat{`.
  **L265 CN**: 继续构造周围的表达式或声明：`const std::optional<std::string> constantFormat{`。
- **L266 EN**: Executes a call or declaration centered on `GetConstExpr<std::string>`.
  **L266 CN**: 执行以 `GetConstExpr<std::string>` 为核心的调用或声明。
- **L267 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L267 CN**: 开始 `if` 控制流语句并计算其条件。
- **L268 EN**: Returns from the current function with `void`.
  **L268 CN**: 以 `void` 从当前函数返回。
- **L269 EN**: Closes the current lexical scope or compound statement.
  **L269 CN**: 结束当前词法作用域或复合语句块。
- **L270 EN**: Comment explains nearby logic, intent, or metadata: `validate constant format -- 12.6.2.2`.
  **L270 CN**: 注释说明附近代码的逻辑、意图或元数据：`validate constant format -- 12.6.2.2`。
- **L271 EN**: Executes a call or declaration centered on `isFolded{constantFormat->size`.
  **L271 CN**: 执行以 `isFolded{constantFormat->size` 为核心的调用或声明。
- **L272 EN**: Continues the surrounding expression or declaration: `parser::CharBlock reporterCharBlock{isFolded`.
  **L272 CN**: 继续构造周围的表达式或声明：`parser::CharBlock reporterCharBlock{isFolded`。
- **L273 EN**: Continues the surrounding expression or declaration: `? parser::CharBlock{format.source}`.
  **L273 CN**: 继续构造周围的表达式或声明：`? parser::CharBlock{format.source}`。
- **L274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: parser::CharBlock{format.source.begin() + 1,`.
  **L274 CN**: 继续一个多行参数列表、初始化器或聚合项：`: parser::CharBlock{format.source.begin() + 1,`。
- **L275 EN**: Executes a call or declaration centered on `static_cast<std::size_t>`.
  **L275 CN**: 执行以 `static_cast<std::size_t>` 为核心的调用或声明。
- **L276 EN**: Executes a standalone statement or declaration: `FormatErrorReporter reporter{context_, reporterCharBlock};`.
  **L276 CN**: 执行一条独立语句或声明：`FormatErrorReporter reporter{context_, reporterCharBlock};`。
- **L277 EN**: Continues the surrounding expression or declaration: `auto reporterWrapper{`.
  **L277 CN**: 继续构造周围的表达式或声明：`auto reporterWrapper{`。
- **L278 EN**: Executes a call or declaration centered on `[&]`.
  **L278 CN**: 执行以 `[&]` 为核心的调用或声明。
- **L279 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L279 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L280 EN**: Introduces a switch dispatch label: `case 1: {`.
  **L280 CN**: 引入一个 switch 分发标签：`case 1: {`。
- **L281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::FormatValidator<char> validator{constantFormat->c_str(),`.
  **L281 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::FormatValidator<char> validator{constantFormat->c_str(),`。
- **L282 EN**: Executes a call or declaration centered on `constantFormat->length`.
  **L282 CN**: 执行以 `constantFormat->length` 为核心的调用或声明。
- **L283 EN**: Executes a call or declaration centered on `validator.Check`.
  **L283 CN**: 执行以 `validator.Check` 为核心的调用或声明。
- **L284 EN**: Exits the nearest loop or switch statement.
  **L284 CN**: 退出最近的循环或 switch 语句。
- **L285 EN**: Closes the current lexical scope or compound statement.
  **L285 CN**: 结束当前词法作用域或复合语句块。
- **L286 EN**: Introduces a switch dispatch label: `case 2: {`.
  **L286 CN**: 引入一个 switch 分发标签：`case 2: {`。
- **L287 EN**: Comment records a pending task or caution: `TODO: Get this to work.  (Maybe combine with earlier instance?)`.
  **L287 CN**: 注释记录待办事项或注意点：`TODO: Get this to work.  (Maybe combine with earlier instance?)`。
- **L288 EN**: Continues the surrounding expression or declaration: `common::FormatValidator<char16_t> validator{`.
  **L288 CN**: 继续构造周围的表达式或声明：`common::FormatValidator<char16_t> validator{`。

### Lines 289-312

````cpp
                  /*???*/ nullptr, /*???*/ 0, reporterWrapper, stmt_};
              validator.Check();
              break;
            }
            case 4: {
              // TODO: Get this to work.  (Maybe combine with earlier instance?)
              common::FormatValidator<char32_t> validator{
                  /*???*/ nullptr, /*???*/ 0, reporterWrapper, stmt_};
              validator.Check();
              break;
            }
            default:
              CRASH_NO_CASE;
            }
          },
      },
      spec.u);
}

void IoChecker::Enter(const parser::IdExpr &) { SetSpecifier(IoSpecKind::Id); }

void IoChecker::Enter(const parser::IdVariable &spec) {
  SetSpecifier(IoSpecKind::Id);
  const auto *expr{GetExpr(context_, spec)};
````
- **L289 EN**: Comment explains nearby logic, intent, or metadata: `???*/ nullptr, /*???*/ 0, reporterWrapper, stmt_};`.
  **L289 CN**: 注释说明附近代码的逻辑、意图或元数据：`???*/ nullptr, /*???*/ 0, reporterWrapper, stmt_};`。
- **L290 EN**: Executes a call or declaration centered on `validator.Check`.
  **L290 CN**: 执行以 `validator.Check` 为核心的调用或声明。
- **L291 EN**: Exits the nearest loop or switch statement.
  **L291 CN**: 退出最近的循环或 switch 语句。
- **L292 EN**: Closes the current lexical scope or compound statement.
  **L292 CN**: 结束当前词法作用域或复合语句块。
- **L293 EN**: Introduces a switch dispatch label: `case 4: {`.
  **L293 CN**: 引入一个 switch 分发标签：`case 4: {`。
- **L294 EN**: Comment records a pending task or caution: `TODO: Get this to work.  (Maybe combine with earlier instance?)`.
  **L294 CN**: 注释记录待办事项或注意点：`TODO: Get this to work.  (Maybe combine with earlier instance?)`。
- **L295 EN**: Continues the surrounding expression or declaration: `common::FormatValidator<char32_t> validator{`.
  **L295 CN**: 继续构造周围的表达式或声明：`common::FormatValidator<char32_t> validator{`。
- **L296 EN**: Comment explains nearby logic, intent, or metadata: `???*/ nullptr, /*???*/ 0, reporterWrapper, stmt_};`.
  **L296 CN**: 注释说明附近代码的逻辑、意图或元数据：`???*/ nullptr, /*???*/ 0, reporterWrapper, stmt_};`。
- **L297 EN**: Executes a call or declaration centered on `validator.Check`.
  **L297 CN**: 执行以 `validator.Check` 为核心的调用或声明。
- **L298 EN**: Exits the nearest loop or switch statement.
  **L298 CN**: 退出最近的循环或 switch 语句。
- **L299 EN**: Closes the current lexical scope or compound statement.
  **L299 CN**: 结束当前词法作用域或复合语句块。
- **L300 EN**: Introduces a switch dispatch label: `default:`.
  **L300 CN**: 引入一个 switch 分发标签：`default:`。
- **L301 EN**: Executes a standalone statement or declaration: `CRASH_NO_CASE;`.
  **L301 CN**: 执行一条独立语句或声明：`CRASH_NO_CASE;`。
- **L302 EN**: Closes the current lexical scope or compound statement.
  **L302 CN**: 结束当前词法作用域或复合语句块。
- **L303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L303 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L304 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L305 EN**: Executes a standalone statement or declaration: `spec.u);`.
  **L305 CN**: 执行一条独立语句或声明：`spec.u);`。
- **L306 EN**: Closes the current lexical scope or compound statement.
  **L306 CN**: 结束当前词法作用域或复合语句块。
- **L307 EN**: Blank line separating nearby declarations or logic blocks.
  **L307 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L308 EN**: Continues logic associated with callable symbol `Enter`.
  **L308 CN**: 继续与可调用符号 `Enter` 相关的逻辑。
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L310 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::IdVariable &spec) {`.
  **L310 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::IdVariable &spec) {`。
- **L311 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L311 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L312 EN**: Executes a call or declaration centered on `*expr{GetExpr`.
  **L312 CN**: 执行以 `*expr{GetExpr` 为核心的调用或声明。

### Lines 313-336

````cpp
  if (!expr || !expr->GetType()) {
    return;
  }
  CheckForDefinableVariable(spec, "ID");
  int kind{expr->GetType()->kind()};
  int defaultKind{context_.GetDefaultKind(TypeCategory::Integer)};
  if (kind < defaultKind) {
    context_.Say(
        "ID kind (%d) is smaller than default INTEGER kind (%d)"_err_en_US,
        std::move(kind), std::move(defaultKind)); // C1229
  }
}

void IoChecker::Enter(const parser::InputItem &spec) {
  flags_.set(Flag::DataList);
  const parser::Variable *var{std::get_if<parser::Variable>(&spec.u)};
  if (!var) {
    return;
  }
  CheckForDefinableVariable(*var, "Input");
  if (auto expr{AnalyzeExpr(context_, *var)}) {
    auto at{var->GetSource()};
    CheckForAssumedRank(UnwrapWholeSymbolDataRef(*expr), at);
    CheckForBadIoType(*expr,
````
- **L313 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L313 CN**: 开始 `if` 控制流语句并计算其条件。
- **L314 EN**: Returns from the current function with `void`.
  **L314 CN**: 以 `void` 从当前函数返回。
- **L315 EN**: Closes the current lexical scope or compound statement.
  **L315 CN**: 结束当前词法作用域或复合语句块。
- **L316 EN**: Executes a call or declaration centered on `CheckForDefinableVariable`.
  **L316 CN**: 执行以 `CheckForDefinableVariable` 为核心的调用或声明。
- **L317 EN**: Executes a call or declaration centered on `kind{expr->GetType`.
  **L317 CN**: 执行以 `kind{expr->GetType` 为核心的调用或声明。
- **L318 EN**: Executes a call or declaration centered on `defaultKind{context_.GetDefaultKind`.
  **L318 CN**: 执行以 `defaultKind{context_.GetDefaultKind` 为核心的调用或声明。
- **L319 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L319 CN**: 开始 `if` 控制流语句并计算其条件。
- **L320 EN**: Continues logic associated with callable symbol `Say`.
  **L320 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ID kind (%d) is smaller than default INTEGER kind (%d)"_err_en_US,`.
  **L321 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ID kind (%d) is smaller than default INTEGER kind (%d)"_err_en_US,`。
- **L322 EN**: Continues logic associated with callable symbol `move`.
  **L322 CN**: 继续与可调用符号 `move` 相关的逻辑。
- **L323 EN**: Closes the current lexical scope or compound statement.
  **L323 CN**: 结束当前词法作用域或复合语句块。
- **L324 EN**: Closes the current lexical scope or compound statement.
  **L324 CN**: 结束当前词法作用域或复合语句块。
- **L325 EN**: Blank line separating nearby declarations or logic blocks.
  **L325 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L326 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::InputItem &spec) {`.
  **L326 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::InputItem &spec) {`。
- **L327 EN**: Executes a call or declaration centered on `flags_.set`.
  **L327 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L328 EN**: Executes a call or declaration centered on `*var{std::get_if<parser::Variable>`.
  **L328 CN**: 执行以 `*var{std::get_if<parser::Variable>` 为核心的调用或声明。
- **L329 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L329 CN**: 开始 `if` 控制流语句并计算其条件。
- **L330 EN**: Returns from the current function with `void`.
  **L330 CN**: 以 `void` 从当前函数返回。
- **L331 EN**: Closes the current lexical scope or compound statement.
  **L331 CN**: 结束当前词法作用域或复合语句块。
- **L332 EN**: Executes a call or declaration centered on `CheckForDefinableVariable`.
  **L332 CN**: 执行以 `CheckForDefinableVariable` 为核心的调用或声明。
- **L333 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L333 CN**: 开始 `if` 控制流语句并计算其条件。
- **L334 EN**: Executes a call or declaration centered on `at{var->GetSource`.
  **L334 CN**: 执行以 `at{var->GetSource` 为核心的调用或声明。
- **L335 EN**: Executes a call or declaration centered on `CheckForAssumedRank`.
  **L335 CN**: 执行以 `CheckForAssumedRank` 为核心的调用或声明。
- **L336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForBadIoType(*expr,`.
  **L336 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForBadIoType(*expr,`。

### Lines 337-360

````cpp
        flags_.test(Flag::FmtOrNml) ? common::DefinedIo::ReadFormatted
                                    : common::DefinedIo::ReadUnformatted,
        at);
  }
}

void IoChecker::Enter(const parser::InquireSpec &spec) {
  // InquireSpec context FileNameExpr
  if (std::get_if<parser::FileNameExpr>(&spec.u)) {
    SetSpecifier(IoSpecKind::File);
  }
}

void IoChecker::Enter(const parser::InquireSpec::CharVar &spec) {
  IoSpecKind specKind{};
  using ParseKind = parser::InquireSpec::CharVar::Kind;
  switch (std::get<ParseKind>(spec.t)) {
  case ParseKind::Access:
    specKind = IoSpecKind::Access;
    break;
  case ParseKind::Action:
    specKind = IoSpecKind::Action;
    break;
  case ParseKind::Asynchronous:
````
- **L337 EN**: Continues logic associated with callable symbol `test`.
  **L337 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: common::DefinedIo::ReadUnformatted,`.
  **L338 CN**: 继续一个多行参数列表、初始化器或聚合项：`: common::DefinedIo::ReadUnformatted,`。
- **L339 EN**: Executes a standalone statement or declaration: `at);`.
  **L339 CN**: 执行一条独立语句或声明：`at);`。
- **L340 EN**: Closes the current lexical scope or compound statement.
  **L340 CN**: 结束当前词法作用域或复合语句块。
- **L341 EN**: Closes the current lexical scope or compound statement.
  **L341 CN**: 结束当前词法作用域或复合语句块。
- **L342 EN**: Blank line separating nearby declarations or logic blocks.
  **L342 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L343 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::InquireSpec &spec) {`.
  **L343 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::InquireSpec &spec) {`。
- **L344 EN**: Comment explains nearby logic, intent, or metadata: `InquireSpec context FileNameExpr`.
  **L344 CN**: 注释说明附近代码的逻辑、意图或元数据：`InquireSpec context FileNameExpr`。
- **L345 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L345 CN**: 开始 `if` 控制流语句并计算其条件。
- **L346 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L346 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L347 EN**: Closes the current lexical scope or compound statement.
  **L347 CN**: 结束当前词法作用域或复合语句块。
- **L348 EN**: Closes the current lexical scope or compound statement.
  **L348 CN**: 结束当前词法作用域或复合语句块。
- **L349 EN**: Blank line separating nearby declarations or logic blocks.
  **L349 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L350 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::InquireSpec::CharVar &spec) {`.
  **L350 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::InquireSpec::CharVar &spec) {`。
- **L351 EN**: Executes a standalone statement or declaration: `IoSpecKind specKind{};`.
  **L351 CN**: 执行一条独立语句或声明：`IoSpecKind specKind{};`。
- **L352 EN**: Defines alias `ParseKind` to simplify later code.
  **L352 CN**: 定义别名 `ParseKind` 以简化后续代码。
- **L353 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L353 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L354 EN**: Introduces a switch dispatch label: `case ParseKind::Access:`.
  **L354 CN**: 引入一个 switch 分发标签：`case ParseKind::Access:`。
- **L355 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Access;`.
  **L355 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Access;`。
- **L356 EN**: Exits the nearest loop or switch statement.
  **L356 CN**: 退出最近的循环或 switch 语句。
- **L357 EN**: Introduces a switch dispatch label: `case ParseKind::Action:`.
  **L357 CN**: 引入一个 switch 分发标签：`case ParseKind::Action:`。
- **L358 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Action;`.
  **L358 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Action;`。
- **L359 EN**: Exits the nearest loop or switch statement.
  **L359 CN**: 退出最近的循环或 switch 语句。
- **L360 EN**: Introduces a switch dispatch label: `case ParseKind::Asynchronous:`.
  **L360 CN**: 引入一个 switch 分发标签：`case ParseKind::Asynchronous:`。

### Lines 361-384

````cpp
    specKind = IoSpecKind::Asynchronous;
    break;
  case ParseKind::Blank:
    specKind = IoSpecKind::Blank;
    break;
  case ParseKind::Decimal:
    specKind = IoSpecKind::Decimal;
    break;
  case ParseKind::Delim:
    specKind = IoSpecKind::Delim;
    break;
  case ParseKind::Direct:
    specKind = IoSpecKind::Direct;
    break;
  case ParseKind::Encoding:
    specKind = IoSpecKind::Encoding;
    break;
  case ParseKind::Form:
    specKind = IoSpecKind::Form;
    break;
  case ParseKind::Formatted:
    specKind = IoSpecKind::Formatted;
    break;
  case ParseKind::Iomsg:
````
- **L361 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Asynchronous;`.
  **L361 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Asynchronous;`。
- **L362 EN**: Exits the nearest loop or switch statement.
  **L362 CN**: 退出最近的循环或 switch 语句。
- **L363 EN**: Introduces a switch dispatch label: `case ParseKind::Blank:`.
  **L363 CN**: 引入一个 switch 分发标签：`case ParseKind::Blank:`。
- **L364 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Blank;`.
  **L364 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Blank;`。
- **L365 EN**: Exits the nearest loop or switch statement.
  **L365 CN**: 退出最近的循环或 switch 语句。
- **L366 EN**: Introduces a switch dispatch label: `case ParseKind::Decimal:`.
  **L366 CN**: 引入一个 switch 分发标签：`case ParseKind::Decimal:`。
- **L367 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Decimal;`.
  **L367 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Decimal;`。
- **L368 EN**: Exits the nearest loop or switch statement.
  **L368 CN**: 退出最近的循环或 switch 语句。
- **L369 EN**: Introduces a switch dispatch label: `case ParseKind::Delim:`.
  **L369 CN**: 引入一个 switch 分发标签：`case ParseKind::Delim:`。
- **L370 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Delim;`.
  **L370 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Delim;`。
- **L371 EN**: Exits the nearest loop or switch statement.
  **L371 CN**: 退出最近的循环或 switch 语句。
- **L372 EN**: Introduces a switch dispatch label: `case ParseKind::Direct:`.
  **L372 CN**: 引入一个 switch 分发标签：`case ParseKind::Direct:`。
- **L373 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Direct;`.
  **L373 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Direct;`。
- **L374 EN**: Exits the nearest loop or switch statement.
  **L374 CN**: 退出最近的循环或 switch 语句。
- **L375 EN**: Introduces a switch dispatch label: `case ParseKind::Encoding:`.
  **L375 CN**: 引入一个 switch 分发标签：`case ParseKind::Encoding:`。
- **L376 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Encoding;`.
  **L376 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Encoding;`。
- **L377 EN**: Exits the nearest loop or switch statement.
  **L377 CN**: 退出最近的循环或 switch 语句。
- **L378 EN**: Introduces a switch dispatch label: `case ParseKind::Form:`.
  **L378 CN**: 引入一个 switch 分发标签：`case ParseKind::Form:`。
- **L379 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Form;`.
  **L379 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Form;`。
- **L380 EN**: Exits the nearest loop or switch statement.
  **L380 CN**: 退出最近的循环或 switch 语句。
- **L381 EN**: Introduces a switch dispatch label: `case ParseKind::Formatted:`.
  **L381 CN**: 引入一个 switch 分发标签：`case ParseKind::Formatted:`。
- **L382 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Formatted;`.
  **L382 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Formatted;`。
- **L383 EN**: Exits the nearest loop or switch statement.
  **L383 CN**: 退出最近的循环或 switch 语句。
- **L384 EN**: Introduces a switch dispatch label: `case ParseKind::Iomsg:`.
  **L384 CN**: 引入一个 switch 分发标签：`case ParseKind::Iomsg:`。

### Lines 385-408

````cpp
    specKind = IoSpecKind::Iomsg;
    break;
  case ParseKind::Leading_Zero:
    specKind = IoSpecKind::Leading_Zero;
    break;
  case ParseKind::Name:
    specKind = IoSpecKind::Name;
    break;
  case ParseKind::Pad:
    specKind = IoSpecKind::Pad;
    break;
  case ParseKind::Position:
    specKind = IoSpecKind::Position;
    break;
  case ParseKind::Read:
    specKind = IoSpecKind::Read;
    break;
  case ParseKind::Readwrite:
    specKind = IoSpecKind::Readwrite;
    break;
  case ParseKind::Round:
    specKind = IoSpecKind::Round;
    break;
  case ParseKind::Sequential:
````
- **L385 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Iomsg;`.
  **L385 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Iomsg;`。
- **L386 EN**: Exits the nearest loop or switch statement.
  **L386 CN**: 退出最近的循环或 switch 语句。
- **L387 EN**: Introduces a switch dispatch label: `case ParseKind::Leading_Zero:`.
  **L387 CN**: 引入一个 switch 分发标签：`case ParseKind::Leading_Zero:`。
- **L388 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Leading_Zero;`.
  **L388 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Leading_Zero;`。
- **L389 EN**: Exits the nearest loop or switch statement.
  **L389 CN**: 退出最近的循环或 switch 语句。
- **L390 EN**: Introduces a switch dispatch label: `case ParseKind::Name:`.
  **L390 CN**: 引入一个 switch 分发标签：`case ParseKind::Name:`。
- **L391 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Name;`.
  **L391 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Name;`。
- **L392 EN**: Exits the nearest loop or switch statement.
  **L392 CN**: 退出最近的循环或 switch 语句。
- **L393 EN**: Introduces a switch dispatch label: `case ParseKind::Pad:`.
  **L393 CN**: 引入一个 switch 分发标签：`case ParseKind::Pad:`。
- **L394 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Pad;`.
  **L394 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Pad;`。
- **L395 EN**: Exits the nearest loop or switch statement.
  **L395 CN**: 退出最近的循环或 switch 语句。
- **L396 EN**: Introduces a switch dispatch label: `case ParseKind::Position:`.
  **L396 CN**: 引入一个 switch 分发标签：`case ParseKind::Position:`。
- **L397 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Position;`.
  **L397 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Position;`。
- **L398 EN**: Exits the nearest loop or switch statement.
  **L398 CN**: 退出最近的循环或 switch 语句。
- **L399 EN**: Introduces a switch dispatch label: `case ParseKind::Read:`.
  **L399 CN**: 引入一个 switch 分发标签：`case ParseKind::Read:`。
- **L400 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Read;`.
  **L400 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Read;`。
- **L401 EN**: Exits the nearest loop or switch statement.
  **L401 CN**: 退出最近的循环或 switch 语句。
- **L402 EN**: Introduces a switch dispatch label: `case ParseKind::Readwrite:`.
  **L402 CN**: 引入一个 switch 分发标签：`case ParseKind::Readwrite:`。
- **L403 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Readwrite;`.
  **L403 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Readwrite;`。
- **L404 EN**: Exits the nearest loop or switch statement.
  **L404 CN**: 退出最近的循环或 switch 语句。
- **L405 EN**: Introduces a switch dispatch label: `case ParseKind::Round:`.
  **L405 CN**: 引入一个 switch 分发标签：`case ParseKind::Round:`。
- **L406 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Round;`.
  **L406 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Round;`。
- **L407 EN**: Exits the nearest loop or switch statement.
  **L407 CN**: 退出最近的循环或 switch 语句。
- **L408 EN**: Introduces a switch dispatch label: `case ParseKind::Sequential:`.
  **L408 CN**: 引入一个 switch 分发标签：`case ParseKind::Sequential:`。

### Lines 409-432

````cpp
    specKind = IoSpecKind::Sequential;
    break;
  case ParseKind::Sign:
    specKind = IoSpecKind::Sign;
    break;
  case ParseKind::Status:
    specKind = IoSpecKind::Status;
    break;
  case ParseKind::Stream:
    specKind = IoSpecKind::Stream;
    break;
  case ParseKind::Unformatted:
    specKind = IoSpecKind::Unformatted;
    break;
  case ParseKind::Write:
    specKind = IoSpecKind::Write;
    break;
  case ParseKind::Carriagecontrol:
    specKind = IoSpecKind::Carriagecontrol;
    break;
  case ParseKind::Convert:
    specKind = IoSpecKind::Convert;
    break;
  case ParseKind::Dispose:
````
- **L409 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Sequential;`.
  **L409 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Sequential;`。
- **L410 EN**: Exits the nearest loop or switch statement.
  **L410 CN**: 退出最近的循环或 switch 语句。
- **L411 EN**: Introduces a switch dispatch label: `case ParseKind::Sign:`.
  **L411 CN**: 引入一个 switch 分发标签：`case ParseKind::Sign:`。
- **L412 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Sign;`.
  **L412 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Sign;`。
- **L413 EN**: Exits the nearest loop or switch statement.
  **L413 CN**: 退出最近的循环或 switch 语句。
- **L414 EN**: Introduces a switch dispatch label: `case ParseKind::Status:`.
  **L414 CN**: 引入一个 switch 分发标签：`case ParseKind::Status:`。
- **L415 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Status;`.
  **L415 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Status;`。
- **L416 EN**: Exits the nearest loop or switch statement.
  **L416 CN**: 退出最近的循环或 switch 语句。
- **L417 EN**: Introduces a switch dispatch label: `case ParseKind::Stream:`.
  **L417 CN**: 引入一个 switch 分发标签：`case ParseKind::Stream:`。
- **L418 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Stream;`.
  **L418 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Stream;`。
- **L419 EN**: Exits the nearest loop or switch statement.
  **L419 CN**: 退出最近的循环或 switch 语句。
- **L420 EN**: Introduces a switch dispatch label: `case ParseKind::Unformatted:`.
  **L420 CN**: 引入一个 switch 分发标签：`case ParseKind::Unformatted:`。
- **L421 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Unformatted;`.
  **L421 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Unformatted;`。
- **L422 EN**: Exits the nearest loop or switch statement.
  **L422 CN**: 退出最近的循环或 switch 语句。
- **L423 EN**: Introduces a switch dispatch label: `case ParseKind::Write:`.
  **L423 CN**: 引入一个 switch 分发标签：`case ParseKind::Write:`。
- **L424 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Write;`.
  **L424 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Write;`。
- **L425 EN**: Exits the nearest loop or switch statement.
  **L425 CN**: 退出最近的循环或 switch 语句。
- **L426 EN**: Introduces a switch dispatch label: `case ParseKind::Carriagecontrol:`.
  **L426 CN**: 引入一个 switch 分发标签：`case ParseKind::Carriagecontrol:`。
- **L427 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Carriagecontrol;`.
  **L427 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Carriagecontrol;`。
- **L428 EN**: Exits the nearest loop or switch statement.
  **L428 CN**: 退出最近的循环或 switch 语句。
- **L429 EN**: Introduces a switch dispatch label: `case ParseKind::Convert:`.
  **L429 CN**: 引入一个 switch 分发标签：`case ParseKind::Convert:`。
- **L430 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Convert;`.
  **L430 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Convert;`。
- **L431 EN**: Exits the nearest loop or switch statement.
  **L431 CN**: 退出最近的循环或 switch 语句。
- **L432 EN**: Introduces a switch dispatch label: `case ParseKind::Dispose:`.
  **L432 CN**: 引入一个 switch 分发标签：`case ParseKind::Dispose:`。

### Lines 433-456

````cpp
    specKind = IoSpecKind::Dispose;
    break;
  }
  const auto &var{parser::UnwrapRef<parser::Variable>(
      std::get<parser::ScalarDefaultCharVariable>(spec.t))};
  std::string what{parser::ToUpperCaseLetters(common::EnumToString(specKind))};
  CheckForDefinableVariable(var, what);
  WarnOnDeferredLengthCharacterScalar(
      context_, GetExpr(context_, var), var.GetSource(), what.c_str());
  SetSpecifier(specKind);
}

void IoChecker::Enter(const parser::InquireSpec::IntVar &spec) {
  IoSpecKind specKind{};
  using ParseKind = parser::InquireSpec::IntVar::Kind;
  switch (std::get<parser::InquireSpec::IntVar::Kind>(spec.t)) {
  case ParseKind::Iostat:
    specKind = IoSpecKind::Iostat;
    break;
  case ParseKind::Nextrec:
    specKind = IoSpecKind::Nextrec;
    break;
  case ParseKind::Number:
    specKind = IoSpecKind::Number;
````
- **L433 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Dispose;`.
  **L433 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Dispose;`。
- **L434 EN**: Exits the nearest loop or switch statement.
  **L434 CN**: 退出最近的循环或 switch 语句。
- **L435 EN**: Closes the current lexical scope or compound statement.
  **L435 CN**: 结束当前词法作用域或复合语句块。
- **L436 EN**: Continues logic associated with callable symbol `Variable>`.
  **L436 CN**: 继续与可调用符号 `Variable>` 相关的逻辑。
- **L437 EN**: Executes a call or declaration centered on `std::get<parser::ScalarDefaultCharVariable>`.
  **L437 CN**: 执行以 `std::get<parser::ScalarDefaultCharVariable>` 为核心的调用或声明。
- **L438 EN**: Executes a call or declaration centered on `what{parser::ToUpperCaseLetters`.
  **L438 CN**: 执行以 `what{parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L439 EN**: Executes a call or declaration centered on `CheckForDefinableVariable`.
  **L439 CN**: 执行以 `CheckForDefinableVariable` 为核心的调用或声明。
- **L440 EN**: Continues logic associated with callable symbol `WarnOnDeferredLengthCharacterScalar`.
  **L440 CN**: 继续与可调用符号 `WarnOnDeferredLengthCharacterScalar` 相关的逻辑。
- **L441 EN**: Executes a call or declaration centered on `GetExpr`.
  **L441 CN**: 执行以 `GetExpr` 为核心的调用或声明。
- **L442 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L442 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L443 EN**: Closes the current lexical scope or compound statement.
  **L443 CN**: 结束当前词法作用域或复合语句块。
- **L444 EN**: Blank line separating nearby declarations or logic blocks.
  **L444 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L445 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::InquireSpec::IntVar &spec) {`.
  **L445 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::InquireSpec::IntVar &spec) {`。
- **L446 EN**: Executes a standalone statement or declaration: `IoSpecKind specKind{};`.
  **L446 CN**: 执行一条独立语句或声明：`IoSpecKind specKind{};`。
- **L447 EN**: Defines alias `ParseKind` to simplify later code.
  **L447 CN**: 定义别名 `ParseKind` 以简化后续代码。
- **L448 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L448 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L449 EN**: Introduces a switch dispatch label: `case ParseKind::Iostat:`.
  **L449 CN**: 引入一个 switch 分发标签：`case ParseKind::Iostat:`。
- **L450 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Iostat;`.
  **L450 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Iostat;`。
- **L451 EN**: Exits the nearest loop or switch statement.
  **L451 CN**: 退出最近的循环或 switch 语句。
- **L452 EN**: Introduces a switch dispatch label: `case ParseKind::Nextrec:`.
  **L452 CN**: 引入一个 switch 分发标签：`case ParseKind::Nextrec:`。
- **L453 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Nextrec;`.
  **L453 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Nextrec;`。
- **L454 EN**: Exits the nearest loop or switch statement.
  **L454 CN**: 退出最近的循环或 switch 语句。
- **L455 EN**: Introduces a switch dispatch label: `case ParseKind::Number:`.
  **L455 CN**: 引入一个 switch 分发标签：`case ParseKind::Number:`。
- **L456 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Number;`.
  **L456 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Number;`。

### Lines 457-480

````cpp
    break;
  case ParseKind::Pos:
    specKind = IoSpecKind::Pos;
    break;
  case ParseKind::Recl:
    specKind = IoSpecKind::Recl;
    break;
  case ParseKind::Size:
    specKind = IoSpecKind::Size;
    break;
  }
  CheckForDefinableVariable(std::get<parser::ScalarIntVariable>(spec.t),
      parser::ToUpperCaseLetters(common::EnumToString(specKind)));
  SetSpecifier(specKind);
}

void IoChecker::Enter(const parser::InquireSpec::LogVar &spec) {
  IoSpecKind specKind{};
  using ParseKind = parser::InquireSpec::LogVar::Kind;
  switch (std::get<parser::InquireSpec::LogVar::Kind>(spec.t)) {
  case ParseKind::Exist:
    specKind = IoSpecKind::Exist;
    break;
  case ParseKind::Named:
````
- **L457 EN**: Exits the nearest loop or switch statement.
  **L457 CN**: 退出最近的循环或 switch 语句。
- **L458 EN**: Introduces a switch dispatch label: `case ParseKind::Pos:`.
  **L458 CN**: 引入一个 switch 分发标签：`case ParseKind::Pos:`。
- **L459 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Pos;`.
  **L459 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Pos;`。
- **L460 EN**: Exits the nearest loop or switch statement.
  **L460 CN**: 退出最近的循环或 switch 语句。
- **L461 EN**: Introduces a switch dispatch label: `case ParseKind::Recl:`.
  **L461 CN**: 引入一个 switch 分发标签：`case ParseKind::Recl:`。
- **L462 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Recl;`.
  **L462 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Recl;`。
- **L463 EN**: Exits the nearest loop or switch statement.
  **L463 CN**: 退出最近的循环或 switch 语句。
- **L464 EN**: Introduces a switch dispatch label: `case ParseKind::Size:`.
  **L464 CN**: 引入一个 switch 分发标签：`case ParseKind::Size:`。
- **L465 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Size;`.
  **L465 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Size;`。
- **L466 EN**: Exits the nearest loop or switch statement.
  **L466 CN**: 退出最近的循环或 switch 语句。
- **L467 EN**: Closes the current lexical scope or compound statement.
  **L467 CN**: 结束当前词法作用域或复合语句块。
- **L468 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForDefinableVariable(std::get<parser::ScalarIntVariable>(spec.t),`.
  **L468 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForDefinableVariable(std::get<parser::ScalarIntVariable>(spec.t),`。
- **L469 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L469 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L470 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L470 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L471 EN**: Closes the current lexical scope or compound statement.
  **L471 CN**: 结束当前词法作用域或复合语句块。
- **L472 EN**: Blank line separating nearby declarations or logic blocks.
  **L472 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L473 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::InquireSpec::LogVar &spec) {`.
  **L473 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::InquireSpec::LogVar &spec) {`。
- **L474 EN**: Executes a standalone statement or declaration: `IoSpecKind specKind{};`.
  **L474 CN**: 执行一条独立语句或声明：`IoSpecKind specKind{};`。
- **L475 EN**: Defines alias `ParseKind` to simplify later code.
  **L475 CN**: 定义别名 `ParseKind` 以简化后续代码。
- **L476 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L476 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L477 EN**: Introduces a switch dispatch label: `case ParseKind::Exist:`.
  **L477 CN**: 引入一个 switch 分发标签：`case ParseKind::Exist:`。
- **L478 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Exist;`.
  **L478 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Exist;`。
- **L479 EN**: Exits the nearest loop or switch statement.
  **L479 CN**: 退出最近的循环或 switch 语句。
- **L480 EN**: Introduces a switch dispatch label: `case ParseKind::Named:`.
  **L480 CN**: 引入一个 switch 分发标签：`case ParseKind::Named:`。

### Lines 481-504

````cpp
    specKind = IoSpecKind::Named;
    break;
  case ParseKind::Opened:
    specKind = IoSpecKind::Opened;
    break;
  case ParseKind::Pending:
    specKind = IoSpecKind::Pending;
    break;
  }
  CheckForDefinableVariable(std::get<parser::ScalarLogicalVariable>(spec.t),
      parser::ToUpperCaseLetters(common::EnumToString(specKind)));
  SetSpecifier(specKind);
}

void IoChecker::Enter(const parser::IoControlSpec &spec) {
  // IoControlSpec context Name
  flags_.set(Flag::IoControlList);
  if (std::holds_alternative<parser::Name>(spec.u)) {
    SetSpecifier(IoSpecKind::Nml);
    flags_.set(Flag::FmtOrNml);
  }
}

void IoChecker::Enter(const parser::IoControlSpec::Asynchronous &spec) {
````
- **L481 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Named;`.
  **L481 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Named;`。
- **L482 EN**: Exits the nearest loop or switch statement.
  **L482 CN**: 退出最近的循环或 switch 语句。
- **L483 EN**: Introduces a switch dispatch label: `case ParseKind::Opened:`.
  **L483 CN**: 引入一个 switch 分发标签：`case ParseKind::Opened:`。
- **L484 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Opened;`.
  **L484 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Opened;`。
- **L485 EN**: Exits the nearest loop or switch statement.
  **L485 CN**: 退出最近的循环或 switch 语句。
- **L486 EN**: Introduces a switch dispatch label: `case ParseKind::Pending:`.
  **L486 CN**: 引入一个 switch 分发标签：`case ParseKind::Pending:`。
- **L487 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Pending;`.
  **L487 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Pending;`。
- **L488 EN**: Exits the nearest loop or switch statement.
  **L488 CN**: 退出最近的循环或 switch 语句。
- **L489 EN**: Closes the current lexical scope or compound statement.
  **L489 CN**: 结束当前词法作用域或复合语句块。
- **L490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForDefinableVariable(std::get<parser::ScalarLogicalVariable>(spec.t),`.
  **L490 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForDefinableVariable(std::get<parser::ScalarLogicalVariable>(spec.t),`。
- **L491 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L491 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L492 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L492 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L493 EN**: Closes the current lexical scope or compound statement.
  **L493 CN**: 结束当前词法作用域或复合语句块。
- **L494 EN**: Blank line separating nearby declarations or logic blocks.
  **L494 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L495 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::IoControlSpec &spec) {`.
  **L495 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::IoControlSpec &spec) {`。
- **L496 EN**: Comment explains nearby logic, intent, or metadata: `IoControlSpec context Name`.
  **L496 CN**: 注释说明附近代码的逻辑、意图或元数据：`IoControlSpec context Name`。
- **L497 EN**: Executes a call or declaration centered on `flags_.set`.
  **L497 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L498 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L498 CN**: 开始 `if` 控制流语句并计算其条件。
- **L499 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L499 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L500 EN**: Executes a call or declaration centered on `flags_.set`.
  **L500 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L501 EN**: Closes the current lexical scope or compound statement.
  **L501 CN**: 结束当前词法作用域或复合语句块。
- **L502 EN**: Closes the current lexical scope or compound statement.
  **L502 CN**: 结束当前词法作用域或复合语句块。
- **L503 EN**: Blank line separating nearby declarations or logic blocks.
  **L503 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L504 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::IoControlSpec::Asynchronous &spec) {`.
  **L504 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::IoControlSpec::Asynchronous &spec) {`。

### Lines 505-528

````cpp
  SetSpecifier(IoSpecKind::Asynchronous);
  if (const std::optional<std::string> charConst{
          GetConstExpr<std::string>(spec)}) {
    flags_.set(Flag::AsynchronousYes, Normalize(*charConst) == "YES");
    CheckStringValue(IoSpecKind::Asynchronous, *charConst,
        parser::FindSourceLocation(spec)); // C1223
  }
}

void IoChecker::Enter(const parser::IoControlSpec::CharExpr &spec) {
  IoSpecKind specKind{};
  using ParseKind = parser::IoControlSpec::CharExpr::Kind;
  switch (std::get<ParseKind>(spec.t)) {
  case ParseKind::Advance:
    specKind = IoSpecKind::Advance;
    break;
  case ParseKind::Blank:
    specKind = IoSpecKind::Blank;
    break;
  case ParseKind::Decimal:
    specKind = IoSpecKind::Decimal;
    break;
  case ParseKind::Delim:
    specKind = IoSpecKind::Delim;
````
- **L505 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L505 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L506 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L506 CN**: 开始 `if` 控制流语句并计算其条件。
- **L507 EN**: Starts a function, method, lambda, or structured scope: `GetConstExpr<std::string>(spec)}) {`.
  **L507 CN**: 开始一个函数、方法、lambda 或结构化作用域：`GetConstExpr<std::string>(spec)}) {`。
- **L508 EN**: Executes a call or declaration centered on `flags_.set`.
  **L508 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L509 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckStringValue(IoSpecKind::Asynchronous, *charConst,`.
  **L509 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckStringValue(IoSpecKind::Asynchronous, *charConst,`。
- **L510 EN**: Continues logic associated with callable symbol `FindSourceLocation`.
  **L510 CN**: 继续与可调用符号 `FindSourceLocation` 相关的逻辑。
- **L511 EN**: Closes the current lexical scope or compound statement.
  **L511 CN**: 结束当前词法作用域或复合语句块。
- **L512 EN**: Closes the current lexical scope or compound statement.
  **L512 CN**: 结束当前词法作用域或复合语句块。
- **L513 EN**: Blank line separating nearby declarations or logic blocks.
  **L513 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L514 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::IoControlSpec::CharExpr &spec) {`.
  **L514 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::IoControlSpec::CharExpr &spec) {`。
- **L515 EN**: Executes a standalone statement or declaration: `IoSpecKind specKind{};`.
  **L515 CN**: 执行一条独立语句或声明：`IoSpecKind specKind{};`。
- **L516 EN**: Defines alias `ParseKind` to simplify later code.
  **L516 CN**: 定义别名 `ParseKind` 以简化后续代码。
- **L517 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L517 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L518 EN**: Introduces a switch dispatch label: `case ParseKind::Advance:`.
  **L518 CN**: 引入一个 switch 分发标签：`case ParseKind::Advance:`。
- **L519 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Advance;`.
  **L519 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Advance;`。
- **L520 EN**: Exits the nearest loop or switch statement.
  **L520 CN**: 退出最近的循环或 switch 语句。
- **L521 EN**: Introduces a switch dispatch label: `case ParseKind::Blank:`.
  **L521 CN**: 引入一个 switch 分发标签：`case ParseKind::Blank:`。
- **L522 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Blank;`.
  **L522 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Blank;`。
- **L523 EN**: Exits the nearest loop or switch statement.
  **L523 CN**: 退出最近的循环或 switch 语句。
- **L524 EN**: Introduces a switch dispatch label: `case ParseKind::Decimal:`.
  **L524 CN**: 引入一个 switch 分发标签：`case ParseKind::Decimal:`。
- **L525 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Decimal;`.
  **L525 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Decimal;`。
- **L526 EN**: Exits the nearest loop or switch statement.
  **L526 CN**: 退出最近的循环或 switch 语句。
- **L527 EN**: Introduces a switch dispatch label: `case ParseKind::Delim:`.
  **L527 CN**: 引入一个 switch 分发标签：`case ParseKind::Delim:`。
- **L528 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Delim;`.
  **L528 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Delim;`。

### Lines 529-552

````cpp
    break;
  case ParseKind::Leading_Zero:
    specKind = IoSpecKind::Leading_Zero;
    break;
  case ParseKind::Pad:
    specKind = IoSpecKind::Pad;
    break;
  case ParseKind::Round:
    specKind = IoSpecKind::Round;
    break;
  case ParseKind::Sign:
    specKind = IoSpecKind::Sign;
    break;
  }
  SetSpecifier(specKind);
  if (const std::optional<std::string> charConst{GetConstExpr<std::string>(
          std::get<parser::ScalarDefaultCharExpr>(spec.t))}) {
    if (specKind == IoSpecKind::Advance) {
      flags_.set(Flag::AdvanceYes, Normalize(*charConst) == "YES");
    }
    CheckStringValue(specKind, *charConst, parser::FindSourceLocation(spec));
  }
}

````
- **L529 EN**: Exits the nearest loop or switch statement.
  **L529 CN**: 退出最近的循环或 switch 语句。
- **L530 EN**: Introduces a switch dispatch label: `case ParseKind::Leading_Zero:`.
  **L530 CN**: 引入一个 switch 分发标签：`case ParseKind::Leading_Zero:`。
- **L531 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Leading_Zero;`.
  **L531 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Leading_Zero;`。
- **L532 EN**: Exits the nearest loop or switch statement.
  **L532 CN**: 退出最近的循环或 switch 语句。
- **L533 EN**: Introduces a switch dispatch label: `case ParseKind::Pad:`.
  **L533 CN**: 引入一个 switch 分发标签：`case ParseKind::Pad:`。
- **L534 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Pad;`.
  **L534 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Pad;`。
- **L535 EN**: Exits the nearest loop or switch statement.
  **L535 CN**: 退出最近的循环或 switch 语句。
- **L536 EN**: Introduces a switch dispatch label: `case ParseKind::Round:`.
  **L536 CN**: 引入一个 switch 分发标签：`case ParseKind::Round:`。
- **L537 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Round;`.
  **L537 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Round;`。
- **L538 EN**: Exits the nearest loop or switch statement.
  **L538 CN**: 退出最近的循环或 switch 语句。
- **L539 EN**: Introduces a switch dispatch label: `case ParseKind::Sign:`.
  **L539 CN**: 引入一个 switch 分发标签：`case ParseKind::Sign:`。
- **L540 EN**: Executes a standalone statement or declaration: `specKind = IoSpecKind::Sign;`.
  **L540 CN**: 执行一条独立语句或声明：`specKind = IoSpecKind::Sign;`。
- **L541 EN**: Exits the nearest loop or switch statement.
  **L541 CN**: 退出最近的循环或 switch 语句。
- **L542 EN**: Closes the current lexical scope or compound statement.
  **L542 CN**: 结束当前词法作用域或复合语句块。
- **L543 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L543 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L544 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L544 CN**: 开始 `if` 控制流语句并计算其条件。
- **L545 EN**: Starts a function, method, lambda, or structured scope: `std::get<parser::ScalarDefaultCharExpr>(spec.t))}) {`.
  **L545 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<parser::ScalarDefaultCharExpr>(spec.t))}) {`。
- **L546 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L546 CN**: 开始 `if` 控制流语句并计算其条件。
- **L547 EN**: Executes a call or declaration centered on `flags_.set`.
  **L547 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L548 EN**: Closes the current lexical scope or compound statement.
  **L548 CN**: 结束当前词法作用域或复合语句块。
- **L549 EN**: Executes a call or declaration centered on `CheckStringValue`.
  **L549 CN**: 执行以 `CheckStringValue` 为核心的调用或声明。
- **L550 EN**: Closes the current lexical scope or compound statement.
  **L550 CN**: 结束当前词法作用域或复合语句块。
- **L551 EN**: Closes the current lexical scope or compound statement.
  **L551 CN**: 结束当前词法作用域或复合语句块。
- **L552 EN**: Blank line separating nearby declarations or logic blocks.
  **L552 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 553-576

````cpp
void IoChecker::Enter(const parser::IoControlSpec::Pos &) {
  SetSpecifier(IoSpecKind::Pos);
}

void IoChecker::Enter(const parser::IoControlSpec::Rec &) {
  SetSpecifier(IoSpecKind::Rec);
}

void IoChecker::Enter(const parser::IoControlSpec::Size &var) {
  CheckForDefinableVariable(var, "SIZE");
  SetSpecifier(IoSpecKind::Size);
}

void IoChecker::Enter(const parser::IoUnit &spec) {
  if (const parser::Variable * var{std::get_if<parser::Variable>(&spec.u)}) {
    // Only now after generic resolution can it be known whether a function
    // call appearing as UNIT=f() is an integer scalar external unit number
    // or a character pointer for internal I/O.
    const auto *expr{GetExpr(context_, *var)};
    std::optional<evaluate::DynamicType> dyType;
    if (expr) {
      dyType = expr->GetType();
    }
    if (dyType && dyType->category() == TypeCategory::Integer) {
````
- **L553 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::IoControlSpec::Pos &) {`.
  **L553 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::IoControlSpec::Pos &) {`。
- **L554 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L554 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L555 EN**: Closes the current lexical scope or compound statement.
  **L555 CN**: 结束当前词法作用域或复合语句块。
- **L556 EN**: Blank line separating nearby declarations or logic blocks.
  **L556 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L557 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::IoControlSpec::Rec &) {`.
  **L557 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::IoControlSpec::Rec &) {`。
- **L558 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L558 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L559 EN**: Closes the current lexical scope or compound statement.
  **L559 CN**: 结束当前词法作用域或复合语句块。
- **L560 EN**: Blank line separating nearby declarations or logic blocks.
  **L560 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L561 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::IoControlSpec::Size &var) {`.
  **L561 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::IoControlSpec::Size &var) {`。
- **L562 EN**: Executes a call or declaration centered on `CheckForDefinableVariable`.
  **L562 CN**: 执行以 `CheckForDefinableVariable` 为核心的调用或声明。
- **L563 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L563 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L564 EN**: Closes the current lexical scope or compound statement.
  **L564 CN**: 结束当前词法作用域或复合语句块。
- **L565 EN**: Blank line separating nearby declarations or logic blocks.
  **L565 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L566 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::IoUnit &spec) {`.
  **L566 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::IoUnit &spec) {`。
- **L567 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L567 CN**: 开始 `if` 控制流语句并计算其条件。
- **L568 EN**: Comment explains nearby logic, intent, or metadata: `Only now after generic resolution can it be known whether a function`.
  **L568 CN**: 注释说明附近代码的逻辑、意图或元数据：`Only now after generic resolution can it be known whether a function`。
- **L569 EN**: Comment explains nearby logic, intent, or metadata: `call appearing as UNIT=f() is an integer scalar external unit number`.
  **L569 CN**: 注释说明附近代码的逻辑、意图或元数据：`call appearing as UNIT=f() is an integer scalar external unit number`。
- **L570 EN**: Comment explains nearby logic, intent, or metadata: `or a character pointer for internal I/O.`.
  **L570 CN**: 注释说明附近代码的逻辑、意图或元数据：`or a character pointer for internal I/O.`。
- **L571 EN**: Executes a call or declaration centered on `*expr{GetExpr`.
  **L571 CN**: 执行以 `*expr{GetExpr` 为核心的调用或声明。
- **L572 EN**: Executes a standalone statement or declaration: `std::optional<evaluate::DynamicType> dyType;`.
  **L572 CN**: 执行一条独立语句或声明：`std::optional<evaluate::DynamicType> dyType;`。
- **L573 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L573 CN**: 开始 `if` 控制流语句并计算其条件。
- **L574 EN**: Executes a call or declaration centered on `expr->GetType`.
  **L574 CN**: 执行以 `expr->GetType` 为核心的调用或声明。
- **L575 EN**: Closes the current lexical scope or compound statement.
  **L575 CN**: 结束当前词法作用域或复合语句块。
- **L576 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L576 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 577-600

````cpp
      if (expr->Rank() != 0) {
        context_.Say(parser::FindSourceLocation(*var),
            "I/O unit number must be scalar"_err_en_US);
      }
      // In the case of an integer unit number variable, rewrite the parse
      // tree as if the unit had been parsed as a FileUnitNumber in order
      // to ease lowering.
      auto &mutableSpec{const_cast<parser::IoUnit &>(spec)};
      auto &mutableVar{std::get<parser::Variable>(mutableSpec.u)};
      auto source{mutableVar.GetSource()};
      auto typedExpr{std::move(mutableVar.typedExpr)};
      auto newExpr{common::visit(
          [](auto &&indirection) {
            return parser::Expr{std::move(indirection)};
          },
          std::move(mutableVar.u))};
      newExpr.source = source;
      newExpr.typedExpr = std::move(typedExpr);
      mutableSpec.u = common::Indirection<parser::Expr>{std::move(newExpr)};
      SetSpecifier(IoSpecKind::Unit);
      flags_.set(Flag::NumberUnit);
    } else if (!dyType || dyType->category() != TypeCategory::Character) {
      SetSpecifier(IoSpecKind::Unit);
      context_.Say(parser::FindSourceLocation(*var),
````
- **L577 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L577 CN**: 开始 `if` 控制流语句并计算其条件。
- **L578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(parser::FindSourceLocation(*var),`.
  **L578 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(parser::FindSourceLocation(*var),`。
- **L579 EN**: Executes a standalone statement or declaration: `"I/O unit number must be scalar"_err_en_US);`.
  **L579 CN**: 执行一条独立语句或声明：`"I/O unit number must be scalar"_err_en_US);`。
- **L580 EN**: Closes the current lexical scope or compound statement.
  **L580 CN**: 结束当前词法作用域或复合语句块。
- **L581 EN**: Comment explains nearby logic, intent, or metadata: `In the case of an integer unit number variable, rewrite the parse`.
  **L581 CN**: 注释说明附近代码的逻辑、意图或元数据：`In the case of an integer unit number variable, rewrite the parse`。
- **L582 EN**: Comment explains nearby logic, intent, or metadata: `tree as if the unit had been parsed as a FileUnitNumber in order`.
  **L582 CN**: 注释说明附近代码的逻辑、意图或元数据：`tree as if the unit had been parsed as a FileUnitNumber in order`。
- **L583 EN**: Comment explains nearby logic, intent, or metadata: `to ease lowering.`.
  **L583 CN**: 注释说明附近代码的逻辑、意图或元数据：`to ease lowering.`。
- **L584 EN**: Executes a call or declaration centered on `&>`.
  **L584 CN**: 执行以 `&>` 为核心的调用或声明。
- **L585 EN**: Executes a call or declaration centered on `&mutableVar{std::get<parser::Variable>`.
  **L585 CN**: 执行以 `&mutableVar{std::get<parser::Variable>` 为核心的调用或声明。
- **L586 EN**: Executes a call or declaration centered on `source{mutableVar.GetSource`.
  **L586 CN**: 执行以 `source{mutableVar.GetSource` 为核心的调用或声明。
- **L587 EN**: Executes a call or declaration centered on `typedExpr{std::move`.
  **L587 CN**: 执行以 `typedExpr{std::move` 为核心的调用或声明。
- **L588 EN**: Continues logic associated with callable symbol `visit`.
  **L588 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L589 EN**: Starts a function, method, lambda, or structured scope: `[](auto &&indirection) {`.
  **L589 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[](auto &&indirection) {`。
- **L590 EN**: Returns from the current function with `parser::Expr{std::move(indirection)}`.
  **L590 CN**: 以 `parser::Expr{std::move(indirection)}` 从当前函数返回。
- **L591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L591 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L592 EN**: Executes a call or declaration centered on `std::move`.
  **L592 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L593 EN**: Executes a standalone statement or declaration: `newExpr.source = source;`.
  **L593 CN**: 执行一条独立语句或声明：`newExpr.source = source;`。
- **L594 EN**: Executes a call or declaration centered on `std::move`.
  **L594 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L595 EN**: Executes a call or declaration centered on `common::Indirection<parser::Expr>{std::move`.
  **L595 CN**: 执行以 `common::Indirection<parser::Expr>{std::move` 为核心的调用或声明。
- **L596 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L596 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L597 EN**: Executes a call or declaration centered on `flags_.set`.
  **L597 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L598 EN**: Transitions from the previous branch into an `else if` condition.
  **L598 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L599 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L599 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(parser::FindSourceLocation(*var),`.
  **L600 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(parser::FindSourceLocation(*var),`。

### Lines 601-624

````cpp
          "I/O unit must be a character variable or a scalar integer expression"_err_en_US);
    } else { // CHARACTER variable (internal I/O)
      if (stmt_ == IoStmtKind::Write) {
        CheckForDefinableVariable(*var, "Internal file");
        WarnOnDeferredLengthCharacterScalar(
            context_, expr, var->GetSource(), "Internal file");
      }
      if (HasVectorSubscript(*expr)) {
        context_.Say(parser::FindSourceLocation(*var), // C1201
            "Internal file must not have a vector subscript"_err_en_US);
      }
      SetSpecifier(IoSpecKind::Unit);
      flags_.set(Flag::InternalUnit);
    }
  } else if (std::get_if<parser::Star>(&spec.u)) {
    SetSpecifier(IoSpecKind::Unit);
    flags_.set(Flag::StarUnit);
  } else if (const common::Indirection<parser::Expr> *pexpr{
                 std::get_if<common::Indirection<parser::Expr>>(&spec.u)}) {
    const auto *expr{GetExpr(context_, *pexpr)};
    std::optional<evaluate::DynamicType> dyType;
    if (expr) {
      dyType = expr->GetType();
    }
````
- **L601 EN**: Executes a standalone statement or declaration: `"I/O unit must be a character variable or a scalar integer expression"_err_en_US);`.
  **L601 CN**: 执行一条独立语句或声明：`"I/O unit must be a character variable or a scalar integer expression"_err_en_US);`。
- **L602 EN**: Transitions from the previous branch into the alternative path.
  **L602 CN**: 从前一个分支过渡到备选路径。
- **L603 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L603 CN**: 开始 `if` 控制流语句并计算其条件。
- **L604 EN**: Executes a call or declaration centered on `CheckForDefinableVariable`.
  **L604 CN**: 执行以 `CheckForDefinableVariable` 为核心的调用或声明。
- **L605 EN**: Continues logic associated with callable symbol `WarnOnDeferredLengthCharacterScalar`.
  **L605 CN**: 继续与可调用符号 `WarnOnDeferredLengthCharacterScalar` 相关的逻辑。
- **L606 EN**: Executes a call or declaration centered on `var->GetSource`.
  **L606 CN**: 执行以 `var->GetSource` 为核心的调用或声明。
- **L607 EN**: Closes the current lexical scope or compound statement.
  **L607 CN**: 结束当前词法作用域或复合语句块。
- **L608 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L608 CN**: 开始 `if` 控制流语句并计算其条件。
- **L609 EN**: Continues logic associated with callable symbol `Say`.
  **L609 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L610 EN**: Executes a standalone statement or declaration: `"Internal file must not have a vector subscript"_err_en_US);`.
  **L610 CN**: 执行一条独立语句或声明：`"Internal file must not have a vector subscript"_err_en_US);`。
- **L611 EN**: Closes the current lexical scope or compound statement.
  **L611 CN**: 结束当前词法作用域或复合语句块。
- **L612 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L612 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L613 EN**: Executes a call or declaration centered on `flags_.set`.
  **L613 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L614 EN**: Closes the current lexical scope or compound statement.
  **L614 CN**: 结束当前词法作用域或复合语句块。
- **L615 EN**: Transitions from the previous branch into an `else if` condition.
  **L615 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L616 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L616 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L617 EN**: Executes a call or declaration centered on `flags_.set`.
  **L617 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L618 EN**: Transitions from the previous branch into an `else if` condition.
  **L618 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L619 EN**: Starts a function, method, lambda, or structured scope: `std::get_if<common::Indirection<parser::Expr>>(&spec.u)}) {`.
  **L619 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get_if<common::Indirection<parser::Expr>>(&spec.u)}) {`。
- **L620 EN**: Executes a call or declaration centered on `*expr{GetExpr`.
  **L620 CN**: 执行以 `*expr{GetExpr` 为核心的调用或声明。
- **L621 EN**: Executes a standalone statement or declaration: `std::optional<evaluate::DynamicType> dyType;`.
  **L621 CN**: 执行一条独立语句或声明：`std::optional<evaluate::DynamicType> dyType;`。
- **L622 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L622 CN**: 开始 `if` 控制流语句并计算其条件。
- **L623 EN**: Executes a call or declaration centered on `expr->GetType`.
  **L623 CN**: 执行以 `expr->GetType` 为核心的调用或声明。
- **L624 EN**: Closes the current lexical scope or compound statement.
  **L624 CN**: 结束当前词法作用域或复合语句块。

### Lines 625-648

````cpp
    if (!expr || !dyType) {
      context_.Say(parser::FindSourceLocation(*pexpr),
          "I/O unit must be a character variable or scalar integer expression"_err_en_US);
    } else if (dyType->category() != TypeCategory::Integer) {
      context_.Say(parser::FindSourceLocation(*pexpr),
          "I/O unit must be a character variable or a scalar integer expression, but is an expression of type %s"_err_en_US,
          parser::ToUpperCaseLetters(dyType->AsFortran()));
    } else if (expr->Rank() != 0) {
      context_.Say(parser::FindSourceLocation(*pexpr),
          "I/O unit number must be scalar"_err_en_US);
    }
    SetSpecifier(IoSpecKind::Unit);
    flags_.set(Flag::NumberUnit);
  }
}

void IoChecker::Enter(const parser::MsgVariable &msgVar) {
  const auto &var{parser::UnwrapRef<parser::Variable>(msgVar)};
  if (stmt_ == IoStmtKind::None) {
    // allocate, deallocate, image control
    CheckForDefinableVariable(var, "ERRMSG");
    WarnOnDeferredLengthCharacterScalar(
        context_, GetExpr(context_, var), var.GetSource(), "ERRMSG=");
  } else {
````
- **L625 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L625 CN**: 开始 `if` 控制流语句并计算其条件。
- **L626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(parser::FindSourceLocation(*pexpr),`.
  **L626 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(parser::FindSourceLocation(*pexpr),`。
- **L627 EN**: Executes a standalone statement or declaration: `"I/O unit must be a character variable or scalar integer expression"_err_en_US);`.
  **L627 CN**: 执行一条独立语句或声明：`"I/O unit must be a character variable or scalar integer expression"_err_en_US);`。
- **L628 EN**: Transitions from the previous branch into an `else if` condition.
  **L628 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(parser::FindSourceLocation(*pexpr),`.
  **L629 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(parser::FindSourceLocation(*pexpr),`。
- **L630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"I/O unit must be a character variable or a scalar integer expression, but is an expression of type %s"_err_en_US,`.
  **L630 CN**: 继续一个多行参数列表、初始化器或聚合项：`"I/O unit must be a character variable or a scalar integer expression, but is an expression of type %s"_err_en_US,`。
- **L631 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L631 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L632 EN**: Transitions from the previous branch into an `else if` condition.
  **L632 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(parser::FindSourceLocation(*pexpr),`.
  **L633 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(parser::FindSourceLocation(*pexpr),`。
- **L634 EN**: Executes a standalone statement or declaration: `"I/O unit number must be scalar"_err_en_US);`.
  **L634 CN**: 执行一条独立语句或声明：`"I/O unit number must be scalar"_err_en_US);`。
- **L635 EN**: Closes the current lexical scope or compound statement.
  **L635 CN**: 结束当前词法作用域或复合语句块。
- **L636 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L636 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L637 EN**: Executes a call or declaration centered on `flags_.set`.
  **L637 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L638 EN**: Closes the current lexical scope or compound statement.
  **L638 CN**: 结束当前词法作用域或复合语句块。
- **L639 EN**: Closes the current lexical scope or compound statement.
  **L639 CN**: 结束当前词法作用域或复合语句块。
- **L640 EN**: Blank line separating nearby declarations or logic blocks.
  **L640 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L641 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::MsgVariable &msgVar) {`.
  **L641 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::MsgVariable &msgVar) {`。
- **L642 EN**: Executes a call or declaration centered on `&var{parser::UnwrapRef<parser::Variable>`.
  **L642 CN**: 执行以 `&var{parser::UnwrapRef<parser::Variable>` 为核心的调用或声明。
- **L643 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L643 CN**: 开始 `if` 控制流语句并计算其条件。
- **L644 EN**: Comment explains nearby logic, intent, or metadata: `allocate, deallocate, image control`.
  **L644 CN**: 注释说明附近代码的逻辑、意图或元数据：`allocate, deallocate, image control`。
- **L645 EN**: Executes a call or declaration centered on `CheckForDefinableVariable`.
  **L645 CN**: 执行以 `CheckForDefinableVariable` 为核心的调用或声明。
- **L646 EN**: Continues logic associated with callable symbol `WarnOnDeferredLengthCharacterScalar`.
  **L646 CN**: 继续与可调用符号 `WarnOnDeferredLengthCharacterScalar` 相关的逻辑。
- **L647 EN**: Executes a call or declaration centered on `GetExpr`.
  **L647 CN**: 执行以 `GetExpr` 为核心的调用或声明。
- **L648 EN**: Transitions from the previous branch into the alternative path.
  **L648 CN**: 从前一个分支过渡到备选路径。

### Lines 649-672

````cpp
    CheckForDefinableVariable(var, "IOMSG");
    WarnOnDeferredLengthCharacterScalar(
        context_, GetExpr(context_, var), var.GetSource(), "IOMSG=");
    SetSpecifier(IoSpecKind::Iomsg);
  }
}

void IoChecker::Enter(const parser::OutputItem &item) {
  flags_.set(Flag::DataList);
  if (const auto *x{std::get_if<parser::Expr>(&item.u)}) {
    if (const auto *expr{GetExpr(context_, *x)}) {
      if (evaluate::IsBOZLiteral(*expr)) {
        context_.Say(parser::FindSourceLocation(*x), // C7109
            "Output item must not be a BOZ literal constant"_err_en_US);
      } else if (IsProcedure(*expr)) {
        context_.Say(parser::FindSourceLocation(*x),
            "Output item must not be a procedure"_err_en_US); // C1233
      } else {
        auto at{parser::FindSourceLocation(item)};
        CheckForAssumedRank(UnwrapWholeSymbolDataRef(*expr), at);
        CheckForBadIoType(*expr,
            flags_.test(Flag::FmtOrNml) ? common::DefinedIo::WriteFormatted
                                        : common::DefinedIo::WriteUnformatted,
            at);
````
- **L649 EN**: Executes a call or declaration centered on `CheckForDefinableVariable`.
  **L649 CN**: 执行以 `CheckForDefinableVariable` 为核心的调用或声明。
- **L650 EN**: Continues logic associated with callable symbol `WarnOnDeferredLengthCharacterScalar`.
  **L650 CN**: 继续与可调用符号 `WarnOnDeferredLengthCharacterScalar` 相关的逻辑。
- **L651 EN**: Executes a call or declaration centered on `GetExpr`.
  **L651 CN**: 执行以 `GetExpr` 为核心的调用或声明。
- **L652 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L652 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L653 EN**: Closes the current lexical scope or compound statement.
  **L653 CN**: 结束当前词法作用域或复合语句块。
- **L654 EN**: Closes the current lexical scope or compound statement.
  **L654 CN**: 结束当前词法作用域或复合语句块。
- **L655 EN**: Blank line separating nearby declarations or logic blocks.
  **L655 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L656 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::OutputItem &item) {`.
  **L656 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::OutputItem &item) {`。
- **L657 EN**: Executes a call or declaration centered on `flags_.set`.
  **L657 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L658 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L658 CN**: 开始 `if` 控制流语句并计算其条件。
- **L659 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L659 CN**: 开始 `if` 控制流语句并计算其条件。
- **L660 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L660 CN**: 开始 `if` 控制流语句并计算其条件。
- **L661 EN**: Continues logic associated with callable symbol `Say`.
  **L661 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L662 EN**: Executes a standalone statement or declaration: `"Output item must not be a BOZ literal constant"_err_en_US);`.
  **L662 CN**: 执行一条独立语句或声明：`"Output item must not be a BOZ literal constant"_err_en_US);`。
- **L663 EN**: Transitions from the previous branch into an `else if` condition.
  **L663 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L664 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(parser::FindSourceLocation(*x),`.
  **L664 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(parser::FindSourceLocation(*x),`。
- **L665 EN**: Continues the surrounding expression or declaration: `"Output item must not be a procedure"_err_en_US); // C1233`.
  **L665 CN**: 继续构造周围的表达式或声明：`"Output item must not be a procedure"_err_en_US); // C1233`。
- **L666 EN**: Transitions from the previous branch into the alternative path.
  **L666 CN**: 从前一个分支过渡到备选路径。
- **L667 EN**: Executes a call or declaration centered on `at{parser::FindSourceLocation`.
  **L667 CN**: 执行以 `at{parser::FindSourceLocation` 为核心的调用或声明。
- **L668 EN**: Executes a call or declaration centered on `CheckForAssumedRank`.
  **L668 CN**: 执行以 `CheckForAssumedRank` 为核心的调用或声明。
- **L669 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForBadIoType(*expr,`.
  **L669 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForBadIoType(*expr,`。
- **L670 EN**: Continues logic associated with callable symbol `test`.
  **L670 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: common::DefinedIo::WriteUnformatted,`.
  **L671 CN**: 继续一个多行参数列表、初始化器或聚合项：`: common::DefinedIo::WriteUnformatted,`。
- **L672 EN**: Executes a standalone statement or declaration: `at);`.
  **L672 CN**: 执行一条独立语句或声明：`at);`。

### Lines 673-696

````cpp
      }
    }
  }
}

void IoChecker::Enter(const parser::StatusExpr &spec) {
  SetSpecifier(IoSpecKind::Status);
  if (const std::optional<std::string> charConst{
          GetConstExpr<std::string>(spec)}) {
    // Status values for Open and Close are different.
    std::string s{Normalize(*charConst)};
    if (stmt_ == IoStmtKind::Open) {
      flags_.set(Flag::KnownStatus);
      flags_.set(Flag::StatusNew, s == "NEW");
      flags_.set(Flag::StatusReplace, s == "REPLACE");
      flags_.set(Flag::StatusScratch, s == "SCRATCH");
      // CheckStringValue compares for OPEN Status string values.
      CheckStringValue(
          IoSpecKind::Status, *charConst, parser::FindSourceLocation(spec));
      return;
    }
    CHECK(stmt_ == IoStmtKind::Close);
    if (s != "DELETE" && s != "KEEP") {
      context_.Say(parser::FindSourceLocation(spec),
````
- **L673 EN**: Closes the current lexical scope or compound statement.
  **L673 CN**: 结束当前词法作用域或复合语句块。
- **L674 EN**: Closes the current lexical scope or compound statement.
  **L674 CN**: 结束当前词法作用域或复合语句块。
- **L675 EN**: Closes the current lexical scope or compound statement.
  **L675 CN**: 结束当前词法作用域或复合语句块。
- **L676 EN**: Closes the current lexical scope or compound statement.
  **L676 CN**: 结束当前词法作用域或复合语句块。
- **L677 EN**: Blank line separating nearby declarations or logic blocks.
  **L677 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L678 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::StatusExpr &spec) {`.
  **L678 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::StatusExpr &spec) {`。
- **L679 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L679 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L680 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L680 CN**: 开始 `if` 控制流语句并计算其条件。
- **L681 EN**: Starts a function, method, lambda, or structured scope: `GetConstExpr<std::string>(spec)}) {`.
  **L681 CN**: 开始一个函数、方法、lambda 或结构化作用域：`GetConstExpr<std::string>(spec)}) {`。
- **L682 EN**: Comment explains nearby logic, intent, or metadata: `Status values for Open and Close are different.`.
  **L682 CN**: 注释说明附近代码的逻辑、意图或元数据：`Status values for Open and Close are different.`。
- **L683 EN**: Executes a call or declaration centered on `s{Normalize`.
  **L683 CN**: 执行以 `s{Normalize` 为核心的调用或声明。
- **L684 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L684 CN**: 开始 `if` 控制流语句并计算其条件。
- **L685 EN**: Executes a call or declaration centered on `flags_.set`.
  **L685 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L686 EN**: Executes a call or declaration centered on `flags_.set`.
  **L686 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L687 EN**: Executes a call or declaration centered on `flags_.set`.
  **L687 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L688 EN**: Executes a call or declaration centered on `flags_.set`.
  **L688 CN**: 执行以 `flags_.set` 为核心的调用或声明。
- **L689 EN**: Comment explains nearby logic, intent, or metadata: `CheckStringValue compares for OPEN Status string values.`.
  **L689 CN**: 注释说明附近代码的逻辑、意图或元数据：`CheckStringValue compares for OPEN Status string values.`。
- **L690 EN**: Continues logic associated with callable symbol `CheckStringValue`.
  **L690 CN**: 继续与可调用符号 `CheckStringValue` 相关的逻辑。
- **L691 EN**: Executes a call or declaration centered on `parser::FindSourceLocation`.
  **L691 CN**: 执行以 `parser::FindSourceLocation` 为核心的调用或声明。
- **L692 EN**: Returns from the current function with `void`.
  **L692 CN**: 以 `void` 从当前函数返回。
- **L693 EN**: Closes the current lexical scope or compound statement.
  **L693 CN**: 结束当前词法作用域或复合语句块。
- **L694 EN**: Executes a call or declaration centered on `CHECK`.
  **L694 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L695 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L695 CN**: 开始 `if` 控制流语句并计算其条件。
- **L696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(parser::FindSourceLocation(spec),`.
  **L696 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(parser::FindSourceLocation(spec),`。

### Lines 697-720

````cpp
          "Invalid STATUS value '%s'"_err_en_US, *charConst);
    }
  }
}

void IoChecker::Enter(const parser::StatVariable &var) {
  if (stmt_ == IoStmtKind::None) {
    // allocate, deallocate, image control
    CheckForDefinableVariable(var, "STAT");
  } else {
    CheckForDefinableVariable(var, "IOSTAT");
    SetSpecifier(IoSpecKind::Iostat);
  }
}

void IoChecker::Leave(const parser::BackspaceStmt &) {
  CheckForPureSubprogram();
  CheckForRequiredSpecifier(
      flags_.test(Flag::NumberUnit), "UNIT number"); // C1240
  CheckForUselessIomsg();
  Done();
}

void IoChecker::Leave(const parser::CloseStmt &) {
````
- **L697 EN**: Executes a standalone statement or declaration: `"Invalid STATUS value '%s'"_err_en_US, *charConst);`.
  **L697 CN**: 执行一条独立语句或声明：`"Invalid STATUS value '%s'"_err_en_US, *charConst);`。
- **L698 EN**: Closes the current lexical scope or compound statement.
  **L698 CN**: 结束当前词法作用域或复合语句块。
- **L699 EN**: Closes the current lexical scope or compound statement.
  **L699 CN**: 结束当前词法作用域或复合语句块。
- **L700 EN**: Closes the current lexical scope or compound statement.
  **L700 CN**: 结束当前词法作用域或复合语句块。
- **L701 EN**: Blank line separating nearby declarations or logic blocks.
  **L701 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L702 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Enter(const parser::StatVariable &var) {`.
  **L702 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Enter(const parser::StatVariable &var) {`。
- **L703 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L703 CN**: 开始 `if` 控制流语句并计算其条件。
- **L704 EN**: Comment explains nearby logic, intent, or metadata: `allocate, deallocate, image control`.
  **L704 CN**: 注释说明附近代码的逻辑、意图或元数据：`allocate, deallocate, image control`。
- **L705 EN**: Executes a call or declaration centered on `CheckForDefinableVariable`.
  **L705 CN**: 执行以 `CheckForDefinableVariable` 为核心的调用或声明。
- **L706 EN**: Transitions from the previous branch into the alternative path.
  **L706 CN**: 从前一个分支过渡到备选路径。
- **L707 EN**: Executes a call or declaration centered on `CheckForDefinableVariable`.
  **L707 CN**: 执行以 `CheckForDefinableVariable` 为核心的调用或声明。
- **L708 EN**: Executes a call or declaration centered on `SetSpecifier`.
  **L708 CN**: 执行以 `SetSpecifier` 为核心的调用或声明。
- **L709 EN**: Closes the current lexical scope or compound statement.
  **L709 CN**: 结束当前词法作用域或复合语句块。
- **L710 EN**: Closes the current lexical scope or compound statement.
  **L710 CN**: 结束当前词法作用域或复合语句块。
- **L711 EN**: Blank line separating nearby declarations or logic blocks.
  **L711 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L712 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Leave(const parser::BackspaceStmt &) {`.
  **L712 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Leave(const parser::BackspaceStmt &) {`。
- **L713 EN**: Executes a call or declaration centered on `CheckForPureSubprogram`.
  **L713 CN**: 执行以 `CheckForPureSubprogram` 为核心的调用或声明。
- **L714 EN**: Continues logic associated with callable symbol `CheckForRequiredSpecifier`.
  **L714 CN**: 继续与可调用符号 `CheckForRequiredSpecifier` 相关的逻辑。
- **L715 EN**: Continues logic associated with callable symbol `test`.
  **L715 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L716 EN**: Executes a call or declaration centered on `CheckForUselessIomsg`.
  **L716 CN**: 执行以 `CheckForUselessIomsg` 为核心的调用或声明。
- **L717 EN**: Executes a call or declaration centered on `Done`.
  **L717 CN**: 执行以 `Done` 为核心的调用或声明。
- **L718 EN**: Closes the current lexical scope or compound statement.
  **L718 CN**: 结束当前词法作用域或复合语句块。
- **L719 EN**: Blank line separating nearby declarations or logic blocks.
  **L719 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L720 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Leave(const parser::CloseStmt &) {`.
  **L720 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Leave(const parser::CloseStmt &) {`。

### Lines 721-744

````cpp
  CheckForPureSubprogram();
  CheckForRequiredSpecifier(
      flags_.test(Flag::NumberUnit), "UNIT number"); // C1208
  CheckForUselessIomsg();
  Done();
}

void IoChecker::Leave(const parser::EndfileStmt &) {
  CheckForPureSubprogram();
  CheckForRequiredSpecifier(
      flags_.test(Flag::NumberUnit), "UNIT number"); // C1240
  CheckForUselessIomsg();
  Done();
}

void IoChecker::Leave(const parser::FlushStmt &) {
  CheckForPureSubprogram();
  CheckForRequiredSpecifier(
      flags_.test(Flag::NumberUnit), "UNIT number"); // C1243
  CheckForUselessIomsg();
  Done();
}

void IoChecker::Leave(const parser::InquireStmt &stmt) {
````
- **L721 EN**: Executes a call or declaration centered on `CheckForPureSubprogram`.
  **L721 CN**: 执行以 `CheckForPureSubprogram` 为核心的调用或声明。
- **L722 EN**: Continues logic associated with callable symbol `CheckForRequiredSpecifier`.
  **L722 CN**: 继续与可调用符号 `CheckForRequiredSpecifier` 相关的逻辑。
- **L723 EN**: Continues logic associated with callable symbol `test`.
  **L723 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L724 EN**: Executes a call or declaration centered on `CheckForUselessIomsg`.
  **L724 CN**: 执行以 `CheckForUselessIomsg` 为核心的调用或声明。
- **L725 EN**: Executes a call or declaration centered on `Done`.
  **L725 CN**: 执行以 `Done` 为核心的调用或声明。
- **L726 EN**: Closes the current lexical scope or compound statement.
  **L726 CN**: 结束当前词法作用域或复合语句块。
- **L727 EN**: Blank line separating nearby declarations or logic blocks.
  **L727 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L728 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Leave(const parser::EndfileStmt &) {`.
  **L728 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Leave(const parser::EndfileStmt &) {`。
- **L729 EN**: Executes a call or declaration centered on `CheckForPureSubprogram`.
  **L729 CN**: 执行以 `CheckForPureSubprogram` 为核心的调用或声明。
- **L730 EN**: Continues logic associated with callable symbol `CheckForRequiredSpecifier`.
  **L730 CN**: 继续与可调用符号 `CheckForRequiredSpecifier` 相关的逻辑。
- **L731 EN**: Continues logic associated with callable symbol `test`.
  **L731 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L732 EN**: Executes a call or declaration centered on `CheckForUselessIomsg`.
  **L732 CN**: 执行以 `CheckForUselessIomsg` 为核心的调用或声明。
- **L733 EN**: Executes a call or declaration centered on `Done`.
  **L733 CN**: 执行以 `Done` 为核心的调用或声明。
- **L734 EN**: Closes the current lexical scope or compound statement.
  **L734 CN**: 结束当前词法作用域或复合语句块。
- **L735 EN**: Blank line separating nearby declarations or logic blocks.
  **L735 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L736 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Leave(const parser::FlushStmt &) {`.
  **L736 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Leave(const parser::FlushStmt &) {`。
- **L737 EN**: Executes a call or declaration centered on `CheckForPureSubprogram`.
  **L737 CN**: 执行以 `CheckForPureSubprogram` 为核心的调用或声明。
- **L738 EN**: Continues logic associated with callable symbol `CheckForRequiredSpecifier`.
  **L738 CN**: 继续与可调用符号 `CheckForRequiredSpecifier` 相关的逻辑。
- **L739 EN**: Continues logic associated with callable symbol `test`.
  **L739 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L740 EN**: Executes a call or declaration centered on `CheckForUselessIomsg`.
  **L740 CN**: 执行以 `CheckForUselessIomsg` 为核心的调用或声明。
- **L741 EN**: Executes a call or declaration centered on `Done`.
  **L741 CN**: 执行以 `Done` 为核心的调用或声明。
- **L742 EN**: Closes the current lexical scope or compound statement.
  **L742 CN**: 结束当前词法作用域或复合语句块。
- **L743 EN**: Blank line separating nearby declarations or logic blocks.
  **L743 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L744 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Leave(const parser::InquireStmt &stmt) {`.
  **L744 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Leave(const parser::InquireStmt &stmt) {`。

### Lines 745-768

````cpp
  if (std::get_if<std::list<parser::InquireSpec>>(&stmt.u)) {
    CheckForPureSubprogram();
    // Inquire by unit or by file (vs. by output list).
    CheckForRequiredSpecifier(
        flags_.test(Flag::NumberUnit) || specifierSet_.test(IoSpecKind::File),
        "UNIT number or FILE"); // C1246
    CheckForProhibitedSpecifier(IoSpecKind::File, IoSpecKind::Unit); // C1246
    CheckForRequiredSpecifier(IoSpecKind::Id, IoSpecKind::Pending); // C1248
    CheckForUselessIomsg();
  }
  Done();
}

void IoChecker::Leave(const parser::OpenStmt &) {
  CheckForPureSubprogram();
  CheckForRequiredSpecifier(specifierSet_.test(IoSpecKind::Unit) ||
          specifierSet_.test(IoSpecKind::Newunit),
      "UNIT or NEWUNIT"); // C1204, C1205
  CheckForProhibitedSpecifier(
      IoSpecKind::Newunit, IoSpecKind::Unit); // C1204, C1205
  CheckForRequiredSpecifier(flags_.test(Flag::StatusNew), "STATUS='NEW'",
      IoSpecKind::File); // 12.5.6.10
  CheckForRequiredSpecifier(flags_.test(Flag::StatusReplace),
      "STATUS='REPLACE'", IoSpecKind::File); // 12.5.6.10
````
- **L745 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L745 CN**: 开始 `if` 控制流语句并计算其条件。
- **L746 EN**: Executes a call or declaration centered on `CheckForPureSubprogram`.
  **L746 CN**: 执行以 `CheckForPureSubprogram` 为核心的调用或声明。
- **L747 EN**: Comment explains nearby logic, intent, or metadata: `Inquire by unit or by file (vs. by output list).`.
  **L747 CN**: 注释说明附近代码的逻辑、意图或元数据：`Inquire by unit or by file (vs. by output list).`。
- **L748 EN**: Continues logic associated with callable symbol `CheckForRequiredSpecifier`.
  **L748 CN**: 继续与可调用符号 `CheckForRequiredSpecifier` 相关的逻辑。
- **L749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `flags_.test(Flag::NumberUnit) || specifierSet_.test(IoSpecKind::File),`.
  **L749 CN**: 继续一个多行参数列表、初始化器或聚合项：`flags_.test(Flag::NumberUnit) || specifierSet_.test(IoSpecKind::File),`。
- **L750 EN**: Continues the surrounding expression or declaration: `"UNIT number or FILE"); // C1246`.
  **L750 CN**: 继续构造周围的表达式或声明：`"UNIT number or FILE"); // C1246`。
- **L751 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L751 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L752 EN**: Continues logic associated with callable symbol `CheckForRequiredSpecifier`.
  **L752 CN**: 继续与可调用符号 `CheckForRequiredSpecifier` 相关的逻辑。
- **L753 EN**: Executes a call or declaration centered on `CheckForUselessIomsg`.
  **L753 CN**: 执行以 `CheckForUselessIomsg` 为核心的调用或声明。
- **L754 EN**: Closes the current lexical scope or compound statement.
  **L754 CN**: 结束当前词法作用域或复合语句块。
- **L755 EN**: Executes a call or declaration centered on `Done`.
  **L755 CN**: 执行以 `Done` 为核心的调用或声明。
- **L756 EN**: Closes the current lexical scope or compound statement.
  **L756 CN**: 结束当前词法作用域或复合语句块。
- **L757 EN**: Blank line separating nearby declarations or logic blocks.
  **L757 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L758 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Leave(const parser::OpenStmt &) {`.
  **L758 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Leave(const parser::OpenStmt &) {`。
- **L759 EN**: Executes a call or declaration centered on `CheckForPureSubprogram`.
  **L759 CN**: 执行以 `CheckForPureSubprogram` 为核心的调用或声明。
- **L760 EN**: Continues logic associated with callable symbol `CheckForRequiredSpecifier`.
  **L760 CN**: 继续与可调用符号 `CheckForRequiredSpecifier` 相关的逻辑。
- **L761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `specifierSet_.test(IoSpecKind::Newunit),`.
  **L761 CN**: 继续一个多行参数列表、初始化器或聚合项：`specifierSet_.test(IoSpecKind::Newunit),`。
- **L762 EN**: Continues the surrounding expression or declaration: `"UNIT or NEWUNIT"); // C1204, C1205`.
  **L762 CN**: 继续构造周围的表达式或声明：`"UNIT or NEWUNIT"); // C1204, C1205`。
- **L763 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L763 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L764 EN**: Continues the surrounding expression or declaration: `IoSpecKind::Newunit, IoSpecKind::Unit); // C1204, C1205`.
  **L764 CN**: 继续构造周围的表达式或声明：`IoSpecKind::Newunit, IoSpecKind::Unit); // C1204, C1205`。
- **L765 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForRequiredSpecifier(flags_.test(Flag::StatusNew), "STATUS='NEW'",`.
  **L765 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForRequiredSpecifier(flags_.test(Flag::StatusNew), "STATUS='NEW'",`。
- **L766 EN**: Continues the surrounding expression or declaration: `IoSpecKind::File); // 12.5.6.10`.
  **L766 CN**: 继续构造周围的表达式或声明：`IoSpecKind::File); // 12.5.6.10`。
- **L767 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForRequiredSpecifier(flags_.test(Flag::StatusReplace),`.
  **L767 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForRequiredSpecifier(flags_.test(Flag::StatusReplace),`。
- **L768 EN**: Continues the surrounding expression or declaration: `"STATUS='REPLACE'", IoSpecKind::File); // 12.5.6.10`.
  **L768 CN**: 继续构造周围的表达式或声明：`"STATUS='REPLACE'", IoSpecKind::File); // 12.5.6.10`。

### Lines 769-792

````cpp
  CheckForProhibitedSpecifier(flags_.test(Flag::StatusScratch),
      "STATUS='SCRATCH'", IoSpecKind::File); // 12.5.6.10
  if (flags_.test(Flag::KnownStatus)) {
    CheckForRequiredSpecifier(IoSpecKind::Newunit,
        specifierSet_.test(IoSpecKind::File) ||
            flags_.test(Flag::StatusScratch),
        "FILE or STATUS='SCRATCH'"); // 12.5.6.12
  } else {
    CheckForRequiredSpecifier(IoSpecKind::Newunit,
        specifierSet_.test(IoSpecKind::File) ||
            specifierSet_.test(IoSpecKind::Status),
        "FILE or STATUS"); // 12.5.6.12
  }
  if (flags_.test(Flag::KnownAccess)) {
    CheckForRequiredSpecifier(flags_.test(Flag::AccessDirect),
        "ACCESS='DIRECT'", IoSpecKind::Recl); // 12.5.6.15
    CheckForProhibitedSpecifier(flags_.test(Flag::AccessStream),
        "STATUS='STREAM'", IoSpecKind::Recl); // 12.5.6.15
  }
  CheckForUselessIomsg();
  Done();
}

void IoChecker::Leave(const parser::PrintStmt &) {
````
- **L769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForProhibitedSpecifier(flags_.test(Flag::StatusScratch),`.
  **L769 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForProhibitedSpecifier(flags_.test(Flag::StatusScratch),`。
- **L770 EN**: Continues the surrounding expression or declaration: `"STATUS='SCRATCH'", IoSpecKind::File); // 12.5.6.10`.
  **L770 CN**: 继续构造周围的表达式或声明：`"STATUS='SCRATCH'", IoSpecKind::File); // 12.5.6.10`。
- **L771 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L771 CN**: 开始 `if` 控制流语句并计算其条件。
- **L772 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForRequiredSpecifier(IoSpecKind::Newunit,`.
  **L772 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForRequiredSpecifier(IoSpecKind::Newunit,`。
- **L773 EN**: Continues logic associated with callable symbol `test`.
  **L773 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `flags_.test(Flag::StatusScratch),`.
  **L774 CN**: 继续一个多行参数列表、初始化器或聚合项：`flags_.test(Flag::StatusScratch),`。
- **L775 EN**: Continues the surrounding expression or declaration: `"FILE or STATUS='SCRATCH'"); // 12.5.6.12`.
  **L775 CN**: 继续构造周围的表达式或声明：`"FILE or STATUS='SCRATCH'"); // 12.5.6.12`。
- **L776 EN**: Transitions from the previous branch into the alternative path.
  **L776 CN**: 从前一个分支过渡到备选路径。
- **L777 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForRequiredSpecifier(IoSpecKind::Newunit,`.
  **L777 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForRequiredSpecifier(IoSpecKind::Newunit,`。
- **L778 EN**: Continues logic associated with callable symbol `test`.
  **L778 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `specifierSet_.test(IoSpecKind::Status),`.
  **L779 CN**: 继续一个多行参数列表、初始化器或聚合项：`specifierSet_.test(IoSpecKind::Status),`。
- **L780 EN**: Continues the surrounding expression or declaration: `"FILE or STATUS"); // 12.5.6.12`.
  **L780 CN**: 继续构造周围的表达式或声明：`"FILE or STATUS"); // 12.5.6.12`。
- **L781 EN**: Closes the current lexical scope or compound statement.
  **L781 CN**: 结束当前词法作用域或复合语句块。
- **L782 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L782 CN**: 开始 `if` 控制流语句并计算其条件。
- **L783 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForRequiredSpecifier(flags_.test(Flag::AccessDirect),`.
  **L783 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForRequiredSpecifier(flags_.test(Flag::AccessDirect),`。
- **L784 EN**: Continues the surrounding expression or declaration: `"ACCESS='DIRECT'", IoSpecKind::Recl); // 12.5.6.15`.
  **L784 CN**: 继续构造周围的表达式或声明：`"ACCESS='DIRECT'", IoSpecKind::Recl); // 12.5.6.15`。
- **L785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForProhibitedSpecifier(flags_.test(Flag::AccessStream),`.
  **L785 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForProhibitedSpecifier(flags_.test(Flag::AccessStream),`。
- **L786 EN**: Continues the surrounding expression or declaration: `"STATUS='STREAM'", IoSpecKind::Recl); // 12.5.6.15`.
  **L786 CN**: 继续构造周围的表达式或声明：`"STATUS='STREAM'", IoSpecKind::Recl); // 12.5.6.15`。
- **L787 EN**: Closes the current lexical scope or compound statement.
  **L787 CN**: 结束当前词法作用域或复合语句块。
- **L788 EN**: Executes a call or declaration centered on `CheckForUselessIomsg`.
  **L788 CN**: 执行以 `CheckForUselessIomsg` 为核心的调用或声明。
- **L789 EN**: Executes a call or declaration centered on `Done`.
  **L789 CN**: 执行以 `Done` 为核心的调用或声明。
- **L790 EN**: Closes the current lexical scope or compound statement.
  **L790 CN**: 结束当前词法作用域或复合语句块。
- **L791 EN**: Blank line separating nearby declarations or logic blocks.
  **L791 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L792 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Leave(const parser::PrintStmt &) {`.
  **L792 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Leave(const parser::PrintStmt &) {`。

### Lines 793-816

````cpp
  CheckForPureSubprogram();
  CheckForUselessIomsg();
  Done();
}

static const parser::Name *FindNamelist(
    const std::list<parser::IoControlSpec> &controls) {
  for (const auto &control : controls) {
    if (const parser::Name * namelist{std::get_if<parser::Name>(&control.u)}) {
      if (namelist->symbol &&
          namelist->symbol->GetUltimate().has<NamelistDetails>()) {
        return namelist;
      }
    }
  }
  return nullptr;
}

static void CheckForDoVariable(
    const parser::ReadStmt &readStmt, SemanticsContext &context) {
  const std::list<parser::InputItem> &items{readStmt.items};
  for (const auto &item : items) {
    if (const parser::Variable *
        variable{std::get_if<parser::Variable>(&item.u)}) {
````
- **L793 EN**: Executes a call or declaration centered on `CheckForPureSubprogram`.
  **L793 CN**: 执行以 `CheckForPureSubprogram` 为核心的调用或声明。
- **L794 EN**: Executes a call or declaration centered on `CheckForUselessIomsg`.
  **L794 CN**: 执行以 `CheckForUselessIomsg` 为核心的调用或声明。
- **L795 EN**: Executes a call or declaration centered on `Done`.
  **L795 CN**: 执行以 `Done` 为核心的调用或声明。
- **L796 EN**: Closes the current lexical scope or compound statement.
  **L796 CN**: 结束当前词法作用域或复合语句块。
- **L797 EN**: Blank line separating nearby declarations or logic blocks.
  **L797 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L798 EN**: Continues logic associated with callable symbol `FindNamelist`.
  **L798 CN**: 继续与可调用符号 `FindNamelist` 相关的逻辑。
- **L799 EN**: Continues the surrounding expression or declaration: `const std::list<parser::IoControlSpec> &controls) {`.
  **L799 CN**: 继续构造周围的表达式或声明：`const std::list<parser::IoControlSpec> &controls) {`。
- **L800 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L800 CN**: 开始 `for` 控制流语句并计算其条件。
- **L801 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L801 CN**: 开始 `if` 控制流语句并计算其条件。
- **L802 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L802 CN**: 开始 `if` 控制流语句并计算其条件。
- **L803 EN**: Starts a function, method, lambda, or structured scope: `namelist->symbol->GetUltimate().has<NamelistDetails>()) {`.
  **L803 CN**: 开始一个函数、方法、lambda 或结构化作用域：`namelist->symbol->GetUltimate().has<NamelistDetails>()) {`。
- **L804 EN**: Returns from the current function with `namelist`.
  **L804 CN**: 以 `namelist` 从当前函数返回。
- **L805 EN**: Closes the current lexical scope or compound statement.
  **L805 CN**: 结束当前词法作用域或复合语句块。
- **L806 EN**: Closes the current lexical scope or compound statement.
  **L806 CN**: 结束当前词法作用域或复合语句块。
- **L807 EN**: Closes the current lexical scope or compound statement.
  **L807 CN**: 结束当前词法作用域或复合语句块。
- **L808 EN**: Returns from the current function with `nullptr`.
  **L808 CN**: 以 `nullptr` 从当前函数返回。
- **L809 EN**: Closes the current lexical scope or compound statement.
  **L809 CN**: 结束当前词法作用域或复合语句块。
- **L810 EN**: Blank line separating nearby declarations or logic blocks.
  **L810 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L811 EN**: Continues logic associated with callable symbol `CheckForDoVariable`.
  **L811 CN**: 继续与可调用符号 `CheckForDoVariable` 相关的逻辑。
- **L812 EN**: Continues the surrounding expression or declaration: `const parser::ReadStmt &readStmt, SemanticsContext &context) {`.
  **L812 CN**: 继续构造周围的表达式或声明：`const parser::ReadStmt &readStmt, SemanticsContext &context) {`。
- **L813 EN**: Executes a standalone statement or declaration: `const std::list<parser::InputItem> &items{readStmt.items};`.
  **L813 CN**: 执行一条独立语句或声明：`const std::list<parser::InputItem> &items{readStmt.items};`。
- **L814 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L814 CN**: 开始 `for` 控制流语句并计算其条件。
- **L815 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L815 CN**: 开始 `if` 控制流语句并计算其条件。
- **L816 EN**: Starts a function, method, lambda, or structured scope: `variable{std::get_if<parser::Variable>(&item.u)}) {`.
  **L816 CN**: 开始一个函数、方法、lambda 或结构化作用域：`variable{std::get_if<parser::Variable>(&item.u)}) {`。

### Lines 817-840

````cpp
      context.CheckIndexVarRedefine(*variable);
    }
  }
}

void IoChecker::Leave(const parser::ReadStmt &readStmt) {
  if (!flags_.test(Flag::InternalUnit)) {
    CheckForPureSubprogram();
  }
  if (const parser::Name * namelist{FindNamelist(readStmt.controls)}) {
    if (namelist->symbol) {
      CheckNamelist(*namelist->symbol, common::DefinedIo::ReadFormatted,
          namelist->source);
    }
  }
  CheckForDoVariable(readStmt, context_);
  if (!flags_.test(Flag::IoControlList)) {
    Done();
    return;
  }
  LeaveReadWrite();
  CheckForProhibitedSpecifier(IoSpecKind::Delim); // C1212
  CheckForProhibitedSpecifier(IoSpecKind::Sign); // C1212
  CheckForProhibitedSpecifier(IoSpecKind::Leading_Zero); // F'2023 C1212
````
- **L817 EN**: Executes a call or declaration centered on `context.CheckIndexVarRedefine`.
  **L817 CN**: 执行以 `context.CheckIndexVarRedefine` 为核心的调用或声明。
- **L818 EN**: Closes the current lexical scope or compound statement.
  **L818 CN**: 结束当前词法作用域或复合语句块。
- **L819 EN**: Closes the current lexical scope or compound statement.
  **L819 CN**: 结束当前词法作用域或复合语句块。
- **L820 EN**: Closes the current lexical scope or compound statement.
  **L820 CN**: 结束当前词法作用域或复合语句块。
- **L821 EN**: Blank line separating nearby declarations or logic blocks.
  **L821 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L822 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Leave(const parser::ReadStmt &readStmt) {`.
  **L822 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Leave(const parser::ReadStmt &readStmt) {`。
- **L823 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L823 CN**: 开始 `if` 控制流语句并计算其条件。
- **L824 EN**: Executes a call or declaration centered on `CheckForPureSubprogram`.
  **L824 CN**: 执行以 `CheckForPureSubprogram` 为核心的调用或声明。
- **L825 EN**: Closes the current lexical scope or compound statement.
  **L825 CN**: 结束当前词法作用域或复合语句块。
- **L826 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L826 CN**: 开始 `if` 控制流语句并计算其条件。
- **L827 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L827 CN**: 开始 `if` 控制流语句并计算其条件。
- **L828 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckNamelist(*namelist->symbol, common::DefinedIo::ReadFormatted,`.
  **L828 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckNamelist(*namelist->symbol, common::DefinedIo::ReadFormatted,`。
- **L829 EN**: Executes a standalone statement or declaration: `namelist->source);`.
  **L829 CN**: 执行一条独立语句或声明：`namelist->source);`。
- **L830 EN**: Closes the current lexical scope or compound statement.
  **L830 CN**: 结束当前词法作用域或复合语句块。
- **L831 EN**: Closes the current lexical scope or compound statement.
  **L831 CN**: 结束当前词法作用域或复合语句块。
- **L832 EN**: Executes a call or declaration centered on `CheckForDoVariable`.
  **L832 CN**: 执行以 `CheckForDoVariable` 为核心的调用或声明。
- **L833 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L833 CN**: 开始 `if` 控制流语句并计算其条件。
- **L834 EN**: Executes a call or declaration centered on `Done`.
  **L834 CN**: 执行以 `Done` 为核心的调用或声明。
- **L835 EN**: Returns from the current function with `void`.
  **L835 CN**: 以 `void` 从当前函数返回。
- **L836 EN**: Closes the current lexical scope or compound statement.
  **L836 CN**: 结束当前词法作用域或复合语句块。
- **L837 EN**: Executes a call or declaration centered on `LeaveReadWrite`.
  **L837 CN**: 执行以 `LeaveReadWrite` 为核心的调用或声明。
- **L838 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L838 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L839 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L839 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L840 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L840 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。

### Lines 841-864

````cpp
  CheckForProhibitedSpecifier(IoSpecKind::Rec, IoSpecKind::End); // C1220
  if (specifierSet_.test(IoSpecKind::Size)) {
    // F'2023 C1214 - allow with a warning
    if (context_.ShouldWarn(common::LanguageFeature::ListDirectedSize)) {
      if (specifierSet_.test(IoSpecKind::Nml)) {
        context_.Say("If NML appears, SIZE should not appear"_port_en_US);
      } else if (flags_.test(Flag::StarFmt)) {
        context_.Say("If FMT=* appears, SIZE should not appear"_port_en_US);
      }
    }
  }
  CheckForRequiredSpecifier(IoSpecKind::Eor,
      specifierSet_.test(IoSpecKind::Advance) && !flags_.test(Flag::AdvanceYes),
      "ADVANCE with value 'NO'"); // C1222 + 12.6.2.1p2
  CheckForRequiredSpecifier(IoSpecKind::Blank, flags_.test(Flag::FmtOrNml),
      "FMT or NML"); // C1227
  CheckForRequiredSpecifier(
      IoSpecKind::Pad, flags_.test(Flag::FmtOrNml), "FMT or NML"); // C1227
  Done();
}

void IoChecker::Leave(const parser::RewindStmt &) {
  CheckForRequiredSpecifier(
      flags_.test(Flag::NumberUnit), "UNIT number"); // C1240
````
- **L841 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L841 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L842 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L842 CN**: 开始 `if` 控制流语句并计算其条件。
- **L843 EN**: Comment explains nearby logic, intent, or metadata: `F'2023 C1214 - allow with a warning`.
  **L843 CN**: 注释说明附近代码的逻辑、意图或元数据：`F'2023 C1214 - allow with a warning`。
- **L844 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L844 CN**: 开始 `if` 控制流语句并计算其条件。
- **L845 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L845 CN**: 开始 `if` 控制流语句并计算其条件。
- **L846 EN**: Executes a call or declaration centered on `context_.Say`.
  **L846 CN**: 执行以 `context_.Say` 为核心的调用或声明。
- **L847 EN**: Transitions from the previous branch into an `else if` condition.
  **L847 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L848 EN**: Executes a call or declaration centered on `context_.Say`.
  **L848 CN**: 执行以 `context_.Say` 为核心的调用或声明。
- **L849 EN**: Closes the current lexical scope or compound statement.
  **L849 CN**: 结束当前词法作用域或复合语句块。
- **L850 EN**: Closes the current lexical scope or compound statement.
  **L850 CN**: 结束当前词法作用域或复合语句块。
- **L851 EN**: Closes the current lexical scope or compound statement.
  **L851 CN**: 结束当前词法作用域或复合语句块。
- **L852 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForRequiredSpecifier(IoSpecKind::Eor,`.
  **L852 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForRequiredSpecifier(IoSpecKind::Eor,`。
- **L853 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `specifierSet_.test(IoSpecKind::Advance) && !flags_.test(Flag::AdvanceYes),`.
  **L853 CN**: 继续一个多行参数列表、初始化器或聚合项：`specifierSet_.test(IoSpecKind::Advance) && !flags_.test(Flag::AdvanceYes),`。
- **L854 EN**: Continues the surrounding expression or declaration: `"ADVANCE with value 'NO'"); // C1222 + 12.6.2.1p2`.
  **L854 CN**: 继续构造周围的表达式或声明：`"ADVANCE with value 'NO'"); // C1222 + 12.6.2.1p2`。
- **L855 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForRequiredSpecifier(IoSpecKind::Blank, flags_.test(Flag::FmtOrNml),`.
  **L855 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForRequiredSpecifier(IoSpecKind::Blank, flags_.test(Flag::FmtOrNml),`。
- **L856 EN**: Continues the surrounding expression or declaration: `"FMT or NML"); // C1227`.
  **L856 CN**: 继续构造周围的表达式或声明：`"FMT or NML"); // C1227`。
- **L857 EN**: Continues logic associated with callable symbol `CheckForRequiredSpecifier`.
  **L857 CN**: 继续与可调用符号 `CheckForRequiredSpecifier` 相关的逻辑。
- **L858 EN**: Continues logic associated with callable symbol `test`.
  **L858 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L859 EN**: Executes a call or declaration centered on `Done`.
  **L859 CN**: 执行以 `Done` 为核心的调用或声明。
- **L860 EN**: Closes the current lexical scope or compound statement.
  **L860 CN**: 结束当前词法作用域或复合语句块。
- **L861 EN**: Blank line separating nearby declarations or logic blocks.
  **L861 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L862 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Leave(const parser::RewindStmt &) {`.
  **L862 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Leave(const parser::RewindStmt &) {`。
- **L863 EN**: Continues logic associated with callable symbol `CheckForRequiredSpecifier`.
  **L863 CN**: 继续与可调用符号 `CheckForRequiredSpecifier` 相关的逻辑。
- **L864 EN**: Continues logic associated with callable symbol `test`.
  **L864 CN**: 继续与可调用符号 `test` 相关的逻辑。

### Lines 865-888

````cpp
  CheckForPureSubprogram();
  CheckForUselessIomsg();
  Done();
}

void IoChecker::Leave(const parser::WaitStmt &) {
  CheckForRequiredSpecifier(
      flags_.test(Flag::NumberUnit), "UNIT number"); // C1237
  CheckForPureSubprogram();
  CheckForUselessIomsg();
  Done();
}

void IoChecker::Leave(const parser::WriteStmt &writeStmt) {
  if (!flags_.test(Flag::InternalUnit)) {
    CheckForPureSubprogram();
  }
  if (const parser::Name * namelist{FindNamelist(writeStmt.controls)}) {
    if (namelist->symbol) {
      CheckNamelist(*namelist->symbol, common::DefinedIo::WriteFormatted,
          namelist->source);
    }
  }
  LeaveReadWrite();
````
- **L865 EN**: Executes a call or declaration centered on `CheckForPureSubprogram`.
  **L865 CN**: 执行以 `CheckForPureSubprogram` 为核心的调用或声明。
- **L866 EN**: Executes a call or declaration centered on `CheckForUselessIomsg`.
  **L866 CN**: 执行以 `CheckForUselessIomsg` 为核心的调用或声明。
- **L867 EN**: Executes a call or declaration centered on `Done`.
  **L867 CN**: 执行以 `Done` 为核心的调用或声明。
- **L868 EN**: Closes the current lexical scope or compound statement.
  **L868 CN**: 结束当前词法作用域或复合语句块。
- **L869 EN**: Blank line separating nearby declarations or logic blocks.
  **L869 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L870 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Leave(const parser::WaitStmt &) {`.
  **L870 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Leave(const parser::WaitStmt &) {`。
- **L871 EN**: Continues logic associated with callable symbol `CheckForRequiredSpecifier`.
  **L871 CN**: 继续与可调用符号 `CheckForRequiredSpecifier` 相关的逻辑。
- **L872 EN**: Continues logic associated with callable symbol `test`.
  **L872 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L873 EN**: Executes a call or declaration centered on `CheckForPureSubprogram`.
  **L873 CN**: 执行以 `CheckForPureSubprogram` 为核心的调用或声明。
- **L874 EN**: Executes a call or declaration centered on `CheckForUselessIomsg`.
  **L874 CN**: 执行以 `CheckForUselessIomsg` 为核心的调用或声明。
- **L875 EN**: Executes a call or declaration centered on `Done`.
  **L875 CN**: 执行以 `Done` 为核心的调用或声明。
- **L876 EN**: Closes the current lexical scope or compound statement.
  **L876 CN**: 结束当前词法作用域或复合语句块。
- **L877 EN**: Blank line separating nearby declarations or logic blocks.
  **L877 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L878 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::Leave(const parser::WriteStmt &writeStmt) {`.
  **L878 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::Leave(const parser::WriteStmt &writeStmt) {`。
- **L879 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L879 CN**: 开始 `if` 控制流语句并计算其条件。
- **L880 EN**: Executes a call or declaration centered on `CheckForPureSubprogram`.
  **L880 CN**: 执行以 `CheckForPureSubprogram` 为核心的调用或声明。
- **L881 EN**: Closes the current lexical scope or compound statement.
  **L881 CN**: 结束当前词法作用域或复合语句块。
- **L882 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L882 CN**: 开始 `if` 控制流语句并计算其条件。
- **L883 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L883 CN**: 开始 `if` 控制流语句并计算其条件。
- **L884 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckNamelist(*namelist->symbol, common::DefinedIo::WriteFormatted,`.
  **L884 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckNamelist(*namelist->symbol, common::DefinedIo::WriteFormatted,`。
- **L885 EN**: Executes a standalone statement or declaration: `namelist->source);`.
  **L885 CN**: 执行一条独立语句或声明：`namelist->source);`。
- **L886 EN**: Closes the current lexical scope or compound statement.
  **L886 CN**: 结束当前词法作用域或复合语句块。
- **L887 EN**: Closes the current lexical scope or compound statement.
  **L887 CN**: 结束当前词法作用域或复合语句块。
- **L888 EN**: Executes a call or declaration centered on `LeaveReadWrite`.
  **L888 CN**: 执行以 `LeaveReadWrite` 为核心的调用或声明。

### Lines 889-912

````cpp
  CheckForProhibitedSpecifier(IoSpecKind::Blank); // C1213
  CheckForProhibitedSpecifier(IoSpecKind::End); // C1213
  CheckForProhibitedSpecifier(IoSpecKind::Eor); // C1213
  CheckForProhibitedSpecifier(IoSpecKind::Pad); // C1213
  CheckForProhibitedSpecifier(IoSpecKind::Size); // C1213
  CheckForRequiredSpecifier(
      IoSpecKind::Sign, flags_.test(Flag::FmtOrNml), "FMT or NML"); // C1227
  CheckForRequiredSpecifier(IoSpecKind::Leading_Zero,
      flags_.test(Flag::FmtOrNml), "FMT or NML"); // F'2023 C1227
  CheckForRequiredSpecifier(IoSpecKind::Delim,
      flags_.test(Flag::StarFmt) || specifierSet_.test(IoSpecKind::Nml),
      "FMT=* or NML"); // C1228
  Done();
}

void IoChecker::LeaveReadWrite() const {
  CheckForRequiredSpecifier(IoSpecKind::Unit); // C1211
  CheckForRequiredSpecifier(flags_.test(Flag::InternalUnit),
      "UNIT=internal-file", flags_.test(Flag::FmtOrNml), "FMT or NML");
  CheckForProhibitedSpecifier(IoSpecKind::Nml, IoSpecKind::Rec); // C1216
  CheckForProhibitedSpecifier(IoSpecKind::Nml, IoSpecKind::Fmt); // C1216
  CheckForProhibitedSpecifier(
      IoSpecKind::Nml, flags_.test(Flag::DataList), "a data list"); // C1216
  CheckForProhibitedSpecifier(flags_.test(Flag::InternalUnit),
````
- **L889 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L889 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L890 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L890 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L891 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L891 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L892 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L892 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L893 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L893 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L894 EN**: Continues logic associated with callable symbol `CheckForRequiredSpecifier`.
  **L894 CN**: 继续与可调用符号 `CheckForRequiredSpecifier` 相关的逻辑。
- **L895 EN**: Continues logic associated with callable symbol `test`.
  **L895 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L896 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForRequiredSpecifier(IoSpecKind::Leading_Zero,`.
  **L896 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForRequiredSpecifier(IoSpecKind::Leading_Zero,`。
- **L897 EN**: Continues logic associated with callable symbol `test`.
  **L897 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L898 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForRequiredSpecifier(IoSpecKind::Delim,`.
  **L898 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForRequiredSpecifier(IoSpecKind::Delim,`。
- **L899 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `flags_.test(Flag::StarFmt) || specifierSet_.test(IoSpecKind::Nml),`.
  **L899 CN**: 继续一个多行参数列表、初始化器或聚合项：`flags_.test(Flag::StarFmt) || specifierSet_.test(IoSpecKind::Nml),`。
- **L900 EN**: Continues the surrounding expression or declaration: `"FMT=* or NML"); // C1228`.
  **L900 CN**: 继续构造周围的表达式或声明：`"FMT=* or NML"); // C1228`。
- **L901 EN**: Executes a call or declaration centered on `Done`.
  **L901 CN**: 执行以 `Done` 为核心的调用或声明。
- **L902 EN**: Closes the current lexical scope or compound statement.
  **L902 CN**: 结束当前词法作用域或复合语句块。
- **L903 EN**: Blank line separating nearby declarations or logic blocks.
  **L903 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L904 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::LeaveReadWrite() const {`.
  **L904 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::LeaveReadWrite() const {`。
- **L905 EN**: Continues logic associated with callable symbol `CheckForRequiredSpecifier`.
  **L905 CN**: 继续与可调用符号 `CheckForRequiredSpecifier` 相关的逻辑。
- **L906 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForRequiredSpecifier(flags_.test(Flag::InternalUnit),`.
  **L906 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForRequiredSpecifier(flags_.test(Flag::InternalUnit),`。
- **L907 EN**: Executes a call or declaration centered on `flags_.test`.
  **L907 CN**: 执行以 `flags_.test` 为核心的调用或声明。
- **L908 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L908 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L909 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L909 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L910 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L910 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L911 EN**: Continues logic associated with callable symbol `test`.
  **L911 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L912 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForProhibitedSpecifier(flags_.test(Flag::InternalUnit),`.
  **L912 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForProhibitedSpecifier(flags_.test(Flag::InternalUnit),`。

### Lines 913-936

````cpp
      "UNIT=internal-file", IoSpecKind::Pos); // C1219
  CheckForProhibitedSpecifier(flags_.test(Flag::InternalUnit),
      "UNIT=internal-file", IoSpecKind::Rec); // C1219
  CheckForProhibitedSpecifier(
      flags_.test(Flag::StarUnit), "UNIT=*", IoSpecKind::Pos); // C1219
  CheckForProhibitedSpecifier(
      flags_.test(Flag::StarUnit), "UNIT=*", IoSpecKind::Rec); // C1219
  CheckForProhibitedSpecifier(
      IoSpecKind::Rec, flags_.test(Flag::StarFmt), "FMT=*"); // C1220
  CheckForRequiredSpecifier(IoSpecKind::Advance,
      flags_.test(Flag::CharFmt) || flags_.test(Flag::LabelFmt) ||
          flags_.test(Flag::AssignFmt),
      "an explicit format"); // C1221
  CheckForProhibitedSpecifier(IoSpecKind::Advance,
      flags_.test(Flag::InternalUnit), "UNIT=internal-file"); // C1221
  CheckForProhibitedSpecifier(IoSpecKind::Advance, IoSpecKind::Rec); // C1221
  CheckForRequiredSpecifier(flags_.test(Flag::AsynchronousYes),
      "ASYNCHRONOUS='YES'", flags_.test(Flag::NumberUnit),
      "UNIT=number"); // C1224
  CheckForRequiredSpecifier(IoSpecKind::Id, flags_.test(Flag::AsynchronousYes),
      "ASYNCHRONOUS='YES'"); // C1225
  CheckForProhibitedSpecifier(IoSpecKind::Pos, IoSpecKind::Rec); // C1226
  CheckForRequiredSpecifier(IoSpecKind::Decimal, flags_.test(Flag::FmtOrNml),
      "FMT or NML"); // C1227
````
- **L913 EN**: Continues the surrounding expression or declaration: `"UNIT=internal-file", IoSpecKind::Pos); // C1219`.
  **L913 CN**: 继续构造周围的表达式或声明：`"UNIT=internal-file", IoSpecKind::Pos); // C1219`。
- **L914 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForProhibitedSpecifier(flags_.test(Flag::InternalUnit),`.
  **L914 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForProhibitedSpecifier(flags_.test(Flag::InternalUnit),`。
- **L915 EN**: Continues the surrounding expression or declaration: `"UNIT=internal-file", IoSpecKind::Rec); // C1219`.
  **L915 CN**: 继续构造周围的表达式或声明：`"UNIT=internal-file", IoSpecKind::Rec); // C1219`。
- **L916 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L916 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L917 EN**: Continues logic associated with callable symbol `test`.
  **L917 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L918 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L918 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L919 EN**: Continues logic associated with callable symbol `test`.
  **L919 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L920 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L920 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L921 EN**: Continues logic associated with callable symbol `test`.
  **L921 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L922 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForRequiredSpecifier(IoSpecKind::Advance,`.
  **L922 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForRequiredSpecifier(IoSpecKind::Advance,`。
- **L923 EN**: Continues logic associated with callable symbol `test`.
  **L923 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L924 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `flags_.test(Flag::AssignFmt),`.
  **L924 CN**: 继续一个多行参数列表、初始化器或聚合项：`flags_.test(Flag::AssignFmt),`。
- **L925 EN**: Continues the surrounding expression or declaration: `"an explicit format"); // C1221`.
  **L925 CN**: 继续构造周围的表达式或声明：`"an explicit format"); // C1221`。
- **L926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForProhibitedSpecifier(IoSpecKind::Advance,`.
  **L926 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForProhibitedSpecifier(IoSpecKind::Advance,`。
- **L927 EN**: Continues logic associated with callable symbol `test`.
  **L927 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L928 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L928 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L929 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForRequiredSpecifier(flags_.test(Flag::AsynchronousYes),`.
  **L929 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForRequiredSpecifier(flags_.test(Flag::AsynchronousYes),`。
- **L930 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ASYNCHRONOUS='YES'", flags_.test(Flag::NumberUnit),`.
  **L930 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ASYNCHRONOUS='YES'", flags_.test(Flag::NumberUnit),`。
- **L931 EN**: Continues the surrounding expression or declaration: `"UNIT=number"); // C1224`.
  **L931 CN**: 继续构造周围的表达式或声明：`"UNIT=number"); // C1224`。
- **L932 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForRequiredSpecifier(IoSpecKind::Id, flags_.test(Flag::AsynchronousYes),`.
  **L932 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForRequiredSpecifier(IoSpecKind::Id, flags_.test(Flag::AsynchronousYes),`。
- **L933 EN**: Continues the surrounding expression or declaration: `"ASYNCHRONOUS='YES'"); // C1225`.
  **L933 CN**: 继续构造周围的表达式或声明：`"ASYNCHRONOUS='YES'"); // C1225`。
- **L934 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L934 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L935 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForRequiredSpecifier(IoSpecKind::Decimal, flags_.test(Flag::FmtOrNml),`.
  **L935 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForRequiredSpecifier(IoSpecKind::Decimal, flags_.test(Flag::FmtOrNml),`。
- **L936 EN**: Continues the surrounding expression or declaration: `"FMT or NML"); // C1227`.
  **L936 CN**: 继续构造周围的表达式或声明：`"FMT or NML"); // C1227`。

### Lines 937-960

````cpp
  CheckForRequiredSpecifier(IoSpecKind::Round, flags_.test(Flag::FmtOrNml),
      "FMT or NML"); // C1227
  CheckForUselessIomsg();
}

void IoChecker::SetSpecifier(IoSpecKind specKind) {
  if (stmt_ == IoStmtKind::None) {
    // FMT may appear on PRINT statements, which don't have any checks.
    // [IO]MSG and [IO]STAT parse symbols are shared with non-I/O statements.
    return;
  }
  // C1203, C1207, C1210, C1236, C1239, C1242, C1245
  if (specifierSet_.test(specKind)) {
    context_.Say("Duplicate %s specifier"_err_en_US,
        parser::ToUpperCaseLetters(common::EnumToString(specKind)));
  }
  specifierSet_.set(specKind);
}

void IoChecker::CheckStringValue(IoSpecKind specKind, const std::string &value,
    const parser::CharBlock &source) const {
  static std::unordered_map<IoSpecKind, const std::set<std::string>> specValues{
      {IoSpecKind::Access, {"DIRECT", "SEQUENTIAL", "STREAM"}},
      {IoSpecKind::Action, {"READ", "READWRITE", "WRITE"}},
````
- **L937 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckForRequiredSpecifier(IoSpecKind::Round, flags_.test(Flag::FmtOrNml),`.
  **L937 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckForRequiredSpecifier(IoSpecKind::Round, flags_.test(Flag::FmtOrNml),`。
- **L938 EN**: Continues the surrounding expression or declaration: `"FMT or NML"); // C1227`.
  **L938 CN**: 继续构造周围的表达式或声明：`"FMT or NML"); // C1227`。
- **L939 EN**: Executes a call or declaration centered on `CheckForUselessIomsg`.
  **L939 CN**: 执行以 `CheckForUselessIomsg` 为核心的调用或声明。
- **L940 EN**: Closes the current lexical scope or compound statement.
  **L940 CN**: 结束当前词法作用域或复合语句块。
- **L941 EN**: Blank line separating nearby declarations or logic blocks.
  **L941 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L942 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::SetSpecifier(IoSpecKind specKind) {`.
  **L942 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::SetSpecifier(IoSpecKind specKind) {`。
- **L943 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L943 CN**: 开始 `if` 控制流语句并计算其条件。
- **L944 EN**: Comment explains nearby logic, intent, or metadata: `FMT may appear on PRINT statements, which don't have any checks.`.
  **L944 CN**: 注释说明附近代码的逻辑、意图或元数据：`FMT may appear on PRINT statements, which don't have any checks.`。
- **L945 EN**: Comment explains nearby logic, intent, or metadata: `[IO]MSG and [IO]STAT parse symbols are shared with non-I/O statements.`.
  **L945 CN**: 注释说明附近代码的逻辑、意图或元数据：`[IO]MSG and [IO]STAT parse symbols are shared with non-I/O statements.`。
- **L946 EN**: Returns from the current function with `void`.
  **L946 CN**: 以 `void` 从当前函数返回。
- **L947 EN**: Closes the current lexical scope or compound statement.
  **L947 CN**: 结束当前词法作用域或复合语句块。
- **L948 EN**: Comment explains nearby logic, intent, or metadata: `C1203, C1207, C1210, C1236, C1239, C1242, C1245`.
  **L948 CN**: 注释说明附近代码的逻辑、意图或元数据：`C1203, C1207, C1210, C1236, C1239, C1242, C1245`。
- **L949 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L949 CN**: 开始 `if` 控制流语句并计算其条件。
- **L950 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say("Duplicate %s specifier"_err_en_US,`.
  **L950 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say("Duplicate %s specifier"_err_en_US,`。
- **L951 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L951 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L952 EN**: Closes the current lexical scope or compound statement.
  **L952 CN**: 结束当前词法作用域或复合语句块。
- **L953 EN**: Executes a call or declaration centered on `specifierSet_.set`.
  **L953 CN**: 执行以 `specifierSet_.set` 为核心的调用或声明。
- **L954 EN**: Closes the current lexical scope or compound statement.
  **L954 CN**: 结束当前词法作用域或复合语句块。
- **L955 EN**: Blank line separating nearby declarations or logic blocks.
  **L955 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L956 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void IoChecker::CheckStringValue(IoSpecKind specKind, const std::string &value,`.
  **L956 CN**: 继续一个多行参数列表、初始化器或聚合项：`void IoChecker::CheckStringValue(IoSpecKind specKind, const std::string &value,`。
- **L957 EN**: Continues the surrounding expression or declaration: `const parser::CharBlock &source) const {`.
  **L957 CN**: 继续构造周围的表达式或声明：`const parser::CharBlock &source) const {`。
- **L958 EN**: Continues the surrounding expression or declaration: `static std::unordered_map<IoSpecKind, const std::set<std::string>> specValues{`.
  **L958 CN**: 继续构造周围的表达式或声明：`static std::unordered_map<IoSpecKind, const std::set<std::string>> specValues{`。
- **L959 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Access, {"DIRECT", "SEQUENTIAL", "STREAM"}},`.
  **L959 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Access, {"DIRECT", "SEQUENTIAL", "STREAM"}},`。
- **L960 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Action, {"READ", "READWRITE", "WRITE"}},`.
  **L960 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Action, {"READ", "READWRITE", "WRITE"}},`。

### Lines 961-984

````cpp
      {IoSpecKind::Advance, {"NO", "YES"}},
      {IoSpecKind::Asynchronous, {"NO", "YES"}},
      {IoSpecKind::Blank, {"NULL", "ZERO"}},
      {IoSpecKind::Decimal, {"COMMA", "POINT"}},
      {IoSpecKind::Delim, {"APOSTROPHE", "NONE", "QUOTE"}},
      {IoSpecKind::Encoding, {"DEFAULT", "UTF-8"}},
      {IoSpecKind::Form, {"FORMATTED", "UNFORMATTED", "BINARY"}},
      {IoSpecKind::Pad, {"NO", "YES"}},
      {IoSpecKind::Position, {"APPEND", "ASIS", "REWIND"}},
      {IoSpecKind::Round,
          {"COMPATIBLE", "DOWN", "NEAREST", "PROCESSOR_DEFINED", "UP", "ZERO"}},
      {IoSpecKind::Sign, {"PLUS", "PROCESSOR_DEFINED", "SUPPRESS"}},
      {IoSpecKind::Leading_Zero, {"PRINT", "PROCESSOR_DEFINED", "SUPPRESS"}},
      {IoSpecKind::Status,
          // Open values; Close values are {"DELETE", "KEEP"}.
          {"NEW", "OLD", "REPLACE", "SCRATCH", "UNKNOWN"}},
      {IoSpecKind::Carriagecontrol, {"LIST", "FORTRAN", "NONE"}},
      {IoSpecKind::Convert, {"BIG_ENDIAN", "LITTLE_ENDIAN", "NATIVE", "SWAP"}},
      {IoSpecKind::Dispose, {"DELETE", "KEEP"}},
  };
  auto upper{Normalize(value)};
  if (specValues.at(specKind).count(upper) == 0) {
    if (specKind == IoSpecKind::Access && upper == "APPEND") {
      context_.Warn(common::LanguageFeature::OpenAccessAppend, source,
````
- **L961 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Advance, {"NO", "YES"}},`.
  **L961 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Advance, {"NO", "YES"}},`。
- **L962 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Asynchronous, {"NO", "YES"}},`.
  **L962 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Asynchronous, {"NO", "YES"}},`。
- **L963 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Blank, {"NULL", "ZERO"}},`.
  **L963 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Blank, {"NULL", "ZERO"}},`。
- **L964 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Decimal, {"COMMA", "POINT"}},`.
  **L964 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Decimal, {"COMMA", "POINT"}},`。
- **L965 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Delim, {"APOSTROPHE", "NONE", "QUOTE"}},`.
  **L965 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Delim, {"APOSTROPHE", "NONE", "QUOTE"}},`。
- **L966 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Encoding, {"DEFAULT", "UTF-8"}},`.
  **L966 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Encoding, {"DEFAULT", "UTF-8"}},`。
- **L967 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Form, {"FORMATTED", "UNFORMATTED", "BINARY"}},`.
  **L967 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Form, {"FORMATTED", "UNFORMATTED", "BINARY"}},`。
- **L968 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Pad, {"NO", "YES"}},`.
  **L968 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Pad, {"NO", "YES"}},`。
- **L969 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Position, {"APPEND", "ASIS", "REWIND"}},`.
  **L969 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Position, {"APPEND", "ASIS", "REWIND"}},`。
- **L970 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Round,`.
  **L970 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Round,`。
- **L971 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"COMPATIBLE", "DOWN", "NEAREST", "PROCESSOR_DEFINED", "UP", "ZERO"}},`.
  **L971 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"COMPATIBLE", "DOWN", "NEAREST", "PROCESSOR_DEFINED", "UP", "ZERO"}},`。
- **L972 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Sign, {"PLUS", "PROCESSOR_DEFINED", "SUPPRESS"}},`.
  **L972 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Sign, {"PLUS", "PROCESSOR_DEFINED", "SUPPRESS"}},`。
- **L973 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Leading_Zero, {"PRINT", "PROCESSOR_DEFINED", "SUPPRESS"}},`.
  **L973 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Leading_Zero, {"PRINT", "PROCESSOR_DEFINED", "SUPPRESS"}},`。
- **L974 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Status,`.
  **L974 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Status,`。
- **L975 EN**: Comment explains nearby logic, intent, or metadata: `Open values; Close values are {"DELETE", "KEEP"}.`.
  **L975 CN**: 注释说明附近代码的逻辑、意图或元数据：`Open values; Close values are {"DELETE", "KEEP"}.`。
- **L976 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"NEW", "OLD", "REPLACE", "SCRATCH", "UNKNOWN"}},`.
  **L976 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"NEW", "OLD", "REPLACE", "SCRATCH", "UNKNOWN"}},`。
- **L977 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Carriagecontrol, {"LIST", "FORTRAN", "NONE"}},`.
  **L977 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Carriagecontrol, {"LIST", "FORTRAN", "NONE"}},`。
- **L978 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Convert, {"BIG_ENDIAN", "LITTLE_ENDIAN", "NATIVE", "SWAP"}},`.
  **L978 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Convert, {"BIG_ENDIAN", "LITTLE_ENDIAN", "NATIVE", "SWAP"}},`。
- **L979 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{IoSpecKind::Dispose, {"DELETE", "KEEP"}},`.
  **L979 CN**: 继续一个多行参数列表、初始化器或聚合项：`{IoSpecKind::Dispose, {"DELETE", "KEEP"}},`。
- **L980 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L980 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L981 EN**: Executes a call or declaration centered on `upper{Normalize`.
  **L981 CN**: 执行以 `upper{Normalize` 为核心的调用或声明。
- **L982 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L982 CN**: 开始 `if` 控制流语句并计算其条件。
- **L983 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L983 CN**: 开始 `if` 控制流语句并计算其条件。
- **L984 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Warn(common::LanguageFeature::OpenAccessAppend, source,`.
  **L984 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Warn(common::LanguageFeature::OpenAccessAppend, source,`。

### Lines 985-1008

````cpp
          "ACCESS='%s' interpreted as POSITION='%s'"_port_en_US, value, upper);
    } else {
      context_.Say(source, "Invalid %s value '%s'"_err_en_US,
          parser::ToUpperCaseLetters(common::EnumToString(specKind)), value);
    }
  }
}

// CheckForRequiredSpecifier and CheckForProhibitedSpecifier functions
// need conditions to check, and string arguments to insert into a message.
// An IoSpecKind provides both an absence/presence condition and a string
// argument (its name).  A (condition, string) pair provides an arbitrary
// condition and an arbitrary string.

void IoChecker::CheckForRequiredSpecifier(IoSpecKind specKind) const {
  if (!specifierSet_.test(specKind)) {
    context_.Say("%s statement must have a %s specifier"_err_en_US,
        parser::ToUpperCaseLetters(common::EnumToString(stmt_)),
        parser::ToUpperCaseLetters(common::EnumToString(specKind)));
  }
}

void IoChecker::CheckForRequiredSpecifier(
    bool condition, const std::string &s) const {
````
- **L985 EN**: Executes a standalone statement or declaration: `"ACCESS='%s' interpreted as POSITION='%s'"_port_en_US, value, upper);`.
  **L985 CN**: 执行一条独立语句或声明：`"ACCESS='%s' interpreted as POSITION='%s'"_port_en_US, value, upper);`。
- **L986 EN**: Transitions from the previous branch into the alternative path.
  **L986 CN**: 从前一个分支过渡到备选路径。
- **L987 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(source, "Invalid %s value '%s'"_err_en_US,`.
  **L987 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(source, "Invalid %s value '%s'"_err_en_US,`。
- **L988 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L988 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L989 EN**: Closes the current lexical scope or compound statement.
  **L989 CN**: 结束当前词法作用域或复合语句块。
- **L990 EN**: Closes the current lexical scope or compound statement.
  **L990 CN**: 结束当前词法作用域或复合语句块。
- **L991 EN**: Closes the current lexical scope or compound statement.
  **L991 CN**: 结束当前词法作用域或复合语句块。
- **L992 EN**: Blank line separating nearby declarations or logic blocks.
  **L992 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L993 EN**: Comment explains nearby logic, intent, or metadata: `CheckForRequiredSpecifier and CheckForProhibitedSpecifier functions`.
  **L993 CN**: 注释说明附近代码的逻辑、意图或元数据：`CheckForRequiredSpecifier and CheckForProhibitedSpecifier functions`。
- **L994 EN**: Comment explains nearby logic, intent, or metadata: `need conditions to check, and string arguments to insert into a message.`.
  **L994 CN**: 注释说明附近代码的逻辑、意图或元数据：`need conditions to check, and string arguments to insert into a message.`。
- **L995 EN**: Comment explains nearby logic, intent, or metadata: `An IoSpecKind provides both an absence/presence condition and a string`.
  **L995 CN**: 注释说明附近代码的逻辑、意图或元数据：`An IoSpecKind provides both an absence/presence condition and a string`。
- **L996 EN**: Comment explains nearby logic, intent, or metadata: `argument (its name).  A (condition, string) pair provides an arbitrary`.
  **L996 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument (its name).  A (condition, string) pair provides an arbitrary`。
- **L997 EN**: Comment explains nearby logic, intent, or metadata: `condition and an arbitrary string.`.
  **L997 CN**: 注释说明附近代码的逻辑、意图或元数据：`condition and an arbitrary string.`。
- **L998 EN**: Blank line separating nearby declarations or logic blocks.
  **L998 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L999 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::CheckForRequiredSpecifier(IoSpecKind specKind) const {`.
  **L999 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::CheckForRequiredSpecifier(IoSpecKind specKind) const {`。
- **L1000 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1000 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1001 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say("%s statement must have a %s specifier"_err_en_US,`.
  **L1001 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say("%s statement must have a %s specifier"_err_en_US,`。
- **L1002 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(common::EnumToString(stmt_)),`.
  **L1002 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(common::EnumToString(stmt_)),`。
- **L1003 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L1003 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L1004 EN**: Closes the current lexical scope or compound statement.
  **L1004 CN**: 结束当前词法作用域或复合语句块。
- **L1005 EN**: Closes the current lexical scope or compound statement.
  **L1005 CN**: 结束当前词法作用域或复合语句块。
- **L1006 EN**: Blank line separating nearby declarations or logic blocks.
  **L1006 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1007 EN**: Continues logic associated with callable symbol `CheckForRequiredSpecifier`.
  **L1007 CN**: 继续与可调用符号 `CheckForRequiredSpecifier` 相关的逻辑。
- **L1008 EN**: Continues the surrounding expression or declaration: `bool condition, const std::string &s) const {`.
  **L1008 CN**: 继续构造周围的表达式或声明：`bool condition, const std::string &s) const {`。

### Lines 1009-1032

````cpp
  if (!condition) {
    context_.Say("%s statement must have a %s specifier"_err_en_US,
        parser::ToUpperCaseLetters(common::EnumToString(stmt_)), s);
  }
}

void IoChecker::CheckForRequiredSpecifier(
    IoSpecKind specKind1, IoSpecKind specKind2) const {
  if (specifierSet_.test(specKind1) && !specifierSet_.test(specKind2)) {
    context_.Say("If %s appears, %s must also appear"_err_en_US,
        parser::ToUpperCaseLetters(common::EnumToString(specKind1)),
        parser::ToUpperCaseLetters(common::EnumToString(specKind2)));
  }
}

void IoChecker::CheckForRequiredSpecifier(
    IoSpecKind specKind, bool condition, const std::string &s) const {
  if (specifierSet_.test(specKind) && !condition) {
    context_.Say("If %s appears, %s must also appear"_err_en_US,
        parser::ToUpperCaseLetters(common::EnumToString(specKind)), s);
  }
}

void IoChecker::CheckForRequiredSpecifier(
````
- **L1009 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1009 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1010 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say("%s statement must have a %s specifier"_err_en_US,`.
  **L1010 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say("%s statement must have a %s specifier"_err_en_US,`。
- **L1011 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L1011 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L1012 EN**: Closes the current lexical scope or compound statement.
  **L1012 CN**: 结束当前词法作用域或复合语句块。
- **L1013 EN**: Closes the current lexical scope or compound statement.
  **L1013 CN**: 结束当前词法作用域或复合语句块。
- **L1014 EN**: Blank line separating nearby declarations or logic blocks.
  **L1014 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1015 EN**: Continues logic associated with callable symbol `CheckForRequiredSpecifier`.
  **L1015 CN**: 继续与可调用符号 `CheckForRequiredSpecifier` 相关的逻辑。
- **L1016 EN**: Continues the surrounding expression or declaration: `IoSpecKind specKind1, IoSpecKind specKind2) const {`.
  **L1016 CN**: 继续构造周围的表达式或声明：`IoSpecKind specKind1, IoSpecKind specKind2) const {`。
- **L1017 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1017 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1018 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say("If %s appears, %s must also appear"_err_en_US,`.
  **L1018 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say("If %s appears, %s must also appear"_err_en_US,`。
- **L1019 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(common::EnumToString(specKind1)),`.
  **L1019 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(common::EnumToString(specKind1)),`。
- **L1020 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L1020 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L1021 EN**: Closes the current lexical scope or compound statement.
  **L1021 CN**: 结束当前词法作用域或复合语句块。
- **L1022 EN**: Closes the current lexical scope or compound statement.
  **L1022 CN**: 结束当前词法作用域或复合语句块。
- **L1023 EN**: Blank line separating nearby declarations or logic blocks.
  **L1023 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1024 EN**: Continues logic associated with callable symbol `CheckForRequiredSpecifier`.
  **L1024 CN**: 继续与可调用符号 `CheckForRequiredSpecifier` 相关的逻辑。
- **L1025 EN**: Continues the surrounding expression or declaration: `IoSpecKind specKind, bool condition, const std::string &s) const {`.
  **L1025 CN**: 继续构造周围的表达式或声明：`IoSpecKind specKind, bool condition, const std::string &s) const {`。
- **L1026 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1026 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1027 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say("If %s appears, %s must also appear"_err_en_US,`.
  **L1027 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say("If %s appears, %s must also appear"_err_en_US,`。
- **L1028 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L1028 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L1029 EN**: Closes the current lexical scope or compound statement.
  **L1029 CN**: 结束当前词法作用域或复合语句块。
- **L1030 EN**: Closes the current lexical scope or compound statement.
  **L1030 CN**: 结束当前词法作用域或复合语句块。
- **L1031 EN**: Blank line separating nearby declarations or logic blocks.
  **L1031 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1032 EN**: Continues logic associated with callable symbol `CheckForRequiredSpecifier`.
  **L1032 CN**: 继续与可调用符号 `CheckForRequiredSpecifier` 相关的逻辑。

### Lines 1033-1056

````cpp
    bool condition, const std::string &s, IoSpecKind specKind) const {
  if (condition && !specifierSet_.test(specKind)) {
    context_.Say("If %s appears, %s must also appear"_err_en_US, s,
        parser::ToUpperCaseLetters(common::EnumToString(specKind)));
  }
}

void IoChecker::CheckForRequiredSpecifier(bool condition1,
    const std::string &s1, bool condition2, const std::string &s2) const {
  if (condition1 && !condition2) {
    context_.Say("If %s appears, %s must also appear"_err_en_US, s1, s2);
  }
}

void IoChecker::CheckForProhibitedSpecifier(IoSpecKind specKind) const {
  if (specifierSet_.test(specKind)) {
    context_.Say("%s statement must not have a %s specifier"_err_en_US,
        parser::ToUpperCaseLetters(common::EnumToString(stmt_)),
        parser::ToUpperCaseLetters(common::EnumToString(specKind)));
  }
}

void IoChecker::CheckForProhibitedSpecifier(
    IoSpecKind specKind1, IoSpecKind specKind2) const {
````
- **L1033 EN**: Continues the surrounding expression or declaration: `bool condition, const std::string &s, IoSpecKind specKind) const {`.
  **L1033 CN**: 继续构造周围的表达式或声明：`bool condition, const std::string &s, IoSpecKind specKind) const {`。
- **L1034 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1034 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1035 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say("If %s appears, %s must also appear"_err_en_US, s,`.
  **L1035 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say("If %s appears, %s must also appear"_err_en_US, s,`。
- **L1036 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L1036 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L1037 EN**: Closes the current lexical scope or compound statement.
  **L1037 CN**: 结束当前词法作用域或复合语句块。
- **L1038 EN**: Closes the current lexical scope or compound statement.
  **L1038 CN**: 结束当前词法作用域或复合语句块。
- **L1039 EN**: Blank line separating nearby declarations or logic blocks.
  **L1039 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1040 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void IoChecker::CheckForRequiredSpecifier(bool condition1,`.
  **L1040 CN**: 继续一个多行参数列表、初始化器或聚合项：`void IoChecker::CheckForRequiredSpecifier(bool condition1,`。
- **L1041 EN**: Continues the surrounding expression or declaration: `const std::string &s1, bool condition2, const std::string &s2) const {`.
  **L1041 CN**: 继续构造周围的表达式或声明：`const std::string &s1, bool condition2, const std::string &s2) const {`。
- **L1042 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1042 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1043 EN**: Executes a call or declaration centered on `context_.Say`.
  **L1043 CN**: 执行以 `context_.Say` 为核心的调用或声明。
- **L1044 EN**: Closes the current lexical scope or compound statement.
  **L1044 CN**: 结束当前词法作用域或复合语句块。
- **L1045 EN**: Closes the current lexical scope or compound statement.
  **L1045 CN**: 结束当前词法作用域或复合语句块。
- **L1046 EN**: Blank line separating nearby declarations or logic blocks.
  **L1046 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1047 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::CheckForProhibitedSpecifier(IoSpecKind specKind) const {`.
  **L1047 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::CheckForProhibitedSpecifier(IoSpecKind specKind) const {`。
- **L1048 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1048 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1049 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say("%s statement must not have a %s specifier"_err_en_US,`.
  **L1049 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say("%s statement must not have a %s specifier"_err_en_US,`。
- **L1050 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(common::EnumToString(stmt_)),`.
  **L1050 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(common::EnumToString(stmt_)),`。
- **L1051 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L1051 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L1052 EN**: Closes the current lexical scope or compound statement.
  **L1052 CN**: 结束当前词法作用域或复合语句块。
- **L1053 EN**: Closes the current lexical scope or compound statement.
  **L1053 CN**: 结束当前词法作用域或复合语句块。
- **L1054 EN**: Blank line separating nearby declarations or logic blocks.
  **L1054 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1055 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L1055 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L1056 EN**: Continues the surrounding expression or declaration: `IoSpecKind specKind1, IoSpecKind specKind2) const {`.
  **L1056 CN**: 继续构造周围的表达式或声明：`IoSpecKind specKind1, IoSpecKind specKind2) const {`。

### Lines 1057-1080

````cpp
  if (specifierSet_.test(specKind1) && specifierSet_.test(specKind2)) {
    context_.Say("If %s appears, %s must not appear"_err_en_US,
        parser::ToUpperCaseLetters(common::EnumToString(specKind1)),
        parser::ToUpperCaseLetters(common::EnumToString(specKind2)));
  }
}

void IoChecker::CheckForProhibitedSpecifier(
    IoSpecKind specKind, bool condition, const std::string &s) const {
  if (specifierSet_.test(specKind) && condition) {
    context_.Say("If %s appears, %s must not appear"_err_en_US,
        parser::ToUpperCaseLetters(common::EnumToString(specKind)), s);
  }
}

void IoChecker::CheckForProhibitedSpecifier(
    bool condition, const std::string &s, IoSpecKind specKind) const {
  if (condition && specifierSet_.test(specKind)) {
    context_.Say("If %s appears, %s must not appear"_err_en_US, s,
        parser::ToUpperCaseLetters(common::EnumToString(specKind)));
  }
}

template <typename A>
````
- **L1057 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1057 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1058 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say("If %s appears, %s must not appear"_err_en_US,`.
  **L1058 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say("If %s appears, %s must not appear"_err_en_US,`。
- **L1059 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(common::EnumToString(specKind1)),`.
  **L1059 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(common::EnumToString(specKind1)),`。
- **L1060 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L1060 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L1061 EN**: Closes the current lexical scope or compound statement.
  **L1061 CN**: 结束当前词法作用域或复合语句块。
- **L1062 EN**: Closes the current lexical scope or compound statement.
  **L1062 CN**: 结束当前词法作用域或复合语句块。
- **L1063 EN**: Blank line separating nearby declarations or logic blocks.
  **L1063 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1064 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L1064 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L1065 EN**: Continues the surrounding expression or declaration: `IoSpecKind specKind, bool condition, const std::string &s) const {`.
  **L1065 CN**: 继续构造周围的表达式或声明：`IoSpecKind specKind, bool condition, const std::string &s) const {`。
- **L1066 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1066 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say("If %s appears, %s must not appear"_err_en_US,`.
  **L1067 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say("If %s appears, %s must not appear"_err_en_US,`。
- **L1068 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L1068 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L1069 EN**: Closes the current lexical scope or compound statement.
  **L1069 CN**: 结束当前词法作用域或复合语句块。
- **L1070 EN**: Closes the current lexical scope or compound statement.
  **L1070 CN**: 结束当前词法作用域或复合语句块。
- **L1071 EN**: Blank line separating nearby declarations or logic blocks.
  **L1071 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1072 EN**: Continues logic associated with callable symbol `CheckForProhibitedSpecifier`.
  **L1072 CN**: 继续与可调用符号 `CheckForProhibitedSpecifier` 相关的逻辑。
- **L1073 EN**: Continues the surrounding expression or declaration: `bool condition, const std::string &s, IoSpecKind specKind) const {`.
  **L1073 CN**: 继续构造周围的表达式或声明：`bool condition, const std::string &s, IoSpecKind specKind) const {`。
- **L1074 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1074 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1075 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say("If %s appears, %s must not appear"_err_en_US, s,`.
  **L1075 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say("If %s appears, %s must not appear"_err_en_US, s,`。
- **L1076 EN**: Executes a call or declaration centered on `parser::ToUpperCaseLetters`.
  **L1076 CN**: 执行以 `parser::ToUpperCaseLetters` 为核心的调用或声明。
- **L1077 EN**: Closes the current lexical scope or compound statement.
  **L1077 CN**: 结束当前词法作用域或复合语句块。
- **L1078 EN**: Closes the current lexical scope or compound statement.
  **L1078 CN**: 结束当前词法作用域或复合语句块。
- **L1079 EN**: Blank line separating nearby declarations or logic blocks.
  **L1079 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1080 EN**: Introduces template parameters or specialization context: `template <typename A>`.
  **L1080 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A>`。

### Lines 1081-1104

````cpp
void IoChecker::CheckForDefinableVariable(
    const A &variable, const std::string &s) const {
  if (const auto *var{parser::Unwrap<parser::Variable>(variable)}) {
    if (auto expr{AnalyzeExpr(context_, *var)}) {
      auto at{var->GetSource()};
      if (auto whyNot{WhyNotDefinable(at, context_.FindScope(at),
              DefinabilityFlags{DefinabilityFlag::VectorSubscriptIsOk},
              *expr)}) {
        if (whyNot->IsFatal()) {
          const Symbol *base{GetFirstSymbol(*expr)};
          context_
              .Say(at, "%s variable '%s' is not definable"_err_en_US, s,
                  (base ? base->name() : at).ToString())
              .Attach(
                  std::move(whyNot->set_severity(parser::Severity::Because)));
        } else {
          context_.Say(std::move(*whyNot));
        }
      }
    }
  }
}

void IoChecker::CheckForPureSubprogram() const { // C1597
````
- **L1081 EN**: Continues logic associated with callable symbol `CheckForDefinableVariable`.
  **L1081 CN**: 继续与可调用符号 `CheckForDefinableVariable` 相关的逻辑。
- **L1082 EN**: Continues the surrounding expression or declaration: `const A &variable, const std::string &s) const {`.
  **L1082 CN**: 继续构造周围的表达式或声明：`const A &variable, const std::string &s) const {`。
- **L1083 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1083 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1084 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1084 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1085 EN**: Executes a call or declaration centered on `at{var->GetSource`.
  **L1085 CN**: 执行以 `at{var->GetSource` 为核心的调用或声明。
- **L1086 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1086 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1087 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefinabilityFlags{DefinabilityFlag::VectorSubscriptIsOk},`.
  **L1087 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefinabilityFlags{DefinabilityFlag::VectorSubscriptIsOk},`。
- **L1088 EN**: Comment explains nearby logic, intent, or metadata: `expr)}) {`.
  **L1088 CN**: 注释说明附近代码的逻辑、意图或元数据：`expr)}) {`。
- **L1089 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1089 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1090 EN**: Executes a call or declaration centered on `*base{GetFirstSymbol`.
  **L1090 CN**: 执行以 `*base{GetFirstSymbol` 为核心的调用或声明。
- **L1091 EN**: Continues the surrounding expression or declaration: `context_`.
  **L1091 CN**: 继续构造周围的表达式或声明：`context_`。
- **L1092 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(at, "%s variable '%s' is not definable"_err_en_US, s,`.
  **L1092 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(at, "%s variable '%s' is not definable"_err_en_US, s,`。
- **L1093 EN**: Continues logic associated with callable symbol `name`.
  **L1093 CN**: 继续与可调用符号 `name` 相关的逻辑。
- **L1094 EN**: Continues logic associated with callable symbol `Attach`.
  **L1094 CN**: 继续与可调用符号 `Attach` 相关的逻辑。
- **L1095 EN**: Executes a call or declaration centered on `std::move`.
  **L1095 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L1096 EN**: Transitions from the previous branch into the alternative path.
  **L1096 CN**: 从前一个分支过渡到备选路径。
- **L1097 EN**: Executes a call or declaration centered on `context_.Say`.
  **L1097 CN**: 执行以 `context_.Say` 为核心的调用或声明。
- **L1098 EN**: Closes the current lexical scope or compound statement.
  **L1098 CN**: 结束当前词法作用域或复合语句块。
- **L1099 EN**: Closes the current lexical scope or compound statement.
  **L1099 CN**: 结束当前词法作用域或复合语句块。
- **L1100 EN**: Closes the current lexical scope or compound statement.
  **L1100 CN**: 结束当前词法作用域或复合语句块。
- **L1101 EN**: Closes the current lexical scope or compound statement.
  **L1101 CN**: 结束当前词法作用域或复合语句块。
- **L1102 EN**: Closes the current lexical scope or compound statement.
  **L1102 CN**: 结束当前词法作用域或复合语句块。
- **L1103 EN**: Blank line separating nearby declarations or logic blocks.
  **L1103 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1104 EN**: Continues logic associated with callable symbol `CheckForPureSubprogram`.
  **L1104 CN**: 继续与可调用符号 `CheckForPureSubprogram` 相关的逻辑。

### Lines 1105-1128

````cpp
  CHECK(context_.location());
  const Scope &scope{context_.FindScope(*context_.location())};
  if (FindPureProcedureContaining(scope)) {
    context_.Say("External I/O is not allowed in a pure subprogram"_err_en_US);
  }
}

void IoChecker::CheckForUselessIomsg() const {
  if (specifierSet_.test(IoSpecKind::Iomsg) &&
      !specifierSet_.test(IoSpecKind::Err) &&
      !specifierSet_.test(IoSpecKind::Iostat) &&
      context_.ShouldWarn(common::UsageWarning::UselessIomsg)) {
    context_.Say("IOMSG= is useless without either ERR= or IOSTAT="_warn_en_US);
  }
}

// Set of derived-type symbols already visited on the current recursion
// path of the component walks below.
using VisitedSymbolSet = std::unordered_set<const Symbol *>;

// Seeks out an allocatable or pointer ultimate component that is not
// nested in a nonallocatable/nonpointer component with a specific defined I/O
// procedure. The 'visited' set tracks derived types to break cycles caused by
// an illegal recursive type definition (F2023 C749).
````
- **L1105 EN**: Executes a call or declaration centered on `CHECK`.
  **L1105 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L1106 EN**: Executes a call or declaration centered on `&scope{context_.FindScope`.
  **L1106 CN**: 执行以 `&scope{context_.FindScope` 为核心的调用或声明。
- **L1107 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1107 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1108 EN**: Executes a call or declaration centered on `context_.Say`.
  **L1108 CN**: 执行以 `context_.Say` 为核心的调用或声明。
- **L1109 EN**: Closes the current lexical scope or compound statement.
  **L1109 CN**: 结束当前词法作用域或复合语句块。
- **L1110 EN**: Closes the current lexical scope or compound statement.
  **L1110 CN**: 结束当前词法作用域或复合语句块。
- **L1111 EN**: Blank line separating nearby declarations or logic blocks.
  **L1111 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1112 EN**: Starts a function, method, lambda, or structured scope: `void IoChecker::CheckForUselessIomsg() const {`.
  **L1112 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void IoChecker::CheckForUselessIomsg() const {`。
- **L1113 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1113 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1114 EN**: Continues logic associated with callable symbol `test`.
  **L1114 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L1115 EN**: Continues logic associated with callable symbol `test`.
  **L1115 CN**: 继续与可调用符号 `test` 相关的逻辑。
- **L1116 EN**: Starts a function, method, lambda, or structured scope: `context_.ShouldWarn(common::UsageWarning::UselessIomsg)) {`.
  **L1116 CN**: 开始一个函数、方法、lambda 或结构化作用域：`context_.ShouldWarn(common::UsageWarning::UselessIomsg)) {`。
- **L1117 EN**: Executes a call or declaration centered on `context_.Say`.
  **L1117 CN**: 执行以 `context_.Say` 为核心的调用或声明。
- **L1118 EN**: Closes the current lexical scope or compound statement.
  **L1118 CN**: 结束当前词法作用域或复合语句块。
- **L1119 EN**: Closes the current lexical scope or compound statement.
  **L1119 CN**: 结束当前词法作用域或复合语句块。
- **L1120 EN**: Blank line separating nearby declarations or logic blocks.
  **L1120 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1121 EN**: Comment explains nearby logic, intent, or metadata: `Set of derived-type symbols already visited on the current recursion`.
  **L1121 CN**: 注释说明附近代码的逻辑、意图或元数据：`Set of derived-type symbols already visited on the current recursion`。
- **L1122 EN**: Comment explains nearby logic, intent, or metadata: `path of the component walks below.`.
  **L1122 CN**: 注释说明附近代码的逻辑、意图或元数据：`path of the component walks below.`。
- **L1123 EN**: Defines alias `VisitedSymbolSet` to simplify later code.
  **L1123 CN**: 定义别名 `VisitedSymbolSet` 以简化后续代码。
- **L1124 EN**: Blank line separating nearby declarations or logic blocks.
  **L1124 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1125 EN**: Comment explains nearby logic, intent, or metadata: `Seeks out an allocatable or pointer ultimate component that is not`.
  **L1125 CN**: 注释说明附近代码的逻辑、意图或元数据：`Seeks out an allocatable or pointer ultimate component that is not`。
- **L1126 EN**: Comment explains nearby logic, intent, or metadata: `nested in a nonallocatable/nonpointer component with a specific defined I/O`.
  **L1126 CN**: 注释说明附近代码的逻辑、意图或元数据：`nested in a nonallocatable/nonpointer component with a specific defined I/O`。
- **L1127 EN**: Comment explains nearby logic, intent, or metadata: `procedure. The 'visited' set tracks derived types to break cycles caused by`.
  **L1127 CN**: 注释说明附近代码的逻辑、意图或元数据：`procedure. The 'visited' set tracks derived types to break cycles caused by`。
- **L1128 EN**: Comment explains nearby logic, intent, or metadata: `an illegal recursive type definition (F2023 C749).`.
  **L1128 CN**: 注释说明附近代码的逻辑、意图或元数据：`an illegal recursive type definition (F2023 C749).`。

### Lines 1129-1152

````cpp
static const Symbol *FindUnsafeIoDirectComponent(common::DefinedIo which,
    const DerivedTypeSpec &derived, const Scope &scope,
    VisitedSymbolSet &visited) {
  if (HasDefinedIo(which, derived, &scope)) {
    return nullptr;
  }
  if (!visited.insert(&derived.typeSymbol()).second) {
    return nullptr;
  }
  if (const Scope * dtScope{derived.scope()}) {
    for (const auto &pair : *dtScope) {
      const Symbol &symbol{*pair.second};
      if (IsAllocatableOrPointer(symbol)) {
        return &symbol;
      }
      if (const auto *details{symbol.detailsIf<ObjectEntityDetails>()}) {
        if (const DeclTypeSpec * type{details->type()}) {
          if (type->category() == DeclTypeSpec::Category::TypeDerived) {
            const DerivedTypeSpec &componentDerived{type->derivedTypeSpec()};
            if (const Symbol *bad{FindUnsafeIoDirectComponent(
                    which, componentDerived, scope, visited)}) {
              return bad;
            }
          }
````
- **L1129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static const Symbol *FindUnsafeIoDirectComponent(common::DefinedIo which,`.
  **L1129 CN**: 继续一个多行参数列表、初始化器或聚合项：`static const Symbol *FindUnsafeIoDirectComponent(common::DefinedIo which,`。
- **L1130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const DerivedTypeSpec &derived, const Scope &scope,`.
  **L1130 CN**: 继续一个多行参数列表、初始化器或聚合项：`const DerivedTypeSpec &derived, const Scope &scope,`。
- **L1131 EN**: Continues the surrounding expression or declaration: `VisitedSymbolSet &visited) {`.
  **L1131 CN**: 继续构造周围的表达式或声明：`VisitedSymbolSet &visited) {`。
- **L1132 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1132 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1133 EN**: Returns from the current function with `nullptr`.
  **L1133 CN**: 以 `nullptr` 从当前函数返回。
- **L1134 EN**: Closes the current lexical scope or compound statement.
  **L1134 CN**: 结束当前词法作用域或复合语句块。
- **L1135 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1135 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1136 EN**: Returns from the current function with `nullptr`.
  **L1136 CN**: 以 `nullptr` 从当前函数返回。
- **L1137 EN**: Closes the current lexical scope or compound statement.
  **L1137 CN**: 结束当前词法作用域或复合语句块。
- **L1138 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1138 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1139 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1139 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1140 EN**: Executes a standalone statement or declaration: `const Symbol &symbol{*pair.second};`.
  **L1140 CN**: 执行一条独立语句或声明：`const Symbol &symbol{*pair.second};`。
- **L1141 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1141 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1142 EN**: Returns from the current function with `&symbol`.
  **L1142 CN**: 以 `&symbol` 从当前函数返回。
- **L1143 EN**: Closes the current lexical scope or compound statement.
  **L1143 CN**: 结束当前词法作用域或复合语句块。
- **L1144 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1144 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1145 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1145 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1146 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1146 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1147 EN**: Executes a call or declaration centered on `&componentDerived{type->derivedTypeSpec`.
  **L1147 CN**: 执行以 `&componentDerived{type->derivedTypeSpec` 为核心的调用或声明。
- **L1148 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1148 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1149 EN**: Continues the surrounding expression or declaration: `which, componentDerived, scope, visited)}) {`.
  **L1149 CN**: 继续构造周围的表达式或声明：`which, componentDerived, scope, visited)}) {`。
- **L1150 EN**: Returns from the current function with `bad`.
  **L1150 CN**: 以 `bad` 从当前函数返回。
- **L1151 EN**: Closes the current lexical scope or compound statement.
  **L1151 CN**: 结束当前词法作用域或复合语句块。
- **L1152 EN**: Closes the current lexical scope or compound statement.
  **L1152 CN**: 结束当前词法作用域或复合语句块。

### Lines 1153-1176

````cpp
        }
      }
    }
  }
  return nullptr;
}

static const Symbol *FindUnsafeIoDirectComponent(common::DefinedIo which,
    const DerivedTypeSpec &derived, const Scope &scope) {
  VisitedSymbolSet visited;
  return FindUnsafeIoDirectComponent(which, derived, scope, visited);
}

// For a type that does not have a defined I/O subroutine, finds a direct
// component that is a witness to an accessibility violation outside the module
// in which the type was defined.  The 'visited' set tracks derived types to
// break cycles caused by an illegal recursive type definition (F2023 C749).
static const Symbol *FindInaccessibleComponent(common::DefinedIo which,
    const DerivedTypeSpec &derived, const Scope &scope,
    VisitedSymbolSet &visited) {
  if (!visited.insert(&derived.typeSymbol()).second) {
    return nullptr;
  }
  if (const Scope * dtScope{derived.scope()}) {
````
- **L1153 EN**: Closes the current lexical scope or compound statement.
  **L1153 CN**: 结束当前词法作用域或复合语句块。
- **L1154 EN**: Closes the current lexical scope or compound statement.
  **L1154 CN**: 结束当前词法作用域或复合语句块。
- **L1155 EN**: Closes the current lexical scope or compound statement.
  **L1155 CN**: 结束当前词法作用域或复合语句块。
- **L1156 EN**: Closes the current lexical scope or compound statement.
  **L1156 CN**: 结束当前词法作用域或复合语句块。
- **L1157 EN**: Returns from the current function with `nullptr`.
  **L1157 CN**: 以 `nullptr` 从当前函数返回。
- **L1158 EN**: Closes the current lexical scope or compound statement.
  **L1158 CN**: 结束当前词法作用域或复合语句块。
- **L1159 EN**: Blank line separating nearby declarations or logic blocks.
  **L1159 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static const Symbol *FindUnsafeIoDirectComponent(common::DefinedIo which,`.
  **L1160 CN**: 继续一个多行参数列表、初始化器或聚合项：`static const Symbol *FindUnsafeIoDirectComponent(common::DefinedIo which,`。
- **L1161 EN**: Continues the surrounding expression or declaration: `const DerivedTypeSpec &derived, const Scope &scope) {`.
  **L1161 CN**: 继续构造周围的表达式或声明：`const DerivedTypeSpec &derived, const Scope &scope) {`。
- **L1162 EN**: Executes a standalone statement or declaration: `VisitedSymbolSet visited;`.
  **L1162 CN**: 执行一条独立语句或声明：`VisitedSymbolSet visited;`。
- **L1163 EN**: Returns from the current function with `FindUnsafeIoDirectComponent(which, derived, scope, visited)`.
  **L1163 CN**: 以 `FindUnsafeIoDirectComponent(which, derived, scope, visited)` 从当前函数返回。
- **L1164 EN**: Closes the current lexical scope or compound statement.
  **L1164 CN**: 结束当前词法作用域或复合语句块。
- **L1165 EN**: Blank line separating nearby declarations or logic blocks.
  **L1165 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1166 EN**: Comment explains nearby logic, intent, or metadata: `For a type that does not have a defined I/O subroutine, finds a direct`.
  **L1166 CN**: 注释说明附近代码的逻辑、意图或元数据：`For a type that does not have a defined I/O subroutine, finds a direct`。
- **L1167 EN**: Comment explains nearby logic, intent, or metadata: `component that is a witness to an accessibility violation outside the module`.
  **L1167 CN**: 注释说明附近代码的逻辑、意图或元数据：`component that is a witness to an accessibility violation outside the module`。
- **L1168 EN**: Comment explains nearby logic, intent, or metadata: `in which the type was defined.  The 'visited' set tracks derived types to`.
  **L1168 CN**: 注释说明附近代码的逻辑、意图或元数据：`in which the type was defined.  The 'visited' set tracks derived types to`。
- **L1169 EN**: Comment explains nearby logic, intent, or metadata: `break cycles caused by an illegal recursive type definition (F2023 C749).`.
  **L1169 CN**: 注释说明附近代码的逻辑、意图或元数据：`break cycles caused by an illegal recursive type definition (F2023 C749).`。
- **L1170 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static const Symbol *FindInaccessibleComponent(common::DefinedIo which,`.
  **L1170 CN**: 继续一个多行参数列表、初始化器或聚合项：`static const Symbol *FindInaccessibleComponent(common::DefinedIo which,`。
- **L1171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const DerivedTypeSpec &derived, const Scope &scope,`.
  **L1171 CN**: 继续一个多行参数列表、初始化器或聚合项：`const DerivedTypeSpec &derived, const Scope &scope,`。
- **L1172 EN**: Continues the surrounding expression or declaration: `VisitedSymbolSet &visited) {`.
  **L1172 CN**: 继续构造周围的表达式或声明：`VisitedSymbolSet &visited) {`。
- **L1173 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1173 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1174 EN**: Returns from the current function with `nullptr`.
  **L1174 CN**: 以 `nullptr` 从当前函数返回。
- **L1175 EN**: Closes the current lexical scope or compound statement.
  **L1175 CN**: 结束当前词法作用域或复合语句块。
- **L1176 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1176 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1177-1200

````cpp
    if (const Scope * module{FindModuleContaining(*dtScope)}) {
      for (const auto &pair : *dtScope) {
        const Symbol &symbol{*pair.second};
        if (IsAllocatableOrPointer(symbol)) {
          continue; // already an error
        }
        if (const auto *details{symbol.detailsIf<ObjectEntityDetails>()}) {
          const DerivedTypeSpec *componentDerived{nullptr};
          if (const DeclTypeSpec * type{details->type()}) {
            if (type->category() == DeclTypeSpec::Category::TypeDerived) {
              componentDerived = &type->derivedTypeSpec();
            }
          }
          if (componentDerived &&
              HasDefinedIo(which, *componentDerived, &scope)) {
            continue; // this component and its descendents are fine
          }
          if (symbol.attrs().test(Attr::PRIVATE) &&
              !symbol.test(Symbol::Flag::ParentComp)) {
            if (!DoesScopeContain(module, scope)) {
              return &symbol;
            }
          }
          if (componentDerived) {
````
- **L1177 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1177 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1178 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1178 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1179 EN**: Executes a standalone statement or declaration: `const Symbol &symbol{*pair.second};`.
  **L1179 CN**: 执行一条独立语句或声明：`const Symbol &symbol{*pair.second};`。
- **L1180 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1180 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1181 EN**: Skips to the next loop iteration.
  **L1181 CN**: 跳到下一次循环迭代。
- **L1182 EN**: Closes the current lexical scope or compound statement.
  **L1182 CN**: 结束当前词法作用域或复合语句块。
- **L1183 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1183 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1184 EN**: Executes a standalone statement or declaration: `const DerivedTypeSpec *componentDerived{nullptr};`.
  **L1184 CN**: 执行一条独立语句或声明：`const DerivedTypeSpec *componentDerived{nullptr};`。
- **L1185 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1185 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1186 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1186 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1187 EN**: Executes a call or declaration centered on `&type->derivedTypeSpec`.
  **L1187 CN**: 执行以 `&type->derivedTypeSpec` 为核心的调用或声明。
- **L1188 EN**: Closes the current lexical scope or compound statement.
  **L1188 CN**: 结束当前词法作用域或复合语句块。
- **L1189 EN**: Closes the current lexical scope or compound statement.
  **L1189 CN**: 结束当前词法作用域或复合语句块。
- **L1190 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1190 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1191 EN**: Starts a function, method, lambda, or structured scope: `HasDefinedIo(which, *componentDerived, &scope)) {`.
  **L1191 CN**: 开始一个函数、方法、lambda 或结构化作用域：`HasDefinedIo(which, *componentDerived, &scope)) {`。
- **L1192 EN**: Skips to the next loop iteration.
  **L1192 CN**: 跳到下一次循环迭代。
- **L1193 EN**: Closes the current lexical scope or compound statement.
  **L1193 CN**: 结束当前词法作用域或复合语句块。
- **L1194 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1194 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1195 EN**: Starts a function, method, lambda, or structured scope: `!symbol.test(Symbol::Flag::ParentComp)) {`.
  **L1195 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!symbol.test(Symbol::Flag::ParentComp)) {`。
- **L1196 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1196 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1197 EN**: Returns from the current function with `&symbol`.
  **L1197 CN**: 以 `&symbol` 从当前函数返回。
- **L1198 EN**: Closes the current lexical scope or compound statement.
  **L1198 CN**: 结束当前词法作用域或复合语句块。
- **L1199 EN**: Closes the current lexical scope or compound statement.
  **L1199 CN**: 结束当前词法作用域或复合语句块。
- **L1200 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1200 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1201-1224

````cpp
            if (const Symbol *bad{FindInaccessibleComponent(
                    which, *componentDerived, scope, visited)}) {
              return bad;
            }
          }
        }
      }
    }
  }
  return nullptr;
}

static const Symbol *FindInaccessibleComponent(common::DefinedIo which,
    const DerivedTypeSpec &derived, const Scope &scope) {
  VisitedSymbolSet visited;
  return FindInaccessibleComponent(which, derived, scope, visited);
}

// Fortran 2018, 12.6.3 paragraphs 5 & 7
parser::Message *IoChecker::CheckForBadIoType(const evaluate::DynamicType &type,
    common::DefinedIo which, parser::CharBlock where) const {
  if (type.IsUnlimitedPolymorphic()) {
    return &context_.Say(
        where, "I/O list item may not be unlimited polymorphic"_err_en_US);
````
- **L1201 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1201 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1202 EN**: Continues the surrounding expression or declaration: `which, *componentDerived, scope, visited)}) {`.
  **L1202 CN**: 继续构造周围的表达式或声明：`which, *componentDerived, scope, visited)}) {`。
- **L1203 EN**: Returns from the current function with `bad`.
  **L1203 CN**: 以 `bad` 从当前函数返回。
- **L1204 EN**: Closes the current lexical scope or compound statement.
  **L1204 CN**: 结束当前词法作用域或复合语句块。
- **L1205 EN**: Closes the current lexical scope or compound statement.
  **L1205 CN**: 结束当前词法作用域或复合语句块。
- **L1206 EN**: Closes the current lexical scope or compound statement.
  **L1206 CN**: 结束当前词法作用域或复合语句块。
- **L1207 EN**: Closes the current lexical scope or compound statement.
  **L1207 CN**: 结束当前词法作用域或复合语句块。
- **L1208 EN**: Closes the current lexical scope or compound statement.
  **L1208 CN**: 结束当前词法作用域或复合语句块。
- **L1209 EN**: Closes the current lexical scope or compound statement.
  **L1209 CN**: 结束当前词法作用域或复合语句块。
- **L1210 EN**: Returns from the current function with `nullptr`.
  **L1210 CN**: 以 `nullptr` 从当前函数返回。
- **L1211 EN**: Closes the current lexical scope or compound statement.
  **L1211 CN**: 结束当前词法作用域或复合语句块。
- **L1212 EN**: Blank line separating nearby declarations or logic blocks.
  **L1212 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static const Symbol *FindInaccessibleComponent(common::DefinedIo which,`.
  **L1213 CN**: 继续一个多行参数列表、初始化器或聚合项：`static const Symbol *FindInaccessibleComponent(common::DefinedIo which,`。
- **L1214 EN**: Continues the surrounding expression or declaration: `const DerivedTypeSpec &derived, const Scope &scope) {`.
  **L1214 CN**: 继续构造周围的表达式或声明：`const DerivedTypeSpec &derived, const Scope &scope) {`。
- **L1215 EN**: Executes a standalone statement or declaration: `VisitedSymbolSet visited;`.
  **L1215 CN**: 执行一条独立语句或声明：`VisitedSymbolSet visited;`。
- **L1216 EN**: Returns from the current function with `FindInaccessibleComponent(which, derived, scope, visited)`.
  **L1216 CN**: 以 `FindInaccessibleComponent(which, derived, scope, visited)` 从当前函数返回。
- **L1217 EN**: Closes the current lexical scope or compound statement.
  **L1217 CN**: 结束当前词法作用域或复合语句块。
- **L1218 EN**: Blank line separating nearby declarations or logic blocks.
  **L1218 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1219 EN**: Comment explains nearby logic, intent, or metadata: `Fortran 2018, 12.6.3 paragraphs 5 & 7`.
  **L1219 CN**: 注释说明附近代码的逻辑、意图或元数据：`Fortran 2018, 12.6.3 paragraphs 5 & 7`。
- **L1220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::Message *IoChecker::CheckForBadIoType(const evaluate::DynamicType &type,`.
  **L1220 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::Message *IoChecker::CheckForBadIoType(const evaluate::DynamicType &type,`。
- **L1221 EN**: Continues the surrounding expression or declaration: `common::DefinedIo which, parser::CharBlock where) const {`.
  **L1221 CN**: 继续构造周围的表达式或声明：`common::DefinedIo which, parser::CharBlock where) const {`。
- **L1222 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1222 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1223 EN**: Returns from the current function with `&context_.Say(`.
  **L1223 CN**: 以 `&context_.Say(` 从当前函数返回。
- **L1224 EN**: Executes a standalone statement or declaration: `where, "I/O list item may not be unlimited polymorphic"_err_en_US);`.
  **L1224 CN**: 执行一条独立语句或声明：`where, "I/O list item may not be unlimited polymorphic"_err_en_US);`。

### Lines 1225-1248

````cpp
  } else if (type.category() == TypeCategory::Derived) {
    const auto &derived{type.GetDerivedTypeSpec()};
    const Scope &scope{context_.FindScope(where)};
    if (const Symbol *
        bad{FindUnsafeIoDirectComponent(which, derived, scope)}) {
      return &context_.SayWithDecl(*bad, where,
          "Derived type '%s' in I/O cannot have an allocatable or pointer direct component '%s' unless using defined I/O"_err_en_US,
          derived.name(), bad->name());
    }
    if (!HasDefinedIo(which, derived, &scope)) {
      if (type.IsPolymorphic()) {
        return &context_.Say(where,
            "Derived type '%s' in I/O may not be polymorphic unless using defined I/O"_err_en_US,
            derived.name());
      }
      if ((IsBuiltinDerivedType(&derived, "c_ptr") ||
              IsBuiltinDerivedType(&derived, "c_devptr")) &&
          !context_.ShouldWarn(common::LanguageFeature::PrintCptr)) {
        // Bypass the check below for c_ptr and c_devptr.
        return nullptr;
      }
      if (const Symbol *
          bad{FindInaccessibleComponent(which, derived, scope)}) {
        return &context_.Say(where,
````
- **L1225 EN**: Transitions from the previous branch into an `else if` condition.
  **L1225 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1226 EN**: Executes a call or declaration centered on `&derived{type.GetDerivedTypeSpec`.
  **L1226 CN**: 执行以 `&derived{type.GetDerivedTypeSpec` 为核心的调用或声明。
- **L1227 EN**: Executes a call or declaration centered on `&scope{context_.FindScope`.
  **L1227 CN**: 执行以 `&scope{context_.FindScope` 为核心的调用或声明。
- **L1228 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1228 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1229 EN**: Starts a function, method, lambda, or structured scope: `bad{FindUnsafeIoDirectComponent(which, derived, scope)}) {`.
  **L1229 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bad{FindUnsafeIoDirectComponent(which, derived, scope)}) {`。
- **L1230 EN**: Returns from the current function with `&context_.SayWithDecl(*bad, where,`.
  **L1230 CN**: 以 `&context_.SayWithDecl(*bad, where,` 从当前函数返回。
- **L1231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Derived type '%s' in I/O cannot have an allocatable or pointer direct component '%s' unless using defined I/O"_err_en_US,`.
  **L1231 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Derived type '%s' in I/O cannot have an allocatable or pointer direct component '%s' unless using defined I/O"_err_en_US,`。
- **L1232 EN**: Executes a call or declaration centered on `derived.name`.
  **L1232 CN**: 执行以 `derived.name` 为核心的调用或声明。
- **L1233 EN**: Closes the current lexical scope or compound statement.
  **L1233 CN**: 结束当前词法作用域或复合语句块。
- **L1234 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1234 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1235 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1235 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1236 EN**: Returns from the current function with `&context_.Say(where,`.
  **L1236 CN**: 以 `&context_.Say(where,` 从当前函数返回。
- **L1237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Derived type '%s' in I/O may not be polymorphic unless using defined I/O"_err_en_US,`.
  **L1237 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Derived type '%s' in I/O may not be polymorphic unless using defined I/O"_err_en_US,`。
- **L1238 EN**: Executes a call or declaration centered on `derived.name`.
  **L1238 CN**: 执行以 `derived.name` 为核心的调用或声明。
- **L1239 EN**: Closes the current lexical scope or compound statement.
  **L1239 CN**: 结束当前词法作用域或复合语句块。
- **L1240 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1240 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1241 EN**: Continues logic associated with callable symbol `IsBuiltinDerivedType`.
  **L1241 CN**: 继续与可调用符号 `IsBuiltinDerivedType` 相关的逻辑。
- **L1242 EN**: Starts a function, method, lambda, or structured scope: `!context_.ShouldWarn(common::LanguageFeature::PrintCptr)) {`.
  **L1242 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!context_.ShouldWarn(common::LanguageFeature::PrintCptr)) {`。
- **L1243 EN**: Comment explains nearby logic, intent, or metadata: `Bypass the check below for c_ptr and c_devptr.`.
  **L1243 CN**: 注释说明附近代码的逻辑、意图或元数据：`Bypass the check below for c_ptr and c_devptr.`。
- **L1244 EN**: Returns from the current function with `nullptr`.
  **L1244 CN**: 以 `nullptr` 从当前函数返回。
- **L1245 EN**: Closes the current lexical scope or compound statement.
  **L1245 CN**: 结束当前词法作用域或复合语句块。
- **L1246 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1246 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1247 EN**: Starts a function, method, lambda, or structured scope: `bad{FindInaccessibleComponent(which, derived, scope)}) {`.
  **L1247 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bad{FindInaccessibleComponent(which, derived, scope)}) {`。
- **L1248 EN**: Returns from the current function with `&context_.Say(where,`.
  **L1248 CN**: 以 `&context_.Say(where,` 从当前函数返回。

### Lines 1249-1272

````cpp
            "I/O of the derived type '%s' may not be performed without defined I/O in a scope in which a direct component like '%s' is inaccessible"_err_en_US,
            derived.name(), bad->name());
      }
    }
  }
  return nullptr;
}

void IoChecker::CheckForBadIoType(const SomeExpr &expr, common::DefinedIo which,
    parser::CharBlock where) const {
  if (auto type{expr.GetType()}) {
    CheckForBadIoType(*type, which, where);
  }
}

parser::Message *IoChecker::CheckForBadIoType(const Symbol &symbol,
    common::DefinedIo which, parser::CharBlock where) const {
  if (auto type{evaluate::DynamicType::From(symbol)}) {
    if (auto *msg{CheckForBadIoType(*type, which, where)}) {
      evaluate::AttachDeclaration(*msg, symbol);
      return msg;
    }
  }
  return nullptr;
````
- **L1249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"I/O of the derived type '%s' may not be performed without defined I/O in a scope in which a direct component like '%s' is inaccessible"_err_en_US,`.
  **L1249 CN**: 继续一个多行参数列表、初始化器或聚合项：`"I/O of the derived type '%s' may not be performed without defined I/O in a scope in which a direct component like '%s' is inaccessible"_err_en_US,`。
- **L1250 EN**: Executes a call or declaration centered on `derived.name`.
  **L1250 CN**: 执行以 `derived.name` 为核心的调用或声明。
- **L1251 EN**: Closes the current lexical scope or compound statement.
  **L1251 CN**: 结束当前词法作用域或复合语句块。
- **L1252 EN**: Closes the current lexical scope or compound statement.
  **L1252 CN**: 结束当前词法作用域或复合语句块。
- **L1253 EN**: Closes the current lexical scope or compound statement.
  **L1253 CN**: 结束当前词法作用域或复合语句块。
- **L1254 EN**: Returns from the current function with `nullptr`.
  **L1254 CN**: 以 `nullptr` 从当前函数返回。
- **L1255 EN**: Closes the current lexical scope or compound statement.
  **L1255 CN**: 结束当前词法作用域或复合语句块。
- **L1256 EN**: Blank line separating nearby declarations or logic blocks.
  **L1256 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void IoChecker::CheckForBadIoType(const SomeExpr &expr, common::DefinedIo which,`.
  **L1257 CN**: 继续一个多行参数列表、初始化器或聚合项：`void IoChecker::CheckForBadIoType(const SomeExpr &expr, common::DefinedIo which,`。
- **L1258 EN**: Continues the surrounding expression or declaration: `parser::CharBlock where) const {`.
  **L1258 CN**: 继续构造周围的表达式或声明：`parser::CharBlock where) const {`。
- **L1259 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1259 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1260 EN**: Executes a call or declaration centered on `CheckForBadIoType`.
  **L1260 CN**: 执行以 `CheckForBadIoType` 为核心的调用或声明。
- **L1261 EN**: Closes the current lexical scope or compound statement.
  **L1261 CN**: 结束当前词法作用域或复合语句块。
- **L1262 EN**: Closes the current lexical scope or compound statement.
  **L1262 CN**: 结束当前词法作用域或复合语句块。
- **L1263 EN**: Blank line separating nearby declarations or logic blocks.
  **L1263 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::Message *IoChecker::CheckForBadIoType(const Symbol &symbol,`.
  **L1264 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::Message *IoChecker::CheckForBadIoType(const Symbol &symbol,`。
- **L1265 EN**: Continues the surrounding expression or declaration: `common::DefinedIo which, parser::CharBlock where) const {`.
  **L1265 CN**: 继续构造周围的表达式或声明：`common::DefinedIo which, parser::CharBlock where) const {`。
- **L1266 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1266 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1267 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1267 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1268 EN**: Executes a call or declaration centered on `evaluate::AttachDeclaration`.
  **L1268 CN**: 执行以 `evaluate::AttachDeclaration` 为核心的调用或声明。
- **L1269 EN**: Returns from the current function with `msg`.
  **L1269 CN**: 以 `msg` 从当前函数返回。
- **L1270 EN**: Closes the current lexical scope or compound statement.
  **L1270 CN**: 结束当前词法作用域或复合语句块。
- **L1271 EN**: Closes the current lexical scope or compound statement.
  **L1271 CN**: 结束当前词法作用域或复合语句块。
- **L1272 EN**: Returns from the current function with `nullptr`.
  **L1272 CN**: 以 `nullptr` 从当前函数返回。

### Lines 1273-1296

````cpp
}

void IoChecker::CheckForAssumedRank(
    const Symbol *symbol, parser::CharBlock namelistLocation) const {
  if (symbol && IsAssumedRank(*symbol)) {
    evaluate::AttachDeclaration(
        context_.Say(namelistLocation,
            "Assumed-rank object '%s' may not be an I/O list item"_err_en_US,
            symbol->name()),
        *symbol);
  }
}

void IoChecker::CheckNamelist(const Symbol &namelist, common::DefinedIo which,
    parser::CharBlock namelistLocation) const {
  if (!context_.HasError(namelist)) {
    const auto &details{namelist.GetUltimate().get<NamelistDetails>()};
    for (const Symbol &object : details.objects()) {
      context_.CheckIndexVarRedefine(namelistLocation, object);
      if (auto *msg{CheckForBadIoType(object, which, namelistLocation)}) {
        evaluate::AttachDeclaration(*msg, namelist);
      } else if (which == common::DefinedIo::ReadFormatted) {
        if (auto why{WhyNotDefinable(namelistLocation, namelist.owner(),
                DefinabilityFlags{}, object)}) {
````
- **L1273 EN**: Closes the current lexical scope or compound statement.
  **L1273 CN**: 结束当前词法作用域或复合语句块。
- **L1274 EN**: Blank line separating nearby declarations or logic blocks.
  **L1274 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1275 EN**: Continues logic associated with callable symbol `CheckForAssumedRank`.
  **L1275 CN**: 继续与可调用符号 `CheckForAssumedRank` 相关的逻辑。
- **L1276 EN**: Continues the surrounding expression or declaration: `const Symbol *symbol, parser::CharBlock namelistLocation) const {`.
  **L1276 CN**: 继续构造周围的表达式或声明：`const Symbol *symbol, parser::CharBlock namelistLocation) const {`。
- **L1277 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1277 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1278 EN**: Continues logic associated with callable symbol `AttachDeclaration`.
  **L1278 CN**: 继续与可调用符号 `AttachDeclaration` 相关的逻辑。
- **L1279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(namelistLocation,`.
  **L1279 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(namelistLocation,`。
- **L1280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Assumed-rank object '%s' may not be an I/O list item"_err_en_US,`.
  **L1280 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Assumed-rank object '%s' may not be an I/O list item"_err_en_US,`。
- **L1281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `symbol->name()),`.
  **L1281 CN**: 继续一个多行参数列表、初始化器或聚合项：`symbol->name()),`。
- **L1282 EN**: Comment explains nearby logic, intent, or metadata: `symbol);`.
  **L1282 CN**: 注释说明附近代码的逻辑、意图或元数据：`symbol);`。
- **L1283 EN**: Closes the current lexical scope or compound statement.
  **L1283 CN**: 结束当前词法作用域或复合语句块。
- **L1284 EN**: Closes the current lexical scope or compound statement.
  **L1284 CN**: 结束当前词法作用域或复合语句块。
- **L1285 EN**: Blank line separating nearby declarations or logic blocks.
  **L1285 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void IoChecker::CheckNamelist(const Symbol &namelist, common::DefinedIo which,`.
  **L1286 CN**: 继续一个多行参数列表、初始化器或聚合项：`void IoChecker::CheckNamelist(const Symbol &namelist, common::DefinedIo which,`。
- **L1287 EN**: Continues the surrounding expression or declaration: `parser::CharBlock namelistLocation) const {`.
  **L1287 CN**: 继续构造周围的表达式或声明：`parser::CharBlock namelistLocation) const {`。
- **L1288 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1288 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1289 EN**: Executes a call or declaration centered on `&details{namelist.GetUltimate`.
  **L1289 CN**: 执行以 `&details{namelist.GetUltimate` 为核心的调用或声明。
- **L1290 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1290 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1291 EN**: Executes a call or declaration centered on `context_.CheckIndexVarRedefine`.
  **L1291 CN**: 执行以 `context_.CheckIndexVarRedefine` 为核心的调用或声明。
- **L1292 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1292 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1293 EN**: Executes a call or declaration centered on `evaluate::AttachDeclaration`.
  **L1293 CN**: 执行以 `evaluate::AttachDeclaration` 为核心的调用或声明。
- **L1294 EN**: Transitions from the previous branch into an `else if` condition.
  **L1294 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1295 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1295 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1296 EN**: Continues the surrounding expression or declaration: `DefinabilityFlags{}, object)}) {`.
  **L1296 CN**: 继续构造周围的表达式或声明：`DefinabilityFlags{}, object)}) {`。

### Lines 1297-1309

````cpp
          context_
              .Say(namelistLocation,
                  "NAMELIST input group must not contain undefinable item '%s'"_err_en_US,
                  object.name())
              .Attach(std::move(why->set_severity(parser::Severity::Because)));
          context_.SetError(namelist);
        }
      }
    }
  }
}

} // namespace Fortran::semantics
````
- **L1297 EN**: Continues the surrounding expression or declaration: `context_`.
  **L1297 CN**: 继续构造周围的表达式或声明：`context_`。
- **L1298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.Say(namelistLocation,`.
  **L1298 CN**: 继续一个多行参数列表、初始化器或聚合项：`.Say(namelistLocation,`。
- **L1299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"NAMELIST input group must not contain undefinable item '%s'"_err_en_US,`.
  **L1299 CN**: 继续一个多行参数列表、初始化器或聚合项：`"NAMELIST input group must not contain undefinable item '%s'"_err_en_US,`。
- **L1300 EN**: Continues logic associated with callable symbol `name`.
  **L1300 CN**: 继续与可调用符号 `name` 相关的逻辑。
- **L1301 EN**: Executes a call or declaration centered on `.Attach`.
  **L1301 CN**: 执行以 `.Attach` 为核心的调用或声明。
- **L1302 EN**: Executes a call or declaration centered on `context_.SetError`.
  **L1302 CN**: 执行以 `context_.SetError` 为核心的调用或声明。
- **L1303 EN**: Closes the current lexical scope or compound statement.
  **L1303 CN**: 结束当前词法作用域或复合语句块。
- **L1304 EN**: Closes the current lexical scope or compound statement.
  **L1304 CN**: 结束当前词法作用域或复合语句块。
- **L1305 EN**: Closes the current lexical scope or compound statement.
  **L1305 CN**: 结束当前词法作用域或复合语句块。
- **L1306 EN**: Closes the current lexical scope or compound statement.
  **L1306 CN**: 结束当前词法作用域或复合语句块。
- **L1307 EN**: Closes the current lexical scope or compound statement.
  **L1307 CN**: 结束当前词法作用域或复合语句块。
- **L1308 EN**: Blank line separating nearby declarations or logic blocks.
  **L1308 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1309 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::semantics`.
  **L1309 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::semantics`。

## Key Concepts / 关键概念

- **Fortran semantic checking / Fortran 语义检查**
- **Parser data structures and diagnostics / 解析器数据结构与诊断**
- **Semantic-context management / 语义上下文管理**
- **Scope and symbol resolution / 作用域与符号解析**
- **Symbol modeling and lookup / 符号建模与查找**
- **Fortran type-spec reasoning / Fortran 类型说明推理**
- **Compile-time evaluation helpers / 编译期求值辅助**
- **Rule-checking workflow / 规则检查流程**

## Dependencies / 依赖关系

- `check-io.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `definable.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Common/format.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Common/indirection.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Evaluate/tools.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Parser/characters.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Parser/tools.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Semantics/expression.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/tools.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `unordered_map`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
- `unordered_set`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
