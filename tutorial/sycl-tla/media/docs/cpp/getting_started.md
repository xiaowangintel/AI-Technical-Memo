# getting_started.rst — Documentation Analysis / 文档分析

## Source / 来源
- **Path:** /root/xw/sycl-tla/media/docs/cpp/getting_started.rst
- **Purpose:** Provides the top-level onboarding map for CUTLASS C++ users. / 为 CUTLASS C++ 用户提供顶层入门导航图。

## Content Analysis / 内容分析
### Getting Started
**EN:** The file defines the main onboarding section and acts as an information architecture node rather than a narrative tutorial.
**CN:** 该文件定义了主要的入门章节，本质上更像信息架构节点，而不是连续叙述式教程。

### toctree
**EN:** The toctree maps the onboarding journey: Quickstart for first build/run, IDE Setup for editor tooling, Build for platform/compiler setup, Functionality and Terminology for conceptual context, Fundamental Types for core abstractions, Programming Guidelines for contributor norms, and GEMM Heuristics for tuning decisions.
**CN:** toctree 展示了完整的入门路径：Quickstart 负责首次构建与运行，IDE Setup 负责编辑器工具链，Build 负责平台与编译器配置，Functionality 与 Terminology 提供概念背景，Fundamental Types 介绍核心抽象，Programming Guidelines 说明贡献规范，GEMM Heuristics 则服务于调优决策。

### Navigation depth
**EN:** The use of maxdepth 2 signals that this page is a broader hub than the build index. It organizes both immediate beginner tasks and follow-on conceptual reading.
**CN:** 使用 maxdepth 2 表明该页面比 build index 覆盖范围更广，它同时组织了初学者的直接任务与后续的概念性阅读材料。

## Key Concepts / 关键概念
- Onboarding information architecture / 入门信息架构
- Task-first learning path / 任务优先的学习路径
- Progression from setup to concepts / 从环境搭建到概念学习的递进

## Related Files / 相关文件
- /root/xw/sycl-tla/media/docs/cpp/quickstart.md — First practical build-and-run guide / 首个可执行的构建运行指南
- /root/xw/sycl-tla/media/docs/cpp/ide_setup.md — Editor and language-server setup guide / 编辑器与语言服务器配置指南
- /root/xw/sycl-tla/media/docs/cpp/build/index.rst — Build subsection linked from this page / 本页链接到的构建子章节
- /root/xw/sycl-tla/media/docs/cpp/programming_guidelines.md — Coding norms referenced as part of onboarding / 作为入门路径一部分的编码规范
