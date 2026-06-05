# test_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/test_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates utils with focused assertions and fixtures. Key symbols include `_load_clip_processor_with_roberta_processing_compat`, `print_value_formatted`, `print_divider`. / 该测试模块通过有针对性的断言与夹具，验证 utils 的实现。 关键符号包括 `_load_clip_processor_with_roberta_processing_compat`, `print_value_formatted`, `print_divider`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-74: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo
import base64
import html
import io
import json
import math
import os
import socket
import subprocess
import sys
import tempfile
import time
from dataclasses import dataclass
from pathlib import Path
# ...
DEFAULT_PSNR_THRESHOLD_VIDEO = 24.0
DEFAULT_MEAN_ABS_DIFF_THRESHOLD_VIDEO = 10.0
_clip_model_cache: dict[str, Any] = {}
_consistency_gt_cache: dict[str, Any] = {}
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 77-97: Function `_load_clip_processor_with_roberta_processing_compat` / 函数 `_load_clip_processor_with_roberta_processing_compat`
```python
def _load_clip_processor_with_roberta_processing_compat(
    clip_processor_cls, *args, **kwargs
):
    from tokenizers import processors

    roberta_processing = processors.RobertaProcessing

    def roberta_processing_compat(*processor_args, **processor_kwargs):
        if "sep" in processor_kwargs and "cls" in processor_kwargs:
            sep = processor_kwargs.pop("sep")
            cls_token = processor_kwargs.pop("cls")
            return roberta_processing(
                sep, cls_token, *processor_args, **processor_kwargs
            )
# ...
    try:
        return clip_processor_cls.from_pretrained(*args, **kwargs)
    finally:
        processors.RobertaProcessing = roberta_processing
```
**EN:** This function drives `_load_clip_processor_with_roberta_processing_compat` with inputs such as `clip_processor_cls`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_load_clip_processor_with_roberta_processing_compat`，主要处理 `clip_processor_cls` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 98-143: Top-level configuration / 顶层配置
```python


# ---------------------------------------------------------------------------
# Common model IDs for diffusion tests
#
# Centralised here so every test file references the same constants instead
# of scattering hard-coded strings. When adding a new model that will be
# reused across tests, define it here.
# ---------------------------------------------------------------------------

DEFAULT_SMALL_MODEL_NAME_FOR_TEST = "Tongyi-MAI/Z-Image-Turbo"

# Qwen image generation models
DEFAULT_QWEN_IMAGE_MODEL_NAME_FOR_TEST = "Qwen/Qwen-Image"
# ...
DEFAULT_WAN_2_2_I2V_A14B_MODEL_NAME_FOR_TEST = "Wan-AI/Wan2.2-I2V-A14B-Diffusers"

# MOVA video generation models
DEFAULT_MOVA_360P_MODEL_NAME_FOR_TEST = "OpenMOSS-Team/MOVA-360p"
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 146-160: Function `print_value_formatted` / 函数 `print_value_formatted`
```python
def print_value_formatted(description: str, value: int | float | str):
    """Helper function to print a metric value formatted."""
    if isinstance(value, int):
        if value >= 1e6:
            value_str = f"{value / 1e6:<30.2f}M"
        elif value >= 1e3:
            value_str = f"{value / 1e3:<30.2f}K"
        else:
            value_str = f"{value:<30}"
    elif isinstance(value, float):
        value_str = f"{value:<30.2f}"
    else:
        value_str = f"{value:<30}"

    print(f"{description:<45} {value_str}")
```
**EN:** This function drives `print_value_formatted` with inputs such as `description`, `value`. Helper function to print a metric value formatted.
**CN:** 这个函数负责 `print_value_formatted`，主要处理 `description`, `value` 等输入。 文档字符串说明：Helper function to print a metric value formatted.

### Lines 163-165: Function `print_divider` / 函数 `print_divider`
```python
def print_divider(length: int, char: str = "-"):
    """Helper function to print a divider line."""
    print(char * length)
```
**EN:** This function drives `print_divider` with inputs such as `length`, `char`. Helper function to print a divider line.
**CN:** 这个函数负责 `print_divider`，主要处理 `length`, `char` 等输入。 文档字符串说明：Helper function to print a divider line.

### Lines 168-174: Function `is_image_url` / 函数 `is_image_url`
```python
def is_image_url(image_path: str | Path | None) -> bool:
    """Check if image_path is a URL."""
    if image_path is None:
        return False
    return isinstance(image_path, str) and (
        image_path.startswith("http://") or image_path.startswith("https://")
    )
```
**EN:** This function drives `is_image_url` with inputs such as `image_path`. Check if image_path is a URL.
**CN:** 这个函数负责 `is_image_url`，主要处理 `image_path` 等输入。 文档字符串说明：Check if image_path is a URL.

### Lines 177-184: Function `probe_port` / 函数 `probe_port`
```python
def probe_port(host="127.0.0.1", port=30010, timeout=2.0) -> bool:
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.settimeout(timeout)
        try:
            s.connect((host, port))
            return True
        except OSError:
            return False
```
**EN:** This function drives `probe_port` with inputs such as `host`, `port`, `timeout`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `probe_port`，主要处理 `host`, `port`, `timeout` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 187-188: Function `is_in_ci` / 函数 `is_in_ci`
```python
def is_in_ci() -> bool:
    return get_bool_env_var("SGLANG_IS_IN_CI")
```
**EN:** This function drives `is_in_ci`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `is_in_ci`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 191-205: Function `get_dynamic_server_port` / 函数 `get_dynamic_server_port`
```python
def get_dynamic_server_port() -> int:
    cuda_devices = os.environ.get("CUDA_VISIBLE_DEVICES", "0")
    if not cuda_devices:
        cuda_devices = "0"
    try:
        first_device_id = int(cuda_devices.split(",")[0].strip()[0])
    except (ValueError, IndexError):
        first_device_id = 0

    if is_in_ci():
        base_port = 10000 + first_device_id * 2000
    else:
        base_port = 20000 + first_device_id * 1000

    return base_port + 1000
```
**EN:** This function drives `get_dynamic_server_port`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_dynamic_server_port`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 208-212: Function `find_free_port` / 函数 `find_free_port`
```python
def find_free_port(host: str = "127.0.0.1") -> int:
    """Bind to port 0 and let the OS assign an available port."""
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.bind((host, 0))
        return s.getsockname()[1]
```
**EN:** This function drives `find_free_port` with inputs such as `host`. Bind to port 0 and let the OS assign an available port.
**CN:** 这个函数负责 `find_free_port`，主要处理 `host` 等输入。 文档字符串说明：Bind to port 0 and let the OS assign an available port.

### Lines 215-237: Function `wait_for_server_health` / 函数 `wait_for_server_health`
```python
def wait_for_server_health(
    base_url: str,
    path: str = "/health",
    timeout: float = 180.0,
    interval: float = 1.0,
) -> None:
    """Poll ``GET <base_url><path>`` until it returns HTTP 200."""
    deadline = time.time() + timeout
    last_err: httpx.RequestError | None = None
    last_status: int | None = None
    while time.time() < deadline:
        try:
            r = httpx.get(urljoin(base_url, path), timeout=5.0)
            last_status = r.status_code
# ...
    raise TimeoutError(
        f"Server at {urljoin(base_url, path)} not healthy after {timeout}s. "
        f"{last_status=} {last_err=}"
    )
```
**EN:** This function drives `wait_for_server_health` with inputs such as `base_url`, `path`, `timeout`, `interval`. Poll ``GET <base_url><path>`` until it returns HTTP 200.
**CN:** 这个函数负责 `wait_for_server_health`，主要处理 `base_url`, `path`, `timeout`, `interval` 等输入。 文档字符串说明：Poll ``GET <base_url><path>`` until it returns HTTP 200.

### Lines 240-247: Function `post_json` / 函数 `post_json`
```python
def post_json(
    base_url: str,
    path: str,
    payload: dict,
    timeout: float = 300.0,
) -> httpx.Response:
    """POST JSON to ``<base_url><path>`` and return the response."""
    return httpx.post(urljoin(base_url, path), json=payload, timeout=timeout)
```
**EN:** This function drives `post_json` with inputs such as `base_url`, `path`, `payload`, `timeout`. POST JSON to ``<base_url><path>`` and return the response.
**CN:** 这个函数负责 `post_json`，主要处理 `base_url`, `path`, `payload`, `timeout` 等输入。 文档字符串说明：POST JSON to ``<base_url><path>`` and return the response.

### Lines 250-267: Function `run_command` / 函数 `run_command`
```python
def run_command(command: list[str]) -> bool:
    """Run a CLI command and return whether it succeeded."""
    print(f"Running command: {' '.join(command)}", flush=True)
    with subprocess.Popen(
        command,
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,
        text=True,
        encoding="utf-8",
    ) as process:
        assert process.stdout is not None
        for line in process.stdout:
            sys.stdout.write(line)
        process.wait()
        if process.returncode == 0:
            return True
        print(f"Command failed with exit code {process.returncode}", flush=True)
    return False
```
**EN:** This function drives `run_command` with inputs such as `command`. Run a CLI command and return whether it succeeded.
**CN:** 这个函数负责 `run_command`，主要处理 `command` 等输入。 文档字符串说明：Run a CLI command and return whether it succeeded.

### Lines 275-297: Function `query_gpu_mem_used_mib` / 函数 `query_gpu_mem_used_mib`
```python
def query_gpu_mem_used_mib(gpu_index: int = 0, required: bool = False) -> int | None:
    """Return GPU memory usage in MiB via ``nvidia-smi``, or *None* on failure.

    When *required* is ``True`` the function raises instead of returning ``None``.
    """
    try:
        out = subprocess.check_output(
            [
                "nvidia-smi",
                f"--id={gpu_index}",
                "--query-gpu=memory.used",
                "--format=csv,noheader,nounits",
            ],
            text=True,
# ...
            "nvidia-smi memory query is unavailable; "
            "cannot enforce GPU memory assertions."
        )
        return None
```
**EN:** This function drives `query_gpu_mem_used_mib` with inputs such as `gpu_index`, `required`. Return GPU memory usage in MiB via ``nvidia-smi``, or *None* on failure.
**CN:** 这个函数负责 `query_gpu_mem_used_mib`，主要处理 `gpu_index`, `required` 等输入。 文档字符串说明：Return GPU memory usage in MiB via ``nvidia-smi``, or *None* on failure.

### Lines 300-308: Function `require_gpu_mem_query` / 函数 `require_gpu_mem_query`
```python
def require_gpu_mem_query(gpu_index: int = 0) -> int:
    """Same as :func:`query_gpu_mem_used_mib` but asserts availability.

    Raises ``AssertionError`` when ``nvidia-smi`` is unavailable instead of
    returning ``None``, so callers can rely on a valid ``int`` result.
    """
    mem = query_gpu_mem_used_mib(gpu_index, required=True)
    assert mem is not None
    return mem
```
**EN:** This function drives `require_gpu_mem_query` with inputs such as `gpu_index`. Same as :func:`query_gpu_mem_used_mib` but asserts availability.
**CN:** 这个函数负责 `require_gpu_mem_query`，主要处理 `gpu_index` 等输入。 文档字符串说明：Same as :func:`query_gpu_mem_used_mib` but asserts availability.

### Lines 311-326: Function `assert_gpu_mem_changed` / 函数 `assert_gpu_mem_changed`
```python
def assert_gpu_mem_changed(
    label: str,
    before_mib: int,
    after_mib: int,
    min_delta_mib: int,
) -> None:
    """Assert that GPU memory changed by at least *min_delta_mib* MiB."""
    delta = abs(after_mib - before_mib)
    logger.debug(
        f"[MEM] {label}: before={before_mib} MiB  after={after_mib} MiB  |delta|={delta} MiB"
    )
    assert delta >= min_delta_mib, (
        f"GPU memory change too small for '{label}': "
        f"|after-before|={delta} MiB < {min_delta_mib} MiB "
        f"(before={before_mib} MiB, after={after_mib} MiB)"
    )
```
**EN:** This function drives `assert_gpu_mem_changed` with inputs such as `label`, `before_mib`, `after_mib`, `min_delta_mib`. Assert that GPU memory changed by at least *min_delta_mib* MiB.
**CN:** 这个函数负责 `assert_gpu_mem_changed`，主要处理 `label`, `before_mib`, `after_mib`, `min_delta_mib` 等输入。 文档字符串说明：Assert that GPU memory changed by at least *min_delta_mib* MiB.

### Lines 329-333: Function `is_mp4` / 函数 `is_mp4`
```python
def is_mp4(data: bytes) -> bool:
    """Check if data represents a valid MP4 file by magic bytes."""
    if len(data) < 8:
        return False
    return data[4:8] == b"ftyp"
```
**EN:** This function drives `is_mp4` with inputs such as `data`. Check if data represents a valid MP4 file by magic bytes.
**CN:** 这个函数负责 `is_mp4`，主要处理 `data` 等输入。 文档字符串说明：Check if data represents a valid MP4 file by magic bytes.

### Lines 336-338: Function `is_jpeg` / 函数 `is_jpeg`
```python
def is_jpeg(data: bytes) -> bool:
    # JPEG files start with: FF D8 FF
    return data.startswith(b"\xff\xd8\xff")
```
**EN:** This function drives `is_jpeg` with inputs such as `data`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `is_jpeg`，主要处理 `data` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 341-343: Function `is_png` / 函数 `is_png`
```python
def is_png(data):
    # PNG files start with: 89 50 4E 47 0D 0A 1A 0A
    return data.startswith(b"\x89PNG\r\n\x1a\n")
```
**EN:** This function drives `is_png` with inputs such as `data`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `is_png`，主要处理 `data` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 346-348: Function `is_webp` / 函数 `is_webp`
```python
def is_webp(data: bytes) -> bool:
    # WebP files start with: RIFF....WEBP
    return data[:4] == b"RIFF" and data[8:12] == b"WEBP"
```
**EN:** This function drives `is_webp` with inputs such as `data`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `is_webp`，主要处理 `data` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 351-361: Function `detect_image_format` / 函数 `detect_image_format`
```python
def detect_image_format(data: bytes) -> str:
    """Detect image format from bytes (magic). Returns 'png'|'jpeg'|'webp'; default 'png'."""
    if len(data) < 12:
        return "png"
    if is_png(data):
        return "png"
    if is_jpeg(data):
        return "jpeg"
    if is_webp(data):
        return "webp"
    return "png"
```
**EN:** This function drives `detect_image_format` with inputs such as `data`. Detect image format from bytes (magic).
**CN:** 这个函数负责 `detect_image_format`，主要处理 `data` 等输入。 文档字符串说明：Detect image format from bytes (magic).

### Lines 364-380: Function `get_expected_image_format` / 函数 `get_expected_image_format`
```python
def get_expected_image_format(
    output_format: str | None = None,
    background: str | None = None,
) -> str:
    """Infer expected image format based on request parameters.
    Args:
        output_format: The output_format parameter from the request (png/jpeg/webp/jpg)
        background: The background parameter from the request (transparent/opaque/auto)
    Returns:
        Expected file extension: "jpg", "png", or "webp"
    """
    fmt = (output_format or "").lower()
    if fmt in {"png", "webp", "jpeg", "jpg"}:
        return "jpg" if fmt == "jpeg" else fmt
    if (background or "auto").lower() == "transparent":
        return "png"
    return "jpg"  # Default
```
**EN:** This function drives `get_expected_image_format` with inputs such as `output_format`, `background`. Infer expected image format based on request parameters.
**CN:** 这个函数负责 `get_expected_image_format`，主要处理 `output_format`, `background` 等输入。 文档字符串说明：Infer expected image format based on request parameters.

### Lines 383-390: Function `wait_for_port` / 函数 `wait_for_port`
```python
def wait_for_port(host="127.0.0.1", port=30010, deadline=300.0, interval=0.5):
    end = time.time() + deadline
    last_err = None
    while time.time() < end:
        if probe_port(host, port, timeout=interval):
            return True
        time.sleep(interval)
    raise TimeoutError(f"Port {host}:{port} not ready. Last error: {last_err}")
```
**EN:** This function drives `wait_for_port` with inputs such as `host`, `port`, `deadline`, `interval`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `wait_for_port`，主要处理 `host`, `port`, `deadline`, `interval` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 393-395: Function `check_image_size` / 函数 `check_image_size`
```python
def check_image_size(ut, image, width, height):
    # check image size
    ut.assertEqual(image.size, (width, height))
```
**EN:** This function drives `check_image_size` with inputs such as `ut`, `image`, `width`, `height`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `check_image_size`，主要处理 `ut`, `image`, `width`, `height` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 398-406: Function `get_perf_log_dir` / 函数 `get_perf_log_dir`
```python
def get_perf_log_dir() -> Path:
    """Gets the performance log directory from the centralized sglang utility."""
    log_dir_str = get_diffusion_perf_log_dir()
    if not log_dir_str:
        raise RuntimeError(
            "Performance logging is disabled (SGLANG_PERF_LOG_DIR is empty), "
            "but a test tried to access the log directory."
        )
    return Path(log_dir_str)
```
**EN:** This function drives `get_perf_log_dir`. Gets the performance log directory from the centralized sglang utility.
**CN:** 这个函数负责 `get_perf_log_dir`。 文档字符串说明：Gets the performance log directory from the centralized sglang utility.

### Lines 409-411: Function `_ensure_log_path` / 函数 `_ensure_log_path`
```python
def _ensure_log_path(log_dir: Path) -> Path:
    log_dir.mkdir(parents=True, exist_ok=True)
    return log_dir / "performance.log"
```
**EN:** This function drives `_ensure_log_path` with inputs such as `log_dir`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_ensure_log_path`，主要处理 `log_dir` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 414-420: Function `clear_perf_log` / 函数 `clear_perf_log`
```python
def clear_perf_log(log_dir: Path) -> Path:
    """Delete the perf log file so tests can watch for fresh entries."""
    log_path = _ensure_log_path(log_dir)
    if log_path.exists():
        log_path.unlink()
    logger.info("[server-test] Monitoring perf log at %s", log_path.as_posix())
    return log_path
```
**EN:** This function drives `clear_perf_log` with inputs such as `log_dir`. Delete the perf log file so tests can watch for fresh entries.
**CN:** 这个函数负责 `clear_perf_log`，主要处理 `log_dir` 等输入。 文档字符串说明：Delete the perf log file so tests can watch for fresh entries.

### Lines 423-427: Function `prepare_perf_log` / 函数 `prepare_perf_log`
```python
def prepare_perf_log() -> tuple[Path, Path]:
    """Convenience helper to resolve and clear the perf log in one call."""
    log_dir = get_perf_log_dir()
    log_path = clear_perf_log(log_dir)
    return log_dir, log_path
```
**EN:** This function drives `prepare_perf_log`. Convenience helper to resolve and clear the perf log in one call.
**CN:** 这个函数负责 `prepare_perf_log`。 文档字符串说明：Convenience helper to resolve and clear the perf log in one call.

### Lines 430-444: Function `read_perf_logs` / 函数 `read_perf_logs`
```python
def read_perf_logs(log_path: Path) -> list[RequestPerfRecord]:
    if not log_path.exists():
        return []
    records: list[RequestPerfRecord] = []
    with log_path.open("r", encoding="utf-8") as fh:
        for line in fh:
            line = line.strip()
            if not line:
                continue
            try:
                record_dict = json.loads(line)
                records.append(RequestPerfRecord(**record_dict))
            except json.JSONDecodeError:
                continue
    return records
```
**EN:** This function drives `read_perf_logs` with inputs such as `log_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `read_perf_logs`，主要处理 `log_path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 447-469: Function `wait_for_req_perf_record` / 函数 `wait_for_req_perf_record`
```python
def wait_for_req_perf_record(
    request_id: str,
    log_path: Path,
    timeout: float = 30.0,
) -> RequestPerfRecord | None:
    """
    the stage metrics of this request should be in the performance_log file with {request-id}
    """
    logger.info(f"Waiting for req perf record with request id: {request_id}")
    deadline = time.time() + timeout
    while time.time() < deadline:
        records = read_perf_logs(log_path)
        for record in records:
            if record.request_id == request_id:
# ...
        return None

    logger.error(f"record: {records}")
    raise AssertionError(f"Timeout waiting for stage metrics for request {request_id} ")
```
**EN:** This function drives `wait_for_req_perf_record` with inputs such as `request_id`, `log_path`, `timeout`. the stage metrics of this request should be in the performance_log file with {request-id}
**CN:** 这个函数负责 `wait_for_req_perf_record`，主要处理 `request_id`, `log_path`, `timeout` 等输入。 文档字符串说明：the stage metrics of this request should be in the performance_log file with {request-id}

### Lines 472-475: Function `validate_image` / 函数 `validate_image`
```python
def validate_image(b64_json: str) -> None:
    """Decode and validate that image is PNG or JPEG."""
    image_bytes = base64.b64decode(b64_json)
    assert is_png(image_bytes) or is_jpeg(image_bytes), "Image must be PNG or JPEG"
```
**EN:** This function drives `validate_image` with inputs such as `b64_json`. Decode and validate that image is PNG or JPEG.
**CN:** 这个函数负责 `validate_image`，主要处理 `b64_json` 等输入。 文档字符串说明：Decode and validate that image is PNG or JPEG.

### Lines 478-482: Function `validate_video` / 函数 `validate_video`
```python
def validate_video(b64_json: str) -> None:
    """Decode and validate that video is a valid format."""
    video_bytes = base64.b64decode(b64_json)
    is_webm = video_bytes[:4] == b"\x1a\x45\xdf\xa3"
    assert is_mp4(video_bytes) or is_webm, "Video must be MP4 or WebM"
```
**EN:** This function drives `validate_video` with inputs such as `b64_json`. Decode and validate that video is a valid format.
**CN:** 这个函数负责 `validate_video`，主要处理 `b64_json` 等输入。 文档字符串说明：Decode and validate that video is a valid format.

### Lines 485-488: Function `validate_openai_video` / 函数 `validate_openai_video`
```python
def validate_openai_video(video_bytes: bytes) -> None:
    """Validate that video is MP4 or WebM by magic bytes."""
    is_webm = video_bytes.startswith(b"\x1a\x45\xdf\xa3")
    assert is_mp4(video_bytes) or is_webm, "Video must be MP4 or WebM"
```
**EN:** This function drives `validate_openai_video` with inputs such as `video_bytes`. Validate that video is MP4 or WebM by magic bytes.
**CN:** 这个函数负责 `validate_openai_video`，主要处理 `video_bytes` 等输入。 文档字符串说明：Validate that video is MP4 or WebM by magic bytes.

### Lines 491-540: Function `validate_image_file` / 函数 `validate_image_file`
```python
def validate_image_file(
    file_path: str,
    expected_filename: str,
    expected_width: int | None = None,
    expected_height: int | None = None,
    output_format: str | None = None,
    background: str | None = None,
) -> None:
    """Validate image output file: existence, extension, size, filename, format, dimensions."""
    # Infer expected format from request parameters
    expected_ext = get_expected_image_format(output_format, background)

    # 1. File existence
    assert os.path.exists(file_path), f"Image file does not exist: {file_path}"
# ...
            ), f"Width mismatch: expected {expected_width}, got {width}"
            assert (
                height == expected_height
            ), f"Height mismatch: expected {expected_height}, got {height}"
```
**EN:** This function drives `validate_image_file` with inputs such as `file_path`, `expected_filename`, `expected_width`, `expected_height`. Validate image output file: existence, extension, size, filename, format, dimensions.
**CN:** 这个函数负责 `validate_image_file`，主要处理 `file_path`, `expected_filename`, `expected_width`, `expected_height` 等输入。 文档字符串说明：Validate image output file: existence, extension, size, filename, format, dimensions.

### Lines 543-560: Function `_get_video_dimensions_from_metadata` / 函数 `_get_video_dimensions_from_metadata`
```python
def _get_video_dimensions_from_metadata(
    cap: cv2.VideoCapture,
) -> tuple[int, int] | None:
    """Get video dimensions from metadata properties.

    Args:
        cap: OpenCV VideoCapture object

    Returns:
        Tuple of (width, height) if successful, None if metadata is invalid
    """
    width = cap.get(cv2.CAP_PROP_FRAME_WIDTH)
    height = cap.get(cv2.CAP_PROP_FRAME_HEIGHT)

    if width == 0 or height == 0:
        return None

    return int(width), int(height)
```
**EN:** This function drives `_get_video_dimensions_from_metadata` with inputs such as `cap`. Get video dimensions from metadata properties.
**CN:** 这个函数负责 `_get_video_dimensions_from_metadata`，主要处理 `cap` 等输入。 文档字符串说明：Get video dimensions from metadata properties.

### Lines 563-579: Function `_get_video_dimensions_from_frame` / 函数 `_get_video_dimensions_from_frame`
```python
def _get_video_dimensions_from_frame(cap: cv2.VideoCapture) -> tuple[int, int]:
    """Get video dimensions by reading the first frame.

    Args:
        cap: OpenCV VideoCapture object

    Returns:
        Tuple of (width, height)

    """
    ret, frame = cap.read()
    if not ret or frame is None:
        raise ValueError("Unable to read video frame to get dimensions")

    # frame.shape is (height, width, channels)
    height, width = frame.shape[:2]
    return int(width), int(height)
```
**EN:** This function drives `_get_video_dimensions_from_frame` with inputs such as `cap`. Get video dimensions by reading the first frame.
**CN:** 这个函数负责 `_get_video_dimensions_from_frame`，主要处理 `cap` 等输入。 文档字符串说明：Get video dimensions by reading the first frame.

### Lines 582-601: Function `get_video_dimensions` / 函数 `get_video_dimensions`
```python
def get_video_dimensions(file_path: str) -> tuple[int, int]:
    """Get video dimensions (width, height) from a video file.

    Tries to get dimensions from metadata first, falls back to reading first frame.

    Returns:
        Tuple of (width, height)

    """
    cap = cv2.VideoCapture(file_path)
    try:
        # Try to get dimensions from metadata first
        dimensions = _get_video_dimensions_from_metadata(cap)
        if dimensions is not None:
# ...
        # Fall back to reading first frame
        return _get_video_dimensions_from_frame(cap)
    finally:
        cap.release()
```
**EN:** This function drives `get_video_dimensions` with inputs such as `file_path`. Get video dimensions (width, height) from a video file.
**CN:** 这个函数负责 `get_video_dimensions`，主要处理 `file_path` 等输入。 文档字符串说明：Get video dimensions (width, height) from a video file.

### Lines 604-617: Function `get_video_frame_count` / 函数 `get_video_frame_count`
```python
def get_video_frame_count(file_path: str) -> int:
    """Return the number of frames in a video file using OpenCV."""
    cap = cv2.VideoCapture(file_path)
    try:
        count = int(cap.get(cv2.CAP_PROP_FRAME_COUNT))
        if count > 0:
            return count
        # Fallback: count frames manually
        n = 0
        while cap.read()[0]:
            n += 1
        return n
    finally:
        cap.release()
```
**EN:** This function drives `get_video_frame_count` with inputs such as `file_path`. Return the number of frames in a video file using OpenCV.
**CN:** 这个函数负责 `get_video_frame_count`，主要处理 `file_path` 等输入。 文档字符串说明：Return the number of frames in a video file using OpenCV.

### Lines 620-656: Function `validate_video_file` / 函数 `validate_video_file`
```python
def validate_video_file(
    file_path: str,
    expected_filename: str,
    expected_width: int | None = None,
    expected_height: int | None = None,
) -> None:
    """Validate video output file: existence, extension, size, filename, format, dimensions."""
    # 1. File existence
    assert os.path.exists(file_path), f"Video file does not exist: {file_path}"

    # 2. Extension check
    assert file_path.endswith(".mp4"), f"Expected .mp4 extension, got: {file_path}"

    # 3. File size > 0
# ...
        ), f"Video width mismatch: expected {expected_width}, got {actual_width}"
        assert (
            actual_height == expected_height
        ), f"Video height mismatch: expected {expected_height}, got {actual_height}"
```
**EN:** This function drives `validate_video_file` with inputs such as `file_path`, `expected_filename`, `expected_width`, `expected_height`. Validate video output file: existence, extension, size, filename, format, dimensions.
**CN:** 这个函数负责 `validate_video_file`，主要处理 `file_path`, `expected_filename`, `expected_width`, `expected_height` 等输入。 文档字符串说明：Validate video output file: existence, extension, size, filename, format, dimensions.

### Lines 659-664: Function `_load_threshold_json` / 函数 `_load_threshold_json`
```python
def _load_threshold_json() -> dict[str, Any]:
    """Load consistency_threshold.json; returns {} if missing."""
    if not CONSISTENCY_THRESHOLD_JSON_PATH.exists():
        return {}
    with CONSISTENCY_THRESHOLD_JSON_PATH.open("r", encoding="utf-8") as f:
        return json.load(f)
```
**EN:** This function drives `_load_threshold_json`. Load consistency_threshold.json; returns {} if missing.
**CN:** 这个函数负责 `_load_threshold_json`。 文档字符串说明：Load consistency_threshold.json; returns {} if missing.

### Lines 667-672: Class `ConsistencyThresholds` / 类 `ConsistencyThresholds`
```python
@dataclass
class ConsistencyThresholds:
    clip_threshold: float
    ssim_threshold: float
    psnr_threshold: float
    mean_abs_diff_threshold: float
```
**EN:** This class models `ConsistencyThresholds`.
**CN:** 该类实现 `ConsistencyThresholds`。

### Lines 675-725: Function `get_consistency_thresholds` / 函数 `get_consistency_thresholds`
```python
def get_consistency_thresholds(
    case_id: str,
    is_video: bool,
    metadata: dict[str, Any] | None = None,
) -> ConsistencyThresholds:
    """Get all consistency thresholds for a case."""
    if metadata is None:
        metadata = _load_threshold_json()

    case_meta = metadata.get("cases", {}).get(case_id, {})
    suffix = "video" if is_video else "image"

    defaults = {
        "clip_threshold": metadata.get(
# ...
                "mean_abs_diff_threshold", defaults["mean_abs_diff_threshold"]
            )
        ),
    )
```
**EN:** This function drives `get_consistency_thresholds` with inputs such as `case_id`, `is_video`, `metadata`. Get all consistency thresholds for a case.
**CN:** 这个函数负责 `get_consistency_thresholds`，主要处理 `case_id`, `is_video`, `metadata` 等输入。 文档字符串说明：Get all consistency thresholds for a case.

### Lines 728-737: Function `get_clip_threshold` / 函数 `get_clip_threshold`
```python
def get_clip_threshold(
    case: "DiffusionTestCase",
    metadata: dict[str, Any] | None = None,
) -> float:
    """Get CLIP similarity threshold for a consistency test case."""
    return get_consistency_thresholds(
        case_id=case.id,
        is_video=case.server_args.modality == "video",
        metadata=metadata,
    ).clip_threshold
```
**EN:** This function drives `get_clip_threshold` with inputs such as `case`, `metadata`. Get CLIP similarity threshold for a consistency test case.
**CN:** 这个函数负责 `get_clip_threshold`，主要处理 `case`, `metadata` 等输入。 文档字符串说明：Get CLIP similarity threshold for a consistency test case.

### Lines 740-750: Class `FrameConsistencyMetrics` / 类 `FrameConsistencyMetrics`
```python
@dataclass
class FrameConsistencyMetrics:
    frame_index: int
    clip_similarity: float
    ssim: float
    psnr: float
    mean_abs_diff: float
    clip_passed: bool
    ssim_passed: bool
    psnr_passed: bool
    mean_abs_diff_passed: bool
```
**EN:** This class models `FrameConsistencyMetrics`.
**CN:** 该类实现 `FrameConsistencyMetrics`。

### Lines 753-766: Class `ConsistencyResult` / 类 `ConsistencyResult`
```python
@dataclass
class ConsistencyResult:
    """Result of a consistency comparison."""

    case_id: str
    passed: bool
    similarity_scores: list[float]
    min_similarity: float
    threshold: float
    min_ssim: float
    min_psnr: float
    max_mean_abs_diff: float
    thresholds: ConsistencyThresholds
    frame_metrics: list[FrameConsistencyMetrics]
```
**EN:** This class models `ConsistencyResult`. Result of a consistency comparison.
**CN:** 该类实现 `ConsistencyResult`。 文档字符串指出：Result of a consistency comparison.

### Lines 769-772: Class `LoadedConsistencyGT` / 类 `LoadedConsistencyGT`
```python
@dataclass
class LoadedConsistencyGT:
    images: list[np.ndarray]
    embeddings: list[np.ndarray]
```
**EN:** This class models `LoadedConsistencyGT`.
**CN:** 该类实现 `LoadedConsistencyGT`。

### Lines 775-816: Function `get_clip_model` / 函数 `get_clip_model`
```python
def get_clip_model() -> tuple[Any, Any]:
    """Get CLIP model and processor."""
    global _clip_model_cache

    if "model" not in _clip_model_cache:
        try:
            import torch
            from transformers import CLIPModel, CLIPProcessor
        except ImportError as exc:
            raise ImportError(
                "transformers and torch are required for CLIP consistency check."
            ) from exc

        logger.info(f"Loading CLIP model: {CLIP_MODEL_NAME}")
# ...
        _clip_model_cache["device"] = device
        logger.info(f"CLIP model loaded on {device}")

    return _clip_model_cache["model"], _clip_model_cache["processor"]
```
**EN:** This function drives `get_clip_model`. Get CLIP model and processor.
**CN:** 这个函数负责 `get_clip_model`。 文档字符串说明：Get CLIP model and processor.

### Lines 819-841: Function `compute_clip_embedding` / 函数 `compute_clip_embedding`
```python
def compute_clip_embedding(image: np.ndarray) -> np.ndarray:
    """Compute a normalized CLIP image embedding."""
    try:
        import torch
    except ImportError as exc:
        raise ImportError("torch is required for CLIP consistency check.") from exc

    model, processor = get_clip_model()
    device = _clip_model_cache["device"]

    pil_image = Image.fromarray(image)
    inputs = processor(images=pil_image, return_tensors="pt")
    inputs = {k: v.to(device) for k, v in inputs.items()}

# ...
            image_features = image_features.pooler_output
        image_features = image_features / image_features.norm(dim=-1, keepdim=True)

    return image_features.cpu().numpy().flatten()
```
**EN:** This function drives `compute_clip_embedding` with inputs such as `image`. Compute a normalized CLIP image embedding.
**CN:** 这个函数负责 `compute_clip_embedding`，主要处理 `image` 等输入。 文档字符串说明：Compute a normalized CLIP image embedding.

### Lines 844-846: Function `compute_clip_similarity` / 函数 `compute_clip_similarity`
```python
def compute_clip_similarity(emb1: np.ndarray, emb2: np.ndarray) -> float:
    """Compute cosine similarity between two CLIP embeddings."""
    return float(np.dot(emb1, emb2))
```
**EN:** This function drives `compute_clip_similarity` with inputs such as `emb1`, `emb2`. Compute cosine similarity between two CLIP embeddings.
**CN:** 这个函数负责 `compute_clip_similarity`，主要处理 `emb1`, `emb2` 等输入。 文档字符串说明：Compute cosine similarity between two CLIP embeddings.

### Lines 849-856: Function `_ensure_rgb_uint8_image` / 函数 `_ensure_rgb_uint8_image`
```python
def _ensure_rgb_uint8_image(image: np.ndarray) -> np.ndarray:
    """Normalize image input for pixel-wise consistency metrics."""
    if image.ndim != 3 or image.shape[2] != 3:
        raise ValueError(f"Expected RGB HWC image, got shape={image.shape}")
    if image.dtype == np.uint8:
        return image
    image = np.clip(image, 0, 255)
    return image.astype(np.uint8)
```
**EN:** This function drives `_ensure_rgb_uint8_image` with inputs such as `image`. Normalize image input for pixel-wise consistency metrics.
**CN:** 这个函数负责 `_ensure_rgb_uint8_image`，主要处理 `image` 等输入。 文档字符串说明：Normalize image input for pixel-wise consistency metrics.

### Lines 859-869: Function `compute_ssim` / 函数 `compute_ssim`
```python
def compute_ssim(image: np.ndarray, gt_image: np.ndarray) -> float:
    """Compute SSIM between two RGB images."""
    from skimage.metrics import structural_similarity

    image = _ensure_rgb_uint8_image(image)
    gt_image = _ensure_rgb_uint8_image(gt_image)
    if image.shape != gt_image.shape:
        raise ValueError(
            f"Image shape mismatch for SSIM: output={image.shape}, gt={gt_image.shape}"
        )
    return float(structural_similarity(image, gt_image, channel_axis=2, data_range=255))
```
**EN:** This function drives `compute_ssim` with inputs such as `image`, `gt_image`. Compute SSIM between two RGB images.
**CN:** 这个函数负责 `compute_ssim`，主要处理 `image`, `gt_image` 等输入。 文档字符串说明：Compute SSIM between two RGB images.

### Lines 872-882: Function `compute_psnr` / 函数 `compute_psnr`
```python
def compute_psnr(image: np.ndarray, gt_image: np.ndarray) -> float:
    """Compute PSNR between two RGB images."""
    from skimage.metrics import peak_signal_noise_ratio

    image = _ensure_rgb_uint8_image(image)
    gt_image = _ensure_rgb_uint8_image(gt_image)
    if image.shape != gt_image.shape:
        raise ValueError(
            f"Image shape mismatch for PSNR: output={image.shape}, gt={gt_image.shape}"
        )
    return float(peak_signal_noise_ratio(gt_image, image, data_range=255))
```
**EN:** This function drives `compute_psnr` with inputs such as `image`, `gt_image`. Compute PSNR between two RGB images.
**CN:** 这个函数负责 `compute_psnr`，主要处理 `image`, `gt_image` 等输入。 文档字符串说明：Compute PSNR between two RGB images.

### Lines 885-893: Function `compute_mean_abs_diff` / 函数 `compute_mean_abs_diff`
```python
def compute_mean_abs_diff(image: np.ndarray, gt_image: np.ndarray) -> float:
    """Compute mean absolute pixel difference between two RGB images."""
    image = _ensure_rgb_uint8_image(image)
    gt_image = _ensure_rgb_uint8_image(gt_image)
    if image.shape != gt_image.shape:
        raise ValueError(
            f"Image shape mismatch for mean_abs_diff: output={image.shape}, gt={gt_image.shape}"
        )
    return float(np.abs(image.astype(np.float32) - gt_image.astype(np.float32)).mean())
```
**EN:** This function drives `compute_mean_abs_diff` with inputs such as `image`, `gt_image`. Compute mean absolute pixel difference between two RGB images.
**CN:** 这个函数负责 `compute_mean_abs_diff`，主要处理 `image`, `gt_image` 等输入。 文档字符串说明：Compute mean absolute pixel difference between two RGB images.

### Lines 896-905: Function `output_format_to_ext` / 函数 `output_format_to_ext`
```python
def output_format_to_ext(output_format: str | None) -> str:
    """Map output_format to file extension. Used by GT naming and consistency check."""
    if not output_format:
        return "jpg"
    of = output_format.lower()
    if of == "jpeg":
        return "jpg"
    if of in ("png", "webp", "jpg"):
        return of
    return "png"
```
**EN:** This function drives `output_format_to_ext` with inputs such as `output_format`. Map output_format to file extension.
**CN:** 这个函数负责 `output_format_to_ext`，主要处理 `output_format` 等输入。 文档字符串说明：Map output_format to file extension.

### Lines 908-920: Function `_consistency_gt_filenames` / 函数 `_consistency_gt_filenames`
```python
def _consistency_gt_filenames(
    case_id: str, num_gpus: int, is_video: bool, output_format: str | None = None
) -> list[str]:
    """Return the list of GT image filenames for a case. Reused by GT generation and consistency check."""
    n = num_gpus
    if is_video:
        return [
            f"{case_id}_{n}gpu_frame_0.png",
            f"{case_id}_{n}gpu_frame_mid.png",
            f"{case_id}_{n}gpu_frame_last.png",
        ]
    ext = output_format_to_ext(output_format)
    return [f"{case_id}_{n}gpu.{ext}"]
```
**EN:** This function drives `_consistency_gt_filenames` with inputs such as `case_id`, `num_gpus`, `is_video`, `output_format`. Return the list of GT image filenames for a case.
**CN:** 这个函数负责 `_consistency_gt_filenames`，主要处理 `case_id`, `num_gpus`, `is_video`, `output_format` 等输入。 文档字符串说明：Return the list of GT image filenames for a case.

### Lines 923-937: Function `get_consistency_gt_candidates` / 函数 `get_consistency_gt_candidates`
```python
def get_consistency_gt_candidates(
    case_id: str, num_gpus: int, is_video: bool, output_format: str | None = None
) -> list[str]:
    """Return candidate GT filenames for local consistency data."""
    n = num_gpus
    if is_video:
        return [
            f"{case_id}_{n}gpu_frame_0.png",
            f"{case_id}_{n}gpu_frame_mid.png",
            f"{case_id}_{n}gpu_frame_last.png",
        ]
    base = f"{case_id}_{n}gpu"
    preferred = output_format_to_ext(output_format)
    exts = [preferred] + [e for e in ("png", "jpg", "webp") if e != preferred]
    return [f"{base}.{e}" for e in exts]
```
**EN:** This function drives `get_consistency_gt_candidates` with inputs such as `case_id`, `num_gpus`, `is_video`, `output_format`. Return candidate GT filenames for local consistency data.
**CN:** 这个函数负责 `get_consistency_gt_candidates`，主要处理 `case_id`, `num_gpus`, `is_video`, `output_format` 等输入。 文档字符串说明：Return candidate GT filenames for local consistency data.

### Lines 940-952: Function `get_consistency_gt_remote_files` / 函数 `get_consistency_gt_remote_files`
```python
def get_consistency_gt_remote_files(
    case_id: str, num_gpus: int, is_video: bool, output_format: str | None = None
) -> list[tuple[str, str]]:
    """Return GT filenames with their remote raw URLs."""
    files = _find_remote_consistency_gt_files(
        case_id, num_gpus, is_video, output_format
    )
    if files:
        return files

    return _remote_consistency_gt_candidates(
        SGL_TEST_FILES_CONSISTENCY_GT_BASE, case_id, num_gpus, is_video, output_format
    )
```
**EN:** This function drives `get_consistency_gt_remote_files` with inputs such as `case_id`, `num_gpus`, `is_video`, `output_format`. Return GT filenames with their remote raw URLs.
**CN:** 这个函数负责 `get_consistency_gt_remote_files`，主要处理 `case_id`, `num_gpus`, `is_video`, `output_format` 等输入。 文档字符串说明：Return GT filenames with their remote raw URLs.

### Lines 955-965: Function `_remote_consistency_gt_candidates` / 函数 `_remote_consistency_gt_candidates`
```python
def _remote_consistency_gt_candidates(
    base_url: str,
    case_id: str,
    num_gpus: int,
    is_video: bool,
    output_format: str | None = None,
) -> list[tuple[str, str]]:
    filenames = get_consistency_gt_candidates(
        case_id, num_gpus, is_video, output_format
    )
    return [(filename, f"{base_url}/{filename}") for filename in filenames]
```
**EN:** This function drives `_remote_consistency_gt_candidates` with inputs such as `base_url`, `case_id`, `num_gpus`, `is_video`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_remote_consistency_gt_candidates`，主要处理 `base_url`, `case_id`, `num_gpus`, `is_video` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 968-990: Function `_remote_file_exists` / 函数 `_remote_file_exists`
```python
def _remote_file_exists(url: str) -> bool:
    for method in ("head", "get"):
        try:
            if method == "head":
                resp = requests.head(url, timeout=10, allow_redirects=True)
            else:
                resp = requests.get(
                    url,
                    timeout=10,
                    allow_redirects=True,
                    headers={"Range": "bytes=0-0"},
                    stream=True,
                )
            try:
# ...
                resp.close()
        except requests.RequestException:
            pass
    return False
```
**EN:** This function drives `_remote_file_exists` with inputs such as `url`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_remote_file_exists`，主要处理 `url` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 993-1015: Function `_find_remote_consistency_gt_files` / 函数 `_find_remote_consistency_gt_files`
```python
def _find_remote_consistency_gt_files(
    case_id: str,
    num_gpus: int,
    is_video: bool,
    output_format: str | None = None,
) -> list[tuple[str, str]]:
    if case_id in SGL_TEST_FILES_OFFICIAL_CONSISTENCY_GT_CASES:
        bases = SGL_TEST_FILES_CONSISTENCY_GT_BASES
    else:
        # Avoid accidentally comparing non-comparable CI cases against official GT.
        bases = (SGL_TEST_FILES_SGLANG_CONSISTENCY_GT_BASE,)
    for base_url in bases:
        candidates = _remote_consistency_gt_candidates(
            base_url, case_id, num_gpus, is_video, output_format
# ...
            for filename, url in candidates:
                if _remote_file_exists(url):
                    return [(filename, url)]
    return []
```
**EN:** This function drives `_find_remote_consistency_gt_files` with inputs such as `case_id`, `num_gpus`, `is_video`, `output_format`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_find_remote_consistency_gt_files`，主要处理 `case_id`, `num_gpus`, `is_video`, `output_format` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1018-1023: Function `_get_consistency_gt_dir` / 函数 `_get_consistency_gt_dir`
```python
def _get_consistency_gt_dir() -> Path | None:
    """Return the local GT directory when configured."""
    d = os.environ.get("SGLANG_CONSISTENCY_GT_DIR")
    if not d:
        return None
    return Path(d).resolve()
```
**EN:** This function drives `_get_consistency_gt_dir`. Return the local GT directory when configured.
**CN:** 这个函数负责 `_get_consistency_gt_dir`。 文档字符串说明：Return the local GT directory when configured.

### Lines 1026-1034: Function `_get_consistency_gt_cache_key` / 函数 `_get_consistency_gt_cache_key`
```python
def _get_consistency_gt_cache_key(
    case_id: str,
    num_gpus: int,
    is_video: bool,
    output_format: str | None,
) -> str:
    gt_dir = _get_consistency_gt_dir()
    source = str(gt_dir) if gt_dir is not None else "remote"
    return f"{case_id}:{num_gpus}:{is_video}:{output_format or ''}:{source}"
```
**EN:** This function drives `_get_consistency_gt_cache_key` with inputs such as `case_id`, `num_gpus`, `is_video`, `output_format`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_get_consistency_gt_cache_key`，主要处理 `case_id`, `num_gpus`, `is_video`, `output_format` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1037-1098: Function `load_consistency_gt` / 函数 `load_consistency_gt`
```python
def load_consistency_gt(
    case_id: str,
    num_gpus: int,
    is_video: bool = False,
    output_format: str | None = None,
) -> LoadedConsistencyGT:
    """Load GT images and CLIP embeddings for consistency checks."""
    cache_key = _get_consistency_gt_cache_key(
        case_id, num_gpus, is_video, output_format
    )
    cached = _consistency_gt_cache.get(cache_key)
    if cached is not None:
        return cached

# ...
    embeddings = [compute_clip_embedding(arr) for arr in images]
    loaded_gt = LoadedConsistencyGT(images=images, embeddings=embeddings)
    _consistency_gt_cache[cache_key] = loaded_gt
    return loaded_gt
```
**EN:** This function drives `load_consistency_gt` with inputs such as `case_id`, `num_gpus`, `is_video`, `output_format`. Load GT images and CLIP embeddings for consistency checks.
**CN:** 这个函数负责 `load_consistency_gt`，主要处理 `case_id`, `num_gpus`, `is_video`, `output_format` 等输入。 文档字符串说明：Load GT images and CLIP embeddings for consistency checks.

### Lines 1101-1113: Function `load_gt_embeddings` / 函数 `load_gt_embeddings`
```python
def load_gt_embeddings(
    case_id: str,
    num_gpus: int,
    is_video: bool = False,
    output_format: str | None = None,
) -> list[np.ndarray]:
    """Load GT images and convert them into CLIP embeddings."""
    return load_consistency_gt(
        case_id=case_id,
        num_gpus=num_gpus,
        is_video=is_video,
        output_format=output_format,
    ).embeddings
```
**EN:** This function drives `load_gt_embeddings` with inputs such as `case_id`, `num_gpus`, `is_video`, `output_format`. Load GT images and convert them into CLIP embeddings.
**CN:** 这个函数负责 `load_gt_embeddings`，主要处理 `case_id`, `num_gpus`, `is_video`, `output_format` 等输入。 文档字符串说明：Load GT images and convert them into CLIP embeddings.

### Lines 1116-1134: Function `gt_exists` / 函数 `gt_exists`
```python
def gt_exists(
    case_id: str,
    num_gpus: int,
    is_video: bool = False,
    output_format: str | None = None,
) -> bool:
    """Check whether GT image(s) exist."""
    gt_dir = _get_consistency_gt_dir()
    if gt_dir is not None:
        candidates = get_consistency_gt_candidates(
            case_id, num_gpus, is_video, output_format
        )
        if is_video:
            return all((gt_dir / c).exists() for c in candidates)
        return any((gt_dir / c).exists() for c in candidates)

    return bool(
        _find_remote_consistency_gt_files(case_id, num_gpus, is_video, output_format)
    )
```
**EN:** This function drives `gt_exists` with inputs such as `case_id`, `num_gpus`, `is_video`, `output_format`. Check whether GT image(s) exist.
**CN:** 这个函数负责 `gt_exists`，主要处理 `case_id`, `num_gpus`, `is_video`, `output_format` 等输入。 文档字符串说明：Check whether GT image(s) exist.

### Lines 1137-1186: Function `extract_key_frames_from_video` / 函数 `extract_key_frames_from_video`
```python
def extract_key_frames_from_video(
    video_bytes: bytes,
    num_frames: int | None = None,
) -> list[np.ndarray]:
    """
    Extract key frames (first, middle, last) from video bytes.

    Args:
        video_bytes: Raw video bytes (MP4 format)
        num_frames: Total number of frames (if known), used for validation

    Returns:
        List of numpy arrays [first_frame, middle_frame, last_frame].
    """
# ...
        return frames

    finally:
        os.unlink(tmp_path)
```
**EN:** This function drives `extract_key_frames_from_video` with inputs such as `video_bytes`, `num_frames`. Extract key frames (first, middle, last) from video bytes.
**CN:** 这个函数负责 `extract_key_frames_from_video`，主要处理 `video_bytes`, `num_frames` 等输入。 文档字符串说明：Extract key frames (first, middle, last) from video bytes.

### Lines 1189-1192: Function `image_bytes_to_numpy` / 函数 `image_bytes_to_numpy`
```python
def image_bytes_to_numpy(image_bytes: bytes) -> np.ndarray:
    """Convert image bytes to numpy array."""
    img = Image.open(io.BytesIO(image_bytes)).convert("RGB")
    return np.array(img)
```
**EN:** This function drives `image_bytes_to_numpy` with inputs such as `image_bytes`. Convert image bytes to numpy array.
**CN:** 这个函数负责 `image_bytes_to_numpy`，主要处理 `image_bytes` 等输入。 文档字符串说明：Convert image bytes to numpy array.

### Lines 1195-1303: Function `compare_with_gt` / 函数 `compare_with_gt`
```python
def compare_with_gt(
    output_frames: list[np.ndarray],
    gt_data: LoadedConsistencyGT,
    thresholds: ConsistencyThresholds,
    case_id: str,
) -> ConsistencyResult:
    """Compare output frames with GT using CLIP and pixel-level metrics."""
    if len(output_frames) != len(gt_data.embeddings):
        raise ValueError(
            f"Frame count mismatch: output={len(output_frames)}, gt={len(gt_data.embeddings)}"
        )

    similarity_scores = []
    frame_metrics: list[FrameConsistencyMetrics] = []
# ...
        )
    print(f"{'=' * 60}\n")

    return result
```
**EN:** This function drives `compare_with_gt` with inputs such as `output_frames`, `gt_data`, `thresholds`, `case_id`. Compare output frames with GT using CLIP and pixel-level metrics.
**CN:** 这个函数负责 `compare_with_gt`，主要处理 `output_frames`, `gt_data`, `thresholds`, `case_id` 等输入。 文档字符串说明：Compare output frames with GT using CLIP and pixel-level metrics.

### Lines 1306-1307: Function `_safe_artifact_name` / 函数 `_safe_artifact_name`
```python
def _safe_artifact_name(name: str) -> str:
    return "".join(c if c.isalnum() or c in "._-" else "_" for c in name)
```
**EN:** This function drives `_safe_artifact_name` with inputs such as `name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_safe_artifact_name`，主要处理 `name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1310-1315: Function `_format_metric_value` / 函数 `_format_metric_value`
```python
def _format_metric_value(value: float) -> str:
    if math.isinf(value):
        return "inf"
    if math.isnan(value):
        return "nan"
    return f"{value:.4f}"
```
**EN:** This function drives `_format_metric_value` with inputs such as `value`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_format_metric_value`，主要处理 `value` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1318-1321: Function `_json_metric_value` / 函数 `_json_metric_value`
```python
def _json_metric_value(value: float) -> float | str:
    if math.isinf(value) or math.isnan(value):
        return _format_metric_value(value)
    return round(value, 6)
```
**EN:** This function drives `_json_metric_value` with inputs such as `value`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_json_metric_value`，主要处理 `value` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1324-1330: Function `_metric_items` / 函数 `_metric_items`
```python
def _metric_items(metric: FrameConsistencyMetrics) -> list[tuple[str, float, bool]]:
    return [
        ("clip", metric.clip_similarity, metric.clip_passed),
        ("ssim", metric.ssim, metric.ssim_passed),
        ("psnr", metric.psnr, metric.psnr_passed),
        ("mean_abs_diff", metric.mean_abs_diff, metric.mean_abs_diff_passed),
    ]
```
**EN:** This function drives `_metric_items` with inputs such as `metric`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_metric_items`，主要处理 `metric` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1333-1335: Function `_text_width` / 函数 `_text_width`
```python
def _text_width(draw: ImageDraw.ImageDraw, text: str, font: ImageFont.ImageFont) -> int:
    box = draw.textbbox((0, 0), text, font=font)
    return box[2] - box[0]
```
**EN:** This function drives `_text_width` with inputs such as `draw`, `text`, `font`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_text_width`，主要处理 `draw`, `text`, `font` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1338-1341: Function `_resize_for_comparison` / 函数 `_resize_for_comparison`
```python
def _resize_for_comparison(image: np.ndarray, max_size: tuple[int, int]) -> Image.Image:
    pil_image = Image.fromarray(_ensure_rgb_uint8_image(image)).copy()
    pil_image.thumbnail(max_size, Image.Resampling.LANCZOS)
    return pil_image
```
**EN:** This function drives `_resize_for_comparison` with inputs such as `image`, `max_size`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_resize_for_comparison`，主要处理 `image`, `max_size` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1344-1360: Function `_draw_metric_items` / 函数 `_draw_metric_items`
```python
def _draw_metric_items(
    draw: ImageDraw.ImageDraw,
    x: int,
    y: int,
    metric: FrameConsistencyMetrics,
    font: ImageFont.ImageFont,
) -> None:
    cursor = x
    for index, (name, value, passed) in enumerate(_metric_items(metric)):
        text = f"{name}={_format_metric_value(value)}"
        fill = (30, 110, 55) if passed else (185, 35, 35)
        draw.text((cursor, y), text, fill=fill, font=font)
        cursor += _text_width(draw, text, font)
        if index != 3:
            separator = " | "
            draw.text((cursor, y), separator, fill=(95, 95, 95), font=font)
            cursor += _text_width(draw, separator, font)
```
**EN:** This function drives `_draw_metric_items` with inputs such as `draw`, `x`, `y`, `metric`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_draw_metric_items`，主要处理 `draw`, `x`, `y`, `metric` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1363-1454: Function `_make_consistency_failure_image` / 函数 `_make_consistency_failure_image`
```python
def _make_consistency_failure_image(
    case_id: str,
    num_gpus: int,
    output_frames: list[np.ndarray],
    gt_data: LoadedConsistencyGT,
    result: ConsistencyResult,
    is_video: bool,
) -> Image.Image:
    font = ImageFont.load_default()
    max_thumb_size = (520, 520) if len(output_frames) == 1 else (480, 320)
    gt_thumbs = [
        _resize_for_comparison(image, max_thumb_size) for image in gt_data.images
    ]
    output_thumbs = [
# ...
        _draw_metric_items(draw, left_x, metric_y, result.frame_metrics[idx], font)
        y += row_height + row_gap

    return image
```
**EN:** This function drives `_make_consistency_failure_image` with inputs such as `case_id`, `num_gpus`, `output_frames`, `gt_data`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_make_consistency_failure_image`，主要处理 `case_id`, `num_gpus`, `output_frames`, `gt_data` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1457-1504: Function `_consistency_failure_record` / 函数 `_consistency_failure_record`
```python
def _consistency_failure_record(
    case_id: str,
    num_gpus: int,
    result: ConsistencyResult,
    is_video: bool,
    output_format: str | None,
    image_name: str,
    generated_files: list[str],
    gt_remote_files: list[tuple[str, str]] | None,
) -> dict[str, Any]:
    return {
        "case_id": case_id,
        "num_gpus": num_gpus,
        "is_video": is_video,
# ...
            {"filename": filename, "url": url}
            for filename, url in (gt_remote_files or [])
        ],
    }
```
**EN:** This function drives `_consistency_failure_record` with inputs such as `case_id`, `num_gpus`, `result`, `is_video`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_consistency_failure_record`，主要处理 `case_id`, `num_gpus`, `result`, `is_video` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1507-1534: Function `_save_generated_artifact_images` / 函数 `_save_generated_artifact_images`
```python
def _save_generated_artifact_images(
    out_dir: Path,
    case_id: str,
    num_gpus: int,
    output_frames: list[np.ndarray],
    is_video: bool,
    output_format: str | None,
) -> list[str]:
    generated_dir = out_dir / "generated"
    generated_dir.mkdir(parents=True, exist_ok=True)

    safe_case_id = _safe_artifact_name(case_id)
    if is_video:
        suffixes = ("frame_0", "frame_mid", "frame_last")
# ...
        path = generated_dir / filename
        Image.fromarray(_ensure_rgb_uint8_image(frame)).save(path)
        generated_files.append(str(path.relative_to(out_dir)))
    return generated_files
```
**EN:** This function drives `_save_generated_artifact_images` with inputs such as `out_dir`, `case_id`, `num_gpus`, `output_frames`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_save_generated_artifact_images`，主要处理 `out_dir`, `case_id`, `num_gpus`, `output_frames` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1537-1582: Function `_write_consistency_failure_index` / 函数 `_write_consistency_failure_index`
```python
def _write_consistency_failure_index(
    out_dir: Path,
    records: list[dict[str, Any]],
) -> None:
    sections = []
    for record in sorted(records, key=lambda r: (r["case_id"], r["num_gpus"])):
        case_id = html.escape(record["case_id"])
        png = html.escape(record["comparison_png"])
        metrics = record["metrics"]
        generated_links = "".join(
            f'<li><a href="{html.escape(path)}">{html.escape(path)}</a></li>'
            for path in record.get("generated_files", [])
        )
        generated_html = (
# ...
        "</style></head><body>"
        "<h1>Diffusion consistency failures</h1>" + "".join(sections) + "</body></html>"
    )
    (out_dir / "index.html").write_text(doc, encoding="utf-8")
```
**EN:** This function drives `_write_consistency_failure_index` with inputs such as `out_dir`, `records`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_write_consistency_failure_index`，主要处理 `out_dir`, `records` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1585-1649: Function `save_consistency_failure_artifact` / 函数 `save_consistency_failure_artifact`
```python
def save_consistency_failure_artifact(
    artifact_dir: str | Path | None,
    case_id: str,
    num_gpus: int,
    output_frames: list[np.ndarray],
    gt_data: LoadedConsistencyGT,
    result: ConsistencyResult,
    is_video: bool,
    output_format: str | None = None,
    gt_remote_files: list[tuple[str, str]] | None = None,
) -> Path | None:
    if not artifact_dir:
        return None

# ...
    records.append(record)
    summary_path.write_text(json.dumps(records, indent=2) + "\n", encoding="utf-8")
    _write_consistency_failure_index(out_dir, records)
    return image_path
```
**EN:** This function drives `save_consistency_failure_artifact` with inputs such as `artifact_dir`, `case_id`, `num_gpus`, `output_frames`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `save_consistency_failure_artifact`，主要处理 `artifact_dir`, `case_id`, `num_gpus`, `output_frames` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Caching strategy / 缓存策略
- Sampling parameter control / 采样参数控制
- Automated verification / 自动化验证
- Command-line interface / 命令行接口

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.utils.common`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.utils.perf_logger`, `sglang.multimodal_gen.test.server.testcase_configs`
- **External / 外部**: `html`, `urllib.parse`, `cv2`, `httpx`, `numpy`, `requests`, `PIL`, `tokenizers`
- **Stdlib / 标准库**: `base64`, `io`, `json`, `math`, `os`, `socket`, `subprocess`, `sys`
