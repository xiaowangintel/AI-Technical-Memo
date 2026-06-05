# test_disagg_trace.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/unit/test_disagg_trace.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates disagg trace with focused assertions and fixtures. Key symbols include `_enable_minimal_otel`, `_traceparent_from`, `_roundtrip_scalar_fields`. / 该测试模块通过有针对性的断言与夹具，验证 disagg trace 的实现。 关键符号包括 `_enable_minimal_otel`, `_traceparent_from`, `_roundtrip_scalar_fields`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-44: Imports and module setup / 导入与模块初始化
```python
"""Unit tests for OTel trace-context propagation across the diffusion disagg
JSON hop (encoder -> denoiser, denoiser -> decoder).

These exercise the serialization contract only (no GPUs, no server, no OTLP
collector required):

 - ``extract_transfer_fields`` emits a JSON-safe ``_trace_state`` (W3C carrier)
   when tracing is enabled, and omits it when tracing is disabled. It never
   serializes the live ``TraceReqContext`` object itself.
 - The ``_trace_state`` payload round-trips through ``codec.pack_tensors``
   (the same ``json.dumps`` path the RDMA metadata frame uses).
 - ``TraceReqContext.__setstate__`` reconstructs a live, ``is_copy=True``
   context whose ``root_span_context`` is an OTel ``Context`` object.
 - ``_build_disagg_req`` pops ``_trace_state`` and installs a rebuilt
# ...
    _OTEL_AVAILABLE = False


_OTEL_BOOTSTRAPPED = False
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 47-56: Function `_enable_minimal_otel` / 函数 `_enable_minimal_otel`
```python
def _enable_minimal_otel() -> None:
    """Bootstrap just enough OTel state for TraceReqContext to produce real
    spans. Idempotent — the TracerProvider can only be set once per process."""
    global _OTEL_BOOTSTRAPPED
    if not _OTEL_BOOTSTRAPPED:
        otel_trace.set_tracer_provider(TracerProvider())
        _OTEL_BOOTSTRAPPED = True
    srt_trace.opentelemetry_initialized = True
    srt_trace.tracer = otel_trace.get_tracer("test-diffusion-disagg")
    srt_trace.trace_set_thread_info("TestThread")
```
**EN:** This function drives `_enable_minimal_otel`. Bootstrap just enough OTel state for TraceReqContext to produce real
**CN:** 这个函数负责 `_enable_minimal_otel`。 文档字符串说明：Bootstrap just enough OTel state for TraceReqContext to produce real

### Lines 59-68: Function `_traceparent_from` / 函数 `_traceparent_from`
```python
def _traceparent_from(ctx) -> str | None:
    """Re-inject a W3C carrier from an OTel Context and return the traceparent.

    Used to assert that a carrier round-trip preserves trace_id/span_id, which
    is the actual correctness property (OTel's Context is a dict subclass, so
    ``isinstance(ctx, dict)`` isn't useful).
    """
    carrier: dict = {}
    otel_propagate.inject(carrier, ctx)
    return carrier.get("traceparent")
```
**EN:** This function drives `_traceparent_from` with inputs such as `ctx`. Re-inject a W3C carrier from an OTel Context and return the traceparent.
**CN:** 这个函数负责 `_traceparent_from`，主要处理 `ctx` 等输入。 文档字符串说明：Re-inject a W3C carrier from an OTel Context and return the traceparent.

### Lines 71-75: Function `_roundtrip_scalar_fields` / 函数 `_roundtrip_scalar_fields`
```python
def _roundtrip_scalar_fields(scalar_fields: dict) -> dict:
    """Run the actual RDMA metadata codec path: pack -> json bytes -> decode."""
    metadata_bytes, _ = pack_tensors({}, scalar_fields)
    decoded = json.loads(metadata_bytes.decode("utf-8"))
    return decoded["scalar_fields"]
```
**EN:** This function drives `_roundtrip_scalar_fields` with inputs such as `scalar_fields`. Run the actual RDMA metadata codec path: pack -> json bytes -> decode.
**CN:** 这个函数负责 `_roundtrip_scalar_fields`，主要处理 `scalar_fields` 等输入。 文档字符串说明：Run the actual RDMA metadata codec path: pack -> json bytes -> decode.

### Lines 78-201: Class `TestDisaggTracePropagation` / 类 `TestDisaggTracePropagation`
```python
class TestDisaggTracePropagation(unittest.TestCase):
    def test_transfer_keeps_seed_needed_to_rebuild_generator(self):
        req = Req(request_id="test-seed", prompt="x")
        req.generator = torch.Generator(device="cpu").manual_seed(req.seed)

        _, scalar_fields = extract_transfer_fields(req)

        self.assertEqual(scalar_fields["seed"], 42)

        rebuilt = SchedulerDisaggMixin._build_disagg_req(None, dict(scalar_fields), {})
        self.assertIsInstance(rebuilt.generator, torch.Generator)
        self.assertEqual(rebuilt.seed, 42)

        expected = torch.rand(
# ...
        self.assertNotIn("_trace_state", scalar_fields)

        rebuilt = SchedulerDisaggMixin._build_disagg_req(None, dict(scalar_fields), {})
        self.assertIsInstance(rebuilt.trace_ctx, TraceNullContext)
```
**EN:** This class models `TestDisaggTracePropagation` as a specialization of `unittest.TestCase`. Important methods include `test_transfer_keeps_seed_needed_to_rebuild_generator`, `test_build_disagg_req_rebuilds_generator_list`, `test_tracing_disabled_omits_trace_state`, `test_tracing_enabled_state_roundtrip`.
**CN:** 该类实现 `TestDisaggTracePropagation`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `test_transfer_keeps_seed_needed_to_rebuild_generator`, `test_build_disagg_req_rebuilds_generator_list`, `test_tracing_disabled_omits_trace_state`, `test_tracing_enabled_state_roundtrip`。

### Lines 202-205: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Request disaggregation / 请求解耦
- Automated verification / 自动化验证

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.disaggregation.scheduler_mixin`, `sglang.multimodal_gen.runtime.disaggregation.transport.codec`, `sglang.multimodal_gen.runtime.pipelines_core`, `sglang.srt.observability`, `sglang.srt.observability.trace`
- **External / 外部**: `__future__`, `unittest`, `torch`, `opentelemetry`, `opentelemetry.sdk.trace`
- **Stdlib / 标准库**: `json`
