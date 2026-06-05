# io_processor_plugins.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/io_processor_plugins.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** Defines IO processor plugins as a way to adapt custom request and response schemas for pooling models without changing the core engine interface. **CN:** 将 IO processor 插件定义为一种适配层，使 pooling 模型可以支持自定义输入输出结构，而无需修改核心引擎接口。
- **EN:** Explains the plugin lifecycle: parse input data, merge request parameters, pre-process into prompts, and post-process `PoolingRequestOutput` back into client-facing data. **CN:** 说明插件生命周期：解析输入数据、合并请求参数、预处理为 prompt，再把 `PoolingRequestOutput` 后处理成面向客户端的数据。
- **EN:** Emphasizes that validation of custom payloads belongs to the plugin itself, while vLLM only validates standard fields such as prompts and request parameters. **CN:** 强调自定义载荷的校验责任由插件承担，而 vLLM 只校验 prompt 与请求参数等标准字段。
- **EN:** Documents configuration precedence and loading paths, where `EngineArgs` can specify a plugin directly and override a Hugging Face model-config declaration. **CN:** 记录插件的配置优先级与加载路径，其中 `EngineArgs` 可直接指定插件，并覆盖 Hugging Face 模型配置中的声明。

## Key Concepts / 关键概念
- **EN:** Pooling-only scope: the design targets `LLM.encode` and the `/pooling` endpoint rather than text generation. **CN:** 作用范围限于 pooling：该设计面向 `LLM.encode` 与 `/pooling` 接口，而非文本生成。
- **EN:** `parse_data`: the entry point where custom raw input is validated and converted into a plugin-specific request object. **CN:** `parse_data`：插件校验并转换自定义原始输入的入口。
- **EN:** Pre/post hooks: `pre_process` and `post_process` let a plugin translate between external schemas and vLLM-native prompt/output types. **CN:** 前后处理钩子：`pre_process` 与 `post_process` 负责在外部数据结构和 vLLM 原生 prompt/输出类型之间转换。
- **EN:** Async support: asynchronous hooks preserve output ordering while allowing non-blocking pre/post work. **CN:** 异步支持：异步钩子在允许非阻塞处理的同时仍保证输出顺序。
- **EN:** Configuration precedence: engine arguments override model-config defaults when both specify an IO processor plugin. **CN:** 配置优先级：当引擎参数与模型配置都指定插件时，以引擎参数为准。
