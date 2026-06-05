# qwen_image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/apps/ComfyUI_SGLDiffusion/executors/qwen_image.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the ComfyUI integration layer. It centers on `QwenImageExecutor`, and `QwenImageEditExecutor`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: QwenImage executor for SGLang Diffusion ComfyUI integration. / 该文件属于ComfyUI 集成层。它围绕 `QwenImageExecutor` 和 `QwenImageEditExecutor` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: module setup and imports / 模块初始化与导入
```python
"""
QwenImage executor for SGLang Diffusion ComfyUI integration.
"""

import torch
```
**EN:** This block establishes the module context and imports `torch`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `torch`。这些依赖为后续实现提供所需符号。

### Lines 7-17: supporting statements / 辅助语句
```python
try:
    from sglang.multimodal_gen.configs.sample.sampling_params import SamplingParams
    from sglang.multimodal_gen.runtime.entrypoints.utils import prepare_request
except ImportError:
    print(
        "Error: sglang.multimodal_gen is not installed. Please install it using 'pip install sglang[diffusion]'"
    )

import comfy.ldm.common_dit

from .base import SGLDiffusionExecutor
```
**EN:** This block gathers supporting statements at module scope. The code collaborates with `print`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 代码会与 `print` 协同工作。

### Lines 20-22: `QwenImageExecutor` class overview / `QwenImageExecutor` 类概览
```python
class QwenImageExecutor(SGLDiffusionExecutor):
    """Executor for QwenImage models in ComfyUI."""
```
**EN:** This block defines class `QwenImageExecutor`. Executor for QwenImage models in ComfyUI. It inherits from `SGLDiffusionExecutor`.
**CN:** 该代码块定义了类 `QwenImageExecutor`。 它用于封装 qwen image executor 相关行为。 它继承自 `SGLDiffusionExecutor`。

### Lines 23-25: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, generator, model_path, model, config):
        super().__init__(generator, model_path, model, config)
        self.patch_size = 2
```
**EN:** This block defines method `__init__` on `QwenImageExecutor`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `generator`, `model_path`, `model`, and `config` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageExecutor` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `generator`、`model_path`、`model` 和 `config` 等参数驱动。

### Lines 27-50: `_pack_latents` implementation / `_pack_latents` 实现
```python
    def _pack_latents(self, x):
        """Process hidden states for QwenImage model."""
        bs, c, t, h, w = x.shape
        patch_size = self.patch_size
        latents = comfy.ldm.common_dit.pad_to_patch_size(
            x, (1, self.patch_size, self.patch_size)
        )
        orig_shape = latents.shape
        latents = latents.view(
            orig_shape[0],
            orig_shape[1],
            orig_shape[-3],
            orig_shape[-2] // 2,
            2,
            orig_shape[-1] // 2,
            2,
        )
        latents = latents.permute(0, 2, 3, 5, 1, 4, 6)
        latents = latents.reshape(
            orig_shape[0],
            orig_shape[-3] * (orig_shape[-2] // 2) * (orig_shape[-1] // 2),
            orig_shape[1] * 4,
        )
        return latents, orig_shape
```
**EN:** This block defines method `_pack_latents` on `QwenImageExecutor`. Process hidden states for QwenImage model. Key calls include `comfy.ldm.common_dit.pad_to_patch_size`, `latents.view`, `latents.permute`, and `latents.reshape`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageExecutor` 的方法 `_pack_latents`。 它用于打包latents。 关键调用包括 `comfy.ldm.common_dit.pad_to_patch_size`、`latents.view`、`latents.permute` 和 `latents.reshape`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 52-65: `_unpack_latents` implementation / `_unpack_latents` 实现
```python
    def _unpack_latents(self, latents, num_embeds, orig_shape, x):
        """Unpack hidden states from packed format to standard format."""
        latents = latents[:, :num_embeds].view(
            orig_shape[0],
            orig_shape[-3],
            orig_shape[-2] // 2,
            orig_shape[-1] // 2,
            orig_shape[1],
            2,
            2,
        )
        latents = latents.permute(0, 4, 1, 2, 5, 3, 6)
        latents = latents.reshape(orig_shape)[:, :, :, : x.shape[-2], : x.shape[-1]]
        return latents
```
**EN:** This block defines method `_unpack_latents` on `QwenImageExecutor`. Unpack hidden states from packed format to standard format. Key calls include `latents.view`, `latents.permute`, and `latents.reshape`. Parameters such as `latents`, `num_embeds`, `orig_shape`, and `x` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageExecutor` 的方法 `_unpack_latents`。 它用于解包latents。 关键调用包括 `latents.view`、`latents.permute` 和 `latents.reshape`。 本段逻辑主要由 `latents`、`num_embeds`、`orig_shape` 和 `x` 等参数驱动。

### Lines 67-105: `forward` implementation / `forward` 实现
```python
    def forward(self, x, timestep, context, **kwargs):
        """Forward pass for QwenImage model."""
        latents, orig_shape = self._pack_latents(x)
        num_embeds = latents.shape[1]
        height = orig_shape[-2] * 8
        width = orig_shape[-1] * 8

        sampling_params = SamplingParams.from_user_sampling_params_args(
            self.model_path,
            server_args=self.generator.server_args,
            prompt=" ",
            guidance_scale=1.0,
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
        # Set ComfyUI-specific inputs directly on the Req object
        req.latents = latents
        req.timesteps = timestep * 1000.0
        req.prompt_embeds = [context]
        req.raw_latent_shape = torch.tensor(latents.shape, dtype=torch.long)
        req.do_classifier_free_guidance = False
        req.generator = [
            torch.Generator("cuda") for _ in range(req.num_outputs_per_prompt)
        ]

        output_batch = self.generator._send_to_scheduler_and_wait_for_response([req])
        noise_pred = output_batch.noise_pred

        return self._unpack_latents(noise_pred, num_embeds, orig_shape, x)
```
**EN:** This block defines method `forward` on `QwenImageExecutor`. Forward pass for QwenImage model. Key calls include `self._pack_latents`, `SamplingParams.from_user_sampling_params_args`, `prepare_request`, `torch.tensor`, and `self.generator._send_to_scheduler_and_wait_for_response`. Parameters such as `x`, `timestep`, and `context` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageExecutor` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `self._pack_latents`、`SamplingParams.from_user_sampling_params_args`、`prepare_request`、`torch.tensor` 和 `self.generator._send_to_scheduler_and_wait_for_response`。 本段逻辑主要由 `x`、`timestep` 和 `context` 等参数驱动。

### Lines 108-110: `QwenImageEditExecutor` class overview / `QwenImageEditExecutor` 类概览
```python
class QwenImageEditExecutor(QwenImageExecutor):
    """Executor for QwenImageEdit models in ComfyUI."""
```
**EN:** This block defines class `QwenImageEditExecutor`. Executor for QwenImageEdit models in ComfyUI. It inherits from `QwenImageExecutor`.
**CN:** 该代码块定义了类 `QwenImageEditExecutor`。 它用于封装 qwen image edit executor 相关行为。 它继承自 `QwenImageExecutor`。

### Lines 111-112: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, generator, model_path, model, config):
        super().__init__(generator, model_path, model, config)
```
**EN:** This block defines method `__init__` on `QwenImageEditExecutor`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `generator`, `model_path`, `model`, and `config` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditExecutor` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `generator`、`model_path`、`model` 和 `config` 等参数驱动。

### Lines 114-174: `forward` implementation / `forward` 实现
```python
    def forward(
        self,
        x,
        timestep,
        context,
        attention_mask=None,
        ref_latents=None,
        additional_t_cond=None,
        transformer_options={},
        **kwargs,
    ):
        """Forward pass for QwenImageEdit model."""
        latents, orig_shape = self._pack_latents(x)
        num_embeds = latents.shape[1]
        height = orig_shape[-2] * 8
        width = orig_shape[-1] * 8

        # Prepare vae_image_sizes for the condition image (ref_latents)
        vae_image_sizes = []
        pack_ref_latents = None

        # TODO: sgld now don't support multiple condition images, so we only support one condition image for now.
        if ref_latents is not None and len(ref_latents) > 0:
            pack_ref_latents, orig_ref_shape = self._pack_latents(ref_latents[0])
            vae_image_sizes = [(orig_ref_shape[-1], orig_ref_shape[-2])]

        sampling_params = SamplingParams.from_user_sampling_params_args(
            self.model_path,
            server_args=self.generator.server_args,
            prompt=" ",
            guidance_scale=1.0,
            image_path="",
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
        # Set ComfyUI-specific inputs directly on the Req object
        req.latents = latents
        req.image_latent = pack_ref_latents
        req.timesteps = timestep * 1000.0
        req.vae_image_sizes = vae_image_sizes
        req.prompt_embeds = [context]
        req.raw_latent_shape = torch.tensor(latents.shape, dtype=torch.long)
        req.do_classifier_free_guidance = False
        req.generator = [
            torch.Generator("cuda") for _ in range(req.num_outputs_per_prompt)
        ]

        output_batch = self.generator._send_to_scheduler_and_wait_for_response([req])
        noise_pred = output_batch.noise_pred

        return self._unpack_latents(noise_pred, num_embeds, orig_shape, x)
```
**EN:** This block defines method `forward` on `QwenImageEditExecutor`. Forward pass for QwenImageEdit model. Key calls include `self._pack_latents`, `SamplingParams.from_user_sampling_params_args`, `prepare_request`, `torch.tensor`, and `self.generator._send_to_scheduler_and_wait_for_response`. The implementation branches on conditions. Parameters such as `x`, `timestep`, `context`, `attention_mask`, and `ref_latents` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditExecutor` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `self._pack_latents`、`SamplingParams.from_user_sampling_params_args`、`prepare_request`、`torch.tensor` 和 `self.generator._send_to_scheduler_and_wait_for_response`。 实现中包含条件分支。 本段逻辑主要由 `x`、`timestep`、`context`、`attention_mask` 和 `ref_latents` 等参数驱动。

## Key Concepts / 关键概念
- `QwenImageExecutor`: Executor for QwenImage models in ComfyUI. / 核心类，用于封装 qwen image executor 相关行为。
- `QwenImageEditExecutor`: Executor for QwenImageEdit models in ComfyUI. / 核心类，用于封装 qwen image edit executor 相关行为。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `torch`, `comfy.ldm.common_dit`
- **Internal modules / 内部模块**: `.base`, `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.entrypoints.utils`

- **Total lines / 总行数**: 174
