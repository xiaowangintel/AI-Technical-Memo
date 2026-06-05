# glm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/glm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for glm, including architecture wrappers and weight loading logic. / 面向推理的 glm vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 5-8)
```python
from vllm.config import VllmConfig
from vllm.model_executor.models.llama import LlamaForCausalLM

from .utils import PPMissingLayer
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `GlmForCausalLM` (lines 11-24)
```python
class GlmForCausalLM(LlamaForCausalLM):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        hf_config = vllm_config.model_config.hf_config
        hf_config.rope_parameters["partial_rotary_factor"] = 0.5
        super().__init__(vllm_config=vllm_config, prefix=prefix)
        # Hack Llama model to fit HF format GLM implementation
        # Attention difference between GLM and Llama:
        # 1. Half partial rotary_dim and no Neox style.
        # 2. There is no bias for o_proj in attention
        for layer in self.model.layers:
            if not isinstance(layer, PPMissingLayer):
                layer.self_attn.rotary_emb.is_neox_style = False
                layer.self_attn.o_proj.bias = None
                layer.self_attn.o_proj.skip_bias_add = True
```
**EN:** Class `GlmForCausalLM` organizes related behavior for this model family or helper component. It inherits from LlamaForCausalLM. Key methods include __init__.
**CN:** 类 `GlmForCausalLM` 用于组织该模型族或辅助组件的相关行为。 它继承自 LlamaForCausalLM。 关键方法包括 __init__。

### Method `GlmForCausalLM.__init__` (lines 12-24)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        hf_config = vllm_config.model_config.hf_config
        hf_config.rope_parameters["partial_rotary_factor"] = 0.5
        super().__init__(vllm_config=vllm_config, prefix=prefix)
        # Hack Llama model to fit HF format GLM implementation
        # Attention difference between GLM and Llama:
        # 1. Half partial rotary_dim and no Neox style.
        # 2. There is no bias for o_proj in attention
        for layer in self.model.layers:
            if not isinstance(layer, PPMissingLayer):
                layer.self_attn.rotary_emb.is_neox_style = False
                layer.self_attn.o_proj.bias = None
                layer.self_attn.o_proj.skip_bias_add = True
```
**EN:** Method `GlmForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GlmForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

## Key Concepts / 关键概念
- **Attention / 注意力**
  - **EN:** Attention layers model token-to-token or token-to-feature interactions inside the architecture.
  - **CN:** 注意力层负责建模 token 与 token，或 token 与特征之间的交互。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.model_executor.models.llama import LlamaForCausalLM`, `from .utils import PPMissingLayer`
- **Module note / 模块说明**: **EN:** Inference-only HF format GLM-4 model compatible with THUDM weights. **CN:** 模块文档字符串给出的原始说明是：Inference-only HF format GLM-4 model compatible with THUDM weights.。
