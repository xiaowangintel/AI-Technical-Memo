# runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/moe_runner/runner.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `MoeRunner` and connects them to backend-specific paths such as `Triton`, `FlashInfer`, and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行。它提供了 `MoeRunner` 等符号，并把这些符号连接到 `Triton`、`FlashInfer` 以及 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

import logging
import os
from typing import TYPE_CHECKING, Any, Optional

from sglang.srt.layers.moe.moe_runner.base import (
    FusedOpPool,
    MoeRunnerConfig,
    PermuteMethodPool,
)
from sglang.srt.layers.moe.moe_runner.deep_gemm import DeepGemmRunnerCore
from sglang.srt.layers.moe.moe_runner.triton import TritonRunnerCore
from sglang.srt.layers.moe.moe_runner.triton_kernels import TritonKernelsRunnerCore
from sglang.srt.layers.moe.utils import get_moe_a2a_backend

if TYPE_CHECKING:
    from sglang.srt.batch_overlap.single_batch_overlap import DownGemmOverlapArgs
    from sglang.srt.layers.moe.moe_runner.base import MoeQuantInfo
    from sglang.srt.layers.moe.token_dispatcher.base import CombineInput, DispatchOutput
    from sglang.srt.layers.moe.utils import MoeRunnerBackend
    from sglang.srt.lora.lora_moe_runners import LoRAHooks

logger = logging.getLogger(__name__)
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `logging`, `os`, `typing.TYPE_CHECKING`, `typing.Any`, and `typing.Optional`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`logging`、`os`、`typing.TYPE_CHECKING`、`typing.Any` 以及 `typing.Optional`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 27-27: Class `MoeRunner` declaration and shared state
```python
class MoeRunner:
```
**EN:** This block introduces class `MoeRunner` and the state shared by its methods.
**CN:** 该代码块引入类 `MoeRunner`，并定义其方法共享的状态。

### Lines 28-94: `MoeRunner` initialization and state setup
```python
    def __init__(
        self,
        runner_backend: MoeRunnerBackend,
        config: MoeRunnerConfig,
        lora_enabled: bool = False,
    ):
        self.runner_backend = runner_backend
        self.config = config
        self.lora_enabled = lora_enabled

        self.fused_func = None

        if runner_backend.is_triton():
            self.runner_core = TritonRunnerCore(config)
        elif runner_backend.is_triton_kernels():
            self.runner_core = TritonKernelsRunnerCore(config)
        elif runner_backend.is_deep_gemm():
            self.runner_core = DeepGemmRunnerCore(config)
        elif runner_backend.is_aiter():
            from sglang.srt.layers.moe.moe_runner.aiter import AiterRunnerCore

            self.runner_core = AiterRunnerCore(config)
        elif runner_backend.is_marlin():
            if lora_enabled:
                from sglang.srt.lora.lora_moe_runner_marlin import MarlinLoraRunnerCore

                self.runner_core = MarlinLoraRunnerCore(config)
            else:
                self.runner_core = None  # Marlin only supports fused path
        elif (
            runner_backend.is_flashinfer_trtllm()
            or runner_backend.is_flashinfer_trtllm_routed()
        ):
            self.runner_core = None  # FlashInfer TRT-LLM only supports fused path
        elif runner_backend.is_flashinfer_cutedsl():
            self.runner_core = None  # FlashInfer CuteDSL only supports fused path
        else:
            raise NotImplementedError(f"Unsupported runner backend: {runner_backend}")

        # Skip fused func if LoRA is enabled (LoRA requires non-fused path)
        if not lora_enabled:
            a2a_backend_name = get_moe_a2a_backend().value
            runner_backend_name = runner_backend.value

            # TODO(cwan): add a server argument to disable fused func
            self.fused_func = FusedOpPool.get_fused_func(
                a2a_backend_name, runner_backend_name
            )

            if self.runner_core is None and self.fused_func is None:
                raise NotImplementedError(
                    f"Runner backend {runner_backend} requires a fused func for a2a backend "
                    f"{a2a_backend_name}, but none is registered."
                )

        self.down_gemm_overlap_args: Optional[DownGemmOverlapArgs] = None
        self.meta_overlap_args: Optional[dict] = None

        SGLANG_CI_DISABLE_MOE_FUSED_FUNC = os.environ.get(
            "SGLANG_CI_DISABLE_MOE_FUSED_FUNC", "0"
        )
        if SGLANG_CI_DISABLE_MOE_FUSED_FUNC == "1":
            logger.info(
                "SGLANG_CI_DISABLE_MOE_FUSED_FUNC is set to 1, disabling fused func"
            )
            self.fused_func = None
```
**EN:** This block defines `MoeRunner.__init__` and contains the main logic for this step. It mainly invokes `runner_backend.is_triton`, `os.environ.get`, `TritonRunnerCore`, `runner_backend.is_triton_kernels`, and `FusedOpPool.get_fused_func`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.runner_backend`, `self.config`, `self.lora_enabled`, `self.fused_func`, and `self.down_gemm_overlap_args` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MoeRunner.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `runner_backend.is_triton`、`os.environ.get`、`TritonRunnerCore`、`runner_backend.is_triton_kernels` 以及 `FusedOpPool.get_fused_func`，说明该流程会编排底层辅助函数或计算内核。 像 `self.runner_backend`、`self.config`、`self.lora_enabled`、`self.fused_func` 以及 `self.down_gemm_overlap_args` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 95-162: Function `MoeRunner.run` and its core logic
```python
    def run(
        self, dispatch_output: DispatchOutput, quant_info: MoeQuantInfo, lora_info=None
    ) -> CombineInput:
        if self.fused_func is not None and not self.lora_enabled:
            return self.fused_func(dispatch_output, quant_info, self.config)

        assert self.runner_core is not None

        def _maybe_build_lora_hooks(_runner_input: Any) -> LoRAHooks:
            from sglang.srt.layers.moe.token_dispatcher.base import DispatchOutput
            from sglang.srt.lora.lora_moe_runners import build_lora_hooks

            if isinstance(_runner_input, DispatchOutput):
                hidden_states, topk_ids = (
                    _runner_input.hidden_states,
                    _runner_input.topk_output.topk_ids,
                )
            else:
                hidden_states = _runner_input.hidden_states
                topk_ids = getattr(_runner_input, "topk_ids", None)
            if self.lora_enabled and lora_info is not None:
                return build_lora_hooks(
                    hidden_states,
                    lora_info,
                    topk_ids,
                )
            return None

        # Runners that handle dispatch_output directly (e.g., MarlinRunnerCore)
        # bypass the pre-permute step and do their own alignment internally.
        if hasattr(self.runner_core, "run_from_dispatch"):
            hooks = _maybe_build_lora_hooks(dispatch_output)
            return self.runner_core.run_from_dispatch(
                dispatch_output, quant_info, self.config, hooks=hooks
            )

        dispatch_format = dispatch_output.format.value
        runner_format = self.runner_core.runner_backend.value
        self.pre_permute_func = PermuteMethodPool.get_pre_permute(
            dispatch_format, runner_format
        )

        running_state = {}
        if self.down_gemm_overlap_args is not None:
            running_state["down_gemm_overlap_args"] = self.down_gemm_overlap_args
        if self.meta_overlap_args is not None:
            running_state["meta_overlap_args"] = self.meta_overlap_args

        runner_input = self.pre_permute_func(
            dispatch_output, quant_info, self.config, running_state
        )

        hooks = _maybe_build_lora_hooks(runner_input)

        runner_output = self.runner_core.run(
            runner_input, quant_info, running_state, hooks=hooks
        )
        runner_format = self.runner_core.runner_backend.value
        combine_format = dispatch_output.format.value
        self.post_permute_func = PermuteMethodPool.get_post_permute(
            runner_format, combine_format
        )
        combine_input = self.post_permute_func(
            runner_output, quant_info, self.config, running_state
        )

        return combine_input
```
**EN:** This block defines `MoeRunner.run` and contains the main logic for this step. It mainly invokes `hasattr`, `PermuteMethodPool.get_pre_permute`, `self.pre_permute_func`, `_maybe_build_lora_hooks`, and `self.runner_core.run`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `dispatch_format`, `runner_format`, `self.pre_permute_func`, `running_state`, and `runner_input` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeRunner.run`，并承载这一阶段的核心逻辑。 它主要调用 `hasattr`、`PermuteMethodPool.get_pre_permute`、`self.pre_permute_func`、`_maybe_build_lora_hooks` 以及 `self.runner_core.run`，说明该流程会编排底层辅助函数或计算内核。 像 `dispatch_format`、`runner_format`、`self.pre_permute_func`、`running_state` 以及 `runner_input` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 163-168: `MoeRunner.set_overlap_args` setter for overlap args
```python
    def set_overlap_args(
        self, down_gemm_overlap_args: DownGemmOverlapArgs, meta_overlap_args: dict
    ):
        self.down_gemm_overlap_args = down_gemm_overlap_args
        self.meta_overlap_args = meta_overlap_args
```
**EN:** This block defines `MoeRunner.set_overlap_args` and contains the main logic for this step. Intermediate names such as `self.down_gemm_overlap_args` and `self.meta_overlap_args` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MoeRunner.set_overlap_args`，并承载这一阶段的核心逻辑。 像 `self.down_gemm_overlap_args` 和 `self.meta_overlap_args` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 169-171: Function `MoeRunner.clear_overlap_args` and its core logic
```python
    def clear_overlap_args(self) -> None:
        self.down_gemm_overlap_args = None
        self.meta_overlap_args = None
```
**EN:** This block defines `MoeRunner.clear_overlap_args` and contains the main logic for this step. Intermediate names such as `self.down_gemm_overlap_args` and `self.meta_overlap_args` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MoeRunner.clear_overlap_args`，并承载这一阶段的核心逻辑。 像 `self.down_gemm_overlap_args` 和 `self.meta_overlap_args` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `MoeRunner`. / **主要符号**：核心入口包括 `MoeRunner`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `logging`, `os`, `typing.TYPE_CHECKING`, `typing.Any`, and `typing.Optional` / **标准库**：`__future__.annotations`、`logging`、`os`、`typing.TYPE_CHECKING`、`typing.Any` 以及 `typing.Optional`
- **Internal SGLang modules**: `sglang.srt.layers.moe.moe_runner.base.FusedOpPool`, `sglang.srt.layers.moe.moe_runner.base.MoeRunnerConfig`, `sglang.srt.layers.moe.moe_runner.base.PermuteMethodPool`, `sglang.srt.layers.moe.moe_runner.deep_gemm.DeepGemmRunnerCore`, `sglang.srt.layers.moe.moe_runner.triton.TritonRunnerCore`, `sglang.srt.layers.moe.moe_runner.triton_kernels.TritonKernelsRunnerCore`, `sglang.srt.layers.moe.utils.get_moe_a2a_backend`, `sglang.srt.batch_overlap.single_batch_overlap.DownGemmOverlapArgs`, `sglang.srt.layers.moe.moe_runner.base.MoeQuantInfo`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInput`, `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput`, and `sglang.srt.layers.moe.utils.MoeRunnerBackend` / **SGLang 内部模块**：`sglang.srt.layers.moe.moe_runner.base.FusedOpPool`、`sglang.srt.layers.moe.moe_runner.base.MoeRunnerConfig`、`sglang.srt.layers.moe.moe_runner.base.PermuteMethodPool`、`sglang.srt.layers.moe.moe_runner.deep_gemm.DeepGemmRunnerCore`、`sglang.srt.layers.moe.moe_runner.triton.TritonRunnerCore`、`sglang.srt.layers.moe.moe_runner.triton_kernels.TritonKernelsRunnerCore`、`sglang.srt.layers.moe.utils.get_moe_a2a_backend`、`sglang.srt.batch_overlap.single_batch_overlap.DownGemmOverlapArgs`、`sglang.srt.layers.moe.moe_runner.base.MoeQuantInfo`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInput`、`sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput` 以及 `sglang.srt.layers.moe.utils.MoeRunnerBackend`
