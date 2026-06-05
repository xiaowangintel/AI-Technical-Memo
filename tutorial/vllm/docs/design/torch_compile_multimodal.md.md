# torch_compile_multimodal.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/torch_compile_multimodal.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This document extends the `torch.compile` story to multimodal encoders and other nn modules, with vision-language models like LLaMA 4 and Qwen-VL used as motivating examples. **CN:** 该文档把 `torch.compile` 的讨论扩展到多模态编码器和其他 nn 模块，并以 LLaMA 4、Qwen-VL 等视觉语言模型为典型示例。
- **EN:** It states that multimodal encoder compilation is currently disabled by default and enabled through `compile_mm_encoder: true` when the model component is decorated appropriately. **CN:** 文档指出多模态编码器编译默认关闭；当模型组件加上合适装饰器后，可通过 `compile_mm_encoder: true` 启用。
- **EN:** Enablement requires `@support_torch_compile` to use `enable_if=should_torch_compile_mm_encoder`, and encoder components should additionally set `is_encoder=True`. **CN:** 启用方式要求 `@support_torch_compile` 使用 `enable_if=should_torch_compile_mm_encoder`，而编码器组件还应设置 `is_encoder=True`。
- **EN:** The document recommends an incremental rollout strategy: start from small modules, use `tlparse` to diagnose recompiles and graph breaks, and configure dynamic arguments carefully. **CN:** 文档建议采用渐进式接入策略：先从小模块开始，使用 `tlparse` 诊断重编译与 graph break，再谨慎配置动态参数。
- **EN:** It also explains that compile ranges for encoders default to `(1, MAX_INT)` because encoder input shapes are harder to infer than text-batch sizes. **CN:** 文档还解释了编码器的编译范围为何默认设为 `(1, MAX_INT)`：因为编码器输入形状通常比文本 batch size 更难推断。
- **EN:** The troubleshooting section focuses on graph breaks from dynamic image sizes, untraceable ops, and conditional logic, and gives concrete debug commands. **CN:** 排障部分重点讨论了动态图像尺寸、不可追踪算子以及条件分支导致的 graph break，并给出了具体调试命令。

## Key Concepts / 关键概念
- **EN:** **`compile_mm_encoder`:** a dedicated config switch that turns multimodal encoder compilation on without changing the rest of the compilation model. **CN:** **`compile_mm_encoder`：** 一个专门的配置开关，用于开启多模态编码器编译，而无需改变其余编译模型。
- **EN:** **`support_torch_compile` decorator:** multimodal components opt in through the same decorator mechanism used by the text backbone. **CN:** **`support_torch_compile` 装饰器：** 多模态组件通过与文本主干相同的装饰器机制接入编译支持。
- **EN:** **`is_encoder=True`:** this flag tells the compile system that encoder range inference differs from standard text-batch assumptions. **CN:** **`is_encoder=True`：** 该标志告诉编译系统，编码器的范围推断不同于标准文本 batch 的假设。
- **EN:** **Graph-break diagnosis:** debug logging and `TORCH_LOGS="+dynamo"` are the main observability tools when vision encoders fail to trace cleanly. **CN:** **Graph break 诊断：** 当视觉编码器无法顺利追踪时，调试日志和 `TORCH_LOGS="+dynamo"` 是主要观测手段。
- **EN:** **Unspecified cudagraph support:** unlike the text path, multimodal encoder compilation has not yet defined a cudagraph integration story. **CN:** **尚未明确的 cudagraph 支持：** 与文本路径不同，多模态编码器编译目前还没有明确的 cudagraph 集成方案。
