# programming_guidelines.md — Documentation Analysis / 文档分析

## Source / 来源
- **Path:** /root/xw/sycl-tla/media/docs/cpp/programming_guidelines.md
- **Purpose:** Defines CUTLASS design patterns, coding style, warning policy, and CUDA/C++ idioms for contributors. / 为贡献者定义 CUTLASS 的设计模式、编码风格、告警策略以及 CUDA/C++ 习惯用法。

## Content Analysis / 内容分析
### Hierarchical Organization
**EN:** Opens by linking CUTLASS 3.0 organization to the GEMM API hierarchy. It contrasts the 3.x conceptual parallelization model with the more hardware-shaped 2.x organization, which frames the rest of the style advice.
**CN:** 开篇把 CUTLASS 3.0 的组织方式与 GEMM API 层级联系起来，并将 3.x 的概念性并行化模型与更贴近硬件结构的 2.x 组织方式做对比，为后续风格与设计建议奠定背景。

### Design Patterns
**EN:** Sets the governing principle: CUTLASS favors performance over simplicity when the two conflict. The rest of the section explains the recurring implementation patterns that preserve composability without giving up speed.
**CN:** 确立总原则：当性能与简洁冲突时，CUTLASS 优先选择性能。接下来的内容解释了那些既保持可组合性、又不牺牲速度的常见实现模式。

### Templates
**EN:** Explains why templates are central: they encode the design space, capture compile-time constants, and enable loop unrolling, inlining, and specialization across data types, layouts, and fused kernels.
**CN:** 解释为什么模板是核心机制：它们用于表达设计空间、承载编译期常量，并支持针对数据类型、布局与融合内核的循环展开、内联和特化。

### Constant Memory
**EN:** Describes the Params pattern for grid-invariant state. The idea is to construct launch-time state on the host, store it in constant memory, and avoid redundant per-thread setup inside the kernel.
**CN:** 说明用于网格不变量状态的 Params 模式。核心思想是在宿主侧构造启动时状态，把它放入常量内存，并避免在 kernel 内部由每个线程重复初始化。

### Composable Shared Memory
**EN:** Introduces the SharedStorage pattern for composing shared-memory needs across nested components. It also warns about careful union usage and the ABI implications of inheritance, especially for host-device compatibility.
**CN:** 介绍用 SharedStorage 组织嵌套组件共享内存需求的模式。同时提醒开发者谨慎使用 union，并注意继承对 host-device ABI 兼容性的影响。

### Loop Unrolling
**EN:** Motivates compile-time loop trip counts as a route to register-resident tiles and better instruction scheduling. The explicit use of CUTLASS_PRAGMA_UNROLL is treated as a deliberate performance signal, not cosmetic syntax.
**CN:** 说明为何需要编译期已知的循环次数：这样更容易把 tile 留在寄存器中，并获得更好的指令调度。显式使用 CUTLASS_PRAGMA_UNROLL 被视为性能意图的表达，而不是语法装饰。

### Style: formatting policy
**EN:** The subsections If you see an issue in code formatting, fix it and No automatic code formatting together define a manual formatting policy: developers may improve formatting, but should not run automatic reformatters like clang-format over CUTLASS code.
**CN:** If you see an issue in code formatting, fix it 与 No automatic code formatting 两个小节共同定义了“手工格式化”策略：开发者可以改进排版，但不应对 CUTLASS 代码直接运行 clang-format 一类自动工具。

### Style: C++ foundations and formatting details
**EN:** The subsections CUTLASS is a C++ project, Follow Standard C++ idioms where possible, C is not a subset of C++, Spacing and line length, Formatting function declarations and definitions, Formatting function calls, and If-else brackets and spacing establish the baseline language identity and formatting discipline.
**CN:** CUTLASS is a C++ project、Follow Standard C++ idioms where possible、C is not a subset of C++、Spacing and line length、Formatting function declarations and definitions、Formatting function calls 以及 If-else brackets and spacing 这些小节共同确立了语言定位与格式纪律。

### Style: types, naming, and interfaces
**EN:** The subsections East const, Alignment of reference and pointer types, Avoid calling functions fast or optimized, and Avoid creating unconstrained templated functions with common names favor explicit, stable, and low-surprise interfaces. This is not just style; it is also about API safety and long-term maintainability.
**CN:** East const、Alignment of reference and pointer types、Avoid calling functions fast or optimized，以及 Avoid creating unconstrained templated functions with common names 这些小节鼓励显式、稳定、低意外性的接口设计。这不仅是风格问题，也关乎 API 安全性和长期可维护性。

### Style: return values and error reporting
**EN:** The group beginning with Function return values and in-out parameters argues for return values over mutable output parameters, recommends struct or tuple for multiple results, discusses optional/variant/expected-like strategies for error handling, and prefers aggregate initialization plus explicit return types where practical.
**CN:** 以 Function return values and in-out parameters 为起点的一组小节，主张用返回值替代可变输出参数，建议通过 struct 或 tuple 返回多个结果，并讨论 optional、variant、expected 类方案来表达错误，同时在可行时优先采用聚合初始化与显式返回类型。

### Style: classes, namespaces, macros, and header rules
**EN:** The subsections Classes and structs, Class members, Class Member Order, For code reuse prefer composition over inheritance, Behavioral subtyping, Use scoped enums, Namespaces, File Names, Macros, and Guard all headers with pragma once define the project's object-model and naming expectations. The composition-over-inheritance examples are especially important because they connect interface design to correctness under parallel behavior.
**CN:** Classes and structs、Class members、Class Member Order、For code reuse prefer composition over inheritance、Behavioral subtyping、Use scoped enums、Namespaces、File Names、Macros 与 Guard all headers with pragma once 这些小节共同定义了项目的对象模型与命名预期。其中“优先组合而非继承”的示例尤其重要，因为它把接口设计与并行行为下的正确性直接联系起来。

### CuTe Layout Comments
**EN:** This section may look minor, but it captures a documentation norm for tensor-heavy code: layout comments should be aligned and readable so that shape and stride information can be scanned quickly.
**CN:** 这一节看似细节，但它实际上规定了张量密集型代码的文档化规范：布局注释应尽量对齐、可读，从而让 shape 与 stride 信息可以被快速扫读。

### Warnings
**EN:** The warning policy aims for warning-free builds while recognizing that some compiler diagnostics are spurious. The sections on missing return statements and unused variables explain when targeted suppression helpers like CUTE_GCC_UNREACHABLE or maybe_unused are justified.
**CN:** 告警策略的目标是尽量做到无告警构建，同时承认有些编译器诊断属于误报。关于缺失 return 和未使用变量的部分解释了何时可以合理使用 CUTE_GCC_UNREACHABLE 或 maybe_unused 这类定向抑制手段。

### CUDA C++ style
**EN:** The CUDA-specific guidance says reusable components should avoid direct dependence on built-in thread/block variables and should instead rely on passed-in linear identifiers. It also encourages consistent use of CUTLASS and CuTe fundamental types instead of ad hoc replacements.
**CN:** CUDA 专项风格建议指出：可复用组件应避免直接依赖 threadIdx 等内建变量，而应使用上传入的线性标识符。同时鼓励统一采用 CUTLASS 与 CuTe 的基础类型与操作，而不是临时造轮子。

### CUTLASS idioms
**EN:** The Detecting major mode subsection gives a concrete example of an approved idiom: use helper traits such as cutlass::detail::is_major instead of making brittle assumptions like stride component equals 1.
**CN:** Detecting major mode 小节给出了一个具体的推荐习惯用法：使用 cutlass::detail::is_major 之类的辅助 traits，而不是做“stride 某一项等于 1”这类脆弱假设。

### Copyright
**EN:** Ends with standard BSD-3-Clause license text.
**CN:** 结尾是标准 BSD-3-Clause 许可证文本。

## Key Concepts / 关键概念
- Performance-oriented design patterns / 以性能为导向的设计模式
- Manual but disciplined code style / 手工但严格的代码风格
- Composition over inheritance / 组合优于继承
- Warning hygiene and targeted suppression / 告警治理与定向抑制
- CUDA-friendly reusable interfaces / 适合 CUDA 的可复用接口
- Preferred CUTLASS coding idioms / 推荐的 CUTLASS 编码习惯

## Related Files / 相关文件
- /root/xw/sycl-tla/media/docs/cpp/code_organization.md — Repository-structure document referenced directly by these guidelines / 这些规范直接关联的仓库结构文档
- /root/xw/sycl-tla/media/docs/cpp/getting_started.rst — Onboarding page that routes contributors toward the guidelines / 把贡献者引导到本规范的入门页面
- /root/xw/sycl-tla/media/docs/cpp/quickstart.md — Practical guide that benefits from the conventions defined here / 会直接受益于这些规范的实践指南
