# test_regular_perf.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/benchmarks/test_regular_perf.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises regular perf behavior in the benchmark suite. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 端到端基准测试 中与 regular perf 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Module docstring
```python
"""Regular router performance benchmark test."""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 3-3: Imports and dependencies
```python
import pytest
```
**EN:** This block imports `pytest`, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 4-29: Test class `TestRegularPerf`
```python


@pytest.mark.e2e
@pytest.mark.workers(count=4)
@pytest.mark.gateway(policy="cache_aware")
@pytest.mark.parametrize("setup_backend", ["http", "grpc"], indirect=True)
class TestRegularPerf:
    """Performance benchmark for regular (non-PD) router."""

    def test_regular_perf(self, setup_backend, genai_bench_runner):
        """Run genai-bench against regular router and validate metrics."""
        backend, model_path, client, gateway = setup_backend
        genai_bench_runner(
            router_url=gateway.base_url,
            model_path=model_path,
            experiment_folder=f"benchmark_cache_aware_regular_{backend}",
            thresholds={
                "ttft_mean_max": 6,
                "e2e_latency_mean_max": 14,
                "input_throughput_mean_min": 800,
                "output_throughput_mean_min": 12,
                # gpu_util_p50_min intentionally omitted: see test_pd_perf.py.
                # On 4-gpu-h100 the median sample lands at 0% for the bursty
                # grpc workload even when mean is healthy (~22%).
            },
        )
```
**EN:** This test class groups related scenarios around `TestRegularPerf` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestRegularPerf` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Performance measurement and comparison / 性能测量与对比
- Primary classes: `TestRegularPerf` / 主要类：`TestRegularPerf`

## Dependencies / 依赖关系
- **Standard library / 标准库**: None explicitly imported / 未显式导入
- **Third-party / 第三方**: `pytest`
- **Internal / 内部模块**: None explicitly imported / 未显式导入
