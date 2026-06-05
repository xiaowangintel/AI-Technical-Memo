# compare_diffusion_trajectory_similarity.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/tools/compare_diffusion_trajectory_similarity.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This script provides developer tooling for compare diffusion trajectory similarity, typically for conversion, inspection, or offline preparation. Key symbols include `parse_component_overrides`, `_cosine_similarity`, `compute_tensor_metrics`. / 该脚本提供与 compare diffusion trajectory similarity 相关的开发工具，通常用于转换、检查或离线准备。 关键符号包括 `parse_component_overrides`, `_cosine_similarity`, `compute_tensor_metrics`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-36: Imports and module setup / 导入与模块初始化
```python
"""Compare diffusion BF16 and quantized runs via trajectory-latent similarity.

This tool runs two SGLang diffusion variants with the same prompt and seed,
captures intermediate denoising latents via `return_trajectory_latents`, and
reports cosine / error metrics for each timestep plus final frame metrics.

The intended use is quant validation with reduced deterministic settings:
- same prompt / seed / resolution / step count for both variants
- BF16 reference on the base model
- FP8 candidate via `--candidate-transformer-path` and/or component overrides

Example:

    python -m sglang.multimodal_gen.tools.compare_diffusion_trajectory_similarity \
# ...
import imageio.v3 as iio
import numpy as np
import torch
import torch.nn.functional as F
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 39-54: Function `parse_component_overrides` / 函数 `parse_component_overrides`
```python
def parse_component_overrides(entries: Sequence[str] | None) -> dict[str, str]:
    overrides: dict[str, str] = {}
    for entry in entries or []:
        if "=" not in entry:
            raise ValueError(
                f"Invalid component override '{entry}'. Expected format component=path."
            )
        component, path = entry.split("=", 1)
        component = component.strip().replace("-", "_")
        path = path.strip()
        if not component or not path:
            raise ValueError(
                f"Invalid component override '{entry}'. Expected format component=path."
            )
        overrides[component] = path
    return overrides
```
**EN:** This function drives `parse_component_overrides` with inputs such as `entries`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `parse_component_overrides`，主要处理 `entries` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 57-64: Function `_cosine_similarity` / 函数 `_cosine_similarity`
```python
def _cosine_similarity(flat_a: torch.Tensor, flat_b: torch.Tensor) -> float:
    norm_a = torch.linalg.vector_norm(flat_a).item()
    norm_b = torch.linalg.vector_norm(flat_b).item()
    if norm_a == 0.0 and norm_b == 0.0:
        return 1.0
    if norm_a == 0.0 or norm_b == 0.0:
        return 0.0
    return float(F.cosine_similarity(flat_a, flat_b, dim=0).item())
```
**EN:** This function drives `_cosine_similarity` with inputs such as `flat_a`, `flat_b`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_cosine_similarity`，主要处理 `flat_a`, `flat_b` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 67-89: Function `compute_tensor_metrics` / 函数 `compute_tensor_metrics`
```python
def compute_tensor_metrics(lhs: Any, rhs: Any) -> dict[str, float]:
    lhs_tensor = torch.as_tensor(lhs).detach().cpu().float()
    rhs_tensor = torch.as_tensor(rhs).detach().cpu().float()
    if lhs_tensor.shape != rhs_tensor.shape:
        raise ValueError(
            f"Metric shape mismatch: {tuple(lhs_tensor.shape)} vs {tuple(rhs_tensor.shape)}"
        )

    diff = lhs_tensor - rhs_tensor
    mse = float(diff.square().mean().item())
    rmse = float(math.sqrt(mse))
    mae = float(diff.abs().mean().item())
    max_abs = float(diff.abs().max().item())
    l2 = float(torch.linalg.vector_norm(diff).item())
# ...
        "rmse": rmse,
        "max_abs": max_abs,
        "l2": l2,
    }
```
**EN:** This function drives `compute_tensor_metrics` with inputs such as `lhs`, `rhs`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `compute_tensor_metrics`，主要处理 `lhs`, `rhs` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 92-98: Function `compute_uint8_frame_metrics` / 函数 `compute_uint8_frame_metrics`
```python
def compute_uint8_frame_metrics(lhs: Any, rhs: Any) -> dict[str, float]:
    metrics = compute_tensor_metrics(lhs, rhs)
    mse = metrics["mse"]
    metrics["psnr_db"] = (
        float("inf") if mse == 0.0 else 20 * math.log10(255.0) - 10 * math.log10(mse)
    )
    return metrics
```
**EN:** This function drives `compute_uint8_frame_metrics` with inputs such as `lhs`, `rhs`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `compute_uint8_frame_metrics`，主要处理 `lhs`, `rhs` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 101-110: Function `_normalize_step_index` / 函数 `_normalize_step_index`
```python
def _normalize_step_index(step_index: int, num_steps: int) -> int:
    if num_steps <= 0:
        raise ValueError("num_steps must be positive.")
    if step_index < 0:
        step_index += num_steps
    if step_index < 0 or step_index >= num_steps:
        raise IndexError(
            f"Requested step index {step_index} is outside the valid range [0, {num_steps})."
        )
    return step_index
```
**EN:** This function drives `_normalize_step_index` with inputs such as `step_index`, `num_steps`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_normalize_step_index`，主要处理 `step_index`, `num_steps` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 113-121: Function `_maybe_scalar` / 函数 `_maybe_scalar`
```python
def _maybe_scalar(timestep: torch.Tensor | None, index: int) -> float | None:
    if timestep is None:
        return None
    value = timestep[index]
    if isinstance(value, torch.Tensor):
        value = value.detach().cpu()
        if value.numel() == 1:
            return float(value.item())
    return float(value)
```
**EN:** This function drives `_maybe_scalar` with inputs such as `timestep`, `index`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_maybe_scalar`，主要处理 `timestep`, `index` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 124-170: Function `summarize_trajectory_metrics` / 函数 `summarize_trajectory_metrics`
```python
def summarize_trajectory_metrics(
    reference_latents: Any,
    candidate_latents: Any,
    *,
    reference_timesteps: Any = None,
    candidate_timesteps: Any = None,
    step_index: int = -1,
) -> dict[str, Any]:
    ref = torch.as_tensor(reference_latents).detach().cpu().float()
    cand = torch.as_tensor(candidate_latents).detach().cpu().float()
    if ref.shape != cand.shape:
        raise ValueError(
            f"Trajectory shape mismatch: {tuple(ref.shape)} vs {tuple(cand.shape)}"
        )
# ...
        "selected_step_index": selected_step,
        "selected_step_metrics": per_step[selected_step],
        "per_step_metrics": per_step,
    }
```
**EN:** This function drives `summarize_trajectory_metrics` with inputs such as `reference_latents`, `candidate_latents`, `reference_timesteps`, `candidate_timesteps`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `summarize_trajectory_metrics`，主要处理 `reference_latents`, `candidate_latents`, `reference_timesteps`, `candidate_timesteps` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 173-200: Function `summarize_output_frame_metrics` / 函数 `summarize_output_frame_metrics`
```python
def summarize_output_frame_metrics(
    reference_frames: Sequence[Any],
    candidate_frames: Sequence[Any],
) -> dict[str, Any]:
    if len(reference_frames) != len(candidate_frames):
        raise ValueError(
            f"Output frame count mismatch: {len(reference_frames)} vs {len(candidate_frames)}"
        )
    if not reference_frames:
        raise ValueError("No output frames available for comparison.")

    ref_stack = np.stack([np.asarray(frame) for frame in reference_frames], axis=0)
    cand_stack = np.stack([np.asarray(frame) for frame in candidate_frames], axis=0)

# ...
        "mid_frame_index": mid_index,
        "mid_frame_metrics": mid_metrics,
        "all_frames_metrics": all_metrics,
    }
```
**EN:** This function drives `summarize_output_frame_metrics` with inputs such as `reference_frames`, `candidate_frames`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `summarize_output_frame_metrics`，主要处理 `reference_frames`, `candidate_frames` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 203-249: Function `extract_result_frames` / 函数 `extract_result_frames`
```python
def extract_result_frames(result: Any) -> list[np.ndarray]:
    if result.frames is not None:
        return [np.asarray(frame) for frame in result.frames]

    sample = result.samples
    if sample is None:
        if result.output_file_path:
            output_path = Path(result.output_file_path)
            if not output_path.exists():
                raise ValueError(
                    "GenerationResult did not contain frames or samples, and its "
                    f"output_file_path does not exist: {output_path}"
                )
            if output_path.suffix.lower() in {".png", ".jpg", ".jpeg", ".webp"}:
# ...
        )
    if array.dtype != np.uint8:
        array = (np.clip(array, 0.0, 1.0) * 255.0).astype(np.uint8)
    return [frame for frame in array]
```
**EN:** This function drives `extract_result_frames` with inputs such as `result`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `extract_result_frames`，主要处理 `result` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 252-277: Function `build_server_kwargs` / 函数 `build_server_kwargs`
```python
def build_server_kwargs(args: argparse.Namespace, *, variant: str) -> dict[str, Any]:
    component_paths = parse_component_overrides(
        getattr(args, f"{variant}_component_path") or []
    )
    transformer_path = getattr(args, f"{variant}_transformer_path")

    kwargs: dict[str, Any] = {
        "model_path": args.model_path,
        "model_id": args.model_id,
        "backend": args.backend,
        "num_gpus": args.num_gpus,
        "dit_cpu_offload": args.dit_cpu_offload,
        "dit_layerwise_offload": args.dit_layerwise_offload,
        "text_encoder_cpu_offload": args.text_encoder_cpu_offload,
# ...
        kwargs["transformer_weights_path"] = transformer_path
    if component_paths:
        kwargs["component_paths"] = component_paths
    return kwargs
```
**EN:** This function drives `build_server_kwargs` with inputs such as `args`, `variant`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `build_server_kwargs`，主要处理 `args`, `variant` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 280-301: Function `build_sampling_kwargs` / 函数 `build_sampling_kwargs`
```python
def build_sampling_kwargs(
    args: argparse.Namespace, *, output_dir: str | None = None
) -> dict[str, Any]:
    kwargs: dict[str, Any] = {
        "prompt": args.prompt,
        "width": args.width,
        "height": args.height,
        "num_inference_steps": args.num_inference_steps,
        "guidance_scale": args.guidance_scale,
        "seed": args.seed,
        "return_frames": True,
        "return_trajectory_latents": True,
        "return_trajectory_decoded": args.return_trajectory_decoded,
        "save_output": output_dir is not None,
# ...
        kwargs["num_frames"] = args.num_frames
    if args.guidance_scale_2 is not None:
        kwargs["guidance_scale_2"] = args.guidance_scale_2
    return kwargs
```
**EN:** This function drives `build_sampling_kwargs` with inputs such as `args`, `output_dir`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `build_sampling_kwargs`，主要处理 `args`, `output_dir` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 304-313: Function `_normalize_single_result` / 函数 `_normalize_single_result`
```python
def _normalize_single_result(result: Any):
    if isinstance(result, list):
        if len(result) != 1:
            raise ValueError(
                f"Expected a single generation result, got {len(result)} results."
            )
        result = result[0]
    if result is None:
        raise RuntimeError("Generation returned no result.")
    return result
```
**EN:** This function drives `_normalize_single_result` with inputs such as `result`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_normalize_single_result`，主要处理 `result` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 316-324: Function `_clear_diffusion_fp4_backend_caches` / 函数 `_clear_diffusion_fp4_backend_caches`
```python
def _clear_diffusion_fp4_backend_caches() -> None:
    from sglang.multimodal_gen.runtime.layers.quantization import (
        modelopt_quant as diffusion_modelopt_quant,
    )
    from sglang.multimodal_gen.runtime.platforms import current_platform

    diffusion_modelopt_quant._get_fp4_gemm_op.cache_clear()
    current_platform.__class__.get_modelopt_fp4_gemm_op.cache_clear()
    current_platform.__class__.get_modelopt_flashinfer_fp4_backend.cache_clear()
```
**EN:** This function drives `_clear_diffusion_fp4_backend_caches`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_clear_diffusion_fp4_backend_caches`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 327-345: Function `override_diffusion_fp4_backend` / 函数 `override_diffusion_fp4_backend`
```python
@contextlib.contextmanager
def override_diffusion_fp4_backend(backend: str | None):
    env_name = "SGLANG_DIFFUSION_FLASHINFER_FP4_GEMM_BACKEND"
    previous = os.environ.get(env_name)

    if backend is None:
        os.environ.pop(env_name, None)
    else:
        os.environ[env_name] = backend

    _clear_diffusion_fp4_backend_caches()
    try:
        yield
    finally:
        if previous is None:
            os.environ.pop(env_name, None)
        else:
            os.environ[env_name] = previous
        _clear_diffusion_fp4_backend_caches()
```
**EN:** This function drives `override_diffusion_fp4_backend` with inputs such as `backend`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `override_diffusion_fp4_backend`，主要处理 `backend` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 348-355: Function `_extract_total_duration_ms` / 函数 `_extract_total_duration_ms`
```python
def _extract_total_duration_ms(result: Any) -> float | None:
    metrics = getattr(result, "metrics", None)
    if not isinstance(metrics, dict):
        return None
    total_duration_ms = metrics.get("total_duration_ms")
    if total_duration_ms is None:
        return None
    return float(total_duration_ms)
```
**EN:** This function drives `_extract_total_duration_ms` with inputs such as `result`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_extract_total_duration_ms`，主要处理 `result` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 358-421: Function `run_variant` / 函数 `run_variant`
```python
def run_variant(
    *,
    server_kwargs: dict[str, Any],
    sampling_kwargs: dict[str, Any],
    fp4_gemm_backend: str | None,
    warmup_runs: int,
    measure_runs: int,
):
    from sglang.multimodal_gen.runtime.entrypoints.diffusion_generator import (
        DiffGenerator,
    )

    if warmup_runs < 0:
        raise ValueError("warmup_runs must be >= 0.")
# ...
            else None
        ),
        "per_run_total_duration_ms": total_duration_ms,
    }
```
**EN:** This function drives `run_variant` with inputs such as `server_kwargs`, `sampling_kwargs`, `fp4_gemm_backend`, `warmup_runs`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `run_variant`，主要处理 `server_kwargs`, `sampling_kwargs`, `fp4_gemm_backend`, `warmup_runs` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 424-425: Function `_to_jsonable` / 函数 `_to_jsonable`
```python
def _to_jsonable(result: dict[str, Any]) -> dict[str, Any]:
    return json.loads(json.dumps(result, allow_nan=True))
```
**EN:** This function drives `_to_jsonable` with inputs such as `result`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_to_jsonable`，主要处理 `result` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 428-627: Function `main` / 函数 `main`
```python
def main() -> None:
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument("--model-path", required=True)
    parser.add_argument(
        "--model-id",
        help=(
            "Optional model ID override passed to DiffGenerator.from_pretrained. "
            "Use this when --model-path points to a local directory whose name "
            "does not match a registered native SGLang model."
        ),
    )
    parser.add_argument("--backend", default="sglang")
    parser.add_argument("--prompt", required=True)
    parser.add_argument("--output-json", required=True)
# ...
            },
            indent=2,
        )
    )
```
**EN:** This function drives `main`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `main`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 628-631: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    main()
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Quantization workflow / 量化工作流
- Caching strategy / 缓存策略
- Sampling parameter control / 采样参数控制
- Command-line interface / 命令行接口

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.layers.quantization`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.entrypoints.diffusion_generator`
- **External / 外部**: `__future__`, `imageio.v3`, `numpy`, `torch`, `torch.nn.functional`
- **Stdlib / 标准库**: `argparse`, `contextlib`, `json`, `math`, `os`, `pathlib`, `typing`
