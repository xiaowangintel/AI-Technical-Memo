# test_flux_pipeline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/apps/ComfyUI_SGLDiffusion/test/test_flux_pipeline.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the ComfyUI integration layer. It centers on `test_comfyui_flux_pipeline_direct`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Test for ComfyUIFluxPipeline with pass-through scheduler. / 该文件属于ComfyUI 集成层。它围绕 `test_comfyui_flux_pipeline_direct` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: module setup and imports / 模块初始化与导入
```python
"""Test for ComfyUIFluxPipeline with pass-through scheduler."""

import os
import sys

import pytest
import torch

from sglang.multimodal_gen.configs.sample.sampling_params import SamplingParams
from sglang.multimodal_gen.runtime.entrypoints.diffusion_generator import DiffGenerator
from sglang.multimodal_gen.runtime.entrypoints.utils import prepare_request
```
**EN:** This block establishes the module context and imports `os`, `sys`, `pytest`, `torch`, `sglang.multimodal_gen.configs.sample.sampling_params`, and `sglang.multimodal_gen.runtime.entrypoints.diffusion_generator`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `os`、`sys`、`pytest`、`torch`、`sglang.multimodal_gen.configs.sample.sampling_params` 和 `sglang.multimodal_gen.runtime.entrypoints.diffusion_generator`。这些依赖为后续实现提供所需符号。

### Lines 14-159: `test_comfyui_flux_pipeline_direct` implementation / `test_comfyui_flux_pipeline_direct` 实现
```python
def test_comfyui_flux_pipeline_direct() -> None:
    """Test ComfyUIFluxPipeline with custom inputs."""
    model_path = os.environ.get(
        "SGLANG_TEST_FLUX_MODEL_PATH",
        "black-forest-labs/FLUX.1-dev",  # Supports both safetensors file and diffusers format
    )

    generator = DiffGenerator.from_pretrained(
        model_path=model_path,
        pipeline_class_name="ComfyUIFluxPipeline",
        num_gpus=2,
        comfyui_mode=True,
    )

    batch_size = 1
    hidden_states_seq_len = 3600
    hidden_states_dim = 64
    height = 1280
    width = 720

    encoder_seq_len = 512
    encoder_dim = 4096
    pooled_dim = 768

    hidden_states = torch.ones(
        batch_size,
        hidden_states_seq_len,
        hidden_states_dim,
        device="cuda",
        dtype=torch.bfloat16,
    )

    encoder_hidden_states = torch.ones(
        batch_size,
        encoder_seq_len,
        encoder_dim,
        device="cuda",
        dtype=torch.bfloat16,
    )

    pooled_projections = torch.ones(
        batch_size,
        pooled_dim,
        device="cuda",
        dtype=torch.bfloat16,
    )

    timesteps = torch.tensor([1000], dtype=torch.long, device="cuda")

    sampling_params = SamplingParams.from_user_sampling_params_args(
        generator.server_args.model_path,
        server_args=generator.server_args,
        prompt="a beautiful girl",
        height=height,
        width=width,
        num_frames=1,
        num_inference_steps=1,
        save_output=True,
        return_trajectory_latents=True,
    )

    req = prepare_request(
        server_args=generator.server_args,
        sampling_params=sampling_params,
    )

    req.latents = hidden_states
    req.timesteps = timesteps
    req.raw_latent_shape = torch.tensor(hidden_states.shape, dtype=torch.long)

    clip_dim = 768
    dummy_clip_embedding = torch.zeros(
        batch_size,
        77,
        clip_dim,
        device="cuda",
        dtype=torch.bfloat16,
    )
    req.prompt_embeds = [pooled_projections, encoder_hidden_states]

    if req.guidance_scale > 1.0:
        dummy_neg_clip_embedding = torch.zeros(
            batch_size,
            77,
            clip_dim,
            device="cuda",
            dtype=torch.bfloat16,
        )
        negative_encoder_hidden_states = torch.ones(
            batch_size,
            encoder_seq_len,
            encoder_dim,
            device="cuda",
            dtype=torch.bfloat16,
        )
        req.negative_prompt_embeds = [
            dummy_neg_clip_embedding,
            negative_encoder_hidden_states,
        ]
    else:
        req.negative_prompt_embeds = None

    req.pooled_embeds = [pooled_projections]
    req.neg_pooled_embeds = []

    if (
        req.guidance_scale > 1.0
        and req.negative_prompt_embeds is not None
        and len(req.negative_prompt_embeds) > 0
    ):
        req.do_classifier_free_guidance = True
    else:
        req.do_classifier_free_guidance = False

    if req.seed is not None:
        generator_device = req.generator_device
        device_str = "cuda" if generator_device == "cuda" else "cpu"
        req.generator = [
            torch.Generator(device_str).manual_seed(req.seed + i)
            for i in range(req.num_outputs_per_prompt)
        ]
    else:
        req.generator = [
            torch.Generator("cuda") for _ in range(req.num_outputs_per_prompt)
        ]

    output_batch = generator._send_to_scheduler_and_wait_for_response([req])
    noise_pred = output_batch.noise_pred

    assert noise_pred is not None, "noise_pred should not be None in OutputBatch"
    assert isinstance(noise_pred, torch.Tensor), "noise_pred should be a torch.Tensor"
    assert (
        noise_pred.device.type == "cuda"
    ), f"noise_pred should be on cuda, got {noise_pred.device}"
    assert (
        noise_pred.dtype == torch.bfloat16
    ), f"noise_pred should be bfloat16, got {noise_pred.dtype}"

    print(f"✓ Successfully retrieved noise_pred from OutputBatch!")
    print(f"  noise_pred shape: {noise_pred.shape}")
    print(f"  noise_pred dtype: {noise_pred.dtype}")
    print(f"  noise_pred device: {noise_pred.device}")

    latents = output_batch.output if output_batch.output is not None else req.latents
    assert latents is not None, "latents should not be None"
    print(f"latents.shape: {latents.shape}")
```
**EN:** This block defines function `test_comfyui_flux_pipeline_direct`. Test ComfyUIFluxPipeline with custom inputs. Key calls include `os.environ.get`, `DiffGenerator.from_pretrained`, `torch.ones`, `torch.tensor`, and `SamplingParams.from_user_sampling_params_args`. The implementation branches on conditions.
**CN:** 该代码块定义了函数 `test_comfyui_flux_pipeline_direct`。 它用于处理 test comfyui flux pipeline direct 相关逻辑。 关键调用包括 `os.environ.get`、`DiffGenerator.from_pretrained`、`torch.ones`、`torch.tensor` 和 `SamplingParams.from_user_sampling_params_args`。 实现中包含条件分支。

### Lines 162-163: supporting statements / 辅助语句
```python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v"]))
```
**EN:** This block gathers supporting statements at module scope. The code collaborates with `sys.exit`, and `pytest.main`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 代码会与 `sys.exit` 和 `pytest.main` 协同工作。

## Key Concepts / 关键概念
- `test_comfyui_flux_pipeline_direct`: Test ComfyUIFluxPipeline with custom inputs. / 顶层函数，用于处理 test comfyui flux pipeline direct 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `sys`
- **Third-party / 第三方依赖**: `pytest`, `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.entrypoints.diffusion_generator`, `sglang.multimodal_gen.runtime.entrypoints.utils`

- **Total lines / 总行数**: 163
