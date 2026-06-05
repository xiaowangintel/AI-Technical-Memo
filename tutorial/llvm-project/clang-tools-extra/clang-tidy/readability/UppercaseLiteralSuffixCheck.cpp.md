# UppercaseLiteralSuffixCheck.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang-tools-extra/clang-tidy/readability/UppercaseLiteralSuffixCheck.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Implements the clang-tidy check `UppercaseLiteralSuffixCheck`, including AST matching, diagnostics, and fix-it behavior.
  - **CN**: 实现 clang-tidy 检查 `UppercaseLiteralSuffixCheck`，包括 AST 匹配、诊断与自动修复行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
 1 | //===----------------------------------------------------------------------===//
 2 | //
 3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4 | // See https://llvm.org/LICENSE.txt for license information.
 5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6 | //
 7 | //===----------------------------------------------------------------------===//
 8 | 
 9 | #include "UppercaseLiteralSuffixCheck.h"
10 | #include "../utils/ASTUtils.h"
11 | #include "../utils/OptionsUtils.h"
12 | #include "clang/AST/ASTContext.h"
13 | #include "clang/ASTMatchers/ASTMatchFinder.h"
14 | #include "clang/Lex/Lexer.h"
15 | #include <optional>
16 | 
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L3**: Comment explains nearby logic, intent, or usage: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑、意图或用法：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic, intent, or usage: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑、意图或用法：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic, intent, or usage: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑、意图或用法：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L8**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L9**: Includes "UppercaseLiteralSuffixCheck.h" to access local declarations from the current tool or check. / 引入 "UppercaseLiteralSuffixCheck.h" 以使用当前工具或检查的本地声明。
- **L10**: Includes "../utils/ASTUtils.h" to access shared clang-tidy utility helpers. / 引入 "../utils/ASTUtils.h" 以使用共享 clang-tidy 工具辅助逻辑。
- **L11**: Includes "../utils/OptionsUtils.h" to access shared clang-tidy utility helpers. / 引入 "../utils/OptionsUtils.h" 以使用共享 clang-tidy 工具辅助逻辑。
- **L12**: Includes "clang/AST/ASTContext.h" to access Clang AST node and semantic interfaces. / 引入 "clang/AST/ASTContext.h" 以使用Clang AST 节点与语义接口。
- **L13**: Includes "clang/ASTMatchers/ASTMatchFinder.h" to access AST matcher construction helpers. / 引入 "clang/ASTMatchers/ASTMatchFinder.h" 以使用AST 匹配器构造辅助逻辑。
- **L14**: Includes "clang/Lex/Lexer.h" to access lexer and preprocessor interfaces. / 引入 "clang/Lex/Lexer.h" 以使用词法分析器与预处理器接口。
- **L15**: Includes <optional> to access C or C++ standard library facilities. / 引入 <optional> 以使用C 或 C++ 标准库设施。
- **L16**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。

### Lines 17-32 / 第 17-32 行

```cpp
17 | using namespace clang::ast_matchers;
18 | 
19 | namespace clang::tidy::readability {
20 | 
21 | namespace {
22 | 
23 | struct NewSuffix {
24 |   SourceRange LiteralLocation;
25 |   StringRef OldSuffix;
26 |   std::optional<FixItHint> FixIt;
27 | };
28 | 
29 | struct LiteralParameters {
30 |   // What characters should be skipped before looking for the Suffixes?
31 |   StringRef SkipFirst;
32 |   // What characters can a suffix start with?
```

- **L17**: Brings namespace `clang::ast_matchers` into the local scope. / 将命名空间 `clang::ast_matchers` 引入当前作用域。
- **L18**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L19**: Opens namespace scope `clang::tidy::readability`. / 打开命名空间作用域 `clang::tidy::readability`。
- **L20**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L21**: Opens an anonymous namespace for internal linkage. / 打开匿名命名空间以提供内部链接。
- **L22**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L23**: Declares struct `NewSuffix`. / 声明 struct `NewSuffix`。
- **L24**: Executes a standalone statement or declaration: `SourceRange LiteralLocation;`. / 执行一条独立语句或声明：`SourceRange LiteralLocation;`。
- **L25**: Executes a standalone statement or declaration: `StringRef OldSuffix;`. / 执行一条独立语句或声明：`StringRef OldSuffix;`。
- **L26**: Constructs an automated source edit to accompany a diagnostic. / 构造与诊断配套的自动化源码编辑。
- **L27**: Closes the current declaration scope such as a class, struct, or enum. / 结束当前声明作用域，例如类、结构体或枚举。
- **L28**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L29**: Declares struct `LiteralParameters`. / 声明 struct `LiteralParameters`。
- **L30**: Comment explains nearby logic, intent, or usage: `What characters should be skipped before looking for the Suffixes?`. / 注释说明了附近代码的逻辑、意图或用法：`What characters should be skipped before looking for the Suffixes?`。
- **L31**: Executes a standalone statement or declaration: `StringRef SkipFirst;`. / 执行一条独立语句或声明：`StringRef SkipFirst;`。
- **L32**: Comment explains nearby logic, intent, or usage: `What characters can a suffix start with?`. / 注释说明了附近代码的逻辑、意图或用法：`What characters can a suffix start with?`。

### Lines 33-48 / 第 33-48 行

```cpp
33 |   StringRef Suffixes;
34 | };
35 | 
36 | } // namespace
37 | 
38 | static constexpr LiteralParameters IntegerParameters = {
39 |     "",
40 |     // Suffix can only consist of 'u', 'l', and 'z' chars, can be a
41 |     // bit-precise integer (wb), and can be a complex number ('i', 'j'). In MS
42 |     // compatibility mode, suffixes like i32 are supported.
43 |     "uUlLzZwWiIjJ",
44 | };
45 | 
46 | static constexpr LiteralParameters FloatParameters = {
47 |     // C++17 introduced hexadecimal floating-point literals, and 'f' is both a
48 |     // valid hexadecimal digit in a hex float literal and a valid floating-point
```

- **L33**: Executes a standalone statement or declaration: `StringRef Suffixes;`. / 执行一条独立语句或声明：`StringRef Suffixes;`。
- **L34**: Closes the current declaration scope such as a class, struct, or enum. / 结束当前声明作用域，例如类、结构体或枚举。
- **L35**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L36**: Closes a namespace scope while preserving the trailing comment: `} // namespace`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L37**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L38**: Continues the surrounding expression or declaration: `static constexpr LiteralParameters IntegerParameters = {`. / 继续构造周围的表达式或声明：`static constexpr LiteralParameters IntegerParameters = {`。
- **L39**: Continues a multi-line argument list, initializer, or aggregate entry: `"",`. / 继续一个多行参数列表、初始化器或聚合项：`"",`。
- **L40**: Comment explains nearby logic, intent, or usage: `Suffix can only consist of 'u', 'l', and 'z' chars, can be a`. / 注释说明了附近代码的逻辑、意图或用法：`Suffix can only consist of 'u', 'l', and 'z' chars, can be a`。
- **L41**: Comment explains nearby logic, intent, or usage: `bit-precise integer (wb), and can be a complex number ('i', 'j'). In MS`. / 注释说明了附近代码的逻辑、意图或用法：`bit-precise integer (wb), and can be a complex number ('i', 'j'). In MS`。
- **L42**: Comment explains nearby logic, intent, or usage: `compatibility mode, suffixes like i32 are supported.`. / 注释说明了附近代码的逻辑、意图或用法：`compatibility mode, suffixes like i32 are supported.`。
- **L43**: Continues a multi-line argument list, initializer, or aggregate entry: `"uUlLzZwWiIjJ",`. / 继续一个多行参数列表、初始化器或聚合项：`"uUlLzZwWiIjJ",`。
- **L44**: Closes the current declaration scope such as a class, struct, or enum. / 结束当前声明作用域，例如类、结构体或枚举。
- **L45**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L46**: Continues the surrounding expression or declaration: `static constexpr LiteralParameters FloatParameters = {`. / 继续构造周围的表达式或声明：`static constexpr LiteralParameters FloatParameters = {`。
- **L47**: Comment explains nearby logic, intent, or usage: `C++17 introduced hexadecimal floating-point literals, and 'f' is both a`. / 注释说明了附近代码的逻辑、意图或用法：`C++17 introduced hexadecimal floating-point literals, and 'f' is both a`。
- **L48**: Comment explains nearby logic, intent, or usage: `valid hexadecimal digit in a hex float literal and a valid floating-point`. / 注释说明了附近代码的逻辑、意图或用法：`valid hexadecimal digit in a hex float literal and a valid floating-point`。

### Lines 49-64 / 第 49-64 行

```cpp
49 |     // literal suffix.
50 |     // So we can't just "skip to the chars that can be in the suffix".
51 |     // Since the exponent ('p'/'P') is mandatory for hexadecimal floating-point
52 |     // literals, we first skip everything before the exponent.
53 |     "pP",
54 |     // Suffix can only consist of 'f', 'l', "f16", "bf16", "df", "dd", "dl",
55 |     // 'h', 'q' chars, and can be a complex number ('i', 'j').
56 |     "fFlLbBdDhHqQiIjJ",
57 | };
58 | 
59 | static std::optional<SourceLocation>
60 | getMacroAwareLocation(SourceLocation Loc, const SourceManager &SM) {
61 |   // Do nothing if the provided location is invalid.
62 |   if (Loc.isInvalid())
63 |     return std::nullopt;
64 |   // Look where the location was *actually* written.
```

- **L49**: Comment explains nearby logic, intent, or usage: `literal suffix.`. / 注释说明了附近代码的逻辑、意图或用法：`literal suffix.`。
- **L50**: Comment explains nearby logic, intent, or usage: `So we can't just "skip to the chars that can be in the suffix".`. / 注释说明了附近代码的逻辑、意图或用法：`So we can't just "skip to the chars that can be in the suffix".`。
- **L51**: Comment explains nearby logic, intent, or usage: `Since the exponent ('p'/'P') is mandatory for hexadecimal floating-point`. / 注释说明了附近代码的逻辑、意图或用法：`Since the exponent ('p'/'P') is mandatory for hexadecimal floating-point`。
- **L52**: Comment explains nearby logic, intent, or usage: `literals, we first skip everything before the exponent.`. / 注释说明了附近代码的逻辑、意图或用法：`literals, we first skip everything before the exponent.`。
- **L53**: Continues a multi-line argument list, initializer, or aggregate entry: `"pP",`. / 继续一个多行参数列表、初始化器或聚合项：`"pP",`。
- **L54**: Comment explains nearby logic, intent, or usage: `Suffix can only consist of 'f', 'l', "f16", "bf16", "df", "dd", "dl",`. / 注释说明了附近代码的逻辑、意图或用法：`Suffix can only consist of 'f', 'l', "f16", "bf16", "df", "dd", "dl",`。
- **L55**: Comment explains nearby logic, intent, or usage: `'h', 'q' chars, and can be a complex number ('i', 'j').`. / 注释说明了附近代码的逻辑、意图或用法：`'h', 'q' chars, and can be a complex number ('i', 'j').`。
- **L56**: Continues a multi-line argument list, initializer, or aggregate entry: `"fFlLbBdDhHqQiIjJ",`. / 继续一个多行参数列表、初始化器或聚合项：`"fFlLbBdDhHqQiIjJ",`。
- **L57**: Closes the current declaration scope such as a class, struct, or enum. / 结束当前声明作用域，例如类、结构体或枚举。
- **L58**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L59**: Continues the surrounding expression or declaration: `static std::optional<SourceLocation>`. / 继续构造周围的表达式或声明：`static std::optional<SourceLocation>`。
- **L60**: Starts a function, method, lambda, or structured scope: `getMacroAwareLocation(SourceLocation Loc, const SourceManager &SM) {`. / 开始一个函数、方法、lambda 或结构化作用域：`getMacroAwareLocation(SourceLocation Loc, const SourceManager &SM) {`。
- **L61**: Comment explains nearby logic, intent, or usage: `Do nothing if the provided location is invalid.`. / 注释说明了附近代码的逻辑、意图或用法：`Do nothing if the provided location is invalid.`。
- **L62**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L63**: Returns from the current function with `std::nullopt`. / 以 `std::nullopt` 从当前函数返回。
- **L64**: Comment explains nearby logic, intent, or usage: `Look where the location was *actually* written.`. / 注释说明了附近代码的逻辑、意图或用法：`Look where the location was *actually* written.`。

### Lines 65-80 / 第 65-80 行

```cpp
65 |   SourceLocation SpellingLoc = SM.getSpellingLoc(Loc);
66 |   if (SpellingLoc.isInvalid())
67 |     return std::nullopt;
68 |   return SpellingLoc;
69 | }
70 | 
71 | static std::optional<SourceRange>
72 | getMacroAwareSourceRange(SourceRange Loc, const SourceManager &SM) {
73 |   std::optional<SourceLocation> Begin =
74 |       getMacroAwareLocation(Loc.getBegin(), SM);
75 |   std::optional<SourceLocation> End = getMacroAwareLocation(Loc.getEnd(), SM);
76 |   if (!Begin || !End)
77 |     return std::nullopt;
78 |   return SourceRange(*Begin, *End);
79 | }
80 | 
```

- **L65**: Initializes variable `SpellingLoc` from the right-hand expression. / 使用右侧表达式初始化变量 `SpellingLoc`。
- **L66**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L67**: Returns from the current function with `std::nullopt`. / 以 `std::nullopt` 从当前函数返回。
- **L68**: Returns from the current function with `SpellingLoc`. / 以 `SpellingLoc` 从当前函数返回。
- **L69**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L70**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L71**: Continues the surrounding expression or declaration: `static std::optional<SourceRange>`. / 继续构造周围的表达式或声明：`static std::optional<SourceRange>`。
- **L72**: Starts a function, method, lambda, or structured scope: `getMacroAwareSourceRange(SourceRange Loc, const SourceManager &SM) {`. / 开始一个函数、方法、lambda 或结构化作用域：`getMacroAwareSourceRange(SourceRange Loc, const SourceManager &SM) {`。
- **L73**: Continues the surrounding expression or declaration: `std::optional<SourceLocation> Begin =`. / 继续构造周围的表达式或声明：`std::optional<SourceLocation> Begin =`。
- **L74**: Executes a call or declaration centered on `getMacroAwareLocation`. / 执行以 `getMacroAwareLocation` 为核心的调用或声明。
- **L75**: Initializes variable `End` from the right-hand expression. / 使用右侧表达式初始化变量 `End`。
- **L76**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L77**: Returns from the current function with `std::nullopt`. / 以 `std::nullopt` 从当前函数返回。
- **L78**: Returns from the current function with `SourceRange(*Begin, *End)`. / 以 `SourceRange(*Begin, *End)` 从当前函数返回。
- **L79**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L80**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。

### Lines 81-96 / 第 81-96 行

```cpp
81 | static std::optional<std::string>
82 | getNewSuffix(StringRef OldSuffix, const std::vector<StringRef> &NewSuffixes) {
83 |   // If there is no config, just uppercase the entirety of the suffix.
84 |   if (NewSuffixes.empty())
85 |     return OldSuffix.upper();
86 |   // Else, find matching suffix, case-*insensitive*ly.
87 |   auto NewSuffix =
88 |       llvm::find_if(NewSuffixes, [OldSuffix](StringRef PotentialNewSuffix) {
89 |         return OldSuffix.equals_insensitive(PotentialNewSuffix);
90 |       });
91 |   // Have a match, return it.
92 |   if (NewSuffix != NewSuffixes.end())
93 |     return NewSuffix->str();
94 |   // Nope, I guess we have to keep it as-is.
95 |   return std::nullopt;
96 | }
```

- **L81**: Continues the surrounding expression or declaration: `static std::optional<std::string>`. / 继续构造周围的表达式或声明：`static std::optional<std::string>`。
- **L82**: Starts a function, method, lambda, or structured scope: `getNewSuffix(StringRef OldSuffix, const std::vector<StringRef> &NewSuffixes) {`. / 开始一个函数、方法、lambda 或结构化作用域：`getNewSuffix(StringRef OldSuffix, const std::vector<StringRef> &NewSuffixes) {`。
- **L83**: Comment explains nearby logic, intent, or usage: `If there is no config, just uppercase the entirety of the suffix.`. / 注释说明了附近代码的逻辑、意图或用法：`If there is no config, just uppercase the entirety of the suffix.`。
- **L84**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L85**: Returns from the current function with `OldSuffix.upper()`. / 以 `OldSuffix.upper()` 从当前函数返回。
- **L86**: Comment explains nearby logic, intent, or usage: `Else, find matching suffix, case-*insensitive*ly.`. / 注释说明了附近代码的逻辑、意图或用法：`Else, find matching suffix, case-*insensitive*ly.`。
- **L87**: Continues the surrounding expression or declaration: `auto NewSuffix =`. / 继续构造周围的表达式或声明：`auto NewSuffix =`。
- **L88**: Starts a function, method, lambda, or structured scope: `llvm::find_if(NewSuffixes, [OldSuffix](StringRef PotentialNewSuffix) {`. / 开始一个函数、方法、lambda 或结构化作用域：`llvm::find_if(NewSuffixes, [OldSuffix](StringRef PotentialNewSuffix) {`。
- **L89**: Returns from the current function with `OldSuffix.equals_insensitive(PotentialNewSuffix)`. / 以 `OldSuffix.equals_insensitive(PotentialNewSuffix)` 从当前函数返回。
- **L90**: Executes a standalone statement or declaration: `});`. / 执行一条独立语句或声明：`});`。
- **L91**: Comment explains nearby logic, intent, or usage: `Have a match, return it.`. / 注释说明了附近代码的逻辑、意图或用法：`Have a match, return it.`。
- **L92**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L93**: Returns from the current function with `NewSuffix->str()`. / 以 `NewSuffix->str()` 从当前函数返回。
- **L94**: Comment explains nearby logic, intent, or usage: `Nope, I guess we have to keep it as-is.`. / 注释说明了附近代码的逻辑、意图或用法：`Nope, I guess we have to keep it as-is.`。
- **L95**: Returns from the current function with `std::nullopt`. / 以 `std::nullopt` 从当前函数返回。
- **L96**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

### Lines 97-112 / 第 97-112 行

```cpp
 97 | 
 98 | static std::optional<NewSuffix>
 99 | shouldReplaceLiteralSuffix(const Expr &Literal,
100 |                            const LiteralParameters &Parameters,
101 |                            const std::vector<StringRef> &NewSuffixes,
102 |                            const SourceManager &SM, const LangOptions &LO) {
103 |   NewSuffix ReplacementDsc;
104 | 
105 |   // The naive location of the literal. Is always valid.
106 |   ReplacementDsc.LiteralLocation = Literal.getSourceRange();
107 | 
108 |   // Was this literal fully spelled or is it a product of macro expansion?
109 |   const bool RangeCanBeFixed =
110 |       utils::rangeCanBeFixed(ReplacementDsc.LiteralLocation, &SM);
111 | 
112 |   // The literal may have macro expansion, we need the final expanded src range.
```

- **L97**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L98**: Continues the surrounding expression or declaration: `static std::optional<NewSuffix>`. / 继续构造周围的表达式或声明：`static std::optional<NewSuffix>`。
- **L99**: Continues a multi-line argument list, initializer, or aggregate entry: `shouldReplaceLiteralSuffix(const Expr &Literal,`. / 继续一个多行参数列表、初始化器或聚合项：`shouldReplaceLiteralSuffix(const Expr &Literal,`。
- **L100**: Continues a multi-line argument list, initializer, or aggregate entry: `const LiteralParameters &Parameters,`. / 继续一个多行参数列表、初始化器或聚合项：`const LiteralParameters &Parameters,`。
- **L101**: Continues a multi-line argument list, initializer, or aggregate entry: `const std::vector<StringRef> &NewSuffixes,`. / 继续一个多行参数列表、初始化器或聚合项：`const std::vector<StringRef> &NewSuffixes,`。
- **L102**: Continues the surrounding expression or declaration: `const SourceManager &SM, const LangOptions &LO) {`. / 继续构造周围的表达式或声明：`const SourceManager &SM, const LangOptions &LO) {`。
- **L103**: Executes a standalone statement or declaration: `NewSuffix ReplacementDsc;`. / 执行一条独立语句或声明：`NewSuffix ReplacementDsc;`。
- **L104**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L105**: Comment explains nearby logic, intent, or usage: `The naive location of the literal. Is always valid.`. / 注释说明了附近代码的逻辑、意图或用法：`The naive location of the literal. Is always valid.`。
- **L106**: Executes a call or declaration centered on `Literal.getSourceRange`. / 执行以 `Literal.getSourceRange` 为核心的调用或声明。
- **L107**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L108**: Comment explains nearby logic, intent, or usage: `Was this literal fully spelled or is it a product of macro expansion?`. / 注释说明了附近代码的逻辑、意图或用法：`Was this literal fully spelled or is it a product of macro expansion?`。
- **L109**: Continues the surrounding expression or declaration: `const bool RangeCanBeFixed =`. / 继续构造周围的表达式或声明：`const bool RangeCanBeFixed =`。
- **L110**: Executes a call or declaration centered on `utils::rangeCanBeFixed`. / 执行以 `utils::rangeCanBeFixed` 为核心的调用或声明。
- **L111**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L112**: Comment explains nearby logic, intent, or usage: `The literal may have macro expansion, we need the final expanded src range.`. / 注释说明了附近代码的逻辑、意图或用法：`The literal may have macro expansion, we need the final expanded src range.`。

### Lines 113-128 / 第 113-128 行

```cpp
113 |   std::optional<SourceRange> Range =
114 |       getMacroAwareSourceRange(ReplacementDsc.LiteralLocation, SM);
115 |   if (!Range)
116 |     return std::nullopt;
117 | 
118 |   if (RangeCanBeFixed)
119 |     ReplacementDsc.LiteralLocation = *Range;
120 |   // Else keep the naive literal location!
121 | 
122 |   // Get the whole literal from the source buffer.
123 |   bool Invalid = false;
124 |   const StringRef LiteralSourceText = Lexer::getSourceText(
125 |       CharSourceRange::getTokenRange(*Range), SM, LO, &Invalid);
126 |   assert(!Invalid && "Failed to retrieve the source text.");
127 | 
128 |   // Make sure the first character is actually a digit, instead of
```

- **L113**: Continues the surrounding expression or declaration: `std::optional<SourceRange> Range =`. / 继续构造周围的表达式或声明：`std::optional<SourceRange> Range =`。
- **L114**: Executes a call or declaration centered on `getMacroAwareSourceRange`. / 执行以 `getMacroAwareSourceRange` 为核心的调用或声明。
- **L115**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L116**: Returns from the current function with `std::nullopt`. / 以 `std::nullopt` 从当前函数返回。
- **L117**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L118**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L119**: Executes a standalone statement or declaration: `ReplacementDsc.LiteralLocation = *Range;`. / 执行一条独立语句或声明：`ReplacementDsc.LiteralLocation = *Range;`。
- **L120**: Comment explains nearby logic, intent, or usage: `Else keep the naive literal location!`. / 注释说明了附近代码的逻辑、意图或用法：`Else keep the naive literal location!`。
- **L121**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L122**: Comment explains nearby logic, intent, or usage: `Get the whole literal from the source buffer.`. / 注释说明了附近代码的逻辑、意图或用法：`Get the whole literal from the source buffer.`。
- **L123**: Initializes variable `Invalid` from the right-hand expression. / 使用右侧表达式初始化变量 `Invalid`。
- **L124**: Continues logic associated with callable symbol `getSourceText`. / 继续与可调用符号 `getSourceText` 相关的逻辑。
- **L125**: Executes a call or declaration centered on `CharSourceRange::getTokenRange`. / 执行以 `CharSourceRange::getTokenRange` 为核心的调用或声明。
- **L126**: Checks an internal invariant in debug builds. / 在调试构建中检查内部不变式。
- **L127**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L128**: Comment explains nearby logic, intent, or usage: `Make sure the first character is actually a digit, instead of`. / 注释说明了附近代码的逻辑、意图或用法：`Make sure the first character is actually a digit, instead of`。

### Lines 129-144 / 第 129-144 行

```cpp
129 |   // something else, like a non-type template parameter.
130 |   if (!std::isdigit(static_cast<unsigned char>(LiteralSourceText.front())))
131 |     return std::nullopt;
132 | 
133 |   size_t Skip = 0;
134 | 
135 |   // Do we need to ignore something before actually looking for the suffix?
136 |   if (!Parameters.SkipFirst.empty()) {
137 |     // E.g. we can't look for 'f' suffix in hexadecimal floating-point literals
138 |     // until after we skip to the exponent (which is mandatory there),
139 |     // because hex-digit-sequence may contain 'f'.
140 |     Skip = LiteralSourceText.find_first_of(Parameters.SkipFirst);
141 |     // We could be in non-hexadecimal floating-point literal, with no exponent.
142 |     if (Skip == StringRef::npos)
143 |       Skip = 0;
144 |   }
```

- **L129**: Comment explains nearby logic, intent, or usage: `something else, like a non-type template parameter.`. / 注释说明了附近代码的逻辑、意图或用法：`something else, like a non-type template parameter.`。
- **L130**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L131**: Returns from the current function with `std::nullopt`. / 以 `std::nullopt` 从当前函数返回。
- **L132**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L133**: Initializes variable `Skip` from the right-hand expression. / 使用右侧表达式初始化变量 `Skip`。
- **L134**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L135**: Comment explains nearby logic, intent, or usage: `Do we need to ignore something before actually looking for the suffix?`. / 注释说明了附近代码的逻辑、意图或用法：`Do we need to ignore something before actually looking for the suffix?`。
- **L136**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L137**: Comment explains nearby logic, intent, or usage: `E.g. we can't look for 'f' suffix in hexadecimal floating-point literals`. / 注释说明了附近代码的逻辑、意图或用法：`E.g. we can't look for 'f' suffix in hexadecimal floating-point literals`。
- **L138**: Comment explains nearby logic, intent, or usage: `until after we skip to the exponent (which is mandatory there),`. / 注释说明了附近代码的逻辑、意图或用法：`until after we skip to the exponent (which is mandatory there),`。
- **L139**: Comment explains nearby logic, intent, or usage: `because hex-digit-sequence may contain 'f'.`. / 注释说明了附近代码的逻辑、意图或用法：`because hex-digit-sequence may contain 'f'.`。
- **L140**: Assigns new state to `Skip` for later logic. / 为后续逻辑给 `Skip` 赋予新状态。
- **L141**: Comment explains nearby logic, intent, or usage: `We could be in non-hexadecimal floating-point literal, with no exponent.`. / 注释说明了附近代码的逻辑、意图或用法：`We could be in non-hexadecimal floating-point literal, with no exponent.`。
- **L142**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L143**: Assigns new state to `Skip` for later logic. / 为后续逻辑给 `Skip` 赋予新状态。
- **L144**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

### Lines 145-160 / 第 145-160 行

```cpp
145 | 
146 |   // Find the beginning of the suffix by looking for the first char that is
147 |   // one of these chars that can be in the suffix, potentially starting looking
148 |   // in the exponent, if we are skipping hex-digit-sequence.
149 |   Skip = LiteralSourceText.find_first_of(Parameters.Suffixes, /*From=*/Skip);
150 | 
151 |   // We can't check whether the *Literal has any suffix or not without actually
152 |   // looking for the suffix. So it is totally possible that there is no suffix.
153 |   if (Skip == StringRef::npos)
154 |     return std::nullopt;
155 | 
156 |   // Move the cursor in the source range to the beginning of the suffix.
157 |   Range->setBegin(Range->getBegin().getLocWithOffset(Skip));
158 |   // And in our textual representation too.
159 |   ReplacementDsc.OldSuffix = LiteralSourceText.drop_front(Skip);
160 |   assert(!ReplacementDsc.OldSuffix.empty() &&
```

- **L145**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L146**: Comment explains nearby logic, intent, or usage: `Find the beginning of the suffix by looking for the first char that is`. / 注释说明了附近代码的逻辑、意图或用法：`Find the beginning of the suffix by looking for the first char that is`。
- **L147**: Comment explains nearby logic, intent, or usage: `one of these chars that can be in the suffix, potentially starting looking`. / 注释说明了附近代码的逻辑、意图或用法：`one of these chars that can be in the suffix, potentially starting looking`。
- **L148**: Comment explains nearby logic, intent, or usage: `in the exponent, if we are skipping hex-digit-sequence.`. / 注释说明了附近代码的逻辑、意图或用法：`in the exponent, if we are skipping hex-digit-sequence.`。
- **L149**: Assigns new state to `Skip` for later logic. / 为后续逻辑给 `Skip` 赋予新状态。
- **L150**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L151**: Comment explains nearby logic, intent, or usage: `We can't check whether the *Literal has any suffix or not without actually`. / 注释说明了附近代码的逻辑、意图或用法：`We can't check whether the *Literal has any suffix or not without actually`。
- **L152**: Comment explains nearby logic, intent, or usage: `looking for the suffix. So it is totally possible that there is no suffix.`. / 注释说明了附近代码的逻辑、意图或用法：`looking for the suffix. So it is totally possible that there is no suffix.`。
- **L153**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L154**: Returns from the current function with `std::nullopt`. / 以 `std::nullopt` 从当前函数返回。
- **L155**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L156**: Comment explains nearby logic, intent, or usage: `Move the cursor in the source range to the beginning of the suffix.`. / 注释说明了附近代码的逻辑、意图或用法：`Move the cursor in the source range to the beginning of the suffix.`。
- **L157**: Executes a call or declaration centered on `Range->setBegin`. / 执行以 `Range->setBegin` 为核心的调用或声明。
- **L158**: Comment explains nearby logic, intent, or usage: `And in our textual representation too.`. / 注释说明了附近代码的逻辑、意图或用法：`And in our textual representation too.`。
- **L159**: Executes a call or declaration centered on `LiteralSourceText.drop_front`. / 执行以 `LiteralSourceText.drop_front` 为核心的调用或声明。
- **L160**: Checks an internal invariant in debug builds. / 在调试构建中检查内部不变式。

### Lines 161-176 / 第 161-176 行

```cpp
161 |          "We still should have some chars left.");
162 | 
163 |   // And get the replacement suffix.
164 |   std::optional<std::string> NewSuffix =
165 |       getNewSuffix(ReplacementDsc.OldSuffix, NewSuffixes);
166 |   if (!NewSuffix || ReplacementDsc.OldSuffix == *NewSuffix)
167 |     return std::nullopt; // The suffix was already the way it should be.
168 | 
169 |   if (RangeCanBeFixed)
170 |     ReplacementDsc.FixIt = FixItHint::CreateReplacement(*Range, *NewSuffix);
171 | 
172 |   return ReplacementDsc;
173 | }
174 | 
175 | UppercaseLiteralSuffixCheck::UppercaseLiteralSuffixCheck(
176 |     StringRef Name, ClangTidyContext *Context)
```

- **L161**: Executes a standalone statement or declaration: `"We still should have some chars left.");`. / 执行一条独立语句或声明：`"We still should have some chars left.");`。
- **L162**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L163**: Comment explains nearby logic, intent, or usage: `And get the replacement suffix.`. / 注释说明了附近代码的逻辑、意图或用法：`And get the replacement suffix.`。
- **L164**: Continues the surrounding expression or declaration: `std::optional<std::string> NewSuffix =`. / 继续构造周围的表达式或声明：`std::optional<std::string> NewSuffix =`。
- **L165**: Executes a call or declaration centered on `getNewSuffix`. / 执行以 `getNewSuffix` 为核心的调用或声明。
- **L166**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L167**: Returns from the current function with `std::nullopt; // The suffix was already the way it should be.`. / 以 `std::nullopt; // The suffix was already the way it should be.` 从当前函数返回。
- **L168**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L169**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L170**: Constructs an automated source edit to accompany a diagnostic. / 构造与诊断配套的自动化源码编辑。
- **L171**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L172**: Returns from the current function with `ReplacementDsc`. / 以 `ReplacementDsc` 从当前函数返回。
- **L173**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L174**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L175**: Continues logic associated with callable symbol `UppercaseLiteralSuffixCheck`. / 继续与可调用符号 `UppercaseLiteralSuffixCheck` 相关的逻辑。
- **L176**: Continues the surrounding expression or declaration: `StringRef Name, ClangTidyContext *Context)`. / 继续构造周围的表达式或声明：`StringRef Name, ClangTidyContext *Context)`。

### Lines 177-192 / 第 177-192 行

```cpp
177 |     : ClangTidyCheck(Name, Context),
178 |       NewSuffixes(
179 |           utils::options::parseStringList(Options.get("NewSuffixes", ""))),
180 |       IgnoreMacros(Options.get("IgnoreMacros", true)) {}
181 | 
182 | void UppercaseLiteralSuffixCheck::storeOptions(
183 |     ClangTidyOptions::OptionMap &Opts) {
184 |   Options.store(Opts, "NewSuffixes",
185 |                 utils::options::serializeStringList(NewSuffixes));
186 |   Options.store(Opts, "IgnoreMacros", IgnoreMacros);
187 | }
188 | 
189 | void UppercaseLiteralSuffixCheck::registerMatchers(MatchFinder *Finder) {
190 |   // Sadly, we can't check whether the literal has suffix or not.
191 |   // E.g. i32 suffix still results in 'BuiltinType::Kind::Int'.
192 |   // And such an info is not stored in the *Literal itself.
```

- **L177**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangTidyCheck(Name, Context),`. / 继续一个多行参数列表、初始化器或聚合项：`: ClangTidyCheck(Name, Context),`。
- **L178**: Continues logic associated with callable symbol `NewSuffixes`. / 继续与可调用符号 `NewSuffixes` 相关的逻辑。
- **L179**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L180**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L181**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L182**: Continues logic associated with callable symbol `storeOptions`. / 继续与可调用符号 `storeOptions` 相关的逻辑。
- **L183**: Continues the surrounding expression or declaration: `ClangTidyOptions::OptionMap &Opts) {`. / 继续构造周围的表达式或声明：`ClangTidyOptions::OptionMap &Opts) {`。
- **L184**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L185**: Executes a call or declaration centered on `utils::options::serializeStringList`. / 执行以 `utils::options::serializeStringList` 为核心的调用或声明。
- **L186**: Reads or writes persistent clang-tidy configuration options. / 读取或写入持久化 clang-tidy 配置选项。
- **L187**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L188**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L189**: Registers AST matcher logic so the check can recognize target code patterns. / 注册 AST 匹配逻辑，使检查能够识别目标代码模式。
- **L190**: Comment explains nearby logic, intent, or usage: `Sadly, we can't check whether the literal has suffix or not.`. / 注释说明了附近代码的逻辑、意图或用法：`Sadly, we can't check whether the literal has suffix or not.`。
- **L191**: Comment explains nearby logic, intent, or usage: `E.g. i32 suffix still results in 'BuiltinType::Kind::Int'.`. / 注释说明了附近代码的逻辑、意图或用法：`E.g. i32 suffix still results in 'BuiltinType::Kind::Int'.`。
- **L192**: Comment explains nearby logic, intent, or usage: `And such an info is not stored in the *Literal itself.`. / 注释说明了附近代码的逻辑、意图或用法：`And such an info is not stored in the *Literal itself.`。

### Lines 193-208 / 第 193-208 行

```cpp
193 | 
194 |   Finder->addMatcher(
195 |       integerLiteral(unless(hasParent(userDefinedLiteral()))).bind("expr"),
196 |       this);
197 |   Finder->addMatcher(
198 |       floatLiteral(unless(hasParent(userDefinedLiteral()))).bind("expr"), this);
199 | }
200 | 
201 | void UppercaseLiteralSuffixCheck::check(
202 |     const MatchFinder::MatchResult &Result) {
203 |   const auto *const Literal = Result.Nodes.getNodeAs<Expr>("expr");
204 |   const bool IsInteger = isa<IntegerLiteral>(Literal);
205 | 
206 |   // We won't *always* want to diagnose.
207 |   // We might have a suffix that is already uppercase.
208 |   if (auto Details = shouldReplaceLiteralSuffix(
```

- **L193**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L194**: Registers AST matcher logic so the check can recognize target code patterns. / 注册 AST 匹配逻辑，使检查能够识别目标代码模式。
- **L195**: Continues a multi-line argument list, initializer, or aggregate entry: `integerLiteral(unless(hasParent(userDefinedLiteral()))).bind("expr"),`. / 继续一个多行参数列表、初始化器或聚合项：`integerLiteral(unless(hasParent(userDefinedLiteral()))).bind("expr"),`。
- **L196**: Executes a standalone statement or declaration: `this);`. / 执行一条独立语句或声明：`this);`。
- **L197**: Registers AST matcher logic so the check can recognize target code patterns. / 注册 AST 匹配逻辑，使检查能够识别目标代码模式。
- **L198**: Executes a call or declaration centered on `floatLiteral`. / 执行以 `floatLiteral` 为核心的调用或声明。
- **L199**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L200**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L201**: Continues logic associated with callable symbol `check`. / 继续与可调用符号 `check` 相关的逻辑。
- **L202**: Continues the surrounding expression or declaration: `const MatchFinder::MatchResult &Result) {`. / 继续构造周围的表达式或声明：`const MatchFinder::MatchResult &Result) {`。
- **L203**: Initializes variable `Literal` from the right-hand expression. / 使用右侧表达式初始化变量 `Literal`。
- **L204**: Initializes variable `IsInteger` from the right-hand expression. / 使用右侧表达式初始化变量 `IsInteger`。
- **L205**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L206**: Comment explains nearby logic, intent, or usage: `We won't *always* want to diagnose.`. / 注释说明了附近代码的逻辑、意图或用法：`We won't *always* want to diagnose.`。
- **L207**: Comment explains nearby logic, intent, or usage: `We might have a suffix that is already uppercase.`. / 注释说明了附近代码的逻辑、意图或用法：`We might have a suffix that is already uppercase.`。
- **L208**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。

### Lines 209-222 / 第 209-222 行

```cpp
209 |           *Literal, IsInteger ? IntegerParameters : FloatParameters,
210 |           NewSuffixes, *Result.SourceManager, getLangOpts())) {
211 |     if (Details->LiteralLocation.getBegin().isMacroID() && IgnoreMacros)
212 |       return;
213 |     auto Complaint = diag(Details->LiteralLocation.getBegin(),
214 |                           "%select{floating point|integer}0 literal has suffix "
215 |                           "'%1', which is not uppercase")
216 |                      << IsInteger << Details->OldSuffix;
217 |     if (Details->FixIt) // Similarly, a fix-it is not always possible.
218 |       Complaint << *(Details->FixIt);
219 |   }
220 | }
221 | 
222 | } // namespace clang::tidy::readability
```

- **L209**: Comment explains nearby logic, intent, or usage: `Literal, IsInteger ? IntegerParameters : FloatParameters,`. / 注释说明了附近代码的逻辑、意图或用法：`Literal, IsInteger ? IntegerParameters : FloatParameters,`。
- **L210**: Starts a function, method, lambda, or structured scope: `NewSuffixes, *Result.SourceManager, getLangOpts())) {`. / 开始一个函数、方法、lambda 或结构化作用域：`NewSuffixes, *Result.SourceManager, getLangOpts())) {`。
- **L211**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L212**: Returns from the current function with `void`. / 以 `void` 从当前函数返回。
- **L213**: Builds or emits a diagnostic message that reports a detected issue. / 构造或发出诊断信息，以报告检测到的问题。
- **L214**: Continues the surrounding expression or declaration: `"%select{floating point|integer}0 literal has suffix "`. / 继续构造周围的表达式或声明：`"%select{floating point|integer}0 literal has suffix "`。
- **L215**: Continues the surrounding expression or declaration: `"'%1', which is not uppercase")`. / 继续构造周围的表达式或声明：`"'%1', which is not uppercase")`。
- **L216**: Executes a standalone statement or declaration: `<< IsInteger << Details->OldSuffix;`. / 执行一条独立语句或声明：`<< IsInteger << Details->OldSuffix;`。
- **L217**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L218**: Executes a call or declaration centered on `*`. / 执行以 `*` 为核心的调用或声明。
- **L219**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L220**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L221**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L222**: Closes a namespace scope while preserving the trailing comment: `} // namespace clang::tidy::readability`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace clang::tidy::readability`。

## Key Concepts / 关键概念

- **Readability checking / 可读性检查**:
  - **EN**: Encourages clearer control flow, naming, and source structure.
  - **CN**: 鼓励更清晰的控制流、命名与源码结构。
- **AST matcher registration / AST 匹配器注册**:
  - **EN**: Builds matcher trees that let the check find relevant AST patterns.
  - **CN**: 构建匹配器树，使检查能够找到相关 AST 模式。
- **AST matcher callbacks / AST 匹配回调**:
  - **EN**: Routes matched AST nodes into check-specific callback logic.
  - **CN**: 把匹配到的 AST 节点路由到检查特定的回调逻辑。
- **Diagnostic emission / 诊断信息发出**:
  - **EN**: Reports issues at source locations with message parameters.
  - **CN**: 在源码位置上携带消息参数报告问题。
- **Automated fix-its / 自动修复提示**:
  - **EN**: Attaches machine-applicable edits to diagnostics whenever the transformation is safe.
  - **CN**: 在转换安全时为诊断附加可由机器应用的编辑。
- **Persistent options / 持久化选项**:
  - **EN**: Saves configurable behavior so checks can be tuned from .clang-tidy.
  - **CN**: 保存可配置行为，以便从 .clang-tidy 调整检查。

## Dependencies / 依赖关系

- `UppercaseLiteralSuffixCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `../utils/ASTUtils.h`: Provides shared clang-tidy utility helpers. / 提供共享 clang-tidy 工具辅助逻辑。
- `../utils/OptionsUtils.h`: Provides shared clang-tidy utility helpers. / 提供共享 clang-tidy 工具辅助逻辑。
- `clang/AST/ASTContext.h`: Provides Clang AST node and semantic interfaces. / 提供Clang AST 节点与语义接口。
- `clang/ASTMatchers/ASTMatchFinder.h`: Provides AST matcher construction helpers. / 提供AST 匹配器构造辅助逻辑。
- `clang/Lex/Lexer.h`: Provides lexer and preprocessor interfaces. / 提供词法分析器与预处理器接口。
- `optional`: Provides C or C++ standard library facilities. / 提供C 或 C++ 标准库设施。
