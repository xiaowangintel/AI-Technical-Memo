# gen_perf_baselines.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/scripts/gen_perf_baselines.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates gen perf baselines with focused assertions and fixtures. Key symbols include `_all_cases`, `_baseline_path`, `_openai_client`. / 该测试模块通过有针对性的断言与夹具，验证 gen perf baselines 的实现。 关键符号包括 `_all_cases`, `_baseline_path`, `_openai_client`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Imports and module setup / 导入与模块初始化
```python
import argparse
import inspect
import json
import os
import re
import sys
from pathlib import Path

from openai import OpenAI

from sglang.multimodal_gen.test.server.test_server_utils import (
    ServerManager,
    get_generate_fn,
)
# ...
from sglang.multimodal_gen.test.test_utils import (
    get_dynamic_server_port,
    wait_for_req_perf_record,
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 25-39: Function `_all_cases` / 函数 `_all_cases`
```python
def _all_cases() -> list[DiffusionTestCase]:
    import sglang.multimodal_gen.test.server.testcase_configs as cfg

    cases: list[DiffusionTestCase] = []
    for _, v in inspect.getmembers(cfg):
        if isinstance(v, list) and v and isinstance(v[0], DiffusionTestCase):
            cases.extend(v)

    seen: set[str] = set()
    out: list[DiffusionTestCase] = []
    for c in cases:
        if c.id not in seen:
            seen.add(c.id)
            out.append(c)
    return out
```
**EN:** This function drives `_all_cases`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_all_cases`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 42-45: Function `_baseline_path` / 函数 `_baseline_path`
```python
def _baseline_path() -> Path:
    import sglang.multimodal_gen.test.server.testcase_configs as cfg

    return Path(cfg.__file__).with_name("perf_baselines.json")
```
**EN:** This function drives `_baseline_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_baseline_path`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 48-49: Function `_openai_client` / 函数 `_openai_client`
```python
def _openai_client(port: int) -> OpenAI:
    return OpenAI(api_key="sglang-anything", base_url=f"http://localhost:{port}/v1")
```
**EN:** This function drives `_openai_client` with inputs such as `port`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_openai_client`，主要处理 `port` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 52-76: Function `_build_server_extra_args` / 函数 `_build_server_extra_args`
```python
def _build_server_extra_args(case: DiffusionTestCase) -> str:
    server_args = case.server_args
    a = os.environ.get("SGLANG_TEST_SERVE_ARGS", "")
    a += f" --num-gpus {server_args.num_gpus}"
    if server_args.tp_size is not None:
        a += f" --tp-size {server_args.tp_size}"
    if server_args.ulysses_degree is not None:
        a += f" --ulysses-degree {server_args.ulysses_degree}"
    if server_args.dit_layerwise_offload:
        a += " --dit-layerwise-offload true"
    if server_args.dit_offload_prefetch_size:
        a += f" --dit-offload-prefetch-size {server_args.dit_offload_prefetch_size}"
    if server_args.text_encoder_cpu_offload:
        a += " --text-encoder-cpu-offload"
# ...

    for extra_arg in server_args.extras:
        a += f" {extra_arg}"
    return a
```
**EN:** This function drives `_build_server_extra_args` with inputs such as `case`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_build_server_extra_args`，主要处理 `case` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 79-82: Function `_build_env_vars` / 函数 `_build_env_vars`
```python
def _build_env_vars(case: DiffusionTestCase) -> dict[str, str]:
    if case.server_args.enable_cache_dit:
        return {"SGLANG_CACHE_DIT_ENABLED": "true"}
    return {}
```
**EN:** This function drives `_build_env_vars` with inputs such as `case`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_build_env_vars`，主要处理 `case` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 85-99: Function `_torch_cleanup` / 函数 `_torch_cleanup`
```python
def _torch_cleanup() -> None:
    try:
        import gc

        gc.collect()
    except Exception:
        pass
    try:
        import torch

        if torch.get_device_module().is_available():
            torch.get_device_module().synchronize()
            torch.get_device_module().empty_cache()
    except Exception:
        pass
```
**EN:** This function drives `_torch_cleanup`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_torch_cleanup`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 102-151: Function `_run_case` / 函数 `_run_case`
```python
def _run_case(case: DiffusionTestCase) -> dict:
    default_port = get_dynamic_server_port()
    port = int(os.environ.get("SGLANG_TEST_SERVER_PORT", default_port))
    mgr = ServerManager(
        model=case.server_args.model_path,
        port=port,
        wait_deadline=float(os.environ.get("SGLANG_TEST_WAIT_SECS", "1200")),
        extra_args=_build_server_extra_args(case),
        env_vars=_build_env_vars(case),
    )
    ctx = mgr.start()
    try:
        sp = case.sampling_params
        output_size = os.environ.get("SGLANG_TEST_OUTPUT_SIZE", sp.output_size)
# ...
            "expected_median_denoise_ms": round(perf.median_denoise_ms, 2),
        }
    finally:
        ctx.cleanup()
```
**EN:** This function drives `_run_case` with inputs such as `case`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_run_case`，主要处理 `case` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 154-208: Function `main` / 函数 `main`
```python
def main() -> int:
    ap = argparse.ArgumentParser()
    ap.add_argument("--baseline", default="")
    ap.add_argument("--out", default="")
    ap.add_argument("--match", default="")
    ap.add_argument("--case", action="append", default=[])
    ap.add_argument("--all-from-baseline", action="store_true")
    ap.add_argument("--timeout", type=float, default=300.0)
    args = ap.parse_args()

    os.environ.setdefault("SGLANG_GEN_BASELINE", "1")
    os.environ["SGLANG_PERF_TIMEOUT"] = str(args.timeout)

    baseline_path = Path(args.baseline) if args.baseline else _baseline_path()
# ...
        _torch_cleanup()

    out_path.write_text(json.dumps(data, indent=4) + "\n", encoding="utf-8")
    return 0
```
**EN:** This function drives `main`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `main`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 209-212: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    sys.exit(main())
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Caching strategy / 缓存策略
- Sampling parameter control / 采样参数控制
- Automated verification / 自动化验证
- Command-line interface / 命令行接口

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.test.server.test_server_utils`, `sglang.multimodal_gen.test.server.testcase_configs`, `sglang.multimodal_gen.test.test_utils`
- **External / 外部**: `openai`, `torch`
- **Stdlib / 标准库**: `argparse`, `inspect`, `json`, `os`, `re`, `sys`, `pathlib`, `gc`
