# LLVM Project Architecture Tutorial / LLVM 项目架构双语教程

This tutorial is based primarily on official in-repo sources, especially `README.md`, `llvm/docs/`, `clang/docs/`, `mlir/docs/`, and the documentation under `lld/`, `lldb/`, `compiler-rt/`, `libcxx/`, `libc/`, `flang/`, `polly/`, `bolt/`, and `clang-tools-extra/`.

本教程主要依据仓库内官方文档与 README 编写，重点参考 `README.md`、`llvm/docs/`、`clang/docs/`、`mlir/docs/`，以及 `lld/`、`lldb/`、`compiler-rt/`、`libcxx/`、`libc/`、`flang/`、`polly/`、`bolt/`、`clang-tools-extra/` 下的子项目文档。

## Table of Contents / 目录

- 1. Introduction / 简介

- 2. Architecture Overview / 架构概览

- 3. LLVM IR / LLVM 中间表示

- 4. LLVM Core Libraries / LLVM 核心库

- 5. Frontend: Clang / 前端：Clang

- 6. Optimization Passes / 优化遍

- 7. Code Generation / 代码生成

- 8. Target Backends / 目标后端

- 9. TableGen / TableGen

- 10. MLIR / 多级中间表示

- 11. LLD Linker / LLD 链接器

- 12. LLDB Debugger / LLDB 调试器

- 13. Compiler-RT / 编译器运行时

- 14. libc++ / C++ 标准库

- 15. libc / C 标准库

- 16. Flang / Fortran 编译器

- 17. Polly / 多面体优化

- 18. BOLT / 二进制优化

- 19. Clang Tools / Clang 工具

- 20. Build System / 构建系统

- 21. Testing Infrastructure / 测试基础设施

- 22. Developer Guide / 开发者指南

## 1. Introduction / 简介

### EN

LLVM is a reusable toolchain ecosystem collected in one monorepo: compilers, IR infrastructure, runtimes, linkers, debuggers, and developer tools evolve together but keep clear subsystem boundaries.

**Core ideas**

- Start with `README.md` and `llvm/docs/index.rst`; they frame the repository as a family of libraries and products.

- The central mental model is pipeline plus reuse: frontends produce IR, passes transform it, backends lower it, and surrounding tools finish the job.

- Most binaries in the tree are thin product layers over reusable libraries.

- The directory layout itself is part of the documentation because code, docs, tests, and build logic live close together.

**Key directories and files**

- `README.md`: umbrella project introduction and common build entry point.

- `llvm/docs/`: central LLVM documentation hub.

- `clang/`, `mlir/`, `lld/`, `lldb/`: major subprojects to study beside core LLVM.

- `runtimes/`: build-time coordination point for runtime libraries.

**Primary docs**

- `README.md`: broad scope and project map.

- `llvm/docs/ProgrammersManual.rst`: practical coding idioms used across the tree.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
sed -n '1,80p' README.md
sed -n '1,120p' llvm/docs/index.rst
ls -1 clang mlir lld lldb compiler-rt libcxx libc flang polly bolt clang-tools-extra
```

**Takeaways**

- Treat LLVM as an ecosystem of layers, not a single compiler.

- When in doubt, ask which layer owns the problem you are looking at.

### 中文

LLVM 是一个汇集在单仓库中的可复用工具链生态：编译器、IR 基础设施、运行时、链接器、调试器和开发者工具一起演化，但仍保持清晰的子系统边界。

**核心理解**

- 先看 `README.md` 与 `llvm/docs/index.rst`，它们把仓库定位为“库与产品的家族”。

- 最重要的心智模型是“流水线 + 复用”：前端产出 IR，pass 变换 IR，后端 lowering，外围工具完成剩余工作。

- 树中的大多数可执行程序，本质上都是可复用库之上的产品层。

- 目录布局本身就是文档的一部分，因为代码、文档、测试与构建逻辑都放得很近。

**关键目录与文件**

- `README.md`：总项目介绍与通用构建入口。

- `llvm/docs/`：LLVM 核心文档中心。

- `clang/`、`mlir/`、`lld/`、`lldb/`：除 LLVM 核心之外最值得先看的大子项目。

- `runtimes/`：运行时库的构建协调位置。

**主要文档**

- `README.md`：项目范围与总体地图。

- `llvm/docs/ProgrammersManual.rst`：贯穿全树的实用编码习惯。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
sed -n '1,80p' README.md
sed -n '1,120p' llvm/docs/index.rst
ls -1 clang mlir lld lldb compiler-rt libcxx libc flang polly bolt clang-tools-extra
```

**结论**

- 把 LLVM 看成分层生态，而不是单一编译器。

- 如果迷路，先问自己：当前问题属于哪一层。

## 2. Architecture Overview / 架构概览

### EN

The monorepo is organized around shared compiler infrastructure: `llvm/` sits at the center, frontends and higher-level IR projects feed it, and toolchain consumers such as linkers, debuggers, and runtimes complete the end-to-end story.

**Core ideas**

- `llvm/` owns the reusable substrate: IR, analysis, passes, codegen, target, MC, support, and utilities.

- `clang/`, `flang/`, and `mlir/` show different ways to map source or domain concepts onto lower-level representations.

- `lld/` and `lldb/` live downstream, but they still reuse LLVM libraries and data structures heavily.

- The build system keeps products configurable while preserving library boundaries.

**Key directories and files**

- `llvm/include/llvm/` and `llvm/lib/`: public interfaces and implementations for the core stack.

- `clang/lib/`: frontend pipeline layers such as driver, parse, sema, AST, and codegen.

- `mlir/lib/`: multi-level IR infrastructure and dialect implementations.

- `llvm/cmake/modules/`: integration layer that turns architecture into buildable targets.

**Primary docs**

- `llvm/docs/index.rst`: map of major LLVM subsystems.

- `clang/docs/InternalsManual.rst`: frontend-oriented architecture view.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
find llvm/lib -maxdepth 1 -mindepth 1 -type d | sort | head -20
find clang/lib -maxdepth 1 -mindepth 1 -type d | sort | head -20
find mlir/lib -maxdepth 1 -mindepth 1 -type d | sort | head -20
```

**Takeaways**

- Follow one artifact across layers to build intuition.

- Architecture boundaries are often visible first in directories and docs, then in code.

### 中文

单仓库围绕共享编译器基础设施组织：`llvm/` 位于中心，前端和更高层 IR 项目向它输入内容，而链接器、调试器和运行时等工具链消费者则补齐端到端故事。

**核心理解**

- `llvm/` 管理可复用地基：IR、analysis、pass、codegen、target、MC、support 与工具库。

- `clang/`、`flang/` 与 `mlir/` 展示了把源语言或领域概念映射到低层表示的不同路径。

- `lld/` 与 `lldb/` 位于下游，但仍大量复用 LLVM 库与数据结构。

- 构建系统在保持库边界的同时，也让最终产品保持高度可配置。

**关键目录与文件**

- `llvm/include/llvm/` 与 `llvm/lib/`：核心栈的公共接口与实现。

- `clang/lib/`：driver、parse、sema、AST、codegen 等前端流水线层。

- `mlir/lib/`：多级 IR 基础设施与 dialect 实现。

- `llvm/cmake/modules/`：把架构转换为可构建目标的集成层。

**主要文档**

- `llvm/docs/index.rst`：LLVM 主要子系统地图。

- `clang/docs/InternalsManual.rst`：偏前端视角的架构说明。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
find llvm/lib -maxdepth 1 -mindepth 1 -type d | sort | head -20
find clang/lib -maxdepth 1 -mindepth 1 -type d | sort | head -20
find mlir/lib -maxdepth 1 -mindepth 1 -type d | sort | head -20
```

**结论**

- 跟踪一个工件跨层流动，是建立直觉的最快方式。

- 架构边界常常先在目录与文档里出现，然后才在代码中体现。

## 3. LLVM IR / LLVM 中间表示

### EN

LLVM IR is the project’s central compiler contract: typed, SSA-based, serializable, and rich enough to support both aggressive optimization and target-sensitive lowering.

**Core ideas**

- `llvm/docs/LangRef.rst` is the normative source for syntax and semantics.

- The same IR exists as in-memory objects, textual `.ll`, and bitcode `.bc`.

- Verifier rules are architectural invariants; passes are expected to preserve them.

- IR is language-neutral, but datalayout, triples, intrinsics, and metadata let it carry machine-relevant information.

**Key directories and files**

- `llvm/include/llvm/IR/`: `Module`, `Function`, `BasicBlock`, `Instruction`, types, attributes, and metadata APIs.

- `llvm/lib/IR/`: implementation of the in-memory IR model.

- `llvm/lib/Bitcode/`: serialization layer for bitcode IO and LTO flows.

- `llvm/lib/IR/Verifier.cpp`: concrete place to study preserved invariants.

**Primary docs**

- `llvm/docs/LangRef.rst`: authoritative IR reference.

- `llvm/docs/Passes.rst`: shows what many passes expect from IR.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
grep -n 'Introduction' llvm/docs/LangRef.rst | head
ls llvm/include/llvm/IR | head -20
grep -n 'Verifier' llvm/lib/IR/Verifier.cpp | head -20
```

**Takeaways**

- If you can read `.ll`, much of LLVM becomes easier.

- Verifier failures usually point to real transformation mistakes.

### 中文

LLVM IR 是整个项目最核心的编译器契约：它带类型、基于 SSA、可序列化，并且足够丰富，既能支撑激进优化，也能支撑目标相关 lowering。

**核心理解**

- `llvm/docs/LangRef.rst` 是语法与语义的规范来源。

- 同一个 IR 可以以内存对象、文本 `.ll` 或 bitcode `.bc` 形式存在。

- Verifier 规则就是架构不变量；pass 必须保持它们。

- IR 虽然语言中立，但 datalayout、triple、intrinsic 与 metadata 让它能携带机器相关信息。

**关键目录与文件**

- `llvm/include/llvm/IR/`：`Module`、`Function`、`BasicBlock`、`Instruction`、类型、属性与元数据 API。

- `llvm/lib/IR/`：内存 IR 模型实现。

- `llvm/lib/Bitcode/`：bitcode 读写与 LTO 序列化层。

- `llvm/lib/IR/Verifier.cpp`：学习 IR 不变量的具体入口。

**主要文档**

- `llvm/docs/LangRef.rst`：IR 的权威参考。

- `llvm/docs/Passes.rst`：展示许多 pass 对 IR 的假设。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
grep -n 'Introduction' llvm/docs/LangRef.rst | head
ls llvm/include/llvm/IR | head -20
grep -n 'Verifier' llvm/lib/IR/Verifier.cpp | head -20
```

**结论**

- 只要能熟练读 `.ll`，很多 LLVM 内容都会容易很多。

- Verifier 失败通常意味着真实的变换错误。

## 4. LLVM Core Libraries / LLVM 核心库

### EN

The core libraries make LLVM reusable: generic containers, platform support, IR classes, analysis frameworks, and pass infrastructure are factored into composable layers instead of one fixed compiler body.

**Core ideas**

- `llvm/include/llvm/ADT/` provides ubiquitous utility types such as `SmallVector`, `DenseMap`, and `StringRef`.

- `llvm/lib/Support/` holds OS abstraction, filesystems, errors, command-line parsing, and low-level services.

- `llvm/lib/Analysis/` captures reusable compiler facts like dominators and loop information.

- `llvm/lib/Passes/` and related headers encode how optimizations are composed.

**Key directories and files**

- `llvm/include/llvm/ADT/`: generic data structures used across subprojects.

- `llvm/lib/Support/`: portable systems programming substrate.

- `llvm/include/llvm/IR/` and `llvm/lib/IR/`: central program representation layer.

- `llvm/lib/Analysis/` and `llvm/lib/Passes/`: reusable reasoning plus scheduling glue.

**Primary docs**

- `llvm/docs/ProgrammersManual.rst`: daily-use idioms for these libraries.

- `llvm/docs/ExtendingLLVM.rst`: where new features plug into existing layers.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
ls llvm/include/llvm/ADT | head -20
ls llvm/lib/Support | head -20
ls llvm/lib/Analysis llvm/lib/Passes
```

**Takeaways**

- Study ADT and Support early; they lower the cognitive cost of the rest of the tree.

- Most higher-level projects are domain logic on top of these layers.

### 中文

LLVM 核心库让“可复用”成为现实：通用容器、平台支持、IR 类、分析框架与 pass 基础设施被拆成可组合层，而不是固定单体编译器。

**核心理解**

- `llvm/include/llvm/ADT/` 提供无处不在的工具类型，如 `SmallVector`、`DenseMap` 与 `StringRef`。

- `llvm/lib/Support/` 保存操作系统抽象、文件系统、错误处理、命令行解析等底层服务。

- `llvm/lib/Analysis/` 封装可复用编译器事实，如支配关系与循环信息。

- `llvm/lib/Passes/` 及相关头文件描述优化如何被组合。

**关键目录与文件**

- `llvm/include/llvm/ADT/`：跨子项目使用的通用数据结构。

- `llvm/lib/Support/`：可移植系统编程基础层。

- `llvm/include/llvm/IR/` 与 `llvm/lib/IR/`：程序表示核心层。

- `llvm/lib/Analysis/` 与 `llvm/lib/Passes/`：可复用推理能力与调度胶水。

**主要文档**

- `llvm/docs/ProgrammersManual.rst`：这些库的日常使用习惯。

- `llvm/docs/ExtendingLLVM.rst`：新能力接入现有层次的路径。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
ls llvm/include/llvm/ADT | head -20
ls llvm/lib/Support | head -20
ls llvm/lib/Analysis llvm/lib/Passes
```

**结论**

- 尽早熟悉 ADT 与 Support，会显著降低后续阅读成本。

- 多数更高层项目，本质上都是这些层之上的领域逻辑。

## 5. Frontend: Clang / 前端：Clang

### EN

Clang demonstrates how a production frontend is layered: driver orchestration, parsing, semantic analysis, AST construction, diagnostics, and IR generation each have their own responsibilities and reusable APIs.

**Core ideas**

- `clang/docs/DriverInternals.rst` separates job construction from actual frontend execution.

- `clang/docs/IntroductionToTheClangAST.rst` explains why Clang keeps a source-faithful AST.

- `clang/lib/Parse/` handles syntax while `clang/lib/Sema/` resolves meaning and type rules.

- `clang/lib/CodeGen/` bridges AST constructs to LLVM IR and runtime conventions.

**Key directories and files**

- `clang/lib/Driver/`: user-facing driver logic and job graphs.

- `clang/lib/Frontend/`: compiler instance setup and `cc1` actions.

- `clang/lib/Parse/`, `clang/lib/Sema/`, `clang/lib/AST/`: source-to-meaning pipeline.

- `clang/lib/CodeGen/`: lowering from frontend structures to IR.

**Primary docs**

- `clang/docs/InternalsManual.rst`: broad internal map.

- `clang/docs/DriverInternals.rst` and `clang/docs/IntroductionToTheClangAST.rst`: two essential entry points.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
ls clang/lib/Driver clang/lib/Frontend clang/lib/Parse clang/lib/Sema clang/lib/AST clang/lib/CodeGen
grep -n 'AST' clang/docs/IntroductionToTheClangAST.rst | head -20
grep -n 'Driver' clang/docs/DriverInternals.rst | head -20
```

**Takeaways**

- Clang proves that frontend architecture needs richer structures than LLVM IR alone.

- Many tools are alternate clients of the same frontend libraries.

### 中文

Clang 展示了生产级前端如何分层：driver 编排、解析、语义分析、AST 构建、诊断与 IR 生成各有明确职责，并暴露可复用 API。

**核心理解**

- `clang/docs/DriverInternals.rst` 区分了任务构造与真正的前端执行。

- `clang/docs/IntroductionToTheClangAST.rst` 解释了为什么 Clang 要保留贴近源码的 AST。

- `clang/lib/Parse/` 处理语法，`clang/lib/Sema/` 负责语义与类型规则。

- `clang/lib/CodeGen/` 把 AST 构造桥接到 LLVM IR 与运行时约定。

**关键目录与文件**

- `clang/lib/Driver/`：面向用户的 driver 逻辑与任务图。

- `clang/lib/Frontend/`：编译实例搭建与 `cc1` action。

- `clang/lib/Parse/`、`clang/lib/Sema/`、`clang/lib/AST/`：从源码到语义的流水线。

- `clang/lib/CodeGen/`：从前端结构 lowering 到 IR。

**主要文档**

- `clang/docs/InternalsManual.rst`：内部结构总览。

- `clang/docs/DriverInternals.rst` 与 `clang/docs/IntroductionToTheClangAST.rst`：两个关键入口。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
ls clang/lib/Driver clang/lib/Frontend clang/lib/Parse clang/lib/Sema clang/lib/AST clang/lib/CodeGen
grep -n 'AST' clang/docs/IntroductionToTheClangAST.rst | head -20
grep -n 'Driver' clang/docs/DriverInternals.rst | head -20
```

**结论**

- Clang 证明了：前端架构需要比 LLVM IR 更丰富的结构。

- 许多工具只是同一前端库的不同客户端。

## 6. Optimization Passes / 优化遍

### EN

LLVM optimizations are structured as passes plus reusable analyses; the modern pass manager makes pipeline granularity, invalidation, and extension points explicit.

**Core ideas**

- `llvm/docs/NewPassManager.rst` explains module, CGSCC, function, and loop pass managers.

- Analyses compute facts; transforms consume them and state what remains preserved.

- `PassBuilder` in `llvm/lib/Passes/` assembles standard pipelines such as `O2`.

- Canonicalization is a recurring theme because later passes depend on stable forms.

**Key directories and files**

- `llvm/lib/Passes/`: pipeline construction and registration.

- `llvm/lib/Transforms/`: transformation implementations grouped by domain.

- `llvm/lib/Analysis/`: shared compiler analyses.

- `llvm/include/llvm/Passes/`: public pipeline APIs.

**Primary docs**

- `llvm/docs/NewPassManager.rst`: modern pass architecture.

- `llvm/docs/Passes.rst`: catalog of common pass goals.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
ls llvm/lib/Passes llvm/lib/Transforms llvm/lib/Analysis
grep -n 'PassBuilder' llvm/docs/NewPassManager.rst | head -20
grep -n 'loop\|scalar\|ipo' llvm/docs/Passes.rst | head -20
```

**Takeaways**

- Optimization works because passes cooperate through preserved analyses and canonical forms.

- Think in pipelines, not isolated pass files.

### 中文

LLVM 优化被组织为“pass + 可复用 analysis”；现代 pass manager 让流水线粒度、失效规则与扩展点都变得显式。

**核心理解**

- `llvm/docs/NewPassManager.rst` 解释了 module、CGSCC、function 与 loop 级别的 pass manager。

- analysis 负责计算事实；transform 消费这些事实，并声明哪些结果仍被保留。

- `llvm/lib/Passes/` 中的 `PassBuilder` 用于组装 `O2` 等标准流水线。

- 规范化是反复出现的主题，因为后续 pass 依赖稳定形式。

**关键目录与文件**

- `llvm/lib/Passes/`：流水线构建与注册。

- `llvm/lib/Transforms/`：按领域组织的变换实现。

- `llvm/lib/Analysis/`：共享编译器分析。

- `llvm/include/llvm/Passes/`：公共流水线 API。

**主要文档**

- `llvm/docs/NewPassManager.rst`：现代 pass 架构。

- `llvm/docs/Passes.rst`：常见 pass 目标目录。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
ls llvm/lib/Passes llvm/lib/Transforms llvm/lib/Analysis
grep -n 'PassBuilder' llvm/docs/NewPassManager.rst | head -20
grep -n 'loop\|scalar\|ipo' llvm/docs/Passes.rst | head -20
```

**结论**

- 优化之所以有效，是因为 pass 通过 preserved analysis 与规范形式进行协作。

- 要从“流水线”角度思考，而不是只看单个 pass 文件。

## 7. Code Generation / 代码生成

### EN

LLVM code generation lowers optimized IR through instruction selection, machine-level optimization, register allocation, frame setup, and MC emission.

**Core ideas**

- `llvm/docs/CodeGenerator.rst` gives the main stage map from IR to machine code.

- SelectionDAG and GlobalISel are two important instruction selection frameworks.

- `MachineFunction` and `MachineInstr` introduce machine-level structure and analyses.

- The MC layer separates encoding, assembly, and object emission from the rest of codegen.

**Key directories and files**

- `llvm/lib/CodeGen/`: target-independent machine code generation framework.

- `llvm/include/llvm/CodeGen/`: public machine-level interfaces.

- `llvm/lib/CodeGen/GlobalISel/`: GlobalISel implementation.

- `llvm/lib/MC/`: assembler, disassembler, and object emission infrastructure.

**Primary docs**

- `llvm/docs/CodeGenerator.rst`: end-to-end codegen guide.

- `llvm/docs/WritingAnLLVMBackend.rst`: backend author view of the same pipeline.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
ls llvm/lib/CodeGen | head -30
ls llvm/lib/MC | head -30
grep -n 'Instruction Selection\|Register Allocation\|MC' llvm/docs/CodeGenerator.rst | head -20
```

**Takeaways**

- Codegen is a staged pipeline, not one opaque lowering step.

- Localize failures by stage before you debug instruction details.

### 中文

LLVM 代码生成通过指令选择、机器级优化、寄存器分配、栈帧设置与 MC 发射，把优化后的 IR lowering 为最终机器代码。

**核心理解**

- `llvm/docs/CodeGenerator.rst` 给出了从 IR 到机器码的主要阶段地图。

- SelectionDAG 与 GlobalISel 是两个重要的指令选择框架。

- `MachineFunction` 与 `MachineInstr` 引入了机器级结构与分析。

- MC 层把编码、汇编与目标文件发射从其余 codegen 中分离出来。

**关键目录与文件**

- `llvm/lib/CodeGen/`：目标无关的机器码生成框架。

- `llvm/include/llvm/CodeGen/`：机器级公共接口。

- `llvm/lib/CodeGen/GlobalISel/`：GlobalISel 实现。

- `llvm/lib/MC/`：汇编、反汇编与目标文件发射基础设施。

**主要文档**

- `llvm/docs/CodeGenerator.rst`：端到端 codegen 指南。

- `llvm/docs/WritingAnLLVMBackend.rst`：从后端作者角度观察同一流水线。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
ls llvm/lib/CodeGen | head -30
ls llvm/lib/MC | head -30
grep -n 'Instruction Selection\|Register Allocation\|MC' llvm/docs/CodeGenerator.rst | head -20
```

**结论**

- codegen 是分阶段流水线，而不是一次性黑盒 lowering。

- 调试时先定位失败阶段，再看具体指令细节。

## 8. Target Backends / 目标后端

### EN

A backend specializes LLVM’s generic code generator for one ISA, ABI, and object model by supplying target data, lowering hooks, register and instruction models, and assembly support.

**Core ideas**

- `llvm/docs/WritingAnLLVMBackend.rst` explains what a target must implement and why.

- `TargetMachine` owns datalayout, relocation model, code model, and subtarget dispatch.

- Backends reuse generic codegen and MC layers whenever possible instead of reimplementing them.

- TableGen-driven descriptions are a major part of backend structure.

**Key directories and files**

- `llvm/lib/Target/`: generic target glue plus per-target directories.

- `llvm/lib/Target/X86/`: rich reference backend with many moving parts.

- `llvm/include/llvm/Target/`: public target interfaces.

- `llvm/lib/Target/*/MCTargetDesc/`: MC-side target descriptions.

**Primary docs**

- `llvm/docs/WritingAnLLVMBackend.rst`: main backend guide.

- `llvm/docs/TableGen/index.rst`: essential because target data is heavily declarative.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
find llvm/lib/Target -maxdepth 1 -mindepth 1 -type d | sort | head -20
ls llvm/lib/Target/X86 | head -30
grep -n 'TargetMachine\|AsmPrinter\|Register' llvm/docs/WritingAnLLVMBackend.rst | head -20
```

**Takeaways**

- A backend is a specialization of generic LLVM machinery.

- Read target code with the generic pipeline in mind.

### 中文

目标后端通过提供目标数据、lowering 钩子、寄存器与指令模型以及汇编支持，把 LLVM 通用代码生成器专门化到某个 ISA、ABI 与对象模型。

**核心理解**

- `llvm/docs/WritingAnLLVMBackend.rst` 解释了目标必须实现什么以及为什么。

- `TargetMachine` 管理 datalayout、重定位模型、代码模型与子目标分派。

- 后端应尽量复用通用 codegen 与 MC 层，而不是重复实现。

- 由 TableGen 驱动的描述是后端结构的重要组成部分。

**关键目录与文件**

- `llvm/lib/Target/`：通用目标胶水与各目标目录。

- `llvm/lib/Target/X86/`：部件丰富的参考后端。

- `llvm/include/llvm/Target/`：公共目标接口。

- `llvm/lib/Target/*/MCTargetDesc/`：MC 侧目标描述。

**主要文档**

- `llvm/docs/WritingAnLLVMBackend.rst`：主要后端指南。

- `llvm/docs/TableGen/index.rst`：目标数据高度声明式，因此也很关键。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
find llvm/lib/Target -maxdepth 1 -mindepth 1 -type d | sort | head -20
ls llvm/lib/Target/X86 | head -30
grep -n 'TargetMachine\|AsmPrinter\|Register' llvm/docs/WritingAnLLVMBackend.rst | head -20
```

**结论**

- 后端是对通用 LLVM 机制的专门化。

- 阅读目标代码时，脑中要始终保留通用流水线。

## 9. TableGen / TableGen

### EN

TableGen is LLVM’s declarative language and generator framework for structured compiler metadata such as instructions, registers, diagnostics, attributes, and patterns.

**Core ideas**

- `llvm/docs/TableGen/index.rst` introduces records, classes, multiclasses, and generation backends.

- Most targets rely on `.td` files to describe instruction sets and scheduling models.

- Clang also uses TableGen for diagnostics and other structured frontend metadata.

- Generated `.inc` files are part of the design story because they show how declarative data feeds C++.

**Key directories and files**

- `llvm/utils/TableGen/`: `llvm-tblgen` implementation.

- `llvm/lib/TableGen/`: shared TableGen library code.

- `llvm/lib/Target/*/*.td`: target descriptions.

- `clang/include/clang/Basic/`: diagnostic and attribute metadata sources.

**Primary docs**

- `llvm/docs/TableGen/index.rst`: language and backend overview.

- `llvm/docs/WritingAnLLVMBackend.rst`: shows where TableGen fits in backend authoring.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
ls llvm/utils/TableGen llvm/lib/TableGen
find llvm/lib/Target/X86 -name '*.td' | head -20
grep -n 'multiclass\|record\|backend' llvm/docs/TableGen/index.rst | head -20
```

**Takeaways**

- Use TableGen for structured facts and C++ for policy.

- Generated files are worth reading when backend behavior looks mysterious.

### 中文

TableGen 是 LLVM 用于结构化编译器元数据的声明式语言与生成框架，可描述指令、寄存器、诊断、属性与模式。

**核心理解**

- `llvm/docs/TableGen/index.rst` 介绍了 record、class、multiclass 与生成 backend。

- 多数目标依赖 `.td` 文件描述指令集与调度模型。

- Clang 也用 TableGen 生成诊断等结构化前端元数据。

- 生成的 `.inc` 文件属于设计故事的一部分，因为它们展示了声明式数据如何流入 C++。

**关键目录与文件**

- `llvm/utils/TableGen/`：`llvm-tblgen` 实现。

- `llvm/lib/TableGen/`：共享 TableGen 库代码。

- `llvm/lib/Target/*/*.td`：各目标描述文件。

- `clang/include/clang/Basic/`：诊断与属性元数据来源。

**主要文档**

- `llvm/docs/TableGen/index.rst`：语言与生成 backend 概览。

- `llvm/docs/WritingAnLLVMBackend.rst`：说明 TableGen 在后端开发中的位置。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
ls llvm/utils/TableGen llvm/lib/TableGen
find llvm/lib/Target/X86 -name '*.td' | head -20
grep -n 'multiclass\|record\|backend' llvm/docs/TableGen/index.rst | head -20
```

**结论**

- 结构化事实适合放在 TableGen，策略逻辑适合放在 C++。

- 当后端行为看起来神秘时，生成文件非常值得读。

## 10. MLIR / 多级中间表示

### EN

MLIR generalizes IR design with extensible dialects and a shared operation/value/block/region model so multiple abstraction levels can coexist and lower progressively.

**Core ideas**

- `mlir/docs/LangRef.md` defines the core object model and syntax.

- `mlir/docs/PassManagement.md` adapts pass ideas to nested operations and regions.

- `mlir/docs/Canonicalization.md` and `DeclarativeRewrites.md` show how local rewrites and patterns drive many transformations.

- `mlir/docs/TargetLLVMIR.md` documents the bridge back to LLVM IR.

**Key directories and files**

- `mlir/include/mlir/IR/`: core operation, type, attribute, and region interfaces.

- `mlir/lib/IR/`: shared MLIR object model implementation.

- `mlir/lib/Dialect/`: dialect-specific operations and transformations.

- `mlir/lib/Conversion/`: progressive lowering paths between dialects.

**Primary docs**

- `mlir/docs/LangRef.md`: the central reference.

- `mlir/docs/Tutorials/UnderstandingTheIRStructure.md` and `CreatingADialect.md`: practical architecture tutorials.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
ls mlir/include/mlir/IR mlir/lib/IR mlir/lib/Dialect mlir/lib/Conversion
grep -n 'Operation\|Region\|Block\|Dialect' mlir/docs/LangRef.md | head -20
grep -n 'canonicalization\|rewrite' mlir/docs/Canonicalization.md mlir/docs/DeclarativeRewrites.md | head -20
```

**Takeaways**

- MLIR makes representation design explicit instead of overloading one IR with every concern.

- Dialects and progressive lowering are the two ideas to internalize first.

### 中文

MLIR 通过可扩展 dialect 与统一的 operation/value/block/region 模型推广了 IR 设计，使多个抽象层级能够并存并渐进 lowering。

**核心理解**

- `mlir/docs/LangRef.md` 定义了核心对象模型与语法。

- `mlir/docs/PassManagement.md` 把 pass 概念适配到嵌套 operation 与 region。

- `mlir/docs/Canonicalization.md` 与 `DeclarativeRewrites.md` 展示了局部 rewrite 与模式如何驱动大量变换。

- `mlir/docs/TargetLLVMIR.md` 记录了回到 LLVM IR 的桥梁。

**关键目录与文件**

- `mlir/include/mlir/IR/`：核心 operation、type、attribute 与 region 接口。

- `mlir/lib/IR/`：共享 MLIR 对象模型实现。

- `mlir/lib/Dialect/`：dialect 特定操作与变换。

- `mlir/lib/Conversion/`：dialect 之间的渐进 lowering 路径。

**主要文档**

- `mlir/docs/LangRef.md`：核心参考文档。

- `mlir/docs/Tutorials/UnderstandingTheIRStructure.md` 与 `CreatingADialect.md`：实践型架构教程。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
ls mlir/include/mlir/IR mlir/lib/IR mlir/lib/Dialect mlir/lib/Conversion
grep -n 'Operation\|Region\|Block\|Dialect' mlir/docs/LangRef.md | head -20
grep -n 'canonicalization\|rewrite' mlir/docs/Canonicalization.md mlir/docs/DeclarativeRewrites.md | head -20
```

**结论**

- MLIR 通过显式表示层设计，避免把所有关注点硬塞进一个 IR。

- 最先要抓住的两个概念是 dialect 与渐进 lowering。

## 11. LLD Linker / LLD 链接器

### EN

LLD is LLVM’s family of native linkers. It favors format-specific implementations plus shared utilities, with tight integration into LLVM’s LTO and object infrastructure.

**Core ideas**

- `lld/docs/NewLLD.rst` explains major linker data structures and design tradeoffs.

- The project is split by object format: ELF, COFF, Mach-O, and WebAssembly.

- Common linker themes recur: symbols, input files, sections, relocation handling, and writers.

- Built-in LTO matters because some inputs re-enter LLVM optimization before final layout.

**Key directories and files**

- `lld/ELF/`, `lld/COFF/`, `lld/MachO/`, `lld/wasm/`: format-specific linker implementations.

- `lld/Common/`: shared support code across linkers.

- `lld/docs/`: project documentation and design notes.

- `llvm/lib/Object/` and related LLVM libraries: nearby dependencies worth knowing.

**Primary docs**

- `lld/docs/index.rst`: overview and entry points.

- `lld/docs/NewLLD.rst`: architecture-oriented internal guide.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
ls lld/ELF lld/COFF lld/MachO lld/wasm lld/Common
grep -n 'symbol\|section\|writer\|LTO' lld/docs/NewLLD.rst | head -20
sed -n '1,80p' lld/docs/index.rst
```

**Takeaways**

- LLD is a family of native linkers, not one generic linker with many modes.

- Format-specific directories are a feature because they keep platform behavior explicit.

### 中文

LLD 是 LLVM 的原生链接器家族。它采用“格式特定实现 + 共享工具”的结构，并与 LLVM 的 LTO 和对象文件基础设施紧密集成。

**核心理解**

- `lld/docs/NewLLD.rst` 解释了主要链接器数据结构与设计取舍。

- 项目按对象文件格式划分：ELF、COFF、Mach-O 与 WebAssembly。

- 链接器中的常见主题会反复出现：symbol、输入文件、section、重定位处理与 writer。

- 内建 LTO 很重要，因为部分输入会在最终布局前重新进入 LLVM 优化。

**关键目录与文件**

- `lld/ELF/`、`lld/COFF/`、`lld/MachO/`、`lld/wasm/`：按格式划分的链接器实现。

- `lld/Common/`：链接器之间共享的支持代码。

- `lld/docs/`：项目文档与设计说明。

- `llvm/lib/Object/` 及相关 LLVM 库：值得了解的相邻依赖。

**主要文档**

- `lld/docs/index.rst`：概览与入口。

- `lld/docs/NewLLD.rst`：偏架构的内部指南。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
ls lld/ELF lld/COFF lld/MachO lld/wasm lld/Common
grep -n 'symbol\|section\|writer\|LTO' lld/docs/NewLLD.rst | head -20
sed -n '1,80p' lld/docs/index.rst
```

**结论**

- LLD 是一组原生链接器，而不是一个通用链接器的多种模式。

- 按格式分目录是一种优点，因为它让平台行为保持显式。

## 12. LLDB Debugger / LLDB 调试器

### EN

LLDB is a layered debugger platform built on reusable libraries, plugin interfaces, and strong language integration rather than a monolithic command loop.

**Core ideas**

- `lldb/docs/resources/overview.rst` maps the main subsystems: target, process, thread, frame, expression, symbol, and plugins.

- The object model mirrors debugging concepts directly, which makes the code easier to navigate once the names are familiar.

- Expression evaluation reuses Clang and LLVM technologies, linking LLDB back to the compiler stack.

- Plugin boundaries matter because debuggers must adapt to platforms, object formats, languages, and remote targets.

**Key directories and files**

- `lldb/include/lldb/`: public debugger API and subsystem interfaces.

- `lldb/source/API/`: API-facing wrappers around reusable core logic.

- `lldb/source/Expression/`, `lldb/source/Symbol/`, `lldb/source/Target/`: major subsystem implementations.

- `lldb/test/`: executable documentation for debugger behavior.

**Primary docs**

- `lldb/docs/index.rst`: documentation hub.

- `lldb/docs/resources/overview.rst`: subsystem-level architecture overview.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
ls lldb/include/lldb lldb/source/API lldb/source/Expression lldb/source/Symbol lldb/source/Target
grep -n 'Target\|Process\|Thread\|Expression\|Plugin' lldb/docs/resources/overview.rst | head -20
find lldb/source -maxdepth 1 -mindepth 1 -type d | sort | head -20
```

**Takeaways**

- LLDB is a reusable debugger framework, not merely a CLI.

- Debugger architecture often mirrors runtime concepts more directly than compiler architecture does.

### 中文

LLDB 是一个建立在可复用库、插件接口与强语言集成之上的分层调试平台，而不是单体命令循环。

**核心理解**

- `lldb/docs/resources/overview.rst` 映射了主要子系统：target、process、thread、frame、expression、symbol 与 plugin。

- 对象模型直接映射调试概念，因此只要名字熟悉，代码导航就会容易很多。

- 表达式求值复用了 Clang 与 LLVM 技术，这让 LLDB 与编译器栈紧密相连。

- 插件边界之所以重要，是因为调试器必须适配平台、对象格式、语言与远程目标。

**关键目录与文件**

- `lldb/include/lldb/`：公共调试器 API 与子系统接口。

- `lldb/source/API/`：面向 API 的封装层。

- `lldb/source/Expression/`、`lldb/source/Symbol/`、`lldb/source/Target/`：主要子系统实现。

- `lldb/test/`：调试器行为的可执行文档。

**主要文档**

- `lldb/docs/index.rst`：文档中心。

- `lldb/docs/resources/overview.rst`：子系统级架构总览。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
ls lldb/include/lldb lldb/source/API lldb/source/Expression lldb/source/Symbol lldb/source/Target
grep -n 'Target\|Process\|Thread\|Expression\|Plugin' lldb/docs/resources/overview.rst | head -20
find lldb/source -maxdepth 1 -mindepth 1 -type d | sort | head -20
```

**结论**

- LLDB 是可复用调试器框架，而不只是 CLI。

- 调试器架构通常比编译器架构更直接地映射运行时概念。

## 13. Compiler-RT / 编译器运行时

### EN

Compiler-RT provides low-level runtime services that compiled programs rely on, including builtins, sanitizers, profiling, and other compiler-facing support libraries.

**Core ideas**

- `compiler-rt/docs/BuildingCompilerRT.rst` shows how these runtimes fit into the larger LLVM build model.

- `compiler-rt/lib/builtins/` supplies helper routines for operations that need library support.

- Sanitizer runtimes combine instrumentation, interceptors, shadow state, and reporting logic.

- Testing is central because runtime behavior is platform- and ABI-sensitive.

**Key directories and files**

- `compiler-rt/lib/builtins/`: helper routines for generated code.

- `compiler-rt/lib/asan/` and `compiler-rt/lib/sanitizer_common/`: sanitizer runtime implementations.

- `compiler-rt/lib/profile/`: profiling support libraries.

- `compiler-rt/test/`: runtime and sanitizer tests.

**Primary docs**

- `compiler-rt/docs/BuildingCompilerRT.rst`: build model.

- `compiler-rt/docs/TestingGuide.rst` and `compiler-rt/lib/asan/README.txt`: testing and design detail.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
ls compiler-rt/lib/builtins compiler-rt/lib/asan compiler-rt/lib/sanitizer_common compiler-rt/lib/profile
grep -n 'AddressSanitizer\|interceptor\|shadow' compiler-rt/lib/asan/README.txt | head -20
sed -n '1,100p' compiler-rt/docs/BuildingCompilerRT.rst
```

**Takeaways**

- Compiler-RT is where compile-time instrumentation meets runtime enforcement.

- Sanitizers are full runtime systems, not just frontend flags.

### 中文

Compiler-RT 提供编译后程序依赖的底层运行时服务，包括 builtins、sanitizer、profiling 以及其他面向编译器的支持库。

**核心理解**

- `compiler-rt/docs/BuildingCompilerRT.rst` 展示这些运行时如何融入更大的 LLVM 构建模型。

- `compiler-rt/lib/builtins/` 提供需要库支持的编译器辅助例程。

- sanitizer 运行时组合了插桩、拦截器、影子状态与报告逻辑。

- 测试是核心，因为运行时行为高度依赖平台与 ABI。

**关键目录与文件**

- `compiler-rt/lib/builtins/`：供生成代码调用的辅助例程。

- `compiler-rt/lib/asan/` 与 `compiler-rt/lib/sanitizer_common/`：sanitizer 运行时实现。

- `compiler-rt/lib/profile/`：profile 支持库。

- `compiler-rt/test/`：运行时与 sanitizer 测试。

**主要文档**

- `compiler-rt/docs/BuildingCompilerRT.rst`：构建模型。

- `compiler-rt/docs/TestingGuide.rst` 与 `compiler-rt/lib/asan/README.txt`：测试与设计细节。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
ls compiler-rt/lib/builtins compiler-rt/lib/asan compiler-rt/lib/sanitizer_common compiler-rt/lib/profile
grep -n 'AddressSanitizer\|interceptor\|shadow' compiler-rt/lib/asan/README.txt | head -20
sed -n '1,100p' compiler-rt/docs/BuildingCompilerRT.rst
```

**结论**

- Compiler-RT 是编译期插桩与运行期执行相接的地方。

- sanitizer 是完整运行时系统，而不只是前端开关。

## 14. libc++ / C++ 标准库

### EN

libc++ is LLVM’s C++ standard library implementation, balancing conformance, performance, ABI stability, and portability across a large and long-lived interface surface.

**Core ideas**

- `libcxx/include/` carries both public headers and a large amount of implementation detail, which is normal for a modern template-heavy standard library.

- `libcxx/src/` contains ABI-sensitive out-of-line pieces.

- `libcxx/test/` is as important architecturally as the code because conformance is a primary goal.

- `libcxx/docs/index.rst` is the right entry point for build, usage, and project organization.

**Key directories and files**

- `libcxx/include/`: public interface plus much implementation.

- `libcxx/src/`: separately compiled library pieces.

- `libcxx/test/`: conformance and regression coverage.

- `libcxx/docs/`: project documentation.

**Primary docs**

- `libcxx/docs/index.rst`: library documentation hub.

- `llvm/docs/CMake.rst`: useful context for runtime-oriented builds.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
ls libcxx/include libcxx/src libcxx/test libcxx/docs
sed -n '1,80p' libcxx/docs/index.rst
find libcxx/include -maxdepth 1 | sort | head -20
```

**Takeaways**

- Headers and tests are first-class architectural artifacts in a standard library.

- Read libc++ with ABI and conformance constraints in mind.

### 中文

libc++ 是 LLVM 的 C++ 标准库实现，需要在标准兼容性、性能、ABI 稳定性与可移植性之间平衡一个庞大且长期存在的接口面。

**核心理解**

- `libcxx/include/` 同时承载公共头文件与大量实现细节，这对模板很重的现代标准库来说很正常。

- `libcxx/src/` 保存 ABI 敏感的非内联实现。

- `libcxx/test/` 在架构上和代码同样重要，因为标准兼容是首要目标。

- `libcxx/docs/index.rst` 是理解构建、使用与项目组织的正确入口。

**关键目录与文件**

- `libcxx/include/`：公共接口与大量实现。

- `libcxx/src/`：单独编译的库实现部分。

- `libcxx/test/`：兼容性与回归测试。

- `libcxx/docs/`：项目文档。

**主要文档**

- `libcxx/docs/index.rst`：库文档入口。

- `llvm/docs/CMake.rst`：理解 runtime 构建时很有帮助。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
ls libcxx/include libcxx/src libcxx/test libcxx/docs
sed -n '1,80p' libcxx/docs/index.rst
find libcxx/include -maxdepth 1 | sort | head -20
```

**结论**

- 在标准库中，头文件与测试都是一等架构工件。

- 阅读 libc++ 时要始终把 ABI 与标准兼容约束放在脑中。

## 15. libc / C 标准库

### EN

LLVM libc is a modular C library effort that emphasizes componentization, explicit build modes, and strong testing over traditional monolithic implementation style.

**Core ideas**

- `libc/docs/index.rst` frames the project and its architectural goals.

- `libc/docs/build_and_test.rst` is especially important because build mode affects how the library is assembled and used.

- `libc/include/` exposes headers, while `libc/src/` is organized by implementation families.

- The project follows LLVM-style factoring: internal helpers are kept small, testable, and reusable.

**Key directories and files**

- `libc/include/`: public C headers.

- `libc/src/`: implementation entry points and internal layers.

- `libc/test/`: behavior validation across configurations.

- `libc/docs/`: architecture, build, and test documentation.

**Primary docs**

- `libc/docs/index.rst`: project overview.

- `libc/docs/build_and_test.rst`: build and test model.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
ls libc/include libc/src libc/test libc/docs
sed -n '1,120p' libc/docs/index.rst
grep -n 'overlay\|full build\|test' libc/docs/build_and_test.rst | head -20
```

**Takeaways**

- Build modes are part of libc architecture, not just packaging details.

- Study headers, source families, and tests together.

### 中文

LLVM libc 是一个强调组件化、显式构建模式与强测试的模块化 C 标准库实现，而不是传统的大型单体实现风格。

**核心理解**

- `libc/docs/index.rst` 给出了项目定位与架构目标。

- `libc/docs/build_and_test.rst` 特别重要，因为构建模式会影响库如何被组装与使用。

- `libc/include/` 暴露头文件，而 `libc/src/` 按实现族组织。

- 该项目体现了 LLVM 风格分解：内部辅助组件尽量保持小、可测、可复用。

**关键目录与文件**

- `libc/include/`：公共 C 头文件。

- `libc/src/`：实现入口与内部层次。

- `libc/test/`：跨配置行为验证。

- `libc/docs/`：架构、构建与测试文档。

**主要文档**

- `libc/docs/index.rst`：项目总览。

- `libc/docs/build_and_test.rst`：构建与测试模型。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
ls libc/include libc/src libc/test libc/docs
sed -n '1,120p' libc/docs/index.rst
grep -n 'overlay\|full build\|test' libc/docs/build_and_test.rst | head -20
```

**结论**

- 构建模式是 libc 架构的一部分，而不只是打包细节。

- 头文件、实现族与测试应该一起阅读。

## 16. Flang / Fortran 编译器

### EN

Flang is LLVM’s Fortran frontend and lowering stack: parser, semantics, FIR/HLFIR, MLIR-based optimization, and runtime support work together to handle a language with rich semantics.

**Core ideas**

- `flang/docs/Overview.md` gives the system-level picture.

- `flang/docs/FortranIR.md` explains why FIR and related IRs sit between semantics and LLVM IR.

- `flang/lib/Parser/` and `flang/lib/Semantics/` separate syntax from meaning.

- `flang/runtime/` matters because not all Fortran semantics can be compiled away.

**Key directories and files**

- `flang/lib/Parser/`: parse tree construction.

- `flang/lib/Semantics/`: language rules and semantic checks.

- `flang/lib/Lower/` and `flang/lib/Optimizer/`: FIR/HLFIR lowering and optimization.

- `flang/runtime/`: runtime support for language features.

**Primary docs**

- `flang/docs/Overview.md`: project overview.

- `flang/docs/FortranIR.md`: FIR and representation strategy.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
ls flang/lib/Parser flang/lib/Semantics flang/lib/Lower flang/lib/Optimizer flang/runtime
sed -n '1,120p' flang/docs/Overview.md
grep -n 'FIR\|HLFIR\|MLIR\|LLVM IR' flang/docs/FortranIR.md flang/docs/Overview.md | head -20
```

**Takeaways**

- Flang is a frontend plus lowering stack, not just a parser.

- FIR exists because source semantics need a richer bridge than raw LLVM IR.

### 中文

Flang 是 LLVM 的 Fortran 前端与 lowering 栈：parser、semantic、FIR/HLFIR、基于 MLIR 的优化与运行时支持协同工作，以处理语义丰富的 Fortran。

**核心理解**

- `flang/docs/Overview.md` 给出系统级图景。

- `flang/docs/FortranIR.md` 解释了为什么 FIR 等 IR 要位于语义层与 LLVM IR 之间。

- `flang/lib/Parser/` 与 `flang/lib/Semantics/` 把语法与语义分离。

- `flang/runtime/` 很重要，因为并非所有 Fortran 语义都能在编译期消除。

**关键目录与文件**

- `flang/lib/Parser/`：parse tree 构建。

- `flang/lib/Semantics/`：语言规则与语义检查。

- `flang/lib/Lower/` 与 `flang/lib/Optimizer/`：FIR/HLFIR lowering 与优化。

- `flang/runtime/`：语言特性所需运行时支持。

**主要文档**

- `flang/docs/Overview.md`：项目概览。

- `flang/docs/FortranIR.md`：FIR 与表示策略。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
ls flang/lib/Parser flang/lib/Semantics flang/lib/Lower flang/lib/Optimizer flang/runtime
sed -n '1,120p' flang/docs/Overview.md
grep -n 'FIR\|HLFIR\|MLIR\|LLVM IR' flang/docs/FortranIR.md flang/docs/Overview.md | head -20
```

**结论**

- Flang 是完整的前端加 lowering 栈，而不只是 parser。

- FIR 的存在，是因为源语言语义需要比原始 LLVM IR 更丰富的桥梁。

## 17. Polly / 多面体优化

### EN

Polly adds a polyhedral optimization framework on top of LLVM IR, focusing on static control parts and mathematically modeled loop and access transformations.

**Core ideas**

- `polly/docs/Architecture.rst` explains both the technique and where Polly sits in the pass pipeline.

- Polly identifies SCoPs, models dependencies, and derives new schedules.

- Its design shows how specialized optimization frameworks can plug into LLVM without replacing the whole middle-end.

- Pipeline placement matters because canonical forms and later vectorization interact with Polly’s work.

**Key directories and files**

- `polly/include/polly/`: public interfaces.

- `polly/lib/Analysis/`: SCoP detection and analysis.

- `polly/lib/Transform/` and `polly/lib/CodeGen/`: transformation and lowering back to IR.

- `polly/test/`: expected detection and transformation behavior.

**Primary docs**

- `polly/docs/Architecture.rst`: architecture and pass placement.

- `llvm/docs/NewPassManager.rst`: useful context for pass composition.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
ls polly/include/polly polly/lib/Analysis polly/lib/Transform polly/lib/CodeGen polly/test
sed -n '1,120p' polly/docs/Architecture.rst
grep -n 'SCoP\|vectorizer\|schedule\|polyhedral' polly/docs/Architecture.rst | head -20
```

**Takeaways**

- Polly is a good case study in domain-specific optimization plugged into generic infrastructure.

- High-level math still has to lower back to legal LLVM IR.

### 中文

Polly 在 LLVM IR 之上增加了多面体优化框架，重点处理静态控制区域以及以数学方式建模的循环与访问变换。

**核心理解**

- `polly/docs/Architecture.rst` 同时解释了技术本身以及 Polly 在 pass 流水线中的位置。

- Polly 识别 SCoP、建模依赖，并推导新的调度。

- 它展示了专门优化框架如何在不替换整个中端的情况下接入 LLVM。

- 流水线放置很重要，因为规范形式与后续向量化都会影响 Polly 的效果。

**关键目录与文件**

- `polly/include/polly/`：公共接口。

- `polly/lib/Analysis/`：SCoP 识别与分析。

- `polly/lib/Transform/` 与 `polly/lib/CodeGen/`：变换及 lowering 回 IR。

- `polly/test/`：预期识别与变换行为。

**主要文档**

- `polly/docs/Architecture.rst`：架构与 pass 放置。

- `llvm/docs/NewPassManager.rst`：理解 pass 组合时的有用背景。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
ls polly/include/polly polly/lib/Analysis polly/lib/Transform polly/lib/CodeGen polly/test
sed -n '1,120p' polly/docs/Architecture.rst
grep -n 'SCoP\|vectorizer\|schedule\|polyhedral' polly/docs/Architecture.rst | head -20
```

**结论**

- Polly 是“领域特定优化接入通用基础设施”的好案例。

- 高层数学模型最终仍必须 lowering 回合法 LLVM IR。

## 18. BOLT / 二进制优化

### EN

BOLT performs post-link optimization on binaries using runtime profiles, extending the optimization story beyond LLVM IR and link-time passes.

**Core ideas**

- `bolt/README.md` explains the post-link model and why final binary layout still has large performance impact.

- BOLT sees the linked binary, so it can optimize around final function ordering and block placement.

- This makes BOLT complementary to middle-end optimization and LTO rather than a replacement.

- Its implementation reconstructs control flow and symbol information from already built binaries.

**Key directories and files**

- `bolt/include/bolt/`: public declarations and reusable interfaces.

- `bolt/lib/`: binary analysis and transformation libraries.

- `bolt/tools/`: end-user tools and drivers.

- `bolt/test/`: regression tests for binary transformations.

**Primary docs**

- `bolt/README.md`: project overview and motivation.

- `lld/docs/NewLLD.rst`: useful contrast for what link-time layout fixes before BOLT runs.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
ls bolt/include/bolt bolt/lib bolt/tools bolt/test
grep -n 'post-link\|profile\|layout\|binary' bolt/README.md | head -20
sed -n '1,120p' bolt/README.md
```

**Takeaways**

- Optimization opportunities remain even after machine code exists.

- Binary layout is a major performance lever that earlier stages only approximate.

### 中文

BOLT 在二进制上执行 post-link 优化，并利用运行时 profile，把优化故事延伸到了 LLVM IR 与链接时 pass 之后。

**核心理解**

- `bolt/README.md` 解释了 post-link 模型，以及为什么最终二进制布局仍会显著影响性能。

- BOLT 看到的是已链接二进制，因此它可以围绕最终函数顺序与基本块布局进行优化。

- 因此 BOLT 与中端优化、LTO 是互补关系，而不是替代关系。

- 它的实现需要从已构建二进制中重建控制流与符号信息。

**关键目录与文件**

- `bolt/include/bolt/`：公共声明与可复用接口。

- `bolt/lib/`：二进制分析与变换库。

- `bolt/tools/`：面向用户的工具与驱动。

- `bolt/test/`：二进制变换回归测试。

**主要文档**

- `bolt/README.md`：项目概览与动机。

- `lld/docs/NewLLD.rst`：理解 BOLT 运行前链接阶段已固定哪些布局的对照材料。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
ls bolt/include/bolt bolt/lib bolt/tools bolt/test
grep -n 'post-link\|profile\|layout\|binary' bolt/README.md | head -20
sed -n '1,120p' bolt/README.md
```

**结论**

- 即使机器码已经存在，优化机会仍然很多。

- 二进制布局是重要性能杠杆，而早期阶段往往只能近似处理。

## 19. Clang Tools / Clang 工具

### EN

The Clang tooling ecosystem turns frontend libraries into developer tools: standalone analyzers, source-to-source refactoring tools, formatters, and language servers all reuse parsing, ASTs, and semantic analysis.

**Core ideas**

- `clang/docs/Tooling.rst` and `clang/docs/LibTooling.rst` explain Clang as a tooling platform.

- `clang-tools-extra/docs/clang-tidy/index.rst` shows a framework plus many AST-based checks.

- `clang-tools-extra/clangd/README.md` shows a long-lived interactive architecture rather than batch compilation.

- Tooling proves the AST and source manager are reusable assets, not internal frontend accidents.

**Key directories and files**

- `clang/lib/Tooling/`: reusable tooling APIs.

- `clang-tools-extra/clang-tidy/`: static analysis checks and fix-it framework.

- `clang-tools-extra/clangd/`: language server implementation.

- `clang-tools-extra/docs/`: extra tool documentation.

**Primary docs**

- `clang/docs/Tooling.rst` and `clang/docs/LibTooling.rst`: platform-level docs.

- `clang-tools-extra/docs/clang-tidy/index.rst` and `clang-tools-extra/clangd/README.md`: concrete tool examples.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
ls clang/lib/Tooling clang-tools-extra/clang-tidy clang-tools-extra/clangd clang-tools-extra/docs
grep -n 'LibTooling\|tooling' clang/docs/Tooling.rst clang/docs/LibTooling.rst | head -20
grep -n 'clangd\|clang-tidy' clang-tools-extra/clangd/README.md clang-tools-extra/docs/clang-tidy/index.rst | head -20
```

**Takeaways**

- Clang is a tooling platform as much as a compiler frontend.

- Tooling is often the easiest entry path into Clang internals.

### 中文

Clang 工具体系把前端库转化为开发者工具：独立分析器、源码到源码重构工具、格式化器与语言服务器都复用了 parsing、AST 与语义分析。

**核心理解**

- `clang/docs/Tooling.rst` 与 `clang/docs/LibTooling.rst` 把 Clang 解释为 tooling 平台。

- `clang-tools-extra/docs/clang-tidy/index.rst` 展示了“框架 + 大量基于 AST 的检查”。

- `clang-tools-extra/clangd/README.md` 展示的是长期交互式架构，而不是批处理式编译。

- tooling 证明了 AST 与 source manager 是可复用资产，而不是前端内部偶然产物。

**关键目录与文件**

- `clang/lib/Tooling/`：可复用 tooling API。

- `clang-tools-extra/clang-tidy/`：静态分析检查与 fix-it 框架。

- `clang-tools-extra/clangd/`：语言服务器实现。

- `clang-tools-extra/docs/`：额外工具文档。

**主要文档**

- `clang/docs/Tooling.rst` 与 `clang/docs/LibTooling.rst`：平台级说明。

- `clang-tools-extra/docs/clang-tidy/index.rst` 与 `clang-tools-extra/clangd/README.md`：具体工具示例。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
ls clang/lib/Tooling clang-tools-extra/clang-tidy clang-tools-extra/clangd clang-tools-extra/docs
grep -n 'LibTooling\|tooling' clang/docs/Tooling.rst clang/docs/LibTooling.rst | head -20
grep -n 'clangd\|clang-tidy' clang-tools-extra/clangd/README.md clang-tools-extra/docs/clang-tidy/index.rst | head -20
```

**结论**

- Clang 既是编译前端，也是 tooling 平台。

- tooling 往往是进入 Clang 内部最容易的路径。

## 20. Build System / 构建系统

### EN

LLVM uses a highly configurable CMake build that can assemble compilers, libraries, runtimes, and tests from one monorepo while keeping source and build trees separate.

**Core ideas**

- `llvm/docs/CMake.rst` is the authoritative build guide.

- `LLVM_ENABLE_PROJECTS` and `LLVM_ENABLE_RUNTIMES` expose the architecture directly at configure time.

- Out-of-tree builds are standard because LLVM generates many files and supports many configurations.

- `runtimes/` exists because some projects are best built in a separate orchestration phase.

**Key directories and files**

- `llvm/CMakeLists.txt`: top-level LLVM build entry point.

- `llvm/cmake/modules/`: shared build logic.

- `runtimes/CMakeLists.txt`: runtime build orchestration.

- `clang/CMakeLists.txt`, `mlir/CMakeLists.txt`, and similar files: subproject integration points.

**Primary docs**

- `llvm/docs/CMake.rst`: main guide.

- `README.md`: concise build examples and project selection context.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
sed -n '1,120p' llvm/docs/CMake.rst
grep -n 'LLVM_ENABLE_PROJECTS\|LLVM_ENABLE_RUNTIMES' llvm/docs/CMake.rst README.md | head -20
ls llvm/cmake/modules runtimes | head -20
```

**Takeaways**

- Build files often reveal product boundaries and integration assumptions.

- Configuration is part of architecture, not just build plumbing.

### 中文

LLVM 使用高度可配置的 CMake 构建系统，可以从同一个单仓库组装编译器、库、运行时与测试，同时保持源码树与构建树分离。

**核心理解**

- `llvm/docs/CMake.rst` 是权威构建指南。

- `LLVM_ENABLE_PROJECTS` 与 `LLVM_ENABLE_RUNTIMES` 在配置阶段就把架构直接暴露出来。

- 推荐 out-of-tree build，因为 LLVM 会生成大量文件并支持多种配置。

- `runtimes/` 存在，是因为某些项目最适合在单独编排阶段构建。

**关键目录与文件**

- `llvm/CMakeLists.txt`：LLVM 顶层构建入口。

- `llvm/cmake/modules/`：共享构建逻辑。

- `runtimes/CMakeLists.txt`：运行时构建编排。

- `clang/CMakeLists.txt`、`mlir/CMakeLists.txt` 等文件：子项目接入点。

**主要文档**

- `llvm/docs/CMake.rst`：主指南。

- `README.md`：简洁构建示例与项目选择背景。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
sed -n '1,120p' llvm/docs/CMake.rst
grep -n 'LLVM_ENABLE_PROJECTS\|LLVM_ENABLE_RUNTIMES' llvm/docs/CMake.rst README.md | head -20
ls llvm/cmake/modules runtimes | head -20
```

**结论**

- 构建文件常常会暴露产品边界与集成假设。

- 配置本身就是架构的一部分，而不只是构建接线。

## 21. Testing Infrastructure / 测试基础设施

### EN

LLVM testing is infrastructure, not afterthought: `lit`, `FileCheck`, unit tests, and project-specific suites encode behavioral contracts across the monorepo.

**Core ideas**

- `llvm/docs/TestingGuide.rst` explains `RUN:` lines, `lit`, `FileCheck`, and feature guards.

- The test tree mirrors the code tree: `llvm/test`, `clang/test`, `mlir/test`, `lldb/test`, and more.

- `llvm/unittests/` complements regression tests with API- and algorithm-oriented coverage.

- Tests are often the fastest executable documentation for diagnostics, transforms, and tool behavior.

**Key directories and files**

- `llvm/test/` and `llvm/unittests/`: core regression and unit tests.

- `llvm/utils/lit/`: test runner implementation.

- `clang/test/`, `mlir/test/`, `compiler-rt/test/`, `lldb/test/`: project-specific suites.

- `llvm/docs/TestingGuide.rst`: central conventions guide.

**Primary docs**

- `llvm/docs/TestingGuide.rst`: main testing guide.

- `compiler-rt/docs/TestingGuide.rst`: useful runtime-focused contrast.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
ls llvm/test llvm/unittests llvm/utils/lit clang/test mlir/test compiler-rt/test lldb/test
grep -n 'RUN:\|FileCheck\|REQUIRES\|XFAIL' llvm/docs/TestingGuide.rst | head -20
find llvm/test -name '*.ll' | head -10
```

**Takeaways**

- Tests are a specification mechanism in LLVM.

- Read tests early if you want to learn subsystem contracts fast.

### 中文

LLVM 的测试是基础设施，而非事后补充：`lit`、`FileCheck`、单元测试与各项目测试套件一起编码了整个单仓库的行为契约。

**核心理解**

- `llvm/docs/TestingGuide.rst` 解释了 `RUN:` 行、`lit`、`FileCheck` 与特性守卫。

- 测试树通常镜像代码树：`llvm/test`、`clang/test`、`mlir/test`、`lldb/test` 等。

- `llvm/unittests/` 用 API 与算法导向覆盖补足回归测试。

- 测试常常是学习诊断、变换与工具行为最快的可执行文档。

**关键目录与文件**

- `llvm/test/` 与 `llvm/unittests/`：核心回归与单元测试。

- `llvm/utils/lit/`：测试运行器实现。

- `clang/test/`、`mlir/test/`、`compiler-rt/test/`、`lldb/test/`：项目特定测试套件。

- `llvm/docs/TestingGuide.rst`：核心约定指南。

**主要文档**

- `llvm/docs/TestingGuide.rst`：主测试指南。

- `compiler-rt/docs/TestingGuide.rst`：有用的运行时测试对照。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
ls llvm/test llvm/unittests llvm/utils/lit clang/test mlir/test compiler-rt/test lldb/test
grep -n 'RUN:\|FileCheck\|REQUIRES\|XFAIL' llvm/docs/TestingGuide.rst | head -20
find llvm/test -name '*.ll' | head -10
```

**结论**

- 在 LLVM 中，测试本身就是规格机制。

- 如果想快速理解子系统契约，应尽早阅读测试。

## 22. Developer Guide / 开发者指南

### EN

Developer guidance keeps this very large repository coherent: contribution workflow, coding standards, extension points, and daily-use programming idioms let many subprojects evolve without losing shared engineering habits.

**Core ideas**

- `llvm/docs/Contributing.rst` explains review expectations and patch workflow.

- `llvm/docs/CodingStandards.rst` is about maintainability and layering discipline, not only style.

- `llvm/docs/ExtendingLLVM.rst` maps common extension surfaces such as passes, targets, and IR features.

- `llvm/docs/ProgrammersManual.rst` captures the idioms that make the codebase feel internally consistent.

**Key directories and files**

- `llvm/docs/Contributing.rst`: contribution workflow and expectations.

- `llvm/docs/CodingStandards.rst`: repository-wide coding norms.

- `llvm/docs/ExtendingLLVM.rst` and `llvm/docs/ProgrammersManual.rst`: extension and coding idioms.

- `llvm/utils/git/`: helper scripts for developer workflows.

**Primary docs**

- `llvm/docs/Contributing.rst`: how changes enter the project.

- `llvm/docs/CodingStandards.rst`: how the project preserves readability at scale.

**Architecture reading checklist**

- Map the public interfaces before diving into implementation details.

- Compare the docs with tests in neighboring directories.

- Separate reusable infrastructure from project-specific policy.

- Trace one artifact through at least two adjacent layers.

**Mini walkthrough**

```bash
sed -n '1,120p' llvm/docs/Contributing.rst
grep -n 'assert\|namespace\|comment\|format' llvm/docs/CodingStandards.rst | head -20
ls llvm/utils/git | head -20
```

**Takeaways**

- In LLVM, process documentation is part of architecture quality.

- When adding features, start from documented extension points and nearby tests.

### 中文

开发者指南保证了这个超大仓库的一致性：贡献流程、编码规范、扩展点与日常编程习惯让众多子项目可以共同演化，而不丢失共享工程习惯。

**核心理解**

- `llvm/docs/Contributing.rst` 解释了 review 期望与补丁流程。

- `llvm/docs/CodingStandards.rst` 关注的不只是风格，还有可维护性与分层纪律。

- `llvm/docs/ExtendingLLVM.rst` 映射了常见扩展面，如 pass、target 与 IR 特性。

- `llvm/docs/ProgrammersManual.rst` 总结了让代码库保持内部一致性的编程习惯。

**关键目录与文件**

- `llvm/docs/Contributing.rst`：贡献流程与期望。

- `llvm/docs/CodingStandards.rst`：全仓库编码规范。

- `llvm/docs/ExtendingLLVM.rst` 与 `llvm/docs/ProgrammersManual.rst`：扩展与编码习惯。

- `llvm/utils/git/`：开发工作流辅助脚本。

**主要文档**

- `llvm/docs/Contributing.rst`：改动如何进入项目。

- `llvm/docs/CodingStandards.rst`：项目如何在大规模下保持可读性。

**阅读清单**

- 先看公共接口，再深入实现细节。

- 把文档与相邻测试目录对照起来阅读。

- 区分可复用基础设施与项目特定策略。

- 至少跟踪一个工件跨越两个相邻层次。

**小练习**

```bash
sed -n '1,120p' llvm/docs/Contributing.rst
grep -n 'assert\|namespace\|comment\|format' llvm/docs/CodingStandards.rst | head -20
ls llvm/utils/git | head -20
```

**结论**

- 在 LLVM 中，流程文档也是架构质量的一部分。

- 新增特性时，优先从文档化扩展点和附近测试开始。

