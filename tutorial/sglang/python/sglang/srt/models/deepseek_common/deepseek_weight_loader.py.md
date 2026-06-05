# deepseek_weight_loader.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/deepseek_common/deepseek_weight_loader.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module adapts DeepSeek checkpoint weights into the tensor-parallel structures expected by SGLang. / 该模块将 DeepSeek 检查点权重适配为 SGLang 所需的张量并行结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Module header and imports / 模块头与导入
```python
# Copyright 2026 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 64-65: Conditional setup: _use_aiter_gfx95 / 条件初始化：_use_aiter_gfx95
```python
if _use_aiter_gfx95:
    from sglang.srt.layers.quantization.quark.utils import quark_post_load_weights
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 67-67: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 70-70: Assignment: NVFP4_CKPT_FP8_ATTN_QUANT_MODULES / 赋值：NVFP4_CKPT_FP8_ATTN_QUANT_MODULES
```python
NVFP4_CKPT_FP8_ATTN_QUANT_MODULES = ["q_b_proj"]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 73-76: Function: _clone_if_runai_streamed_tensor() / 函数：_clone_if_runai_streamed_tensor()
```python
def _clone_if_runai_streamed_tensor(tensor: torch.Tensor) -> torch.Tensor:
    if getattr(tensor, RUNAI_STREAMER_TENSOR_ATTR, False):
        return tensor.clone().detach()
    return tensor
```
**EN:** This function implements clone if runai streamed tensor for the surrounding model/runtime logic. Key parameters include tensor.
**CN:** 该函数实现了 clone if runai streamed tensor 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 80-84: Class: NextNEnabledConfig / 类：NextNEnabledConfig
```python
class NextNEnabledConfig:
    num_nextn_layers: int
    nextn_layer_id: int
    nextn_layer_prefix: str
    nextn_spec_weight_names: List[str]
```
**EN:** This class defines Next N Enabled Config, grouping state and methods for this model component.
**CN:** 该类定义了 Next N Enabled Config，用于封装该模型组件的状态与方法。

### Lines 88-89: Class: NextNDisabledConfig / 类：NextNDisabledConfig
```python
class NextNDisabledConfig:
    pass
```
**EN:** This class defines Next N Disabled Config, grouping state and methods for this model component.
**CN:** 该类定义了 Next N Disabled Config，用于封装该模型组件的状态与方法。

### Lines 92-92: Code block / 代码块
```python
"""Union type for NextN configuration, including enabled and disabled configurations."""
```
**EN:** This block provides supporting module logic.
**CN:** 该代码块提供辅助性的模块逻辑。

### Lines 93-93: Assignment: NextNConfig / 赋值：NextNConfig
```python
NextNConfig = NextNEnabledConfig | NextNDisabledConfig
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 96-737: Class: DeepseekV2WeightLoaderMixin / 类：DeepseekV2WeightLoaderMixin
```python
class DeepseekV2WeightLoaderMixin:
    """Mixin for loading weights in DeepSeek V2/V3 models."""

    model: nn.Module
    config: PretrainedConfig
    quant_config: Optional[QuantizationConfig]
    pp_group: GroupCoordinator
    num_fused_shared_experts: int

    def do_load_weights(
        self,
        weights: Iterable[Tuple[str, torch.Tensor]],
        is_nextn: bool = False,
    ):
        """Load model weights from checkpoint.

        Args:
            weights: Iterable of (weight_name, weight_tensor) pairs
            is_nextn: Whether loading NextN speculative decoding weights
        """
        nextn_conf = self._initialize_nextn_conf(is_nextn)

        weights = self._maybe_quant_weights_to_fp8_ue8m0(
            weights, NVFP4_CKPT_FP8_ATTN_QUANT_MODULES, nextn_conf
        )

        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]

        # Params for weights, fp8 weight scales, fp8 activation scales
        # (param_name, weight_name, expert_id, shard_id)
        expert_params_mapping = FusedMoE.make_expert_params_mapping(
            ckpt_gate_proj_name="gate_proj",
            ckpt_down_proj_name="down_proj",
            ckpt_up_proj_name="up_proj",
            num_experts=self.config.n_routed_experts + self.num_fused_shared_experts,
        )
# ... truncated for brevity ...
```
**EN:** This class defines Deepseek V2 Weight Loader Mixin, grouping state and methods for this model component. Docstring summary: Mixin for loading weights in DeepSeek V2/V3 models..
**CN:** 该类定义了 Deepseek V2 Weight Loader Mixin，用于封装该模型组件的状态与方法。 文档字符串摘要：Mixin for loading weights in DeepSeek V2/V3 models.。

## Key Concepts / 关键概念
- **EN:** Backend-aware dispatch that selects specialized attention implementations for different hardware and execution modes. / **CN:** 具备后端感知能力的分派机制，可针对不同硬件与执行模式选择特化注意力实现。
- **EN:** Shared utilities that reduce duplication across DeepSeek-family model integrations. / **CN:** 在 DeepSeek 系列模型集成中复用的共享工具，以减少重复实现。

## Dependencies / 依赖关系
- `concurrent.futures`
- `logging`
- `dataclasses: dataclass`
- `typing: Dict, Iterable, List, Optional, Tuple`
- `torch`
- `torch.nn`
- `tqdm`
- `transformers: PretrainedConfig`
- `sglang.srt.distributed.parallel_state: GroupCoordinator`
- `sglang.srt.environ: envs`
- `sglang.srt.layers: deep_gemm_wrapper`
- `sglang.srt.layers.moe.fused_moe_triton.layer: FusedMoE`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.quantization.fp8_utils: block_quant_dequant, block_quant_to_tensor_quant, channel_quant_to_tensor_quant, inverse_transform_scale_ue8m0, normalize_e4m3fn_to_e4m3fnuz, quant_weight_ue8m0`
- `sglang.srt.layers.quantization.int8_utils: block_dequant`
- `sglang.srt.layers.utils: get_layer_id`
- `sglang.srt.model_loader.utils: maybe_executor_submit, should_async_load, should_deepgemm_weight_requant_ue8m0`
- `sglang.srt.model_loader.weight_utils: RUNAI_STREAMER_TENSOR_ATTR, default_weight_loader`
- `sglang.srt.models.deepseek_common.utils: _is_cuda, _is_fp8_fnuz, _is_hip, _is_musa, _is_npu, _is_xpu, _use_aiter_gfx95, awq_dequantize_func, enable_nextn_moe_bf16_cast_to_fp8`
- `sglang.srt.utils: bind_or_assign, get_bool_env_var, log_info_on_rank0`
