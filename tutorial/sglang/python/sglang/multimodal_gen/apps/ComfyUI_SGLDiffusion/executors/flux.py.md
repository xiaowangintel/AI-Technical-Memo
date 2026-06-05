# flux.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/apps/ComfyUI_SGLDiffusion/executors/flux.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the ComfyUI integration layer. It centers on `FluxExecutor`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Flux executor for SGLang Diffusion ComfyUI integration. / 该文件属于ComfyUI 集成层。它围绕 `FluxExecutor` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: module setup and imports / 模块初始化与导入
```python
"""
Flux executor for SGLang Diffusion ComfyUI integration.
"""

import torch
```
**EN:** This block establishes the module context and imports `torch`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `torch`。这些依赖为后续实现提供所需符号。

### Lines 7-15: supporting statements / 辅助语句
```python
try:
    from sglang.multimodal_gen.configs.sample.sampling_params import SamplingParams
    from sglang.multimodal_gen.runtime.entrypoints.utils import prepare_request
except ImportError:
    print(
        "Error: sglang.multimodal_gen is not installed. Please install it using 'pip install sglang[diffusion]'"
    )

from .base import SGLDiffusionExecutor
```
**EN:** This block gathers supporting statements at module scope. The code collaborates with `print`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 代码会与 `print` 协同工作。

### Lines 18-20: `FluxExecutor` class overview / `FluxExecutor` 类概览
```python
class FluxExecutor(SGLDiffusionExecutor):
    """Executor for Flux models in ComfyUI."""
```
**EN:** This block defines class `FluxExecutor`. Executor for Flux models in ComfyUI. It inherits from `SGLDiffusionExecutor`.
**CN:** 该代码块定义了类 `FluxExecutor`。 它用于封装 flux executor 相关行为。 它继承自 `SGLDiffusionExecutor`。

### Lines 21-22: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, generator, model_path, model, config):
        super().__init__(generator, model_path, model, config)
```
**EN:** This block defines method `__init__` on `FluxExecutor`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `generator`, `model_path`, `model`, and `config` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxExecutor` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `generator`、`model_path`、`model` 和 `config` 等参数驱动。

### Lines 24-69: `forward` implementation / `forward` 实现
```python
    def forward(self, x, timestep, context, y=None, guidance=None, **kwargs):
        """Forward pass for Flux model."""
        hidden_states = self._pack_latents(x)
        timesteps = timestep * 1000.0
        encoder_hidden_states = context
        pooled_projections = y
        guidance = guidance * 1000.0

        B, C, H, W = x.shape
        height = H * 8
        width = W * 8
        # Create SamplingParams
        sampling_params = SamplingParams.from_user_sampling_params_args(
            self.model_path,
            server_args=self.generator.server_args,
            prompt=" ",
            guidance_scale=3.5,  # Flux typically uses embedded_cfg_scale=3.5
            height=height,
            width=width,
            num_frames=1,
            num_inference_steps=1,
            save_output=False,
            suppress_logs=self.should_suppress_logs(timestep),
        )

        # Prepare request (converts SamplingParams to Req)
        req = prepare_request(
            server_args=self.generator.server_args,
            sampling_params=sampling_params,
        )
        req.latents = hidden_states  # Set as [B, S, D] format directly
        req.timesteps = timesteps  # ComfyUI's timesteps parameter
        req.prompt_embeds = [pooled_projections, encoder_hidden_states]  # [CLIP, T5]
        req.raw_latent_shape = torch.tensor(hidden_states.shape, dtype=torch.long)

        # Set pooled_projections (required by Flux)
        req.pooled_embeds = [pooled_projections]  # List format as per Req definition
        req.do_classifier_free_guidance = False
        req.generator = [
            torch.Generator("cuda") for _ in range(req.num_outputs_per_prompt)
        ]

        # Send request to scheduler
        output_batch = self.generator._send_to_scheduler_and_wait_for_response([req])
        noise_pred = output_batch.noise_pred
        return self._unpack_latents(noise_pred, H, W, C).to(x.device)
```
**EN:** This block defines method `forward` on `FluxExecutor`. Forward pass for Flux model. Key calls include `self._pack_latents`, `SamplingParams.from_user_sampling_params_args`, `prepare_request`, `torch.tensor`, and `self.generator._send_to_scheduler_and_wait_for_response`. Parameters such as `x`, `timestep`, `context`, `y`, and `guidance` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxExecutor` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `self._pack_latents`、`SamplingParams.from_user_sampling_params_args`、`prepare_request`、`torch.tensor` 和 `self.generator._send_to_scheduler_and_wait_for_response`。 本段逻辑主要由 `x`、`timestep`、`context`、`y` 和 `guidance` 等参数驱动。

## Key Concepts / 关键概念
- `FluxExecutor`: Executor for Flux models in ComfyUI. / 核心类，用于封装 flux executor 相关行为。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `.base`, `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.entrypoints.utils`

- **Total lines / 总行数**: 69
