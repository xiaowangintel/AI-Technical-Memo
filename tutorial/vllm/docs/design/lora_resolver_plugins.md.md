# lora_resolver_plugins.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/lora_resolver_plugins.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** Explains how LoRA resolver plugins let vLLM load adapters on demand when a request references an adapter that is not already present in memory. **CN:** 说明 LoRA resolver 插件如何让 vLLM 在请求引用尚未驻留内存的适配器时按需加载该适配器。
- **EN:** Documents the runtime prerequisites, especially the environment variables that enable dynamic LoRA updates, activate plugins, and choose a resolver cache directory. **CN:** 记录运行前提条件，重点说明启用动态 LoRA 更新、激活插件以及设置 resolver 缓存目录所需的环境变量。
- **EN:** Uses the filesystem resolver as the main example, showing expected adapter directory structure, server startup, and the request flow that triggers resolution. **CN:** 以文件系统 resolver 为主要示例，展示适配器目录结构、服务启动方式，以及触发解析加载的请求流程。
- **EN:** Covers multiple-resolver ordering, custom resolver implementation, and troubleshooting for missing files, permissions, or malformed adapter packages. **CN:** 涵盖多 resolver 的顺序、自定义 resolver 的实现方式，以及缺文件、权限问题或适配器包格式错误时的排障方法。

## Key Concepts / 关键概念
- **EN:** Runtime LoRA loading: adapter resolution happens at request time instead of only at server startup. **CN:** 运行时 LoRA 加载：适配器解析发生在请求期间，而不只是在服务启动时完成。
- **EN:** Resolver chain: vLLM can try several resolver plugins in sequence until one can materialize the requested adapter. **CN:** Resolver 链：vLLM 可以按顺序尝试多个 resolver 插件，直到其中一个成功提供所请求的适配器。
- **EN:** Environment-driven setup: dynamic behavior is controlled through `VLLM_ALLOW_RUNTIME_LORA_UPDATING`, `VLLM_PLUGINS`, and the resolver cache path. **CN:** 环境变量驱动的配置：动态行为由 `VLLM_ALLOW_RUNTIME_LORA_UPDATING`、`VLLM_PLUGINS` 与 resolver 缓存路径共同控制。
- **EN:** Adapter package schema: resolvers are expected to produce a directory with config and weight files in the format vLLM can load. **CN:** 适配器包结构：resolver 需要产出包含配置文件和权重文件、且符合 vLLM 加载格式的目录。
- **EN:** Custom resolver API: users can extend the system by implementing their own resolver class and registering it as a plugin. **CN:** 自定义 resolver API：用户可以通过实现自己的 resolver 类并注册为插件来扩展系统。
