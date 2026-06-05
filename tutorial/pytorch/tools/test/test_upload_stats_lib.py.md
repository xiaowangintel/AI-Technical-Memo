# test_upload_stats_lib.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/test/test_upload_stats_lib.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides test fixtures, sample inputs, and validation helpers for the tooling layer itself.
- **Purpose (CN)**: 为工具层本身提供测试夹具、示例输入和验证辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```python
from __future__ import annotations

import gzip
import inspect
import json
import sys
import unittest
from pathlib import Path
from typing import Any
from unittest import mock
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, gzip, inspect, and 5 more. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、gzip、inspect 等共 8 项。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 12-21
```python

REPO_ROOT = Path(__file__).resolve().parents[2]
sys.path.insert(0, str(REPO_ROOT))

from tools.stats.upload_metrics import add_global_metric, emit_metric, global_metrics
from tools.stats.upload_stats_lib import get_s3_resource, remove_nan_inf


sys.path.remove(str(REPO_ROOT))
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.stats.upload_metrics, tools.stats.upload_stats_lib. Configuration constants such as REPO_ROOT centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.stats.upload_metrics、tools.stats.upload_stats_lib。 REPO_ROOT 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 22-34
```python
# default values
REPO = "some/repo"
BUILD_ENV = "cuda-10.2"
TEST_CONFIG = "test-config"
WORKFLOW = "some-workflow"
JOB = "some-job"
RUN_ID = 56
RUN_NUMBER = 123
RUN_ATTEMPT = 3
PR_NUMBER = 6789
JOB_ID = 234
JOB_NAME = "some-job-name"
```
- **EN**: Configuration constants such as REPO, BUILD_ENV, TEST_CONFIG, and 8 more centralize defaults so later functions share the same policy knobs.
- **CN**: REPO、BUILD_ENV、TEST_CONFIG 等共 11 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 35-46
```python

@mock.patch("boto3.resource")
class TestUploadStats(unittest.TestCase):
    emitted_metric: dict[str, Any] = {"did_not_emit": True}

    def mock_put_item(self, **kwargs: Any) -> None:
        # Utility for mocking putting items into s3.  THis will save the emitted
        # metric so tests can check it
        self.emitted_metric = json.loads(
            gzip.decompress(kwargs["Body"]).decode("utf-8")
        )
```
- **EN**: It introduces classes such as TestUploadStats, which package state and behavior for this tooling task. This chunk defines `mock_put_item`, which implements a focused step inside the tooling tests pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 它引入了 TestUploadStats 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `mock_put_item`，其作用是实现工具测试流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 47-64
```python
    # Before each test, set the env vars to their default values
    def setUp(self) -> None:
        get_s3_resource.cache_clear()
        global_metrics.clear()

        mock.patch.dict(
            "os.environ",
            {
                "CI": "true",
                "BUILD_ENVIRONMENT": BUILD_ENV,
                "TEST_CONFIG": TEST_CONFIG,
                "GITHUB_REPOSITORY": REPO,
                "GITHUB_WORKFLOW": WORKFLOW,
                "GITHUB_JOB": JOB,
                "GITHUB_RUN_ID": str(RUN_ID),
                "GITHUB_RUN_NUMBER": str(RUN_NUMBER),
                "GITHUB_RUN_ATTEMPT": str(RUN_ATTEMPT),
                "JOB_ID": str(JOB_ID),
```
- **EN**: This chunk defines `setUp`, which prepares build-system state and translates configuration into downstream tool invocations.
- **CN**: 这一段定义了 `setUp`，其作用是准备构建系统状态，并把配置转换为后续工具调用。

### Lines 65-75
```python
                "JOB_NAME": str(JOB_NAME),
            },
            clear=True,  # Don't read any preset env vars
        ).start()

    def test_emits_default_and_given_metrics(self, mock_resource: Any) -> None:
        metric = {
            "some_number": 123,
            "float_number": 32.34,
        }
```
- **EN**: This chunk defines `test_emits_default_and_given_metrics`, which generates derived source text, templates, or metadata outputs.
- **CN**: 这一段定义了 `test_emits_default_and_given_metrics`，其作用是生成派生源码文本、模板或元数据输出。

### Lines 76-93
```python
        # Querying for this instead of hard coding it b/c this will change
        # based on whether we run this test directly from python or from
        # pytest
        current_module = inspect.getmodule(inspect.currentframe()).__name__  # type: ignore[union-attr]

        emit_should_include = {
            "metric_name": "metric_name",
            "calling_file": "test_upload_stats_lib.py",
            "calling_module": current_module,
            "calling_function": "test_emits_default_and_given_metrics",
            "repo": REPO,
            "workflow": WORKFLOW,
            "build_environment": BUILD_ENV,
            "job": JOB,
            "test_config": TEST_CONFIG,
            "run_id": RUN_ID,
            "run_number": RUN_NUMBER,
            "run_attempt": RUN_ATTEMPT,
```
- **EN**: This chunk continues `test_emits_default_and_given_metrics` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `test_emits_default_and_given_metrics`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 94-102
```python
            "job_id": JOB_ID,
            "job_name": JOB_NAME,
            "info": metric,
        }

        mock_resource.return_value.Object.return_value.put = self.mock_put_item

        emit_metric("metric_name", metric)
```
- **EN**: This chunk continues `test_emits_default_and_given_metrics` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_emits_default_and_given_metrics`，进一步展开其内部控制流或数据流转。

### Lines 103-114
```python
        self.assertEqual(
            self.emitted_metric,
            {**self.emitted_metric, **emit_should_include},
        )

    def test_when_global_metric_specified_then_it_emits_it(
        self, mock_resource: Any
    ) -> None:
        metric = {
            "some_number": 123,
        }
```
- **EN**: This chunk defines `test_when_global_metric_specified_then_it_emits_it`, which generates derived source text, templates, or metadata outputs.
- **CN**: 这一段定义了 `test_when_global_metric_specified_then_it_emits_it`，其作用是生成派生源码文本、模板或元数据输出。

### Lines 115-124
```python
        global_metric_name = "global_metric"
        global_metric_value = "global_value"

        add_global_metric(global_metric_name, global_metric_value)

        emit_should_include = {
            **metric,
            global_metric_name: global_metric_value,
        }
```
- **EN**: This chunk continues `test_when_global_metric_specified_then_it_emits_it` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_when_global_metric_specified_then_it_emits_it`，进一步展开其内部控制流或数据流转。

### Lines 125-133
```python
        mock_resource.return_value.Object.return_value.put = self.mock_put_item

        emit_metric("metric_name", metric)

        self.assertEqual(
            self.emitted_metric,
            {**self.emitted_metric, "info": emit_should_include},
        )
```
- **EN**: This chunk continues `test_when_global_metric_specified_then_it_emits_it` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_when_global_metric_specified_then_it_emits_it`，进一步展开其内部控制流或数据流转。

### Lines 134-142
```python
    def test_when_local_and_global_metric_specified_then_global_is_overridden(
        self, mock_resource: Any
    ) -> None:
        global_metric_name = "global_metric"
        global_metric_value = "global_value"
        local_override = "local_override"

        add_global_metric(global_metric_name, global_metric_value)
```
- **EN**: This chunk defines `test_when_local_and_global_metric_specified_then_global_is_overridden`, which implements a focused step inside the tooling tests pipeline.
- **CN**: 这一段定义了 `test_when_local_and_global_metric_specified_then_global_is_overridden`，其作用是实现工具测试流水线中的一个关键步骤。

### Lines 143-152
```python
        metric = {
            "some_number": 123,
            global_metric_name: local_override,
        }

        emit_should_include = {
            **metric,
            global_metric_name: local_override,
        }
```
- **EN**: This chunk continues `test_when_local_and_global_metric_specified_then_global_is_overridden` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_when_local_and_global_metric_specified_then_global_is_overridden`，进一步展开其内部控制流或数据流转。

### Lines 153-161
```python
        mock_resource.return_value.Object.return_value.put = self.mock_put_item

        emit_metric("metric_name", metric)

        self.assertEqual(
            self.emitted_metric,
            {**self.emitted_metric, "info": emit_should_include},
        )
```
- **EN**: This chunk continues `test_when_local_and_global_metric_specified_then_global_is_overridden` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_when_local_and_global_metric_specified_then_global_is_overridden`，进一步展开其内部控制流或数据流转。

### Lines 162-173
```python
    def test_when_optional_envvar_set_to_actual_value_then_emit_vars_emits_it(
        self, mock_resource: Any
    ) -> None:
        metric = {
            "some_number": 123,
        }

        emit_should_include = {
            "info": {**metric},
            "pr_number": PR_NUMBER,
        }
```
- **EN**: This chunk defines `test_when_optional_envvar_set_to_actual_value_then_emit_vars_emits_it`, which generates derived source text, templates, or metadata outputs.
- **CN**: 这一段定义了 `test_when_optional_envvar_set_to_actual_value_then_emit_vars_emits_it`，其作用是生成派生源码文本、模板或元数据输出。

### Lines 174-182
```python
        mock.patch.dict(
            "os.environ",
            {
                "PR_NUMBER": str(PR_NUMBER),
            },
        ).start()

        mock_resource.return_value.Object.return_value.put = self.mock_put_item
```
- **EN**: This chunk continues `test_when_optional_envvar_set_to_actual_value_then_emit_vars_emits_it` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_when_optional_envvar_set_to_actual_value_then_emit_vars_emits_it`，进一步展开其内部控制流或数据流转。

### Lines 183-194
```python
        emit_metric("metric_name", metric)

        self.assertEqual(
            self.emitted_metric,
            {**self.emitted_metric, **emit_should_include},
        )

    def test_when_optional_envvar_set_to_a_empty_str_then_emit_vars_ignores_it(
        self, mock_resource: Any
    ) -> None:
        metric = {"some_number": 123}
```
- **EN**: This chunk defines `test_when_optional_envvar_set_to_a_empty_str_then_emit_vars_ignores_it`, which generates derived source text, templates, or metadata outputs.
- **CN**: 这一段定义了 `test_when_optional_envvar_set_to_a_empty_str_then_emit_vars_ignores_it`，其作用是生成派生源码文本、模板或元数据输出。

### Lines 195-205
```python
        emit_should_include: dict[str, Any] = metric.copy()

        # Github Actions defaults some env vars to an empty string
        default_val = ""
        mock.patch.dict(
            "os.environ",
            {
                "PR_NUMBER": default_val,
            },
        ).start()
```
- **EN**: This chunk continues `test_when_optional_envvar_set_to_a_empty_str_then_emit_vars_ignores_it` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_when_optional_envvar_set_to_a_empty_str_then_emit_vars_ignores_it`，进一步展开其内部控制流或数据流转。

### Lines 206-219
```python
        mock_resource.return_value.Object.return_value.put = self.mock_put_item

        emit_metric("metric_name", metric)

        self.assertEqual(
            self.emitted_metric,
            {**self.emitted_metric, "info": emit_should_include},
            f"Metrics should be emitted when an option parameter is set to '{default_val}'",
        )
        self.assertFalse(
            self.emitted_metric.get("pr_number"),
            f"Metrics should not include optional item 'pr_number' when it's envvar is set to '{default_val}'",
        )
```
- **EN**: This chunk continues `test_when_optional_envvar_set_to_a_empty_str_then_emit_vars_ignores_it` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_when_optional_envvar_set_to_a_empty_str_then_emit_vars_ignores_it`，进一步展开其内部控制流或数据流转。

### Lines 220-233
```python
    def test_no_metrics_emitted_if_required_env_var_not_set(
        self, mock_resource: Any
    ) -> None:
        metric = {"some_number": 123}

        mock.patch.dict(
            "os.environ",
            {
                "CI": "true",
                "BUILD_ENVIRONMENT": BUILD_ENV,
            },
            clear=True,
        ).start()
```
- **EN**: This chunk defines `test_no_metrics_emitted_if_required_env_var_not_set`, which generates derived source text, templates, or metadata outputs.
- **CN**: 这一段定义了 `test_no_metrics_emitted_if_required_env_var_not_set`，其作用是生成派生源码文本、模板或元数据输出。

### Lines 234-244
```python
        mock_resource.return_value.Object.return_value.put = self.mock_put_item

        emit_metric("metric_name", metric)

        self.assertTrue(self.emitted_metric["did_not_emit"])

    def test_no_metrics_emitted_if_required_env_var_set_to_empty_string(
        self, mock_resource: Any
    ) -> None:
        metric = {"some_number": 123}
```
- **EN**: This chunk defines `test_no_metrics_emitted_if_required_env_var_set_to_empty_string`, which generates derived source text, templates, or metadata outputs.
- **CN**: 这一段定义了 `test_no_metrics_emitted_if_required_env_var_set_to_empty_string`，其作用是生成派生源码文本、模板或元数据输出。

### Lines 245-253
```python
        mock.patch.dict(
            "os.environ",
            {
                "GITHUB_JOB": "",
            },
        ).start()

        mock_resource.return_value.Object.return_value.put = self.mock_put_item
```
- **EN**: This chunk continues `test_no_metrics_emitted_if_required_env_var_set_to_empty_string` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `test_no_metrics_emitted_if_required_env_var_set_to_empty_string`，进一步展开其内部控制流或数据流转。

### Lines 254-266
```python
        emit_metric("metric_name", metric)

        self.assertTrue(self.emitted_metric["did_not_emit"])

    def test_remove_nan_inf(self, _mocked_resource: Any) -> None:
        checks = [
            (float("inf"), '"inf"', "Infinity"),
            (float("nan"), '"nan"', "NaN"),
            ({1: float("inf")}, '{"1": "inf"}', '{"1": Infinity}'),
            ([float("nan")], '["nan"]', "[NaN]"),
            ({1: [float("nan")]}, '{"1": ["nan"]}', '{"1": [NaN]}'),
        ]
```
- **EN**: This chunk defines `test_remove_nan_inf`, which implements a focused step inside the tooling tests pipeline.
- **CN**: 这一段定义了 `test_remove_nan_inf`，其作用是实现工具测试流水线中的一个关键步骤。

### Lines 267-280
```python
        for input, clean, unclean in checks:
            clean_output = json.dumps(remove_nan_inf(input))
            unclean_output = json.dumps(input)
            self.assertEqual(
                clean_output,
                clean,
                f"Expected {clean} when input is {unclean}, got {clean_output}",
            )
            self.assertEqual(
                unclean_output,
                unclean,
                f"Expected {unclean} when input is {unclean}, got {unclean_output}",
            )
```
- **EN**: This chunk continues `test_remove_nan_inf` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `test_remove_nan_inf`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 281-283
```python

if __name__ == "__main__":
    unittest.main()
```
- **EN**: This chunk continues `test_remove_nan_inf` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `test_remove_nan_inf`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

## Key Concepts / 关键概念

- **Tooling tests**
  - EN: This file belongs to the tooling tests layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于工具测试层，应结合同一子目录中的相邻脚本一起理解。
- **GitHub automation**
  - EN: The file integrates with pull requests, workflow metadata, or repository automation flows.
  - CN: 该文件与 Pull Request、工作流元数据或仓库自动化流程集成。
- **Statistics reporting**
  - EN: The implementation aggregates measurements and turns them into summaries or dashboards.
  - CN: 该实现聚合度量数据，并将其转化为摘要或仪表板。
- **Pytest integration**
  - EN: The file plugs into pytest collection, execution, or reporting behavior.
  - CN: 该文件接入 pytest 的收集、执行或报告行为。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.stats.upload_metrics`, `tools.stats.upload_stats_lib`
- **Python standard library / Python 标准库**: `__future__`, `gzip`, `inspect`, `json`, `sys`, `unittest`, `pathlib`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `REPO`, `BUILD_ENV`, `TEST_CONFIG`, `WORKFLOW`, `JOB`, `RUN_ID`, `RUN_NUMBER`, `RUN_ATTEMPT`, `PR_NUMBER`, `JOB_ID`, `JOB_NAME`
