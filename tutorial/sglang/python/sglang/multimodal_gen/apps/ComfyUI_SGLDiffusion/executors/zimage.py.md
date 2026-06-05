# zimage.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/apps/ComfyUI_SGLDiffusion/executors/zimage.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the ComfyUI integration layer. It centers on `ZImageExecutor`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: ZImage executor for SGLang Diffusion ComfyUI integration. / 该文件属于ComfyUI 集成层。它围绕 `ZImageExecutor` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: module setup and imports / 模块初始化与导入
```python
"""
ZImage executor for SGLang Diffusion ComfyUI integration.
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

### Lines 18-20: `ZImageExecutor` class overview / `ZImageExecutor` 类概览
```python
class ZImageExecutor(SGLDiffusionExecutor):
    """Executor for ZImage models in ComfyUI."""
```
**EN:** This block defines class `ZImageExecutor`. Executor for ZImage models in ComfyUI. It inherits from `SGLDiffusionExecutor`.
**CN:** 该代码块定义了类 `ZImageExecutor`。 它用于封装 zimage executor 相关行为。 它继承自 `SGLDiffusionExecutor`。

### Lines 21-22: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, generator, model_path, model, config):
        super().__init__(generator, model_path, model, config)
```
**EN:** This block defines method `__init__` on `ZImageExecutor`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `generator`, `model_path`, `model`, and `config` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImageExecutor` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `generator`、`model_path`、`model` 和 `config` 等参数驱动。

### Lines 24-64: `forward` implementation / `forward` 实现
```python
    def forward(self, x, timesteps, context, **kwargs):
        """Forward pass for ZImage model."""
        B, C, H, W = x.shape
        height = H * 8
        width = W * 8
        sampling_params = SamplingParams.from_user_sampling_params_args(
            self.model_path,
            server_args=self.generator.server_args,
            prompt=" ",
            guidance_scale=1.0,
            height=height,
            width=width,
            num_frames=1,  # For images
            num_inference_steps=1,  # Single step for ComfyUI
            save_output=False,
            suppress_logs=self.should_suppress_logs(timesteps),
        )

        # Prepare request (converts SamplingParams to Req)
        req = prepare_request(
            server_args=self.generator.server_args,
            sampling_params=sampling_params,
        )
        latents = x.unsqueeze(2)
        context = context.squeeze(0)
        # Set ComfyUI-specific inputs directly on the Req object
        req.latents = latents  # ComfyUI's x parameter
        req.timesteps = timesteps * 1000.0  # ComfyUI's timesteps parameter
        req.prompt_embeds = [
            context
        ]  # ComfyUI's context parameter (must be List[Tensor])
        req.raw_latent_shape = torch.tensor(latents.shape, dtype=torch.long)
        req.do_classifier_free_guidance = False
        req.generator = [
            torch.Generator("cuda") for _ in range(req.num_outputs_per_prompt)
        ]

        output_batch = self.generator._send_to_scheduler_and_wait_for_response([req])
        noise_pred = output_batch.noise_pred

        return noise_pred.permute(1, 0, 2, 3).to(x.device)
```
**EN:** This block defines method `forward` on `ZImageExecutor`. Forward pass for ZImage model. Key calls include `SamplingParams.from_user_sampling_params_args`, `prepare_request`, `x.unsqueeze`, `context.squeeze`, and `torch.tensor`. Parameters such as `x`, `timesteps`, and `context` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImageExecutor` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `SamplingParams.from_user_sampling_params_args`、`prepare_request`、`x.unsqueeze`、`context.squeeze` 和 `torch.tensor`。 本段逻辑主要由 `x`、`timesteps` 和 `context` 等参数驱动。

## Key Concepts / 关键概念
- `ZImageExecutor`: Executor for ZImage models in ComfyUI. / 核心类，用于封装 zimage executor 相关行为。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `.base`, `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.entrypoints.utils`

- **Total lines / 总行数**: 64
