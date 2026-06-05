# verify_diffusion_coverage.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/utils/diffusion/verify_diffusion_coverage.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `verify_diffusion_coverage` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `verify_diffusion_coverage` 流程，主要负责CI 编排、测试执行。它属于 `diffusion` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-14: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Verify 100% coverage of diffusion test cases.

This script checks that all expected test cases were executed across all partitions.
Designed to run in the CI summary job after all partition jobs complete.

Usage:
    python scripts/ci/utils/diffusion/verify_diffusion_coverage.py --reports-dir <path>

Exit codes:
    0 - All cases executed (100% coverage)
    1 - Missing cases detected (coverage < 100%)
"""
```
**EN:** Verify 100% coverage of diffusion test cases.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 16-26: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import json
import sys
from pathlib import Path

from diffusion_case_parser import (
    BASELINE_REL_PATH,
    RUN_SUITE_REL_PATH,
    collect_diffusion_suites,
    resolve_case_config_path,
)
```
**EN:** This block loads argparse, json, sys, pathlib, diffusion_case_parser. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, json, sys, pathlib, diffusion_case_parser。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 28-28: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
DYNAMIC_SUITES = {"1-gpu", "2-gpu"}
```
**EN:** This section defines DYNAMIC_SUITES, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 DYNAMIC_SUITES，用于把可复用的默认值集中在模块顶部。

### Lines 31-37: Defines the `load_execution_reports` routine / 定义 `load_execution_reports` 例程
```python
def load_execution_reports(reports_dir: Path) -> list[dict]:
    """Load all execution report JSON files from the given directory."""
    reports = []
    for json_file in reports_dir.glob("**/execution_report_*.json"):
        with open(json_file, "r", encoding="utf-8") as f:
            reports.append(json.load(f))
    return reports
```
**EN:** This block defines `load_execution_reports`. Load all execution report JSON files from the given directory. It accepts 1 parameter(s): reports_dir. Internally it loads structured data, reads or writes files.
**CN:** 该代码块定义了 `load_execution_reports`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：reports_dir。内部会加载结构化数据、读写文件。

### Lines 40-83: Defines the `get_expected_cases` routine / 定义 `get_expected_cases` 例程
```python
def get_expected_cases(repo_root: Path) -> dict[str, set[str]]:
    """
    Get all expected cases from case config and run_suite.py.

    Returns:
        Dictionary mapping suite name to set of expected case IDs.
        Standalone files are represented as "standalone:<filename>".
    """
    baseline_path = repo_root / BASELINE_REL_PATH
    run_suite_path = repo_root / RUN_SUITE_REL_PATH
    case_config_path = resolve_case_config_path(repo_root, run_suite_path)

    suites = collect_diffusion_suites(
        case_config_path,
        run_suite_path,
        baseline_path,
    )

    expected = {}
    for suite_name, suite_info in suites.items():
        if suite_name not in DYNAMIC_SUITES:
            continue
        case_ids = set(case.case_id for case in suite_info.cases)
        # Add standalone files as special case IDs
        for standalone_file in suite_info.standalone_files:
            case_ids.add(f"standalone:{standalone_file}")
        expected[suite_name] = case_ids

    empty_dynamic_suites = [
        suite_name
        for suite_name in DYNAMIC_SUITES
        if suite_name in expected
        and not any(
            not case_id.startswith("standalone:") for case_id in expected[suite_name]
        )
    ]
    if empty_dynamic_suites:
        raise RuntimeError(
            "Parsed zero parametrized cases for diffusion suites: "
            + ", ".join(sorted(empty_dynamic_suites))
            + ". Refuse to pass coverage verification."
        )

    return expected
```
**EN:** This block defines `get_expected_cases`. Get all expected cases from case config and run_suite.py. It accepts 1 parameter(s): repo_root. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `get_expected_cases`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：repo_root。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 86-106: Defines the `collect_executed_cases` routine / 定义 `collect_executed_cases` 例程
```python
def collect_executed_cases(reports: list[dict]) -> dict[str, set[str]]:
    """
    Collect all executed cases from execution reports.

    Returns:
        Dictionary mapping suite name to set of executed case IDs.
    """
    executed = {}
    for report in reports:
        suite = report["suite"]
        if suite not in executed:
            executed[suite] = set()

        executed_cases = report.get("executed_cases", [])
        if executed_cases:
            executed[suite].update(executed_cases)
        elif report["is_standalone"]:
            standalone_file = report["standalone_file"]
            executed[suite].add(f"standalone:{standalone_file}")

    return executed
```
**EN:** This block defines `collect_executed_cases`. Collect all executed cases from execution reports. It accepts 1 parameter(s): reports. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `collect_executed_cases`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：reports。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 109-126: Defines the `collect_case_results` routine / 定义 `collect_case_results` 例程
```python
def collect_case_results(reports: list[dict]) -> dict[str, dict[str, str]]:
    """
    Collect case results (pass/fail/error status) from execution reports.

    Returns:
        Dictionary mapping suite name to {case_id: status} dictionary.
    """
    results = {}
    for report in reports:
        suite = report["suite"]
        if suite not in results:
            results[suite] = {}

        # Get case_results from report (empty dict for legacy reports)
        case_results = report.get("case_results", {})
        results[suite].update(case_results)

    return results
```
**EN:** This block defines `collect_case_results`. Collect case results (pass/fail/error status) from execution reports. It accepts 1 parameter(s): reports. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `collect_case_results`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：reports。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 129-137: Defines the `collect_missing_standalone_estimates` routine / 定义 `collect_missing_standalone_estimates` 例程
```python
def collect_missing_standalone_estimates(reports: list[dict]) -> dict[str, set[str]]:
    missing_by_suite: dict[str, set[str]] = {}
    for report in reports:
        suite = report["suite"]
        missing = report.get("missing_standalone_estimates", [])
        if not missing:
            continue
        missing_by_suite.setdefault(suite, set()).update(missing)
    return missing_by_suite
```
**EN:** This block defines `collect_missing_standalone_estimates`. It accepts 1 parameter(s): reports. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `collect_missing_standalone_estimates`。它接收 1 个参数：reports。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 140-146: Defines the `collect_standalone_measurements` routine / 定义 `collect_standalone_measurements` 例程
```python
def collect_standalone_measurements(reports: list[dict]) -> dict[tuple[str, str], dict]:
    measurements: dict[tuple[str, str], dict] = {}
    for report in reports:
        for measurement in report.get("standalone_measurements", []):
            key = (measurement["suite"], measurement["standalone_file"])
            measurements[key] = measurement
    return measurements
```
**EN:** This block defines `collect_standalone_measurements`. It accepts 1 parameter(s): reports. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `collect_standalone_measurements`。它接收 1 个参数：reports。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 149-173: Defines the `print_missing_standalone_estimates_summary` routine / 定义 `print_missing_standalone_estimates_summary` 例程
```python
def print_missing_standalone_estimates_summary(
    missing_by_suite: dict[str, set[str]],
    measurements: dict[tuple[str, str], dict],
) -> None:
    if not missing_by_suite:
        return

    print("\n" + "=" * 60)
    print(
        "Add standalone estimate(s) to "
        "python/sglang/multimodal_gen/test/run_suite.py"
    )
    print("=" * 60)
    print("The following standalone file(s) used fallback estimate 300.0s.")
    print("Update STANDALONE_FILE_EST_TIMES with the measured runtime below:\n")

    for suite in sorted(missing_by_suite):
        print(f'"{suite}": {{')
        for standalone_file in sorted(missing_by_suite[suite]):
            measurement = measurements.get((suite, standalone_file))
            measured_time = (
                measurement["measured_full_test_time_s"] if measurement else 300.0
            )
            print(f'    "{standalone_file}": {measured_time:.1f},')
        print("}\n")
```
**EN:** This block defines `print_missing_standalone_estimates_summary`. It accepts 2 parameter(s): missing_by_suite, measurements. Internally it emits status messages.
**CN:** 该代码块定义了 `print_missing_standalone_estimates_summary`。它接收 2 个参数：missing_by_suite, measurements。内部会输出状态信息。

### Lines 176-193: Defines the `verify_coverage` routine / 定义 `verify_coverage` 例程
```python
def verify_coverage(
    expected: dict[str, set[str]],
    executed: dict[str, set[str]],
) -> tuple[bool, dict[str, set[str]]]:
    """
    Verify that all expected cases were executed.

    Returns:
        Tuple of (is_complete, missing_cases_by_suite)
    """
    missing = {}
    for suite, expected_cases in expected.items():
        executed_cases = executed.get(suite, set())
        suite_missing = expected_cases - executed_cases
        if suite_missing:
            missing[suite] = suite_missing

    return len(missing) == 0, missing
```
**EN:** This block defines `verify_coverage`. Verify that all expected cases were executed. It accepts 2 parameter(s): expected, executed. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `verify_coverage`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：expected, executed。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 196-251: Defines the `print_results_summary` routine / 定义 `print_results_summary` 例程
```python
def print_results_summary(
    case_results: dict[str, dict[str, str]],
) -> tuple[int, int, int]:
    """
    Print test results summary and return counts.

    Returns:
        Tuple of (passed_count, failed_count, error_count)
    """
    # Check if we have any results data
    total_results = sum(len(results) for results in case_results.values())
    if total_results == 0:
        print("\nTest Results: No results data available (legacy reports)")
        return (0, 0, 0)

    # Count by status
    passed_count = 0
    failed_count = 0
    error_count = 0
    failed_cases: dict[str, list[str]] = {}

    for suite, results in case_results.items():
        for case_id, status in results.items():
            if status == "pass":
                passed_count += 1
            elif status == "fail":
                failed_count += 1
                if suite not in failed_cases:
                    failed_cases[suite] = []
                failed_cases[suite].append(case_id)
            elif status == "error":
                error_count += 1
                if suite not in failed_cases:
                    failed_cases[suite] = []
                failed_cases[suite].append(f"{case_id} (error)")

    # Print summary
    total = passed_count + failed_count + error_count
    print("\n" + "=" * 60)
    print("Test Results Summary")
    print("=" * 60)
    print(f"  Total executed: {total}")
    print(f"  ✅ Passed: {passed_count}")
    print(f"  ❌ Failed: {failed_count}")
    if error_count > 0:
        print(f"  ⚠️  Errors: {error_count}")

    # Print failed cases if any
    if failed_cases:
        print("\nFailed cases:")
        for suite, cases in sorted(failed_cases.items()):
            print(f"  {suite}:")
            for case_id in sorted(cases):
                print(f"    - {case_id}")

    return (passed_count, failed_count, error_count)
```
**EN:** This block defines `print_results_summary`. Print test results summary and return counts. It accepts 1 parameter(s): case_results. Internally it emits status messages.
**CN:** 该代码块定义了 `print_results_summary`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：case_results。内部会输出状态信息。

### Lines 254-339: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser(
        description="Verify 100% coverage of diffusion test cases"
    )
    parser.add_argument(
        "--reports-dir",
        type=str,
        required=True,
        help="Directory containing execution report JSON files",
    )
    args = parser.parse_args()

    # Determine repository root
    script_dir = Path(__file__).resolve().parent
    repo_root = script_dir.parent.parent.parent.parent

    reports_dir = Path(args.reports_dir)

    print("=" * 60)
    print("Diffusion CI Coverage Verification")
    print("=" * 60)

    # Load execution reports
    reports = load_execution_reports(reports_dir)
    print(f"\nLoaded {len(reports)} execution reports")

    if not reports:
        print("\nERROR: No execution reports found!")
        print(f"Expected reports in: {reports_dir}")
        sys.exit(1)

    # Get expected cases
    try:
        expected = get_expected_cases(repo_root)
    except (RuntimeError, FileNotFoundError) as exc:
        print(f"\nERROR: {exc}")
        sys.exit(1)
    print("\nExpected cases by suite:")
    for suite, cases in expected.items():
        print(f"  {suite}: {len(cases)} cases")

    # Collect executed cases
    executed = collect_executed_cases(reports)
    print("\nExecuted cases by suite:")
    for suite, cases in executed.items():
        print(f"  {suite}: {len(cases)} cases")

    # Collect case results
    case_results = collect_case_results(reports)
    missing_standalone_estimates = collect_missing_standalone_estimates(reports)
    standalone_measurements = collect_standalone_measurements(reports)

    # Verify coverage
    is_complete, missing = verify_coverage(expected, executed)

    if is_complete:
        print("\n" + "=" * 60)
        print("✅ COVERAGE: 100% - All test cases executed")
        print("=" * 60)
    else:
        print("\n" + "=" * 60)
        print("❌ COVERAGE FAILURE: Missing test cases detected")
        print("=" * 60)
        for suite, cases in missing.items():
            print(f"\n{suite.upper()} suite - Missing {len(cases)} case(s):")
            for case_id in sorted(cases):
                print(f"  - {case_id}")

    # Print test results summary
    passed_count, failed_count, error_count = print_results_summary(case_results)
    print_missing_standalone_estimates_summary(
        missing_standalone_estimates, standalone_measurements
    )

    # Exit with appropriate code
    if not is_complete:
        sys.exit(1)
    elif missing_standalone_estimates:
        sys.exit(1)
    elif failed_count > 0 or error_count > 0:
        print("\n" + "=" * 60)
        print("⚠️  WARNING: Some tests failed but coverage is complete")
        print("=" * 60)
        sys.exit(0)  # Coverage is complete, failures are visible in results
    else:
        sys.exit(0)
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it touches filesystem paths, parses CLI arguments, emits status messages, controls process exit status.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会处理文件系统路径、解析命令行参数、输出状态信息、控制进程退出状态。

### Lines 342-343: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **CLI parsing** / 命令行解析
- **Filesystem coordination** / 文件系统协同
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Structured data handling** / 结构化数据处理

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `json`, `pathlib`, `sys`
- **Third-party modules / 第三方模块**: `diffusion_case_parser`
