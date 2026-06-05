# run_suite.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/run_suite.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates run suite with focused assertions and fixtures. Key symbols include `_discover_unit_tests`, `PartitionAssignment`, `get_case_est_time`. / 该测试模块通过有针对性的断言与夹具，验证 run suite 的实现。 关键符号包括 `_discover_unit_tests`, `PartitionAssignment`, `get_case_est_time`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-47: Imports and module setup / 导入与模块初始化
```python
"""
Test runner for multimodal_gen that manages test suites and parallel execution.

For diffusion 1-gpu/2-gpu suites, cases are partitioned by estimated runtime
using LPT so each CI shard has a similar total runtime.
"""

import argparse
import copy
import json
import os
import random
import subprocess
import sys
# ...
_UPDATE_WEIGHTS_MODEL_PAIR_IDS = (
    "FLUX.2-klein-base-4B",
    "Qwen-Image",
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 50-56: Function `_discover_unit_tests` / 函数 `_discover_unit_tests`
```python
def _discover_unit_tests() -> list[str]:
    unit_dir = Path(__file__).resolve().parent / "unit"
    if not unit_dir.is_dir():
        return []
    return sorted(
        f"../unit/{f.name}" for f in unit_dir.glob("test_*.py") if f.is_file()
    )
```
**EN:** This function drives `_discover_unit_tests`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_discover_unit_tests`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 57-132: Top-level configuration / 顶层配置
```python


FILE_SUITES = {
    "unit": _discover_unit_tests(),
    "component-accuracy": [
        "test_component_accuracy_1_gpu.py",
        "test_component_accuracy_2_gpu.py",
    ],
    "component-accuracy-1-gpu": [
        "test_component_accuracy_1_gpu.py",
    ],
    "component-accuracy-2-gpu": [
        "test_component_accuracy_2_gpu.py",
    ],
# ...
COMPONENT_ACCURACY_FILE_NUM_GPUS = {
    "test_component_accuracy_1_gpu.py": 1,
    "test_component_accuracy_2_gpu.py": 2,
}
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 135-140: Class `PartitionAssignment` / 类 `PartitionAssignment`
```python
@dataclass(frozen=True)
class PartitionAssignment:
    case_ids: list[str]
    standalone_files: list[str]
    estimated_time: float | None = None
    missing_standalone_estimates: list[str] | None = None
```
**EN:** This class models `PartitionAssignment`.
**CN:** 该类实现 `PartitionAssignment`。

### Lines 143-149: Function `get_case_est_time` / 函数 `get_case_est_time`
```python
def get_case_est_time(case_id: str) -> float:
    scenario = BASELINE_CONFIG.scenarios.get(case_id)
    if scenario is None:
        return DEFAULT_EST_TIME_SECONDS
    if scenario.estimated_full_test_time_s is not None:
        return scenario.estimated_full_test_time_s
    return scenario.expected_e2e_ms / 1000.0 + STARTUP_OVERHEAD_SECONDS
```
**EN:** This function drives `get_case_est_time` with inputs such as `case_id`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_case_est_time`，主要处理 `case_id` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 152-158: Function `get_standalone_file_est_time` / 函数 `get_standalone_file_est_time`
```python
def get_standalone_file_est_time(
    suite: str, standalone_file: str
) -> tuple[float, bool]:
    suite_est_times = STANDALONE_FILE_EST_TIMES.get(suite, {})
    if standalone_file not in suite_est_times:
        return DEFAULT_STANDALONE_EST_TIME_SECONDS, True
    return suite_est_times[standalone_file], False
```
**EN:** This function drives `get_standalone_file_est_time` with inputs such as `suite`, `standalone_file`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_standalone_file_est_time`，主要处理 `suite`, `standalone_file` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 161-162: Function `get_all_standalone_file_est_times` / 函数 `get_all_standalone_file_est_times`
```python
def get_all_standalone_file_est_times() -> dict[str, dict[str, float]]:
    return copy.deepcopy(STANDALONE_FILE_EST_TIMES)
```
**EN:** This function drives `get_all_standalone_file_est_times`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_all_standalone_file_est_times`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 165-176: Function `validate_standalone_file_est_times` / 函数 `validate_standalone_file_est_times`
```python
def validate_standalone_file_est_times() -> dict[str, list[str]]:
    missing_by_suite: dict[str, list[str]] = {}
    for suite, standalone_files in STANDALONE_FILES.items():
        suite_est_times = STANDALONE_FILE_EST_TIMES.get(suite, {})
        missing = [
            standalone_file
            for standalone_file in standalone_files
            if standalone_file not in suite_est_times
        ]
        if missing:
            missing_by_suite[suite] = missing
    return missing_by_suite
```
**EN:** This function drives `validate_standalone_file_est_times`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `validate_standalone_file_est_times`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 179-193: Function `auto_partition` / 函数 `auto_partition`
```python
def auto_partition(
    cases: list[DiffusionTestCase], rank: int, size: int
) -> list[DiffusionTestCase]:
    if not cases or size <= 0:
        return []

    case_by_id = {case.id: case for case in cases}
    items = [
        PartitionItem(kind="case", item_id=case.id, est_time=get_case_est_time(case.id))
        for case in cases
    ]
    partitions = partition_items_by_lpt(items, size)
    if rank >= len(partitions):
        return []
    return [case_by_id[item.item_id] for item in partitions[rank]]
```
**EN:** This function drives `auto_partition` with inputs such as `cases`, `rank`, `size`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `auto_partition`，主要处理 `cases`, `rank`, `size` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 196-199: Function `get_suite_files_rel` / 函数 `get_suite_files_rel`
```python
def get_suite_files_rel(suite: str, parametrized_only: bool = False) -> list[str]:
    if parametrized_only and suite in PARAMETRIZED_CASE_GROUPS:
        return [filename for filename, _ in PARAMETRIZED_CASE_GROUPS[suite]]
    return SUITES[suite]
```
**EN:** This function drives `get_suite_files_rel` with inputs such as `suite`, `parametrized_only`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_suite_files_rel`，主要处理 `suite`, `parametrized_only` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 202-203: Function `_normalize_standalone_key` / 函数 `_normalize_standalone_key`
```python
def _normalize_standalone_key(standalone_file: str) -> str:
    return f"standalone:{standalone_file}"
```
**EN:** This function drives `_normalize_standalone_key` with inputs such as `standalone_file`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_normalize_standalone_key`，主要处理 `standalone_file` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 206-245: Function `parse_partition_plan` / 函数 `parse_partition_plan`
```python
def parse_partition_plan(
    suite: str,
    partition_id: int,
    total_partitions: int,
    plan_json: str,
) -> PartitionAssignment:
    plan = json.loads(plan_json)
    if plan.get("suite") != suite:
        raise ValueError(
            f"Partition plan suite mismatch: expected {suite!r}, "
            f"got {plan.get('suite')!r}"
        )

    partition_count = plan.get("partition_count")
# ...
        missing_standalone_estimates=list(
            selected_partition.get("missing_standalone_estimates", [])
        ),
    )
```
**EN:** This function drives `parse_partition_plan` with inputs such as `suite`, `partition_id`, `total_partitions`, `plan_json`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `parse_partition_plan`，主要处理 `suite`, `partition_id`, `total_partitions`, `plan_json` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 248-257: Function `_merge_execution_results` / 函数 `_merge_execution_results`
```python
def _merge_execution_results(
    executed_cases: list[str],
    case_results: dict[str, str],
    new_executed_cases: list[str],
    new_case_results: dict[str, str],
) -> None:
    executed_cases.extend(
        case_id for case_id in new_executed_cases if case_id not in executed_cases
    )
    case_results.update(new_case_results)
```
**EN:** This function drives `_merge_execution_results` with inputs such as `executed_cases`, `case_results`, `new_executed_cases`, `new_case_results`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_merge_execution_results`，主要处理 `executed_cases`, `case_results`, `new_executed_cases`, `new_case_results` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 260-267: Function `_format_standalone_estimate_snippet` / 函数 `_format_standalone_estimate_snippet`
```python
def _format_standalone_estimate_snippet(
    suite: str, standalone_file: str, measured_full_test_time_s: float
) -> str:
    return (
        f'"{suite}": {{\n'
        f'    "{standalone_file}": {measured_full_test_time_s:.1f},\n'
        f"}}"
    )
```
**EN:** This function drives `_format_standalone_estimate_snippet` with inputs such as `suite`, `standalone_file`, `measured_full_test_time_s`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_format_standalone_estimate_snippet`，主要处理 `suite`, `standalone_file`, `measured_full_test_time_s` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 270-286: Function `_print_missing_standalone_estimate_message` / 函数 `_print_missing_standalone_estimate_message`
```python
def _print_missing_standalone_estimate_message(
    suite: str,
    standalone_file: str,
    measured_full_test_time_s: float,
) -> None:
    snippet = _format_standalone_estimate_snippet(
        suite, standalone_file, measured_full_test_time_s
    )
    logger.error(
        f'\n{"=" * 60}\n'
        f'Add standalone estimate for suite "{suite}" and file "{standalone_file}":\n\n'
        f"File: python/sglang/multimodal_gen/test/run_suite.py\n\n"
        f"Current partition used fallback estimate: "
        f"{DEFAULT_STANDALONE_EST_TIME_SECONDS:.1f}s\n\n"
        f"{snippet}\n"
        f'{"=" * 60}\n'
    )
```
**EN:** This function drives `_print_missing_standalone_estimate_message` with inputs such as `suite`, `standalone_file`, `measured_full_test_time_s`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_print_missing_standalone_estimate_message`，主要处理 `suite`, `standalone_file`, `measured_full_test_time_s` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 289-331: Function `_run_standalone_file` / 函数 `_run_standalone_file`
```python
def _run_standalone_file(
    suite: str,
    standalone_rel: str,
    target_dir: Path,
    extra_filter: str | None = None,
) -> tuple[int, list[str], dict[str, str], dict]:
    if standalone_rel == _UPDATE_WEIGHTS_FROM_DISK_TEST_FILE:
        _maybe_pin_update_weights_model_pair([standalone_rel])

    est_time, used_fallback_estimate = get_standalone_file_est_time(
        suite, standalone_rel
    )
    standalone_file = _resolve_suite_files(target_dir, [standalone_rel], strict=True)[0]
    junit_xml_path = str(
# ...
        [standalone_key],
        {standalone_key: "pass" if exit_code == 0 else "fail"},
        measurement,
    )
```
**EN:** This function drives `_run_standalone_file` with inputs such as `suite`, `standalone_rel`, `target_dir`, `extra_filter`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_run_standalone_file`，主要处理 `suite`, `standalone_rel`, `target_dir`, `extra_filter` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 334-381: Function `parse_args` / 函数 `parse_args`
```python
def parse_args():
    suite_choices = sorted(set(FILE_SUITES) | set(PARAMETRIZED_CASE_GROUPS))
    parser = argparse.ArgumentParser(description="Run multimodal_gen test suite")
    parser.add_argument(
        "--suite",
        type=str,
        required=True,
        choices=suite_choices,
        help="The test suite to run.",
    )
    parser.add_argument(
        "--partition-id",
        type=int,
        default=0,
# ...
        default=None,
        help="Full partition plan JSON for the current suite.",
    )
    return parser.parse_args()
```
**EN:** This function drives `parse_args`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `parse_args`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 384-421: Function `collect_test_items` / 函数 `collect_test_items`
```python
def collect_test_items(files: list[str], filter_expr: str | None = None) -> list[str]:
    """Collect test node IDs from the given files using pytest --collect-only."""
    cmd = [sys.executable, "-m", "pytest", "--collect-only", "-q"]
    if filter_expr:
        cmd.extend(["-k", filter_expr])
    cmd.extend(files)

    filter_note = f" with filter: {filter_expr}" if filter_expr else ""
    print(f"Collecting tests from {len(files)} file(s){filter_note}")
    result = subprocess.run(cmd, capture_output=True, text=True)

    if result.returncode not in (0, 5):
        error_msg = (
            f"pytest --collect-only failed with exit code {result.returncode}\n"
# ...
                test_items.append(test_id)

    print(f"Collected {len(test_items)} test items")
    return test_items
```
**EN:** This function drives `collect_test_items` with inputs such as `files`, `filter_expr`. Collect test node IDs from the given files using pytest --collect-only.
**CN:** 这个函数负责 `collect_test_items`，主要处理 `files`, `filter_expr` 等输入。 文档字符串说明：Collect test node IDs from the given files using pytest --collect-only.

### Lines 424-441: Function `parse_junit_xml_for_executed_cases` / 函数 `parse_junit_xml_for_executed_cases`
```python
def parse_junit_xml_for_executed_cases(xml_path: str) -> list[str]:
    if not Path(xml_path).exists():
        return []

    executed_cases = []
    tree = ET.parse(xml_path)
    root = tree.getroot()

    for testcase in root.iter("testcase"):
        if testcase.find("skipped") is not None:
            continue

        name = testcase.get("name", "")
        if "[" in name and "]" in name:
            case_id = name[name.index("[") + 1 : name.index("]")]
            executed_cases.append(case_id)

    return executed_cases
```
**EN:** This function drives `parse_junit_xml_for_executed_cases` with inputs such as `xml_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `parse_junit_xml_for_executed_cases`，主要处理 `xml_path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 444-468: Function `parse_junit_xml_for_case_results` / 函数 `parse_junit_xml_for_case_results`
```python
def parse_junit_xml_for_case_results(xml_path: str) -> dict[str, str]:
    if not Path(xml_path).exists():
        return {}

    case_results = {}
    tree = ET.parse(xml_path)
    root = tree.getroot()

    for testcase in root.iter("testcase"):
        if testcase.find("skipped") is not None:
            continue

        name = testcase.get("name", "")
        if "[" not in name or "]" not in name:
# ...
        else:
            case_results[case_id] = "pass"

    return case_results
```
**EN:** This function drives `parse_junit_xml_for_case_results` with inputs such as `xml_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `parse_junit_xml_for_case_results`，主要处理 `xml_path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 471-500: Function `write_execution_report` / 函数 `write_execution_report`
```python
def write_execution_report(
    suite: str,
    partition_id: int,
    total_partitions: int,
    executed_cases: list[str],
    is_standalone: bool = False,
    standalone_file: str | None = None,
    case_results: dict[str, str] | None = None,
    missing_standalone_estimates: list[str] | None = None,
    standalone_measurements: list[dict] | None = None,
) -> str:
    report = {
        "suite": suite,
        "partition_id": partition_id,
# ...
        json.dump(report, f, indent=2)

    logger.info("Execution report written to: %s", report_path)
    return str(report_path)
```
**EN:** This function drives `write_execution_report` with inputs such as `suite`, `partition_id`, `total_partitions`, `executed_cases`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `write_execution_report`，主要处理 `suite`, `partition_id`, `total_partitions`, `executed_cases` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 503-521: Function `_run_pytest_attempt` / 函数 `_run_pytest_attempt`
```python
def _run_pytest_attempt(cmd: list[str]) -> tuple[int, str]:
    process = subprocess.Popen(
        cmd,
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,
        bufsize=0,
    )

    output_bytes = bytearray()
    while True:
        chunk = process.stdout.read(4096)
        if not chunk:
            break
        sys.stdout.buffer.write(chunk)
        sys.stdout.buffer.flush()
        output_bytes.extend(chunk)

    process.wait()
    return process.returncode, output_bytes.decode("utf-8", errors="replace")
```
**EN:** This function drives `_run_pytest_attempt` with inputs such as `cmd`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_run_pytest_attempt`，主要处理 `cmd` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 524-529: Function `_extract_collection_line` / 函数 `_extract_collection_line`
```python
def _extract_collection_line(full_output: str) -> str | None:
    for line in full_output.splitlines():
        stripped = line.strip()
        if stripped.startswith("collected "):
            return stripped
    return None
```
**EN:** This function drives `_extract_collection_line` with inputs such as `full_output`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_extract_collection_line`，主要处理 `full_output` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 532-547: Function `_extract_short_test_summary` / 函数 `_extract_short_test_summary`
```python
def _extract_short_test_summary(full_output: str) -> list[str]:
    summary_lines = []
    in_summary = False
    for line in full_output.splitlines():
        stripped = line.strip()
        if "short test summary info" in stripped:
            in_summary = True
            continue
        if not in_summary:
            continue
        if stripped.startswith("="):
            break
        if not stripped or stripped.startswith("!"):
            continue
        summary_lines.append(stripped)
    return summary_lines
```
**EN:** This function drives `_extract_short_test_summary` with inputs such as `full_output`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_extract_short_test_summary`，主要处理 `full_output` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 550-556: Function `_extract_failure_tail` / 函数 `_extract_failure_tail`
```python
def _extract_failure_tail(full_output: str, max_lines: int = 20) -> list[str]:
    summary_lines = _extract_short_test_summary(full_output)
    if summary_lines:
        return summary_lines

    lines = [line.rstrip() for line in full_output.splitlines() if line.strip()]
    return lines[-max_lines:]
```
**EN:** This function drives `_extract_failure_tail` with inputs such as `full_output`, `max_lines`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_extract_failure_tail`，主要处理 `full_output`, `max_lines` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 559-571: Function `_summary_has_retryable_failure` / 函数 `_summary_has_retryable_failure`
```python
def _summary_has_retryable_failure(summary_lines: list[str]) -> bool:
    for line in summary_lines:
        lowered = line.lower()
        if (
            "[performance]" in line
            or "SafetensorError" in line
            or "FileNotFoundError" in line
            or "TimeoutError" in line
            or "out of memory" in lowered
            or "oom killer" in lowered
        ):
            return True
    return False
```
**EN:** This function drives `_summary_has_retryable_failure` with inputs such as `summary_lines`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_summary_has_retryable_failure`，主要处理 `summary_lines` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 574-584: Function `_is_consistency_failure` / 函数 `_is_consistency_failure`
```python
def _is_consistency_failure(full_output: str) -> bool:
    summary_lines = _extract_short_test_summary(full_output)
    for line in summary_lines:
        if "Consistency check failed for" in line or "GT not found for" in line:
            return True

    return (
        "Consistency check failed for " in full_output
        or "GT not found for " in full_output
        or "--- MISSING GROUND TRUTH DETECTED ---" in full_output
    )
```
**EN:** This function drives `_is_consistency_failure` with inputs such as `full_output`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_is_consistency_failure`，主要处理 `full_output` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 587-613: Function `_is_retryable_failure` / 函数 `_is_retryable_failure`
```python
def _is_retryable_failure(full_output: str) -> bool:
    if _is_consistency_failure(full_output):
        return False

    summary_lines = _extract_short_test_summary(full_output)
    is_perf_assertion = (
        "multimodal_gen/test/server/test_server_utils.py" in full_output
        and "AssertionError" in full_output
    )
    is_aggregated_retryable_failure = _summary_has_retryable_failure(summary_lines)

    is_flaky_ci_assertion = (
        "SafetensorError" in full_output
        or "FileNotFoundError" in full_output
# ...
        or is_aggregated_retryable_failure
        or is_flaky_ci_assertion
        or is_oom_error
    )
```
**EN:** This function drives `_is_retryable_failure` with inputs such as `full_output`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_is_retryable_failure`，主要处理 `full_output` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 616-657: Function `_print_attempt_tail_summary` / 函数 `_print_attempt_tail_summary`
```python
def _print_attempt_tail_summary(
    attempt_reports: list[dict], assigned_count: int
) -> None:
    if len(attempt_reports) == 1 and attempt_reports[0]["returncode"] in (0, 5):
        return

    rows = []
    for report in attempt_reports:
        if report["returncode"] in (0, 5):
            result = "success"
        elif report["retryable"]:
            result = "retryable failure"
        else:
            result = "failure"
# ...
        for line in report["failure_tail"]:
            print(f"  {line}", flush=True)

    print("=" * 84, flush=True)
```
**EN:** This function drives `_print_attempt_tail_summary` with inputs such as `attempt_reports`, `assigned_count`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_print_attempt_tail_summary`，主要处理 `attempt_reports`, `assigned_count` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 660-756: Function `run_pytest` / 函数 `run_pytest`
```python
def run_pytest(
    files: list[str],
    filter_expr: str | None = None,
    junit_xml_path: str | None = None,
) -> tuple[int, list[str], dict[str, str]]:
    if not files:
        print("No files to run.")
        return (0, [], {})

    all_executed_cases: set[str] = set()
    all_case_results: dict[str, str] = {}

    base_cmd = [
        sys.executable,
# ...
        attempt_reports[-1]["returncode"],
        list(all_executed_cases),
        all_case_results,
    )
```
**EN:** This function drives `run_pytest` with inputs such as `files`, `filter_expr`, `junit_xml_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `run_pytest`，主要处理 `files`, `filter_expr`, `junit_xml_path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 759-764: Function `partition_items_by_index` / 函数 `partition_items_by_index`
```python
def partition_items_by_index(
    items: list[str], partition_id: int, total_partitions: int
) -> list[str]:
    return [
        item for i, item in enumerate(items) if i % total_partitions == partition_id
    ]
```
**EN:** This function drives `partition_items_by_index` with inputs such as `items`, `partition_id`, `total_partitions`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `partition_items_by_index`，主要处理 `items`, `partition_id`, `total_partitions` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 767-768: Function `partition_test_files` / 函数 `partition_test_files`
```python
def partition_test_files(files, partition_id, total_partitions):
    return partition_items_by_index(files, partition_id, total_partitions)
```
**EN:** This function drives `partition_test_files` with inputs such as `files`, `partition_id`, `total_partitions`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `partition_test_files`，主要处理 `files`, `partition_id`, `total_partitions` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 771-806: Function `run_component_accuracy_files` / 函数 `run_component_accuracy_files`
```python
def run_component_accuracy_files(files, filter_expr=None, continue_on_error=False):
    exit_code = 0
    for file_path in files:
        file_name = Path(file_path).name
        num_gpus = COMPONENT_ACCURACY_FILE_NUM_GPUS.get(file_name, 1)
        if num_gpus > 1:
            cmd = [
                sys.executable,
                "-m",
                "torch.distributed.run",
                f"--nproc_per_node={num_gpus}",
                "-m",
                "pytest",
                "-s",
# ...
            exit_code = file_exit_code
        if file_exit_code != 0 and not continue_on_error:
            return file_exit_code
    return exit_code
```
**EN:** This function drives `run_component_accuracy_files` with inputs such as `files`, `filter_expr`, `continue_on_error`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `run_component_accuracy_files`，主要处理 `files`, `filter_expr`, `continue_on_error` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 809-810: Function `_is_in_ci` / 函数 `_is_in_ci`
```python
def _is_in_ci() -> bool:
    return os.environ.get("SGLANG_IS_IN_CI", "").lower() in ("1", "true", "yes", "on")
```
**EN:** This function drives `_is_in_ci`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_is_in_ci`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 813-827: Function `_maybe_pin_update_weights_model_pair` / 函数 `_maybe_pin_update_weights_model_pair`
```python
def _maybe_pin_update_weights_model_pair(suite_files_rel: list[str]) -> None:
    if not _is_in_ci():
        return
    if _UPDATE_WEIGHTS_FROM_DISK_TEST_FILE not in suite_files_rel:
        return
    if os.environ.get(_UPDATE_WEIGHTS_MODEL_PAIR_ENV):
        print(
            f"Using preset {_UPDATE_WEIGHTS_MODEL_PAIR_ENV}="
            f"{os.environ[_UPDATE_WEIGHTS_MODEL_PAIR_ENV]}"
        )
        return

    selected_pair = random.choice(_UPDATE_WEIGHTS_MODEL_PAIR_IDS)
    os.environ[_UPDATE_WEIGHTS_MODEL_PAIR_ENV] = selected_pair
    print(f"Selected {_UPDATE_WEIGHTS_MODEL_PAIR_ENV}={selected_pair} for this CI run")
```
**EN:** This function drives `_maybe_pin_update_weights_model_pair` with inputs such as `suite_files_rel`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_maybe_pin_update_weights_model_pair`，主要处理 `suite_files_rel` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 830-844: Function `_resolve_suite_files` / 函数 `_resolve_suite_files`
```python
def _resolve_suite_files(
    target_dir: Path, suite_files_rel: list[str], strict: bool
) -> list[str]:
    suite_files_abs = []
    for f_rel in suite_files_rel:
        f_abs = target_dir / f_rel
        if not f_abs.exists():
            msg = f"Test file {f_rel} not found in {target_dir}."
            if strict:
                print(f"Error: {msg}")
                sys.exit(1)
            print(f"Warning: {msg} Skipping.")
            continue
        suite_files_abs.append(str(f_abs))
    return suite_files_abs
```
**EN:** This function drives `_resolve_suite_files` with inputs such as `target_dir`, `suite_files_rel`, `strict`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_resolve_suite_files`，主要处理 `target_dir`, `suite_files_rel`, `strict` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 847-863: Function `_run_file_suite` / 函数 `_run_file_suite`
```python
def _run_file_suite(args, target_dir: Path) -> int:
    suite_files_rel = FILE_SUITES[args.suite]
    _maybe_pin_update_weights_model_pair(suite_files_rel)
    suite_files_abs = _resolve_suite_files(
        target_dir, suite_files_rel, args.suite in STRICT_SUITES
    )

    if not suite_files_abs:
        print(f"No valid test files found for suite '{args.suite}'.")
        return 1 if args.suite in STRICT_SUITES else 0

    exit_code, _, _ = run_pytest(
        suite_files_abs,
        filter_expr=args.filter,
        junit_xml_path=None,
    )
    return exit_code
```
**EN:** This function drives `_run_file_suite` with inputs such as `args`, `target_dir`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_run_file_suite`，主要处理 `args`, `target_dir` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 866-870: Function `_get_dynamic_suite_cases` / 函数 `_get_dynamic_suite_cases`
```python
def _get_dynamic_suite_cases(suite: str) -> list[DiffusionTestCase]:
    cases = []
    for _, case_group in PARAMETRIZED_CASE_GROUPS[suite]:
        cases.extend(case_group)
    return cases
```
**EN:** This function drives `_get_dynamic_suite_cases` with inputs such as `suite`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_get_dynamic_suite_cases`，主要处理 `suite` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 873-884: Function `_get_parametrized_files_for_case_ids` / 函数 `_get_parametrized_files_for_case_ids`
```python
def _get_parametrized_files_for_case_ids(
    suite: str, case_ids: set[str], target_dir: Path
) -> list[str]:
    files = []
    for filename, case_group in PARAMETRIZED_CASE_GROUPS[suite]:
        if any(case.id in case_ids for case in case_group):
            file_path = target_dir / filename
            if file_path.exists():
                files.append(str(file_path))
            else:
                logger.warning("Test file %s not found in %s", filename, target_dir)
    return files
```
**EN:** This function drives `_get_parametrized_files_for_case_ids` with inputs such as `suite`, `case_ids`, `target_dir`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_get_parametrized_files_for_case_ids`，主要处理 `suite`, `case_ids`, `target_dir` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 887-897: Function `_get_standalone_file` / 函数 `_get_standalone_file`
```python
def _get_standalone_file(target_dir: Path, suite: str, index: int) -> str | None:
    standalone_files = STANDALONE_FILES.get(suite, [])
    if index < 0 or index >= len(standalone_files):
        return None
    file_path = target_dir / standalone_files[index]
    if file_path.exists():
        return str(file_path)
    logger.warning(
        "Standalone test file %s not found in %s", standalone_files[index], target_dir
    )
    return None
```
**EN:** This function drives `_get_standalone_file` with inputs such as `target_dir`, `suite`, `index`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_get_standalone_file`，主要处理 `target_dir`, `suite`, `index` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 900-1152: Function `_run_dynamic_suite` / 函数 `_run_dynamic_suite`
```python
def _run_dynamic_suite(args, target_dir: Path) -> int:
    if args.partition_plan_json:
        assignment = parse_partition_plan(
            suite=args.suite,
            partition_id=args.partition_id,
            total_partitions=args.total_partitions,
            plan_json=args.partition_plan_json,
        )

        rows = [[args.suite, f"{args.partition_id + 1}/{args.total_partitions}"]]
        print(tabulate.tabulate(rows, headers=["Suite", "Partition"], tablefmt="psql"))

        total_est_time = 0.0
        executed_cases: list[str] = []
# ...
        ),
        standalone_measurements=[measurement],
    )
    return exit_code
```
**EN:** This function drives `_run_dynamic_suite` with inputs such as `args`, `target_dir`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_run_dynamic_suite`，主要处理 `args`, `target_dir` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1155-1218: Function `main` / 函数 `main`
```python
def main():
    args = parse_args()
    validate_standalone_file_est_times()
    test_root_dir = Path(__file__).resolve().parent
    target_dir = test_root_dir / args.base_dir

    if not target_dir.exists():
        print(f"Error: Target directory {target_dir} does not exist.")
        sys.exit(1)

    if args.suite in COMPONENT_ACCURACY_SUITES:
        suite_files_rel = FILE_SUITES[args.suite]
        suite_files_abs = _resolve_suite_files(
            target_dir, suite_files_rel, args.suite in STRICT_SUITES
# ...
    else:
        exit_code = _run_file_suite(args, target_dir)

    sys.exit(exit_code)
```
**EN:** This function drives `main`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `main`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1219-1222: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    main()
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Distributed execution / 分布式执行
- Automated verification / 自动化验证
- Pytest-based assertions / 基于 Pytest 的断言
- Command-line interface / 命令行接口

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.test.partitioning`, `sglang.multimodal_gen.test.server.gpu_cases`, `sglang.multimodal_gen.test.server.testcase_configs`
- **External / 外部**: `xml.etree.ElementTree`, `tabulate`
- **Stdlib / 标准库**: `argparse`, `copy`, `json`, `os`, `random`, `subprocess`, `sys`, `time`
