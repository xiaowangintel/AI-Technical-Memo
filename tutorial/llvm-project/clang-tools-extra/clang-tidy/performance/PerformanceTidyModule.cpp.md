# PerformanceTidyModule.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang-tools-extra/clang-tidy/performance/PerformanceTidyModule.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Registers the clang-tidy module represented by `PerformanceTidyModule` and exposes its checks.
  - **CN**: 注册由 `PerformanceTidyModule` 表示的 clang-tidy 模块并暴露其中的检查。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

```cpp
 1 | //===----------------------------------------------------------------------===//
 2 | //
 3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4 | // See https://llvm.org/LICENSE.txt for license information.
 5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6 | //
 7 | //===----------------------------------------------------------------------===//
 8 | 
 9 | #include "../ClangTidy.h"
10 | #include "../ClangTidyModule.h"
11 | #include "AvoidEndlCheck.h"
12 | #include "EnumSizeCheck.h"
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L3**: Comment explains nearby logic, intent, or usage: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑、意图或用法：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic, intent, or usage: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑、意图或用法：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic, intent, or usage: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑、意图或用法：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used for visual grouping. / 分隔注释，用于视觉分组。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L8**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L9**: Includes "../ClangTidy.h" to access neighbor declarations from the same subsystem. / 引入 "../ClangTidy.h" 以使用同一子系统中的相邻声明。
- **L10**: Includes "../ClangTidyModule.h" to access neighbor declarations from the same subsystem. / 引入 "../ClangTidyModule.h" 以使用同一子系统中的相邻声明。
- **L11**: Includes "AvoidEndlCheck.h" to access local declarations from the current tool or check. / 引入 "AvoidEndlCheck.h" 以使用当前工具或检查的本地声明。
- **L12**: Includes "EnumSizeCheck.h" to access local declarations from the current tool or check. / 引入 "EnumSizeCheck.h" 以使用当前工具或检查的本地声明。

### Lines 13-24 / 第 13-24 行

```cpp
13 | #include "ForRangeCopyCheck.h"
14 | #include "ImplicitConversionInLoopCheck.h"
15 | #include "InefficientAlgorithmCheck.h"
16 | #include "InefficientStringConcatenationCheck.h"
17 | #include "InefficientVectorOperationCheck.h"
18 | #include "MoveConstArgCheck.h"
19 | #include "MoveConstructorInitCheck.h"
20 | #include "NoAutomaticMoveCheck.h"
21 | #include "NoIntToPtrCheck.h"
22 | #include "NoexceptDestructorCheck.h"
23 | #include "NoexceptMoveConstructorCheck.h"
24 | #include "NoexceptSwapCheck.h"
```

- **L13**: Includes "ForRangeCopyCheck.h" to access local declarations from the current tool or check. / 引入 "ForRangeCopyCheck.h" 以使用当前工具或检查的本地声明。
- **L14**: Includes "ImplicitConversionInLoopCheck.h" to access local declarations from the current tool or check. / 引入 "ImplicitConversionInLoopCheck.h" 以使用当前工具或检查的本地声明。
- **L15**: Includes "InefficientAlgorithmCheck.h" to access local declarations from the current tool or check. / 引入 "InefficientAlgorithmCheck.h" 以使用当前工具或检查的本地声明。
- **L16**: Includes "InefficientStringConcatenationCheck.h" to access local declarations from the current tool or check. / 引入 "InefficientStringConcatenationCheck.h" 以使用当前工具或检查的本地声明。
- **L17**: Includes "InefficientVectorOperationCheck.h" to access local declarations from the current tool or check. / 引入 "InefficientVectorOperationCheck.h" 以使用当前工具或检查的本地声明。
- **L18**: Includes "MoveConstArgCheck.h" to access local declarations from the current tool or check. / 引入 "MoveConstArgCheck.h" 以使用当前工具或检查的本地声明。
- **L19**: Includes "MoveConstructorInitCheck.h" to access local declarations from the current tool or check. / 引入 "MoveConstructorInitCheck.h" 以使用当前工具或检查的本地声明。
- **L20**: Includes "NoAutomaticMoveCheck.h" to access local declarations from the current tool or check. / 引入 "NoAutomaticMoveCheck.h" 以使用当前工具或检查的本地声明。
- **L21**: Includes "NoIntToPtrCheck.h" to access local declarations from the current tool or check. / 引入 "NoIntToPtrCheck.h" 以使用当前工具或检查的本地声明。
- **L22**: Includes "NoexceptDestructorCheck.h" to access local declarations from the current tool or check. / 引入 "NoexceptDestructorCheck.h" 以使用当前工具或检查的本地声明。
- **L23**: Includes "NoexceptMoveConstructorCheck.h" to access local declarations from the current tool or check. / 引入 "NoexceptMoveConstructorCheck.h" 以使用当前工具或检查的本地声明。
- **L24**: Includes "NoexceptSwapCheck.h" to access local declarations from the current tool or check. / 引入 "NoexceptSwapCheck.h" 以使用当前工具或检查的本地声明。

### Lines 25-36 / 第 25-36 行

```cpp
25 | #include "PreferSingleCharOverloadsCheck.h"
26 | #include "StringViewConversionsCheck.h"
27 | #include "TriviallyDestructibleCheck.h"
28 | #include "TypePromotionInMathFnCheck.h"
29 | #include "UnnecessaryCopyInitializationCheck.h"
30 | #include "UnnecessaryValueParamCheck.h"
31 | #include "UseStdMoveCheck.h"
32 | 
33 | namespace clang::tidy {
34 | namespace performance {
35 | namespace {
36 | 
```

- **L25**: Includes "PreferSingleCharOverloadsCheck.h" to access local declarations from the current tool or check. / 引入 "PreferSingleCharOverloadsCheck.h" 以使用当前工具或检查的本地声明。
- **L26**: Includes "StringViewConversionsCheck.h" to access local declarations from the current tool or check. / 引入 "StringViewConversionsCheck.h" 以使用当前工具或检查的本地声明。
- **L27**: Includes "TriviallyDestructibleCheck.h" to access local declarations from the current tool or check. / 引入 "TriviallyDestructibleCheck.h" 以使用当前工具或检查的本地声明。
- **L28**: Includes "TypePromotionInMathFnCheck.h" to access local declarations from the current tool or check. / 引入 "TypePromotionInMathFnCheck.h" 以使用当前工具或检查的本地声明。
- **L29**: Includes "UnnecessaryCopyInitializationCheck.h" to access local declarations from the current tool or check. / 引入 "UnnecessaryCopyInitializationCheck.h" 以使用当前工具或检查的本地声明。
- **L30**: Includes "UnnecessaryValueParamCheck.h" to access local declarations from the current tool or check. / 引入 "UnnecessaryValueParamCheck.h" 以使用当前工具或检查的本地声明。
- **L31**: Includes "UseStdMoveCheck.h" to access local declarations from the current tool or check. / 引入 "UseStdMoveCheck.h" 以使用当前工具或检查的本地声明。
- **L32**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L33**: Opens namespace scope `clang::tidy`. / 打开命名空间作用域 `clang::tidy`。
- **L34**: Opens namespace scope `performance`. / 打开命名空间作用域 `performance`。
- **L35**: Opens an anonymous namespace for internal linkage. / 打开匿名命名空间以提供内部链接。
- **L36**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。

### Lines 37-48 / 第 37-48 行

```cpp
37 | class PerformanceModule : public ClangTidyModule {
38 | public:
39 |   void addCheckFactories(ClangTidyCheckFactories &CheckFactories) override {
40 |     CheckFactories.registerCheck<AvoidEndlCheck>("performance-avoid-endl");
41 |     CheckFactories.registerCheck<EnumSizeCheck>("performance-enum-size");
42 |     CheckFactories.registerCheck<PreferSingleCharOverloadsCheck>(
43 |         "performance-faster-string-find");
44 |     CheckFactories.registerCheck<ForRangeCopyCheck>(
45 |         "performance-for-range-copy");
46 |     CheckFactories.registerCheck<ImplicitConversionInLoopCheck>(
47 |         "performance-implicit-conversion-in-loop");
48 |     CheckFactories.registerCheck<InefficientAlgorithmCheck>(
```

- **L37**: Declares class `PerformanceModule`. / 声明类 `PerformanceModule`。
- **L38**: Sets the following members to `public` access. / 将后续成员的访问级别设为 `public`。
- **L39**: Starts a function, method, lambda, or structured scope: `void addCheckFactories(ClangTidyCheckFactories &CheckFactories) override {`. / 开始一个函数、方法、lambda 或结构化作用域：`void addCheckFactories(ClangTidyCheckFactories &CheckFactories) override {`。
- **L40**: Executes a call or declaration centered on `CheckFactories.registerCheck<AvoidEndlCheck>`. / 执行以 `CheckFactories.registerCheck<AvoidEndlCheck>` 为核心的调用或声明。
- **L41**: Executes a call or declaration centered on `CheckFactories.registerCheck<EnumSizeCheck>`. / 执行以 `CheckFactories.registerCheck<EnumSizeCheck>` 为核心的调用或声明。
- **L42**: Continues logic associated with callable symbol `registerCheck<PreferSingleCharOverloadsCheck>`. / 继续与可调用符号 `registerCheck<PreferSingleCharOverloadsCheck>` 相关的逻辑。
- **L43**: Executes a standalone statement or declaration: `"performance-faster-string-find");`. / 执行一条独立语句或声明：`"performance-faster-string-find");`。
- **L44**: Continues logic associated with callable symbol `registerCheck<ForRangeCopyCheck>`. / 继续与可调用符号 `registerCheck<ForRangeCopyCheck>` 相关的逻辑。
- **L45**: Executes a standalone statement or declaration: `"performance-for-range-copy");`. / 执行一条独立语句或声明：`"performance-for-range-copy");`。
- **L46**: Continues logic associated with callable symbol `registerCheck<ImplicitConversionInLoopCheck>`. / 继续与可调用符号 `registerCheck<ImplicitConversionInLoopCheck>` 相关的逻辑。
- **L47**: Executes a standalone statement or declaration: `"performance-implicit-conversion-in-loop");`. / 执行一条独立语句或声明：`"performance-implicit-conversion-in-loop");`。
- **L48**: Continues logic associated with callable symbol `registerCheck<InefficientAlgorithmCheck>`. / 继续与可调用符号 `registerCheck<InefficientAlgorithmCheck>` 相关的逻辑。

### Lines 49-60 / 第 49-60 行

```cpp
49 |         "performance-inefficient-algorithm");
50 |     CheckFactories.registerCheck<InefficientStringConcatenationCheck>(
51 |         "performance-inefficient-string-concatenation");
52 |     CheckFactories.registerCheck<InefficientVectorOperationCheck>(
53 |         "performance-inefficient-vector-operation");
54 |     CheckFactories.registerCheck<MoveConstArgCheck>(
55 |         "performance-move-const-arg");
56 |     CheckFactories.registerCheck<MoveConstructorInitCheck>(
57 |         "performance-move-constructor-init");
58 |     CheckFactories.registerCheck<NoAutomaticMoveCheck>(
59 |         "performance-no-automatic-move");
60 |     CheckFactories.registerCheck<NoIntToPtrCheck>("performance-no-int-to-ptr");
```

- **L49**: Executes a standalone statement or declaration: `"performance-inefficient-algorithm");`. / 执行一条独立语句或声明：`"performance-inefficient-algorithm");`。
- **L50**: Continues logic associated with callable symbol `registerCheck<InefficientStringConcatenationCheck>`. / 继续与可调用符号 `registerCheck<InefficientStringConcatenationCheck>` 相关的逻辑。
- **L51**: Executes a standalone statement or declaration: `"performance-inefficient-string-concatenation");`. / 执行一条独立语句或声明：`"performance-inefficient-string-concatenation");`。
- **L52**: Continues logic associated with callable symbol `registerCheck<InefficientVectorOperationCheck>`. / 继续与可调用符号 `registerCheck<InefficientVectorOperationCheck>` 相关的逻辑。
- **L53**: Executes a standalone statement or declaration: `"performance-inefficient-vector-operation");`. / 执行一条独立语句或声明：`"performance-inefficient-vector-operation");`。
- **L54**: Continues logic associated with callable symbol `registerCheck<MoveConstArgCheck>`. / 继续与可调用符号 `registerCheck<MoveConstArgCheck>` 相关的逻辑。
- **L55**: Executes a standalone statement or declaration: `"performance-move-const-arg");`. / 执行一条独立语句或声明：`"performance-move-const-arg");`。
- **L56**: Continues logic associated with callable symbol `registerCheck<MoveConstructorInitCheck>`. / 继续与可调用符号 `registerCheck<MoveConstructorInitCheck>` 相关的逻辑。
- **L57**: Executes a standalone statement or declaration: `"performance-move-constructor-init");`. / 执行一条独立语句或声明：`"performance-move-constructor-init");`。
- **L58**: Continues logic associated with callable symbol `registerCheck<NoAutomaticMoveCheck>`. / 继续与可调用符号 `registerCheck<NoAutomaticMoveCheck>` 相关的逻辑。
- **L59**: Executes a standalone statement or declaration: `"performance-no-automatic-move");`. / 执行一条独立语句或声明：`"performance-no-automatic-move");`。
- **L60**: Executes a call or declaration centered on `CheckFactories.registerCheck<NoIntToPtrCheck>`. / 执行以 `CheckFactories.registerCheck<NoIntToPtrCheck>` 为核心的调用或声明。

### Lines 61-72 / 第 61-72 行

```cpp
61 |     CheckFactories.registerCheck<NoexceptDestructorCheck>(
62 |         "performance-noexcept-destructor");
63 |     CheckFactories.registerCheck<NoexceptMoveConstructorCheck>(
64 |         "performance-noexcept-move-constructor");
65 |     CheckFactories.registerCheck<NoexceptSwapCheck>(
66 |         "performance-noexcept-swap");
67 |     CheckFactories.registerCheck<PreferSingleCharOverloadsCheck>(
68 |         "performance-prefer-single-char-overloads");
69 |     CheckFactories.registerCheck<StringViewConversionsCheck>(
70 |         "performance-string-view-conversions");
71 |     CheckFactories.registerCheck<TriviallyDestructibleCheck>(
72 |         "performance-trivially-destructible");
```

- **L61**: Continues logic associated with callable symbol `registerCheck<NoexceptDestructorCheck>`. / 继续与可调用符号 `registerCheck<NoexceptDestructorCheck>` 相关的逻辑。
- **L62**: Executes a standalone statement or declaration: `"performance-noexcept-destructor");`. / 执行一条独立语句或声明：`"performance-noexcept-destructor");`。
- **L63**: Continues logic associated with callable symbol `registerCheck<NoexceptMoveConstructorCheck>`. / 继续与可调用符号 `registerCheck<NoexceptMoveConstructorCheck>` 相关的逻辑。
- **L64**: Executes a standalone statement or declaration: `"performance-noexcept-move-constructor");`. / 执行一条独立语句或声明：`"performance-noexcept-move-constructor");`。
- **L65**: Continues logic associated with callable symbol `registerCheck<NoexceptSwapCheck>`. / 继续与可调用符号 `registerCheck<NoexceptSwapCheck>` 相关的逻辑。
- **L66**: Executes a standalone statement or declaration: `"performance-noexcept-swap");`. / 执行一条独立语句或声明：`"performance-noexcept-swap");`。
- **L67**: Continues logic associated with callable symbol `registerCheck<PreferSingleCharOverloadsCheck>`. / 继续与可调用符号 `registerCheck<PreferSingleCharOverloadsCheck>` 相关的逻辑。
- **L68**: Executes a standalone statement or declaration: `"performance-prefer-single-char-overloads");`. / 执行一条独立语句或声明：`"performance-prefer-single-char-overloads");`。
- **L69**: Continues logic associated with callable symbol `registerCheck<StringViewConversionsCheck>`. / 继续与可调用符号 `registerCheck<StringViewConversionsCheck>` 相关的逻辑。
- **L70**: Executes a standalone statement or declaration: `"performance-string-view-conversions");`. / 执行一条独立语句或声明：`"performance-string-view-conversions");`。
- **L71**: Continues logic associated with callable symbol `registerCheck<TriviallyDestructibleCheck>`. / 继续与可调用符号 `registerCheck<TriviallyDestructibleCheck>` 相关的逻辑。
- **L72**: Executes a standalone statement or declaration: `"performance-trivially-destructible");`. / 执行一条独立语句或声明：`"performance-trivially-destructible");`。

### Lines 73-84 / 第 73-84 行

```cpp
73 |     CheckFactories.registerCheck<TypePromotionInMathFnCheck>(
74 |         "performance-type-promotion-in-math-fn");
75 |     CheckFactories.registerCheck<UnnecessaryCopyInitializationCheck>(
76 |         "performance-unnecessary-copy-initialization");
77 |     CheckFactories.registerCheck<UnnecessaryValueParamCheck>(
78 |         "performance-unnecessary-value-param");
79 |     CheckFactories.registerCheck<UseStdMoveCheck>("performance-use-std-move");
80 |   }
81 | };
82 | 
83 | } // namespace
84 | 
```

- **L73**: Continues logic associated with callable symbol `registerCheck<TypePromotionInMathFnCheck>`. / 继续与可调用符号 `registerCheck<TypePromotionInMathFnCheck>` 相关的逻辑。
- **L74**: Executes a standalone statement or declaration: `"performance-type-promotion-in-math-fn");`. / 执行一条独立语句或声明：`"performance-type-promotion-in-math-fn");`。
- **L75**: Continues logic associated with callable symbol `registerCheck<UnnecessaryCopyInitializationCheck>`. / 继续与可调用符号 `registerCheck<UnnecessaryCopyInitializationCheck>` 相关的逻辑。
- **L76**: Executes a standalone statement or declaration: `"performance-unnecessary-copy-initialization");`. / 执行一条独立语句或声明：`"performance-unnecessary-copy-initialization");`。
- **L77**: Continues logic associated with callable symbol `registerCheck<UnnecessaryValueParamCheck>`. / 继续与可调用符号 `registerCheck<UnnecessaryValueParamCheck>` 相关的逻辑。
- **L78**: Executes a standalone statement or declaration: `"performance-unnecessary-value-param");`. / 执行一条独立语句或声明：`"performance-unnecessary-value-param");`。
- **L79**: Executes a call or declaration centered on `CheckFactories.registerCheck<UseStdMoveCheck>`. / 执行以 `CheckFactories.registerCheck<UseStdMoveCheck>` 为核心的调用或声明。
- **L80**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L81**: Closes the current declaration scope such as a class, struct, or enum. / 结束当前声明作用域，例如类、结构体或枚举。
- **L82**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L83**: Closes a namespace scope while preserving the trailing comment: `} // namespace`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L84**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。

### Lines 85-96 / 第 85-96 行

```cpp
85 | // Register the PerformanceModule using this statically initialized variable.
86 | static ClangTidyModuleRegistry::Add<PerformanceModule>
87 |     X("performance-module", "Adds performance checks.");
88 | 
89 | } // namespace performance
90 | 
91 | // This anchor is used to force the linker to link in the generated object file
92 | // and thus register the PerformanceModule.
93 | // NOLINTNEXTLINE(misc-use-internal-linkage)
94 | volatile int PerformanceModuleAnchorSource = 0;
95 | 
96 | } // namespace clang::tidy
```

- **L85**: Comment explains nearby logic, intent, or usage: `Register the PerformanceModule using this statically initialized variable.`. / 注释说明了附近代码的逻辑、意图或用法：`Register the PerformanceModule using this statically initialized variable.`。
- **L86**: Continues the surrounding expression or declaration: `static ClangTidyModuleRegistry::Add<PerformanceModule>`. / 继续构造周围的表达式或声明：`static ClangTidyModuleRegistry::Add<PerformanceModule>`。
- **L87**: Executes a call or declaration centered on `X`. / 执行以 `X` 为核心的调用或声明。
- **L88**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L89**: Closes a namespace scope while preserving the trailing comment: `} // namespace performance`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace performance`。
- **L90**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L91**: Comment explains nearby logic, intent, or usage: `This anchor is used to force the linker to link in the generated object file`. / 注释说明了附近代码的逻辑、意图或用法：`This anchor is used to force the linker to link in the generated object file`。
- **L92**: Comment explains nearby logic, intent, or usage: `and thus register the PerformanceModule.`. / 注释说明了附近代码的逻辑、意图或用法：`and thus register the PerformanceModule.`。
- **L93**: Comment explains nearby logic, intent, or usage: `NOLINTNEXTLINE(misc-use-internal-linkage)`. / 注释说明了附近代码的逻辑、意图或用法：`NOLINTNEXTLINE(misc-use-internal-linkage)`。
- **L94**: Initializes variable `PerformanceModuleAnchorSource` from the right-hand expression. / 使用右侧表达式初始化变量 `PerformanceModuleAnchorSource`。
- **L95**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻声明或逻辑块。
- **L96**: Closes a namespace scope while preserving the trailing comment: `} // namespace clang::tidy`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace clang::tidy`。

## Key Concepts / 关键概念

- **Performance diagnostics / 性能诊断**:
  - **EN**: Looks for unnecessary copies, allocations, conversions, or other avoidable costs.
  - **CN**: 查找不必要的拷贝、分配、转换或其他可避免的开销。

## Dependencies / 依赖关系

- `../ClangTidy.h`: Provides neighbor declarations from the same subsystem. / 提供同一子系统中的相邻声明。
- `../ClangTidyModule.h`: Provides neighbor declarations from the same subsystem. / 提供同一子系统中的相邻声明。
- `AvoidEndlCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `EnumSizeCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `ForRangeCopyCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `ImplicitConversionInLoopCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `InefficientAlgorithmCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `InefficientStringConcatenationCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `InefficientVectorOperationCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `MoveConstArgCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `MoveConstructorInitCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `NoAutomaticMoveCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `NoIntToPtrCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `NoexceptDestructorCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `NoexceptMoveConstructorCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `NoexceptSwapCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `PreferSingleCharOverloadsCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `StringViewConversionsCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `TriviallyDestructibleCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `TypePromotionInMathFnCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `UnnecessaryCopyInitializationCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `UnnecessaryValueParamCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
- `UseStdMoveCheck.h`: Provides local declarations from the current tool or check. / 提供当前工具或检查的本地声明。
