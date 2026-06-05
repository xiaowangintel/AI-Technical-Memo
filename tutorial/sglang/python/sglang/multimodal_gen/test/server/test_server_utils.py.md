# test_server_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/server/test_server_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates server utils with focused assertions and fixtures. Key symbols include `_urlopen_with_retry`, `download_image_from_url`, `parse_dimensions`. / 该测试模块通过有针对性的断言与夹具，验证 server utils 的实现。 关键符号包括 `_urlopen_with_retry`, `download_image_from_url`, `parse_dimensions`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-55: Imports and module setup / 导入与模块初始化
```python
"""
Server management and performance validation for diffusion tests.
"""

from __future__ import annotations

import base64
import os
import shlex
import subprocess
import sys
import tempfile
import threading
import time
# ...

# Tracks mesh output file paths from generate_mesh for later correctness validation.
# Keyed by case_id, cleaned up after use.
MESH_OUTPUT_PATHS: dict[str, str] = {}
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 58-77: Function `_urlopen_with_retry` / 函数 `_urlopen_with_retry`
```python
def _urlopen_with_retry(url: str, timeout: int = 30, max_retries: int = 3) -> bytes:
    """Download content from a URL with retry on transient failures."""
    for attempt in range(max_retries + 1):
        try:
            with urlopen(url, timeout=timeout) as response:
                return response.read()
        except (TimeoutError, OSError) as e:
            if attempt < max_retries:
                wait = 2**attempt
                logger.warning(
                    f"Download attempt {attempt + 1}/{max_retries + 1} failed "
                    f"for {url}: {e}. Retrying in {wait}s..."
                )
                time.sleep(wait)
# ...
                    f"Failed to download from {url} after "
                    f"{max_retries + 1} attempts: {e}"
                )
                raise
```
**EN:** This function drives `_urlopen_with_retry` with inputs such as `url`, `timeout`, `max_retries`. Download content from a URL with retry on transient failures.
**CN:** 这个函数负责 `_urlopen_with_retry`，主要处理 `url`, `timeout`, `max_retries` 等输入。 文档字符串说明：Download content from a URL with retry on transient failures.

### Lines 80-104: Function `download_image_from_url` / 函数 `download_image_from_url`
```python
def download_image_from_url(url: str) -> Path:
    """Download an image from a URL to a temporary file.

    Args:
        url: The URL of the image to download

    Returns:
        Path to the downloaded temporary file
    """
    logger.info(f"Downloading image from URL: {url}")

    # Determine file extension from URL
    ext = ".jpg"  # default
    if url.lower().endswith((".png", ".jpeg", ".jpg", ".webp", ".gif")):
# ...
    data = _urlopen_with_retry(url)
    temp_file.write_bytes(data)
    logger.info(f"Downloaded image to: {temp_file}")
    return temp_file
```
**EN:** This function drives `download_image_from_url` with inputs such as `url`. Download an image from a URL to a temporary file.
**CN:** 这个函数负责 `download_image_from_url`，主要处理 `url` 等输入。 文档字符串说明：Download an image from a URL to a temporary file.

### Lines 107-147: Function `parse_dimensions` / 函数 `parse_dimensions`
```python
def parse_dimensions(size_string: str | None) -> tuple[int | None, int | None]:
    """Parse a size string in "widthxheight" format to (width, height) tuple.

    Args:
        size_string: Size string in "widthxheight" format (e.g., "1024x1024") or None.
                    Spaces are automatically stripped.

    Returns:
        Tuple of (width, height) as integers if parsing succeeds, (None, None) otherwise.
    """
    if not size_string:
        return (None, None)

    # Strip spaces from the entire string
# ...

        return (width, height)
    except ValueError:
        return (None, None)
```
**EN:** This function drives `parse_dimensions` with inputs such as `size_string`. Parse a size string in "widthxheight" format to (width, height) tuple.
**CN:** 这个函数负责 `parse_dimensions`，主要处理 `size_string` 等输入。 文档字符串说明：Parse a size string in "widthxheight" format to (width, height) tuple.

### Lines 150-267: Class `ServerContext` / 类 `ServerContext`
```python
@dataclass
class ServerContext:
    """Context for a running diffusion server."""

    port: int
    process: subprocess.Popen
    model: str
    stdout_file: Path
    perf_log_path: Path
    log_dir: Path
    _stdout_fh: Any = field(repr=False)
    _log_thread: threading.Thread | None = field(default=None, repr=False)

    def cleanup(self) -> None:
# ...

        # Wait for GPU memory to be released (ROCm can be much slower than CUDA)
        # The GPU driver needs time to reclaim memory from killed processes
        time.sleep(15)
```
**EN:** This class models `ServerContext`. Context for a running diffusion server. Important methods include `cleanup`, `_cleanup_hf_cache_if_not_persistent`, `_cleanup_rocm_gpu_memory`.
**CN:** 该类实现 `ServerContext`。 文档字符串指出：Context for a running diffusion server. 其中较重要的方法包括 `cleanup`, `_cleanup_hf_cache_if_not_persistent`, `_cleanup_rocm_gpu_memory`。

### Lines 270-478: Class `ServerManager` / 类 `ServerManager`
```python
class ServerManager:
    """Manages diffusion server lifecycle."""

    def __init__(
        self,
        model: str,
        port: int,
        wait_deadline: float = 1200.0,
        extra_args: str = "",
        env_vars: dict[str, str] | None = None,
    ):
        self.model = model
        self.port = port
        self.wait_deadline = wait_deadline
# ...
            content = path.read_text(encoding="utf-8", errors="ignore")
            return "\n".join(content.splitlines()[-lines:])
        except Exception:
            return ""
```
**EN:** This class models `ServerManager`. Manages diffusion server lifecycle. Important methods include `__init__`, `_wait_for_rocm_gpu_memory_clear`, `start`, `_wait_for_ready`.
**CN:** 该类实现 `ServerManager`。 文档字符串指出：Manages diffusion server lifecycle. 其中较重要的方法包括 `__init__`, `_wait_for_rocm_gpu_memory_clear`, `start`, `_wait_for_ready`。

### Lines 481-633: Class `PerformanceValidator` / 类 `PerformanceValidator`
```python
class PerformanceValidator:
    """Validates performance metrics against expectations."""

    is_video_gen: bool = False

    def __init__(
        self,
        scenario: ScenarioConfig,
        tolerances: ToleranceConfig,
        step_fractions: Sequence[float],
    ):
        self.scenario = scenario
        self.tolerances = tolerances
        self.step_fractions = step_fractions
# ...
                expected,
                tolerance,
                min_abs_tolerance_ms=min_abs_tolerance_ms,
            )
```
**EN:** This class models `PerformanceValidator`. Validates performance metrics against expectations. Important methods include `__init__`, `_assert_le`, `validate`, `collect_metrics`.
**CN:** 该类实现 `PerformanceValidator`。 文档字符串指出：Validates performance metrics against expectations. 其中较重要的方法包括 `__init__`, `_assert_le`, `validate`, `collect_metrics`。

### Lines 636-668: Class `VideoPerformanceValidator` / 类 `VideoPerformanceValidator`
```python
class VideoPerformanceValidator(PerformanceValidator):
    """Extended validator for video diffusion with frame-level metrics."""

    is_video_gen = True

    def validate(
        self,
        perf_record: RequestPerfRecord,
        num_frames: int | None = None,
    ) -> PerformanceSummary:
        """Validate video metrics including frame generation rates."""
        summary = super().validate(perf_record)

        if num_frames and summary.e2e_ms > 0:
# ...
                summary.avg_frame_time_ms,
                expected_frame_time,
                self.tolerances.denoise_stage,
            )
```
**EN:** This class models `VideoPerformanceValidator` as a specialization of `PerformanceValidator`. Extended validator for video diffusion with frame-level metrics. Important methods include `validate`, `_validate_frame_rate`.
**CN:** 该类实现 `VideoPerformanceValidator`，并继承/扩展 `PerformanceValidator`。 文档字符串指出：Extended validator for video diffusion with frame-level metrics. 其中较重要的方法包括 `validate`, `_validate_frame_rate`。

### Lines 671-674: Class `MeshValidator` / 类 `MeshValidator`
```python
class MeshValidator(PerformanceValidator):
    """Validator for 3D mesh generation. Inherits perf validation from PerformanceValidator."""

    pass
```
**EN:** This class models `MeshValidator` as a specialization of `PerformanceValidator`. Validator for 3D mesh generation.
**CN:** 该类实现 `MeshValidator`，并继承/扩展 `PerformanceValidator`。 文档字符串指出：Validator for 3D mesh generation.

### Lines 675-680: Top-level configuration / 顶层配置
```python


HUNYUAN3D_REFERENCE_URL = (
    "https://raw.githubusercontent.com/sgl-project/sgl-test-files/"
    "main/diffusion-ci/consistency_gt/1-gpu/hunyuan3d_2_0/hunyuan3d.glb"
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 683-696: Function `_download_reference_mesh` / 函数 `_download_reference_mesh`
```python
def _download_reference_mesh(url: str) -> Path:
    """Download a reference mesh from URL, caching in temp dir."""
    import hashlib

    cache_name = f"ref_mesh_{hashlib.md5(url.encode()).hexdigest()}.glb"
    cache_path = Path(tempfile.gettempdir()) / cache_name
    if cache_path.exists():
        logger.info(f"Using cached reference mesh: {cache_path}")
        return cache_path

    logger.info(f"Downloading reference mesh from: {url}")
    cache_path.write_bytes(_urlopen_with_retry(url, timeout=60))
    logger.info(f"Reference mesh cached at: {cache_path}")
    return cache_path
```
**EN:** This function drives `_download_reference_mesh` with inputs such as `url`. Download a reference mesh from URL, caching in temp dir.
**CN:** 这个函数负责 `_download_reference_mesh`，主要处理 `url` 等输入。 文档字符串说明：Download a reference mesh from URL, caching in temp dir.

### Lines 699-755: Function `validate_mesh_correctness` / 函数 `validate_mesh_correctness`
```python
def validate_mesh_correctness(
    generated_mesh_path: str,
    reference_url: str = HUNYUAN3D_REFERENCE_URL,
    num_sample_points: int = 4096,
    cd_threshold_ratio: float = 0.01,
    random_seed: int = 42,
):
    """Validate mesh geometric similarity against a reference via Chamfer Distance.

    Downloads the reference mesh from a URL (cached), samples point clouds from
    both meshes, and asserts Chamfer Distance is within threshold.
    """
    import numpy as np

# ...
    assert total_cd <= cd_threshold, (
        f"Chamfer Distance check failed: total_cd={total_cd:.6f}, "
        f"threshold={cd_threshold:.6f} ({cd_threshold_ratio * 100:.2f}% of bbox diagonal {bbox_diagonal:.4f})"
    )
```
**EN:** This function drives `validate_mesh_correctness` with inputs such as `generated_mesh_path`, `reference_url`, `num_sample_points`, `cd_threshold_ratio`. Validate mesh geometric similarity against a reference via Chamfer Distance.
**CN:** 这个函数负责 `validate_mesh_correctness`，主要处理 `generated_mesh_path`, `reference_url`, `num_sample_points`, `cd_threshold_ratio` 等输入。 文档字符串说明：Validate mesh geometric similarity against a reference via Chamfer Distance.

### Lines 756-763: Top-level configuration / 顶层配置
```python


# Registry of validators by name
VALIDATOR_REGISTRY = {
    "default": PerformanceValidator,
    "video": VideoPerformanceValidator,
    "mesh": MeshValidator,
}
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 766-785: Function `_extract_async_job_error_message` / 函数 `_extract_async_job_error_message`
```python
def _extract_async_job_error_message(job: Any) -> str | None:
    error = getattr(job, "error", None)
    if error is None and isinstance(job, dict):
        error = job.get("error")

    if error is None:
        return None

    if isinstance(error, dict):
        for key in ("message", "detail", "error"):
            value = error.get(key)
            if value:
                return str(value)
        return str(error)
# ...
    if message:
        return str(message)

    return str(error)
```
**EN:** This function drives `_extract_async_job_error_message` with inputs such as `job`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_extract_async_job_error_message`，主要处理 `job` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 788-1373: Function `get_generate_fn` / 函数 `get_generate_fn`
```python
def get_generate_fn(
    model_path: str,
    modality: str,
    sampling_params: DiffusionSamplingParams,
) -> Callable[[str, Client], tuple[str, bytes]]:
    """Return appropriate generation function for the case."""
    # Allow override via environment variable (useful for AMD where large resolutions cause slow VAE)
    output_size = os.environ.get("SGLANG_TEST_OUTPUT_SIZE", sampling_params.output_size)
    n = sampling_params.num_outputs_per_prompt

    def _create_and_download_video(
        client,
        case_id,
        *,
# ...
    else:
        fn = generate_image

    return fn
```
**EN:** This function drives `get_generate_fn` with inputs such as `model_path`, `modality`, `sampling_params`. Return appropriate generation function for the case.
**CN:** 这个函数负责 `get_generate_fn`，主要处理 `model_path`, `modality`, `sampling_params` 等输入。 文档字符串说明：Return appropriate generation function for the case.

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Caching strategy / 缓存策略
- Sampling parameter control / 采样参数控制
- Automated verification / 自动化验证
- Pytest-based assertions / 基于 Pytest 的断言

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.benchmarks.compare_perf`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.common`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.utils.perf_logger`, `sglang.multimodal_gen.test.server.testcase_configs`, `sglang.multimodal_gen.test.slack_utils`, `sglang.multimodal_gen.test.test_utils`
- **External / 外部**: `__future__`, `urllib.request`, `pytest`, `openai`, `numpy`, `scipy.spatial`, `trimesh`, `requests`
- **Stdlib / 标准库**: `base64`, `os`, `shlex`, `subprocess`, `sys`, `tempfile`, `threading`, `time`
