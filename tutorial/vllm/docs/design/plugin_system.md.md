# plugin_system.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/plugin_system.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** The document motivates plugins as a way to extend vLLM without patching the main codebase, which is important because vLLM often launches multiple processes during inference. **CN:** 文档将插件机制定位为无需修改主代码库即可扩展 vLLM 的方式；这很重要，因为 vLLM 在推理时通常会启动多个进程。
- **EN:** It explains that plugins are discovered through Python `entry_points`, and every vLLM process loads them through `load_plugins_by_group`. **CN:** 文档说明插件通过 Python 的 `entry_points` 机制发现，并由 `load_plugins_by_group` 在每个 vLLM 进程中加载。
- **EN:** It defines the three registration elements—plugin group, plugin name, and plugin value—and notes that `VLLM_PLUGINS` can filter loading by plugin name. **CN:** 文档定义了注册的三个核心元素——plugin group、plugin name 和 plugin value，并指出可以通过 `VLLM_PLUGINS` 按插件名过滤加载。
- **EN:** The supported plugin categories are general plugins, platform plugins, IO processor plugins, and stat logger plugins, each aimed at a different extension surface. **CN:** 支持的插件类型包括通用插件、平台插件、IO 处理器插件和统计日志插件，分别对应不同的扩展入口。
- **EN:** A large portion of the document is a platform-plugin implementation guide covering project layout, entry-point registration, required platform/worker/backend hooks, optional features, and custom-op integration. **CN:** 文档的大部分篇幅都在讲平台插件实现指南，包括项目结构、入口点注册、平台/worker/backend 的必需接口、可选特性以及自定义算子集成。
- **EN:** It closes with compatibility expectations and explicit deprecation notices for several older interfaces. **CN:** 文档最后补充了兼容性约定，并明确列出了一些旧接口的弃用说明。

## Key Concepts / 关键概念
- **EN:** **Entry-point discovery:** vLLM relies on standard Python packaging metadata rather than a custom plugin registry. **CN:** **入口点发现：** vLLM 依赖标准 Python 打包元数据，而不是自建插件注册中心。
- **EN:** **Plugin groups:** `vllm.general_plugins`, `vllm.platform_plugins`, `vllm.io_processor_plugins`, and `vllm.stat_logger_plugins` separate different extension purposes. **CN:** **插件分组：** `vllm.general_plugins`、`vllm.platform_plugins`、`vllm.io_processor_plugins` 和 `vllm.stat_logger_plugins` 用于区分不同扩展目的。
- **EN:** **Re-entrant registration:** plugin entry functions must tolerate repeated execution because multiple vLLM processes may invoke them. **CN:** **可重入注册：** 插件入口函数必须能被重复调用，因为多个 vLLM 进程都可能执行它们。
- **EN:** **Platform integration:** a platform plugin typically defines a `Platform`, a `WorkerBase` implementation, an attention backend, and possibly communicator/custom ops. **CN:** **平台集成：** 一个平台插件通常需要定义 `Platform`、`WorkerBase` 实现、attention backend，以及可能的通信器和自定义算子。
- **EN:** **Version responsibility:** vLLM keeps documented registration interfaces available, but plugin authors must keep their implementations compatible with target vLLM versions. **CN:** **版本责任：** vLLM 会保持文档化注册接口可用，但插件作者需要自行保证实现与目标 vLLM 版本兼容。
