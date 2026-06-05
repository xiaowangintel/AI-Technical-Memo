# lora.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/lora/lora.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements core LoRA runtime logic such as adapter metadata, memory management, layer wrapping, or orchestration. It is part of SGLang's infrastructure for serving models with dynamic adapters. / 该文件实现了核心 LoRA 运行时逻辑，例如适配器元数据、内存管理、层封装或整体编排。它属于 SGLang 支持动态适配器服务能力的基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-42: Module header, imports, and shared constants
```python
# Copyright 2023-2024 SGLang Team
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

# Integrates "S-LoRA: Serving Thousands of Concurrent LoRA Adapters"
# and "Punica: Multi-Tenant LoRA Serving"

# LoRA layers class inheritance adapted from:
# https://github.com/vllm-project/vllm/blob/4abf6336ec65c270343eb895e7b18786e9274176/vllm/lora/layers.py

import logging
import re
from typing import Dict, List, Optional

import torch
from torch import nn

from sglang.srt.configs.load_config import LoadConfig
from sglang.srt.layers.utils import get_layer_id
from sglang.srt.lora.backend.base_backend import BaseLoRABackend
from sglang.srt.lora.lora_config import LoRAConfig
from sglang.srt.model_loader.loader import DefaultModelLoader
from sglang.srt.utils.hf_transformers_utils import AutoConfig

# Matches both per-expert keys ("...experts.0.<module>...") and shared-outer
# keys ("...experts.<module>..."), while excluding "shared_experts." (where the
# preceding char is "_", not ".").
_ROUTED_EXPERT_PATTERN = re.compile(r"\.experts\.")

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 43-43: LoRALayer class declaration
```python
class LoRALayer(nn.Module):
```
**EN:** This block declares the `LoRALayer` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `LoRALayer` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 44-50: LoRALayer initializer
```python
    def __init__(self, config: LoRAConfig, base_hf_config: AutoConfig):
        super().__init__()
        self.config: LoRAConfig = config
        self.base_hf_config: AutoConfig = base_hf_config

        # lora weights in cpu. The weights are loaded from checkpoint.
        self.weights: Dict[str, torch.Tensor] = {}
```
**EN:** This block initializes the `LoRALayer` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `LoRALayer` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 53-53: LoRAAdapter class declaration
```python
class LoRAAdapter(nn.Module):
```
**EN:** This block declares the `LoRAAdapter` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `LoRAAdapter` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 54-90: LoRAAdapter initializer
```python
    def __init__(
        self,
        uid: str,
        config: LoRAConfig,
        base_hf_config: AutoConfig,
        load_config: LoadConfig,
        lora_backend: BaseLoRABackend,
        base_model: Optional[torch.nn.Module] = None,
    ):
        super().__init__()
        self.uid: str = uid
        self.config: LoRAConfig = config
        assert self.config.hf_config["peft_type"].lower() == "lora"
        self.base_hf_config: AutoConfig = base_hf_config
        self.load_config: LoadConfig = load_config
        self.lora_backend: BaseLoRABackend = lora_backend
        self.scaling: float = self.config.lora_alpha / self.config.r

        # Bypass nn.Module.__setattr__ so the base model is held as a plain
        # reference rather than auto-registered as a submodule (which would
        # leak its parameters into our state_dict / parameters() / .to()).
        object.__setattr__(self, "base_model", base_model)
        object.__setattr__(
            self,
            "_moe_is_gated_by_layer",
            self._build_moe_gated_map(base_model) if base_model is not None else {},
        )

        self.layers: List[LoRALayer] = nn.ModuleList(
            [
                LoRALayer(config, base_hf_config)
                for _ in range(base_hf_config.num_hidden_layers)
            ]
        )

        self.embedding_layers: Dict[str, torch.Tensor] = {}
        self.added_tokens_embeddings: Dict[str, torch.Tensor] = {}
```
**EN:** This block initializes the `LoRAAdapter` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `LoRAAdapter` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 92-119: LoRAAdapter._build_moe_gated_map method
```python
    @staticmethod
    def _build_moe_gated_map(base_model: torch.nn.Module) -> Dict[int, bool]:
        """Map layer_id -> moe_runner_config.is_gated for FusedMoE base layers.

        Only used by normalize_gate_up_proj to decide whether per-expert
        gate_proj weights should be zero-padded and stacked (gated → c=2 buffer)
        or just renamed (non-gated → c=1 buffer via model's get_stacked_multiply
        override on gate_up_proj_moe).

        Adapters can be loaded both before `init_lora_modules` (initial
        --lora-paths) and after (dynamic API loads), so the FusedMoE may
        appear either directly or under a `BaseLayerWithLoRA.base_layer`.
        """
        from sglang.srt.layers.moe.fused_moe_triton.layer import FusedMoE

        gated_map: Dict[int, bool] = {}
        for name, module in base_model.named_modules():
            inner = (
                module
                if isinstance(module, FusedMoE)
                else getattr(module, "base_layer", None)
            )
            if not isinstance(inner, FusedMoE):
                continue
            layer_id = get_layer_id(name)
            if layer_id is not None:
                gated_map[layer_id] = bool(inner.moe_runner_config.is_gated)
        return gated_map
```
**EN:** This block uses `LoRAAdapter._build_moe_gated_map` to construct helper objects or layouts. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRAAdapter._build_moe_gated_map` 来构建辅助对象或布局。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 121-138: LoRAAdapter._is_non_gated_moe_weight method
```python
    def _is_non_gated_moe_weight(self, weight_name: str) -> bool:
        """True iff this adapter weight targets a non-gated MoE expert.

        Such weights flow into the `gate_up_proj_moe` buffer, which the model
        overrides to stacked_multiply=1 — so the weight must be stored without
        being stacked with a synthetic up_proj zero-pad.

        Matches both adapter key conventions:
        - per-expert: ``...experts.0.<module>...`` (one tensor per expert)
        - shared-outer: ``...experts.<module>...`` (3D tensor with the expert
          dim baked into the shape)
        """
        if not _ROUTED_EXPERT_PATTERN.search(weight_name):
            return False
        layer_id = get_layer_id(weight_name)
        if layer_id is None:
            return False
        return self._moe_is_gated_by_layer.get(layer_id) is False
```
**EN:** This block uses `LoRAAdapter._is_non_gated_moe_weight` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRAAdapter._is_non_gated_moe_weight` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 140-153: LoRAAdapter.initialize_weights method
```python
    def initialize_weights(self):
        model_path = self.config.path
        loader = DefaultModelLoader(self.load_config)
        revision = getattr(self.config.hf_config, "revision", None)

        # Get normalized target modules for filtering
        for name, loaded_weight in loader._get_weights_iterator(
            DefaultModelLoader.Source(
                model_path, revision=revision, fall_back_to_pt=True
            )
        ):
            self._process_weight(name, loaded_weight)

        self._normalize_weights()
```
**EN:** This block uses `LoRAAdapter.initialize_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRAAdapter.initialize_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 155-159: LoRAAdapter.initialize_weights_from_tensors method
```python
    def initialize_weights_from_tensors(self, tensors: Dict[str, torch.Tensor]):
        for name, tensor in tensors.items():
            self._process_weight(name, tensor)

        self._normalize_weights()
```
**EN:** This block uses `LoRAAdapter.initialize_weights_from_tensors` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRAAdapter.initialize_weights_from_tensors` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 161-197: LoRAAdapter._process_weight method
```python
    def _process_weight(self, name: str, loaded_weight: torch.Tensor):
        from sglang.srt.lora.utils import get_normalized_target_modules

        normalized_target_modules = get_normalized_target_modules(
            self.config.target_modules
        )

        # Remap PEFT "unembed_tokens" key to "lm_head" so the weight is
        # recognized and loaded into the correct buffer.
        if "unembed_tokens" in name:
            name = name.replace("unembed_tokens", "lm_head")

        layer_id = get_layer_id(name)
        if layer_id is not None:
            self.layers[layer_id].weights[name] = loaded_weight.cpu()
        elif "embed_tokens" in name or "lm_head" in name:
            # Check if this module is declared in target_modules before loading.
            # When normalized_target_modules is {"all"} (e.g. target_modules was
            # "all-linear"), we allow loading since the server-level
            # --lora-target-modules will govern which modules are active.
            module_name = "embed_tokens" if "embed_tokens" in name else "lm_head"
            if (
                "all" in normalized_target_modules
                or module_name in normalized_target_modules
            ):
                self.embedding_layers[name] = loaded_weight.cpu()
            else:
                logger.debug(
                    f"Skipping {name} as '{module_name}' is not in adapter's target_modules: {self.config.target_modules}"
                )
        elif "input_embeddings" in name or "output_embeddings" in name:
            # added/extra token emb
            self.added_tokens_embeddings[name] = loaded_weight.cpu()
            assert loaded_weight.shape[0] == self.config.lora_added_tokens_size, (
                f"LoRA adapter {self.uid} has lora_added_tokens_size {self.config.lora_added_tokens_size} specified in the config, "
                f"but the loaded weight '{name}' has shape {loaded_weight.shape[0]} in first dimension"
            )
```
**EN:** This block uses `LoRAAdapter._process_weight` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRAAdapter._process_weight` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 199-211: LoRAAdapter._normalize_weights method
```python
    def _normalize_weights(self):
        for layer in self.layers:
            weight_names = list(layer.weights.keys())
            self.normalize_qkv_proj(weight_names, layer.weights)
            self._rename_expert_w_to_proj(layer.weights)
            # Stack gate_proj + x_proj → in_proj for Mamba layers (before gate_up normalization)
            self._normalize_in_proj(layer.weights)
            # Stack in_proj_q + in_proj_k + in_proj_v + in_proj_z → in_proj_qkvz for GDN layers
            self._normalize_in_proj_qkvz(layer.weights)
            weight_names = list(layer.weights.keys())
            self.normalize_gate_up_proj(weight_names, layer.weights)
            weight_names = list(layer.weights.keys())
            self.normalize_fused_qkv_a_proj(weight_names, layer.weights)
```
**EN:** This block uses `LoRAAdapter._normalize_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRAAdapter._normalize_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 213-263: LoRAAdapter.normalize_qkv_proj method
```python
    def normalize_qkv_proj(
        self, weight_names: List[str], weights: Dict[str, torch.Tensor]
    ):
        # Collect target q/k/v modules. This process is necessary since there might be no lora attached to k_proj
        target_module = set()
        for weight_name in weight_names:
            if "k_proj" in weight_name:
                target_module.add("k_proj")
            if "q_proj" in weight_name:
                target_module.add("q_proj")
            if "v_proj" in weight_name:
                target_module.add("v_proj")
            if "qkv_proj" in weight_name:
                target_module.add("qkv_proj")
        if len(target_module) == 0:
            return

        for weight_name in weight_names:
            # We assume every lora adaptor should contain lora modules for q_proj
            if "q_proj" in weight_name:
                q_name = weight_name
                k_name = weight_name.replace("q_proj", "k_proj")
                v_name = weight_name.replace("q_proj", "v_proj")
                qkv_name = weight_name.replace("q_proj", "qkv_proj")

                # If k_proj doesn't have lora, initialize it to zero
                k_proj_weight = (
                    weights[k_name]
                    if "k_proj" in target_module
                    else torch.zeros_like(weights[v_name])
                )
                weights[qkv_name] = torch.cat(
                    (
                        weights[q_name],
                        k_proj_weight,
                        weights[v_name],
                    ),
                    0,
                )
                weights.pop(q_name)
                if "k_proj" in target_module:
                    weights.pop(k_name)
                weights.pop(v_name)
            elif "qkv_proj" in weight_name:
                # If qkv_proj is already stacked, we normalize it following the SGL convention.
                qkv_name = weight_name
                q_name = weight_name.replace("qkv_proj", "q_proj")
                k_name = weight_name.replace("qkv_proj", "k_proj")
                v_name = weight_name.replace("qkv_proj", "v_proj")
                if "lora_A" in weight_name:
                    weights[qkv_name] = weights[qkv_name].repeat(3, 1)
```
**EN:** This block uses `LoRAAdapter.normalize_qkv_proj` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRAAdapter.normalize_qkv_proj` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 264-265: Class-level supporting statements
```python
                # else: no-op as LoRA B weight is already stacked.
```
**EN:** This block contains supporting statements for the `LoRAAdapter` class, such as constants, attributes, registrations, or small helper expressions that shape the surrounding control flow.
**CN:** 该代码块包含`LoRAAdapter` 类的配套语句，例如常量、属性、注册逻辑或小型辅助表达式，它们共同塑造了周边控制流程。

### Lines 266-281: LoRAAdapter._rename_expert_w_to_proj method
```python
    def _rename_expert_w_to_proj(self, weights: Dict[str, torch.Tensor]):
        """Rename w1 -> gate_proj, w3 -> up_proj, w2 -> down_proj so that
        normalize_gate_up_proj can stack them into gate_up_proj."""
        renames = {}
        for name in list(weights.keys()):
            new_name = name
            if ".w1." in name:
                new_name = name.replace(".w1.", ".gate_proj.")
            elif ".w3." in name:
                new_name = name.replace(".w3.", ".up_proj.")
            elif ".w2." in name:
                new_name = name.replace(".w2.", ".down_proj.")
            if new_name != name:
                renames[name] = new_name
        for old_name, new_name in renames.items():
            weights[new_name] = weights.pop(old_name)
```
**EN:** This block uses `LoRAAdapter._rename_expert_w_to_proj` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRAAdapter._rename_expert_w_to_proj` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 283-304: LoRAAdapter._normalize_in_proj method
```python
    def _normalize_in_proj(self, weights: Dict[str, torch.Tensor]):
        """Stack gate_proj + x_proj → in_proj for Mamba layers.

        Detects Mamba layers by the presence of both gate_proj and x_proj.
        Must run BEFORE normalize_gate_up_proj to prevent gate_proj from
        being consumed by the gate+up stacking.
        """
        # Find gate_proj weights that have a matching x_proj (Mamba pattern)
        for weight_name in list(weights.keys()):
            if "gate_proj" not in weight_name:
                continue
            x_name = weight_name.replace("gate_proj", "x_proj")
            if x_name not in weights:
                continue
            # This is a Mamba layer: stack gate_proj + x_proj → in_proj
            in_proj_name = weight_name.replace("gate_proj", "in_proj")
            cat_dim = weights[weight_name].dim() - 2
            weights[in_proj_name] = torch.cat(
                (weights[weight_name], weights[x_name]), cat_dim
            )
            weights.pop(weight_name)
            weights.pop(x_name)
```
**EN:** This block uses `LoRAAdapter._normalize_in_proj` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRAAdapter._normalize_in_proj` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 306-353: LoRAAdapter._normalize_in_proj_qkvz method
```python
    def _normalize_in_proj_qkvz(self, weights: Dict[str, torch.Tensor]):
        """Normalize in_proj_qkvz weights for GDN (GatedDeltaNet) layers like
        Qwen3.5.

        Two adapter formats are handled:

        1. Split: ``in_proj_q + in_proj_k + in_proj_v + in_proj_z`` are present
           as separate weights → concatenate them into ``in_proj_qkvz``.

        2. Already-merged: the adapter has a single ``in_proj_qkvz`` weight
           (PEFT trained against SGLang's fused Linear). The stacked buffer
           expects four per-slice ``A`` blocks, so repeat ``lora_A`` 4× along
           the rank dim. ``lora_B`` is already full-output-dim and matches
           the buffer directly.
        """
        for weight_name in list(weights.keys()):
            if "in_proj_q." in weight_name:
                k_name = weight_name.replace("in_proj_q", "in_proj_k")
                v_name = weight_name.replace("in_proj_q", "in_proj_v")
                z_name = weight_name.replace("in_proj_q", "in_proj_z")
                if (
                    k_name not in weights
                    or v_name not in weights
                    or z_name not in weights
                ):
                    continue
                qkvz_name = weight_name.replace("in_proj_q", "in_proj_qkvz")
                cat_dim = weights[weight_name].dim() - 2
                weights[qkvz_name] = torch.cat(
                    (
                        weights[weight_name],
                        weights[k_name],
                        weights[v_name],
                        weights[z_name],
                    ),
                    cat_dim,
                )
                weights.pop(weight_name)
                weights.pop(k_name)
                weights.pop(v_name)
                weights.pop(z_name)
            elif "in_proj_qkvz" in weight_name and "lora_A" in weight_name:
                # Already-merged adapter: replicate the shared A across the 4
                # stacked slots the buffer expects (q, k, v, z).
                ndim = weights[weight_name].dim()
                repeat_dims = [1] * ndim
                repeat_dims[ndim - 2] = 4
                weights[weight_name] = weights[weight_name].repeat(*repeat_dims)
```
**EN:** This block uses `LoRAAdapter._normalize_in_proj_qkvz` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRAAdapter._normalize_in_proj_qkvz` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 354-355: Class-level supporting statements
```python
            # else (in_proj_qkvz lora_B, or unrelated): no-op.
```
**EN:** This block contains supporting statements for the `LoRAAdapter` class, such as constants, attributes, registrations, or small helper expressions that shape the surrounding control flow.
**CN:** 该代码块包含`LoRAAdapter` 类的配套语句，例如常量、属性、注册逻辑或小型辅助表达式，它们共同塑造了周边控制流程。

### Lines 356-391: LoRAAdapter.normalize_gate_up_proj method
```python
    def normalize_gate_up_proj(
        self, weight_names: List[str], weights: Dict[str, torch.Tensor]
    ):
        for weight_name in weight_names:
            if "gate_proj" in weight_name:
                up_name = weight_name.replace("gate_proj", "up_proj")
                gate_up_name = weight_name.replace("gate_proj", "gate_up_proj")
                # PEFT can ship up_proj in two forms when there's no real
                # up_proj content: the key may be absent, or present as a
                # numel-zero placeholder. Treat both as "no up_proj".
                if up_name not in weights or weights[up_name].numel() == 0:
                    if self._is_non_gated_moe_weight(weight_name):
                        # Non-gated MoE expert: the gate_up_proj_moe buffer
                        # uses stacked_multiply=1 (per model override), so just
                        # rename without stacking.
                        weights[gate_up_name] = weights.pop(weight_name)
                        if up_name in weights:
                            weights.pop(up_name)
                        continue
                    # Gated path: buffer expects stacked [2r, hidden] (c=2);
                    # synthesize a properly-shaped zero up_proj.
                    weights[up_name] = torch.zeros_like(weights[weight_name])
                cat_dim = weights[weight_name].dim() - 2
                weights[gate_up_name] = torch.cat(
                    (weights[weight_name], weights[up_name]), cat_dim
                )
                weights.pop(weight_name)
                weights.pop(up_name)
            elif "gate_up_proj" in weight_name:
                # If gate_up_proj is already stacked, we normalize it following the SGL convention
                gate_up_name = weight_name
                if "lora_A" in weight_name:
                    ndim = weights[gate_up_name].dim()
                    repeat_dims = [1] * ndim
                    repeat_dims[ndim - 2] = 2
                    weights[gate_up_name] = weights[gate_up_name].repeat(*repeat_dims)
```
**EN:** This block uses `LoRAAdapter.normalize_gate_up_proj` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRAAdapter.normalize_gate_up_proj` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 392-396: Class-level supporting statements
```python
                # else: no-op as LoRA B weight is already stacked.
        # Orphan up_proj weights (no matching gate_proj) are kept as-is.
        # Models with non-gated MLP/shared-experts declare up_proj in
        # supported_lora_modules so they get their own buffer and wrapping.
```
**EN:** This block contains supporting statements for the `LoRAAdapter` class, such as constants, attributes, registrations, or small helper expressions that shape the surrounding control flow.
**CN:** 该代码块包含`LoRAAdapter` 类的配套语句，例如常量、属性、注册逻辑或小型辅助表达式，它们共同塑造了周边控制流程。

### Lines 397-422: LoRAAdapter.normalize_fused_qkv_a_proj method
```python
    def normalize_fused_qkv_a_proj(
        self, weight_names: List[str], weights: Dict[str, torch.Tensor]
    ):
        """Fuse separate q_a_proj and kv_a_proj_with_mqa LoRA weights into
        a single fused_qkv_a_proj_with_mqa entry (concat along dim 0 for
        both A and B), matching the DeepSeek MLA fused projection layout."""
        for weight_name in weight_names:
            if "q_a_proj" not in weight_name:
                continue
            if "fused_qkv_a_proj_with_mqa" in weight_name:
                continue

            q_a_name = weight_name
            kv_a_name = weight_name.replace("q_a_proj", "kv_a_proj_with_mqa")
            fused_name = weight_name.replace("q_a_proj", "fused_qkv_a_proj_with_mqa")

            kv_a_weight = (
                weights[kv_a_name]
                if kv_a_name in weights
                else torch.zeros_like(weights[q_a_name])
            )

            weights[fused_name] = torch.cat((weights[q_a_name], kv_a_weight), dim=0)
            weights.pop(q_a_name)
            if kv_a_name in weights:
                weights.pop(kv_a_name)
```
**EN:** This block uses `LoRAAdapter.normalize_fused_qkv_a_proj` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRAAdapter.normalize_fused_qkv_a_proj` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 424-433: LoRAAdapter.pin_weights_in_cpu method
```python
    def pin_weights_in_cpu(self):
        for layer in self.layers:
            for name, weight in layer.weights.items():
                layer.weights[name] = weight.pin_memory()

        for name, weight in self.embedding_layers.items():
            self.embedding_layers[name] = weight.pin_memory()

        for name, weight in self.added_tokens_embeddings.items():
            self.added_tokens_embeddings[name] = weight.pin_memory()
```
**EN:** This block uses `LoRAAdapter.pin_weights_in_cpu` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRAAdapter.pin_weights_in_cpu` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- LoRA adapter execution / LoRA 适配器执行
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.configs.load_config`
- `sglang.srt.layers.moe.fused_moe_triton.layer`
- `sglang.srt.layers.utils`
- `sglang.srt.lora.backend.base_backend`
- `sglang.srt.lora.lora_config`
- `sglang.srt.lora.utils`
- `sglang.srt.model_loader.loader`
- `sglang.srt.utils.hf_transformers_utils`
### External / 外部
- `torch`
- `logging` (stdlib)
- `re` (stdlib)
- `typing` (stdlib)
