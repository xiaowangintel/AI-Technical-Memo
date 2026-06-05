# test_disagg_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/server/test_disagg_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates disagg server with focused assertions and fixtures. Key symbols include `_kill_tree`, `_wait_for_log`, `_tail_log`. / 该测试模块通过有针对性的断言与夹具，验证 disagg server 的实现。 关键符号包括 `_kill_tree`, `_wait_for_log`, `_tail_log`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-48: Imports and module setup / 导入与模块初始化
```python
"""End-to-end tests for disaggregated diffusion.

Launches encoder / denoiser / decoder role instances plus a DiffusionServer
head, sends a generation request through the HTTP front-end, and verifies
that a non-empty output comes back.

Two configurations are covered:

1. :class:`TestDisaggZImage1Rank` — 1 rank per role (baseline disagg path).
2. :class:`TestDisaggZImage2RankDenoiser` — denoiser with
   ``--denoiser-sp 2`` across 2 GPUs. Exercises the multi-rank receive path
   where only rank 0 owns the RDMA TransferManager and must broadcast
   prompt/image tensors to non-rank-0 ranks before
   ``execute_forward`` — without that broadcast the denoising stage fails
# ...
_LOG_DIR = Path(os.environ.get("SGLANG_TEST_LOG_DIR", "/tmp"))

# Env knob: bump if a cold HF download is needed on a fresh CI runner.
_STARTUP_TIMEOUT_S = float(os.environ.get("SGLANG_DISAGG_STARTUP_TIMEOUT", "600"))
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 56-60: Function `_kill_tree` / 函数 `_kill_tree`
```python
def _kill_tree(pid: int) -> None:
    try:
        os.killpg(os.getpgid(pid), signal.SIGKILL)
    except (ProcessLookupError, PermissionError):
        pass
```
**EN:** This function drives `_kill_tree` with inputs such as `pid`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_kill_tree`，主要处理 `pid` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 63-73: Function `_wait_for_log` / 函数 `_wait_for_log`
```python
def _wait_for_log(path: Path, message: str, timeout: float) -> bool:
    deadline = time.time() + timeout
    while time.time() < deadline:
        if path.exists():
            try:
                if message in path.read_text(errors="ignore"):
                    return True
            except OSError:
                pass
        time.sleep(2)
    return False
```
**EN:** This function drives `_wait_for_log` with inputs such as `path`, `message`, `timeout`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_wait_for_log`，主要处理 `path`, `message`, `timeout` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 76-83: Function `_tail_log` / 函数 `_tail_log`
```python
def _tail_log(path: Path, n: int = 50) -> str:
    if not path.exists():
        return f"<no log at {path}>"
    try:
        lines = path.read_text(errors="ignore").splitlines()
    except OSError as e:
        return f"<log read failed: {e}>"
    return "\n".join(lines[-n:])
```
**EN:** This function drives `_tail_log` with inputs such as `path`, `n`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_tail_log`，主要处理 `path`, `n` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 91-268: Class `DisaggCluster` / 类 `DisaggCluster`
```python
class DisaggCluster:
    """Launch encoder / denoiser / decoder / server as separate processes.

    ``gpu_layout`` is a mapping role → list of physical GPU ids. The length
    of each list determines ``--num-gpus`` for that role, and the first id is
    passed as ``--base-gpu-id``. For a multi-rank role the GPUs must be
    contiguous starting from ``base-gpu-id`` (sglang derives local_rank from
    ``base-gpu-id + rank``).
    """

    def __init__(
        self,
        model: str,
        name: str,
# ...
        # Give OS a moment to release ports before the next test.
        time.sleep(3)
        self._procs.clear()
        self._fhs.clear()
```
**EN:** This class models `DisaggCluster`. Launch encoder / denoiser / decoder / server as separate processes. Important methods include `__init__`, `_alloc_ports`, `__enter__`, `__exit__`.
**CN:** 该类实现 `DisaggCluster`。 文档字符串指出：Launch encoder / denoiser / decoder / server as separate processes. 其中较重要的方法包括 `__init__`, `_alloc_ports`, `__enter__`, `__exit__`。

### Lines 276-296: Function `_generate_image` / 函数 `_generate_image`
```python
def _generate_image(api_port: int, model: str) -> bytes:
    # Use raw requests (openai SDK pulls in a lot and complicates CI deps).
    resp = requests.post(
        f"http://{HOST}:{api_port}/v1/images/generations",
        json={
            "model": model,
            "prompt": "A sunset over mountains",
            "n": 1,
            "size": "1024x1024",
            "response_format": "b64_json",
        },
        timeout=600,
    )
    if resp.status_code != 200:
# ...
        )
    resp.raise_for_status()
    data = resp.json()
    return base64.b64decode(data["data"][0]["b64_json"])
```
**EN:** This function drives `_generate_image` with inputs such as `api_port`, `model`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_generate_image`，主要处理 `api_port`, `model` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 304-307: Function `_require_gpus` / 函数 `_require_gpus`
```python
def _require_gpus(n: int) -> None:
    available = torch.cuda.device_count() if torch.cuda.is_available() else 0
    if available < n:
        raise unittest.SkipTest(f"need {n} GPUs, have {available}")
```
**EN:** This function drives `_require_gpus` with inputs such as `n`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_require_gpus`，主要处理 `n` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 310-345: Class `_DisaggTestBase` / 类 `_DisaggTestBase`
```python
class _DisaggTestBase(CustomTestCase):
    """Shared setup: launch cluster once per class, tear down at the end."""

    model: str = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
    required_gpus: int = 2
    cluster_name: str = ""
    gpu_layout: dict[str, list[int]] = {}
    extra_role_args: dict[str, list[str]] = {}

    cluster: DisaggCluster | None = None

    @classmethod
    def setUpClass(cls) -> None:
        super().setUpClass()
# ...
                print(_tail_log(log_path, n=80), flush=True)
            cls.cluster.stop()
            cls.cluster = None
        super().tearDownClass()
```
**EN:** This class models `_DisaggTestBase` as a specialization of `CustomTestCase`. Shared setup: launch cluster once per class, tear down at the end. Important methods include `setUpClass`, `tearDownClass`.
**CN:** 该类实现 `_DisaggTestBase`，并继承/扩展 `CustomTestCase`。 文档字符串指出：Shared setup: launch cluster once per class, tear down at the end. 其中较重要的方法包括 `setUpClass`, `tearDownClass`。

### Lines 348-363: Class `TestDisaggZImage1Rank` / 类 `TestDisaggZImage1Rank`
```python
class TestDisaggZImage1Rank(_DisaggTestBase):
    """Baseline: 1 rank per role, 2 physical GPUs."""

    cluster_name = "zimage_1rank"
    required_gpus = 2
    gpu_layout = {
        "encoder": [0],
        "denoiser": [1],
        "decoder": [0],
    }

    def test_generates_image(self) -> None:
        assert self.cluster is not None
        img = _generate_image(self.cluster.api_port, self.model)
        # A real PNG is well above 1 KB; catches empty / error responses.
        self.assertGreater(len(img), 1_000, f"image too small: {len(img)} bytes")
```
**EN:** This class models `TestDisaggZImage1Rank` as a specialization of `_DisaggTestBase`. Baseline: 1 rank per role, 2 physical GPUs. Important methods include `test_generates_image`.
**CN:** 该类实现 `TestDisaggZImage1Rank`，并继承/扩展 `_DisaggTestBase`。 文档字符串指出：Baseline: 1 rank per role, 2 physical GPUs. 其中较重要的方法包括 `test_generates_image`。

### Lines 366-389: Class `TestDisaggZImage2RankDenoiser` / 类 `TestDisaggZImage2RankDenoiser`
```python
class TestDisaggZImage2RankDenoiser(_DisaggTestBase):
    """Multi-rank denoiser (``--denoiser-sp 2``) on 2 GPUs.

    Regression guard for the bug where non-rank-0 denoiser ranks entered
    ``execute_forward`` with an empty Req because ``ParallelExecutor``'s
    REPLICATED stage does not broadcast the batch. With the fix, rank 0
    broadcasts both scalar and tensor fields over NCCL before compute.
    """

    cluster_name = "zimage_sp2"
    required_gpus = 2
    gpu_layout = {
        "encoder": [0],
        "denoiser": [0, 1],
# ...
    def test_generates_image_with_sp2_denoiser(self) -> None:
        assert self.cluster is not None
        img = _generate_image(self.cluster.api_port, self.model)
        self.assertGreater(len(img), 1_000, f"image too small: {len(img)} bytes")
```
**EN:** This class models `TestDisaggZImage2RankDenoiser` as a specialization of `_DisaggTestBase`. Multi-rank denoiser (``--denoiser-sp 2``) on 2 GPUs. Important methods include `test_generates_image_with_sp2_denoiser`.
**CN:** 该类实现 `TestDisaggZImage2RankDenoiser`，并继承/扩展 `_DisaggTestBase`。 文档字符串指出：Multi-rank denoiser (``--denoiser-sp 2``) on 2 GPUs. 其中较重要的方法包括 `test_generates_image_with_sp2_denoiser`。

### Lines 397-420: Function `_generate_image_with_traceparent` / 函数 `_generate_image_with_traceparent`
```python
def _generate_image_with_traceparent(
    api_port: int, model: str, trace_id_hex: str, span_id_hex: str
) -> tuple[int, bytes]:
    """Same as :func:`_generate_image` but seeds a known W3C traceparent.

    Returns ``(status_code, image_bytes)``. Kept separate so the tracing test
    can tolerate non-200 responses while still reporting useful diagnostics.
    """
    traceparent = f"00-{trace_id_hex}-{span_id_hex}-01"
    resp = requests.post(
        f"http://{HOST}:{api_port}/v1/images/generations",
        headers={"traceparent": traceparent},
        json={
            "model": model,
# ...
    )
    if resp.status_code != 200:
        return resp.status_code, b""
    return resp.status_code, base64.b64decode(resp.json()["data"][0]["b64_json"])
```
**EN:** This function drives `_generate_image_with_traceparent` with inputs such as `api_port`, `model`, `trace_id_hex`, `span_id_hex`. Same as :func:`_generate_image` but seeds a known W3C traceparent.
**CN:** 这个函数负责 `_generate_image_with_traceparent`，主要处理 `api_port`, `model`, `trace_id_hex`, `span_id_hex` 等输入。 文档字符串说明：Same as :func:`_generate_image` but seeds a known W3C traceparent.

### Lines 423-430: Function `_as_hex` / 函数 `_as_hex`
```python
def _as_hex(v) -> str:
    """OTLP span trace_id/span_id/parent_span_id come back as raw bytes over
    gRPC and as hex strings over HTTP; normalize to lowercase hex."""
    if isinstance(v, (bytes, bytearray)):
        return v.hex()
    if isinstance(v, str):
        return v.lower()
    return ""
```
**EN:** This function drives `_as_hex` with inputs such as `v`. OTLP span trace_id/span_id/parent_span_id come back as raw bytes over
**CN:** 这个函数负责 `_as_hex`，主要处理 `v` 等输入。 文档字符串说明：OTLP span trace_id/span_id/parent_span_id come back as raw bytes over

### Lines 433-554: Class `TestDisaggZImageTracing` / 类 `TestDisaggZImageTracing`
```python
class TestDisaggZImageTracing(_DisaggTestBase):
    """End-to-end verification of OTel trace propagation across disagg roles.

    Spins up the same 1-rank cluster as :class:`TestDisaggZImage1Rank` with
    ``--enable-trace`` wired to an in-process OTLP collector on every role and
    the server head, sends one image-generation request with a controlled
    ``traceparent``, and asserts the server head plus all three role worker
    processes emit per-role ``scheduler_dispatch``/``gpu_forward`` spans under
    the same trace_id. This is the regression guard for trace-context
    propagation over the encoder→denoiser→decoder JSON hops.
    """

    cluster_name = "zimage_trace"
    required_gpus = 2
# ...
            trace_ids,
            {trace_id},
            f"spans split across multiple traces: {trace_ids}",
        )
```
**EN:** This class models `TestDisaggZImageTracing` as a specialization of `_DisaggTestBase`. End-to-end verification of OTel trace propagation across disagg roles. Important methods include `setUpClass`, `tearDownClass`, `test_disagg_spans_share_trace_id`.
**CN:** 该类实现 `TestDisaggZImageTracing`，并继承/扩展 `_DisaggTestBase`。 文档字符串指出：End-to-end verification of OTel trace propagation across disagg roles. 其中较重要的方法包括 `setUpClass`, `tearDownClass`, `test_disagg_spans_share_trace_id`。

### Lines 555-558: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Scheduling and batching / 调度与批处理
- Automated verification / 自动化验证
- Pytest-based assertions / 基于 Pytest 的断言
- Command-line interface / 命令行接口

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.test.test_utils`, `sglang.test.test_utils`, `sglang.test.otel_collector`
- **External / 外部**: `__future__`, `unittest`, `requests`, `torch`
- **Stdlib / 标准库**: `base64`, `os`, `signal`, `subprocess`, `time`, `pathlib`
