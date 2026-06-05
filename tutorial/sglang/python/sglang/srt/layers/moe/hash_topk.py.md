# hash_topk.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/hash_topk.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `HashTopK` and connects them to backend-specific paths such as `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行。它提供了 `HashTopK` 等符号，并把这些符号连接到 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Imports, constants, and runtime setup
```python
from __future__ import annotations

import logging
from typing import Optional, Tuple

import torch
from torch import nn

from sglang.srt.environ import envs
from sglang.srt.eplb.expert_location_dispatch import (
    ExpertLocationDispatchInfo,
    topk_ids_logical_to_physical,
)
from sglang.srt.layers.moe.topk import (
    StandardTopKOutput,
    _mask_topk_ids_padded_region,
)
from sglang.srt.utils import is_hip

logger = logging.getLogger(__name__)
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `logging`, `typing.Optional`, `typing.Tuple`, `torch`, and `torch.nn`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`logging`、`typing.Optional`、`typing.Tuple`、`torch` 以及 `torch.nn`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 23-23: Class `HashTopK` declaration and shared state
```python
class HashTopK(nn.Module):
```
**EN:** This block introduces class `HashTopK` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `HashTopK`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。

### Lines 24-46: `HashTopK` initialization and state setup
```python
    def __init__(
        self,
        topk,
        num_experts,
        num_fused_shared_experts,
        vocab_size,
        scoring_func="sqrtsoftplus",
        routed_scaling_factor=1.5,
        apply_routed_scaling_factor_on_output=False,
    ):
        super().__init__()
        self.num_experts = num_experts
        self.topk = topk
        self.routed_scaling_factor = routed_scaling_factor
        self.num_fused_shared_experts = num_fused_shared_experts
        self.score_func = scoring_func
        self.tid2eid = nn.Parameter(
            torch.empty(vocab_size, topk - num_fused_shared_experts, dtype=torch.int32),
            requires_grad=False,
        )

        assert not apply_routed_scaling_factor_on_output, "not implemented"
```
**EN:** This block defines `HashTopK.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `nn.Parameter`, and `torch.empty`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.num_experts`, `self.topk`, `self.routed_scaling_factor`, `self.num_fused_shared_experts`, and `self.score_func` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `HashTopK.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`nn.Parameter` 以及 `torch.empty`，说明该流程会编排底层辅助函数或计算内核。 像 `self.num_experts`、`self.topk`、`self.routed_scaling_factor`、`self.num_fused_shared_experts` 以及 `self.score_func` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 47-53: Function `HashTopK.empty_topk_output` and its core logic
```python
    def empty_topk_output(self, device: torch.device):
        topk = self.topk - self.num_fused_shared_experts
        topk_weights = torch.empty((0, topk), dtype=torch.float32, device=device)
        topk_ids = torch.full((0, topk), -1, dtype=torch.int32, device=device)
        router_logits = torch.empty((0, topk), dtype=torch.float32, device=device)
        return StandardTopKOutput(topk_weights, topk_ids, router_logits)
```
**EN:** This block defines `HashTopK.empty_topk_output` and contains the main logic for this step. It mainly invokes `torch.empty`, `torch.full`, and `StandardTopKOutput`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `topk`, `topk_weights`, `topk_ids`, and `router_logits` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `HashTopK.empty_topk_output`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`、`torch.full` 以及 `StandardTopKOutput`，说明该流程会编排底层辅助函数或计算内核。 像 `topk`、`topk_weights`、`topk_ids` 以及 `router_logits` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 54-98: Internal helper `HashTopK._forward_torch`
```python
    def _forward_torch(
        self, router_logits: torch.Tensor, input_ids: torch.Tensor
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        if self.score_func == "softmax":
            scores = router_logits.softmax(dim=-1)
        elif self.score_func == "sigmoid":
            scores = router_logits.sigmoid()
        else:
            scores = torch.nn.functional.softplus(router_logits).sqrt()

        num_token = scores.shape[0]

        topk_ids = torch.zeros(
            (num_token, self.topk), dtype=torch.int32, device=scores.device
        )
        topk_weights = torch.zeros(
            (num_token, self.topk), dtype=scores.dtype, device=scores.device
        )

        if self.num_fused_shared_experts == 1:
            topk_ids[:, :-1] = self.tid2eid[input_ids]
            topk_weights[:, :-1] = scores.gather(1, topk_ids[:, :-1])

            if self.score_func != "softmax":
                topk_weights[:, :-1] /= topk_weights[:, :-1].sum(dim=-1, keepdim=True)

            topk_ids[:, -1] = torch.randint(
                low=self.num_experts,
                high=self.num_experts + self.num_fused_shared_experts,
                size=(num_token,),
                dtype=topk_ids.dtype,
                device=topk_ids.device,
            )

            topk_weights[:, -1] = (
                topk_weights[:, :-1].sum(dim=-1) / self.routed_scaling_factor
            )
        else:
            topk_ids[:, :] = self.tid2eid[input_ids]
            topk_weights[:, :] = scores.gather(1, topk_ids[:, :])
            if self.score_func != "softmax":
                topk_weights[:, :] /= topk_weights[:, :].sum(dim=-1, keepdim=True)

        return topk_weights, topk_ids
```
**EN:** This block defines `HashTopK._forward_torch` and contains the main logic for this step. It mainly invokes `torch.zeros`, `router_logits.softmax`, `scores.gather`, `torch.randint`, and `router_logits.sigmoid`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_token`, `topk_ids`, `topk_weights`, and `scores` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `HashTopK._forward_torch`，并承载这一阶段的核心逻辑。 它主要调用 `torch.zeros`、`router_logits.softmax`、`scores.gather`、`torch.randint` 以及 `router_logits.sigmoid`，说明该流程会编排底层辅助函数或计算内核。 像 `num_token`、`topk_ids`、`topk_weights` 以及 `scores` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 99-133: `HashTopK.forward` main forward path
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        input_ids: torch.Tensor,
        num_token_non_padded: Optional[torch.Tensor] = None,
        expert_location_dispatch_info: Optional[ExpertLocationDispatchInfo] = None,
    ):
        assert (
            input_ids.shape[0] == hidden_states.shape[0] == router_logits.shape[0]
        ), f"{input_ids.shape=} {hidden_states.shape=} {router_logits.shape=}"

        if envs.SGLANG_OPT_USE_FUSED_HASH_TOPK.get():
            from sglang.jit_kernel.deepseek_v4 import hash_topk

            topk_weights, topk_ids = hash_topk(
                router_logits=router_logits,
                input_ids=input_ids,
                tid2eid=self.tid2eid,
                num_fused_shared_experts=self.num_fused_shared_experts,
                routed_scaling_factor=self.routed_scaling_factor,
                scoring_func=self.score_func,
            )
        else:
            topk_weights, topk_ids = self._forward_torch(router_logits, input_ids)

        if is_hip():
            topk_weights = topk_weights.to(torch.float32)

        topk_ids = topk_ids_logical_to_physical(topk_ids, expert_location_dispatch_info)
        _mask_topk_ids_padded_region(topk_ids, num_token_non_padded)
        topk_output = StandardTopKOutput(
            topk_weights=topk_weights, topk_ids=topk_ids, router_logits=router_logits
        )
        return topk_output
```
**EN:** This block defines `HashTopK.forward` and contains the main logic for this step. It mainly invokes `envs.SGLANG_OPT_USE_FUSED_HASH_TOPK.get`, `is_hip`, `topk_ids_logical_to_physical`, `_mask_topk_ids_padded_region`, and `StandardTopKOutput`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `topk_ids`, `topk_output`, and `topk_weights` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `HashTopK.forward`，并承载这一阶段的核心逻辑。 它主要调用 `envs.SGLANG_OPT_USE_FUSED_HASH_TOPK.get`、`is_hip`、`topk_ids_logical_to_physical`、`_mask_topk_ids_padded_region` 以及 `StandardTopKOutput`，说明该流程会编排底层辅助函数或计算内核。 像 `topk_ids`、`topk_output` 以及 `topk_weights` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `HashTopK`. / **主要符号**：核心入口包括 `HashTopK`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Sampling policy**: Covers post-processing of logits and token selection strategies. / **采样策略**：涵盖 logits 后处理与 token 选择策略。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `logging`, `typing.Optional`, and `typing.Tuple` / **标准库**：`__future__.annotations`、`logging`、`typing.Optional` 以及 `typing.Tuple`
- **Third-party**: `torch` and `torch.nn` / **第三方依赖**：`torch` 和 `torch.nn`
- **Internal SGLang modules**: `sglang.srt.environ.envs`, `sglang.srt.eplb.expert_location_dispatch.ExpertLocationDispatchInfo`, `sglang.srt.eplb.expert_location_dispatch.topk_ids_logical_to_physical`, `sglang.srt.layers.moe.topk.StandardTopKOutput`, `sglang.srt.layers.moe.topk._mask_topk_ids_padded_region`, `sglang.srt.utils.is_hip`, and `sglang.jit_kernel.deepseek_v4.hash_topk` / **SGLang 内部模块**：`sglang.srt.environ.envs`、`sglang.srt.eplb.expert_location_dispatch.ExpertLocationDispatchInfo`、`sglang.srt.eplb.expert_location_dispatch.topk_ids_logical_to_physical`、`sglang.srt.layers.moe.topk.StandardTopKOutput`、`sglang.srt.layers.moe.topk._mask_topk_ids_padded_region`、`sglang.srt.utils.is_hip` 以及 `sglang.jit_kernel.deepseek_v4.hash_topk`
