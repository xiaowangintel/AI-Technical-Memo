# chatglm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/chatglm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for chatglm so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 chatglm 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8: Comments and module notes
```python
# Adapted from
# https://github.com/THUDM/ChatGLM2-6B
# https://github.com/vllm-project/vllm/blob/main/vllm/transformers_utils/configs/chatglm.py

# ChatGLM2 and ChatGLM3 share the same config.
# ChatGLM4 is officially supported by Huggingface
# transformers >= 4.46.0 is required
# https://huggingface.co/docs/transformers/en/model_doc/glm
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 9-9: Imports dependencies
```python
from transformers import PretrainedConfig
```
**EN:** This block imports dependencies for the module, including transformers.PretrainedConfig. These imports supply standard utilities, third-party packages, or SGLang runtime components used later in the file.
**CN:** 该代码块为 模块 导入依赖，包括 transformers.PretrainedConfig。这些导入为后续实现提供标准库工具、第三方包或 SGLang 运行时组件。

### Lines 10-11: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 12-12: Declares class ChatGLMConfig
```python
class ChatGLMConfig(PretrainedConfig):
```
**EN:** This block introduces class `ChatGLMConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `ChatGLMConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 13-17: Declares model_type, attribute_map
```python
    model_type = "chatglm"
    attribute_map = {
        "num_hidden_layers": "num_layers",
        "n_head_kv": "multi_query_group_num",
    }
```
**EN:** This block initializes a related set of values in the ChatGLMConfig, including model_type, attribute_map. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 ChatGLMConfig 中初始化一组相关值，包括 model_type, attribute_map。将这些赋值集中在一起有助于理解周边配置。

### Lines 18-18: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the ChatGLMConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 ChatGLMConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 19-78: Defines function ChatGLMConfig.__init__
```python
    def __init__(
        self,
        num_layers=28,
        padded_vocab_size=65024,
        hidden_size=4096,
        ffn_hidden_size=13696,
        kv_channels=128,
        num_attention_heads=32,
        seq_length=2048,
        hidden_dropout=0.0,
        attention_dropout=0.0,
        layernorm_epsilon=1e-5,
        rmsnorm=True,
        apply_residual_connection_post_layernorm=False,
        post_layer_norm=True,
        add_bias_linear=False,
        add_qkv_bias=False,
        interleaved_qkv=False,
        bias_dropout_fusion=True,
        multi_query_attention=False,
        multi_query_group_num=1,
        apply_query_key_layer_scaling=True,
        attention_softmax_in_fp32=True,
        fp32_residual_connection=False,
        quantization_bit=0,
        pre_seq_len=None,
        prefix_projection=False,
        **kwargs,
    ):
        self.num_layers = num_layers
        self.vocab_size = padded_vocab_size
        self.padded_vocab_size = padded_vocab_size
        self.hidden_size = hidden_size
        self.ffn_hidden_size = ffn_hidden_size
        self.kv_channels = kv_channels
        self.num_attention_heads = num_attention_heads
        self.seq_length = seq_length
        # It is to be compatible with long lora.
        self.max_position_embeddings = seq_length
        self.hidden_dropout = hidden_dropout
        self.attention_dropout = attention_dropout
        self.layernorm_epsilon = layernorm_epsilon
        self.rmsnorm = rmsnorm
        self.apply_residual_connection_post_layernorm = (
            apply_residual_connection_post_layernorm
        )
        self.post_layer_norm = post_layer_norm
        self.add_bias_linear = add_bias_linear
        self.add_qkv_bias = add_qkv_bias
        self.bias_dropout_fusion = bias_dropout_fusion
        self.multi_query_attention = multi_query_attention
        self.multi_query_group_num = multi_query_group_num
        self.apply_query_key_layer_scaling = apply_query_key_layer_scaling
        self.attention_softmax_in_fp32 = attention_softmax_in_fp32
        self.fp32_residual_connection = fp32_residual_connection
        self.quantization_bit = quantization_bit
        self.pre_seq_len = pre_seq_len
        self.prefix_projection = prefix_projection
        self.interleaved_qkv = interleaved_qkv
        super().__init__(**kwargs)
```
**EN:** This block defines function `ChatGLMConfig.__init__`. Parameters: self, num_layers, padded_vocab_size, hidden_size, ffn_hidden_size, kv_channels, num_attention_heads, seq_length, hidden_dropout, attention_dropout, layernorm_epsilon, rmsnorm, apply_residual_connection_post_layernorm, post_layer_norm, add_bias_linear, add_qkv_bias, interleaved_qkv, bias_dropout_fusion, multi_query_attention, multi_query_group_num, apply_query_key_layer_scaling, attention_softmax_in_fp32, fp32_residual_connection, quantization_bit, pre_seq_len, prefix_projection.
**CN:** 该代码块定义函数 `ChatGLMConfig.__init__`。 参数包括 self、num_layers、padded_vocab_size、hidden_size、ffn_hidden_size、kv_channels、num_attention_heads、seq_length、hidden_dropout、attention_dropout、layernorm_epsilon、rmsnorm、apply_residual_connection_post_layernorm、post_layer_norm、add_bias_linear、add_qkv_bias、interleaved_qkv、bias_dropout_fusion、multi_query_attention、multi_query_group_num、apply_query_key_layer_scaling、attention_softmax_in_fp32、fp32_residual_connection、quantization_bit、pre_seq_len、prefix_projection。

## Key Concepts / 关键概念
- **Classes / 类**: `ChatGLMConfig`

## Dependencies / 依赖关系
- **Third-Party / 第三方**: `transformers`
