# test_evs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/vlm/test_evs.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates evs behavior in SGLang's vlm area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 vlm 领域中与 evs 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: module imports and dependencies / 模块导入与依赖
```python
from dataclasses import asdict, dataclass
from types import SimpleNamespace

import pytest

from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import run_doctests
```
**EN:** This block imports the modules needed by the rest of the file, including `dataclasses`, `types`, `pytest`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `dataclasses`, `types`, `pytest`, `sglang.test.ci.ci_register`。

### Lines 9-10: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=11, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=20, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 13-48: test case resolve evs config / 测试用例 resolve evs config
```python
def test_resolve_evs_config():
    from sglang.srt.multimodal.evs import EVS, EVSConfig, EVSProcessor

    @dataclass(frozen=True, kw_only=True)
    class EVSModelConfig:
        video_pruning_rate: float = 0.1
        spatial_merge_size: int = 2

    class EVSModel(EVS):
        @staticmethod
        def create_evs_config(hf_config: EVSModelConfig) -> EVSConfig:
            return EVSConfig(
                video_pruning_rate=hf_config.video_pruning_rate,
                spatial_merge_size=hf_config.spatial_merge_size,
            )

    processor = EVSProcessor(
        hf_config=EVSModelConfig(spatial_merge_size=3),
        config_to_evs_model={EVSModelConfig: EVSModel},
    )
    expected = EVSConfig(video_pruning_rate=0.1, spatial_merge_size=3)
    assert asdict(processor.evs_config) == asdict(expected)

    # No EVS for pruning rate 0.0
    processor = EVSProcessor(
        hf_config=EVSModelConfig(video_pruning_rate=0.0),
        config_to_evs_model={EVSModelConfig: EVSModel},
    )
    assert processor.evs_config is None

    # No EVS for non-EVS config
    processor = EVSProcessor(
        hf_config=SimpleNamespace(),
        config_to_evs_model={EVSModelConfig: EVSModel},
    )
    assert processor.evs_config is None
```
**EN:** This test exercises `test_resolve_evs_config` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_resolve_evs_config`。

### Lines 51-54: test case replace offsets with tokens per frame / 测试用例 replace offsets with tokens per frame
```python
def test_replace_offsets_with_tokens_per_frame():
    from sglang.srt.multimodal.evs.evs_core import replace_offsets_with_tokens_per_frame

    run_doctests(replace_offsets_with_tokens_per_frame)
```
**EN:** This test exercises `test_replace_offsets_with_tokens_per_frame` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_replace_offsets_with_tokens_per_frame`。

### Lines 57-60: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    import sys

    sys.exit(pytest.main([__file__]))
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `test_resolve_evs_config`: This test exercises `test_resolve_evs_config` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_resolve_evs_config`。
- `test_replace_offsets_with_tokens_per_frame`: This test exercises `test_replace_offsets_with_tokens_per_frame` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_replace_offsets_with_tokens_per_frame`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `types`
- **Third-party modules / 第三方模块**: `pytest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 60
