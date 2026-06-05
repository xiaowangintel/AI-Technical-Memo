# diffusion_case_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/utils/diffusion/diffusion_case_parser.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `diffusion_case_parser` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `diffusion_case_parser` 流程，主要负责CI 编排、测试执行。它属于 `diffusion` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-15: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
AST-based parser for diffusion test cases.

This module parses the diffusion case source and run_suite.py using AST to
extract test case information without requiring sglang dependencies. The case
source file is discovered from ONE_GPU_CASES/TWO_GPU_CASES imports in
run_suite.py so CI keeps a single source of truth.

Usage:
    # From sibling scripts in this directory:
    from diffusion_case_parser import collect_diffusion_suites, resolve_case_config_path
    case_config_path = resolve_case_config_path(repo_root, run_suite_path)
    suites = collect_diffusion_suites(case_config_path, run_suite_path, baseline_path)
"""
```
**EN:** AST-based parser for diffusion test cases.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 17-21: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import ast
import json
from dataclasses import dataclass
from pathlib import Path
from typing import Dict, List, Optional
```
**EN:** This block loads ast, json, dataclasses, pathlib, typing. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 ast, json, dataclasses, pathlib, typing。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 24-45: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
CASE_LIST_TO_SUITE = {
    "ONE_GPU_CASES": "1-gpu",
    "ONE_GPU_CASES_A": "1-gpu",
    "ONE_GPU_CASES_B": "1-gpu",
    "ONE_GPU_CASES_C": "1-gpu-b200",
    "ONE_GPU_MODELOPT_FP8_CASES": "1-gpu",
    "ONE_GPU_MODELOPT_CASES": "1-gpu-b200",
    "ONE_GPU_B200_CASES": "1-gpu-b200",
    "TWO_GPU_CASES": "2-gpu",
    "TWO_GPU_CASES_A": "2-gpu",
    "TWO_GPU_CASES_B": "2-gpu",
}

# Default estimated time for cases without baseline (5 minutes)
DEFAULT_EST_TIME_SECONDS = 300.0

# Fixed overhead for server startup when estimated_full_test_time_s is not set
STARTUP_OVERHEAD_SECONDS = 120.0

# Paths relative to repository root
BASELINE_REL_PATH = "python/sglang/multimodal_gen/test/server/perf_baselines.json"
RUN_SUITE_REL_PATH = "python/sglang/multimodal_gen/test/run_suite.py"
```
**EN:** This section defines CASE_LIST_TO_SUITE, DEFAULT_EST_TIME_SECONDS, STARTUP_OVERHEAD_SECONDS, BASELINE_REL_PATH, RUN_SUITE_REL_PATH, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 CASE_LIST_TO_SUITE, DEFAULT_EST_TIME_SECONDS, STARTUP_OVERHEAD_SECONDS, BASELINE_REL_PATH, RUN_SUITE_REL_PATH，用于把可复用的默认值集中在模块顶部。

### Lines 49-54: Declares the `DiffusionCaseInfo` class / 声明 `DiffusionCaseInfo` 类
```python
class DiffusionCaseInfo:
    """Information about a single diffusion test case."""

    case_id: str  # e.g., "qwen_image_t2i"
    suite: str  # "1-gpu" or "2-gpu"
    est_time: float  # estimated time in seconds
```
**EN:** This block declares the `DiffusionCaseInfo` class. Its body mainly serves as a namespace or lightweight data container.
**CN:** 该代码块声明了 `DiffusionCaseInfo` 类。它的主体更像命名空间或轻量数据容器。

### Lines 58-67: Declares the `DiffusionSuiteInfo` class / 声明 `DiffusionSuiteInfo` 类
```python
class DiffusionSuiteInfo:
    """Complete information for a test suite."""

    suite: str  # "1-gpu" or "2-gpu"
    cases: List[DiffusionCaseInfo]  # parametrized test cases
    standalone_files: List[str]  # standalone test files
    standalone_est_times: Dict[str, float]  # standalone file -> estimated seconds
    missing_standalone_estimates: List[
        str
    ]  # standalone files without configured estimate
```
**EN:** This block declares the `DiffusionSuiteInfo` class. Its body mainly serves as a namespace or lightweight data container.
**CN:** 该代码块声明了 `DiffusionSuiteInfo` 类。它的主体更像命名空间或轻量数据容器。

### Lines 70-171: Declares the `DiffusionTestCaseVisitor` class / 声明 `DiffusionTestCaseVisitor` 类
```python
class DiffusionTestCaseVisitor(ast.NodeVisitor):
    """
    AST visitor to extract DiffusionTestCase definitions from the case config.

    Parses assignments like:
        ONE_GPU_CASES_A: list[DiffusionTestCase] = [
            DiffusionTestCase("case_id", ...),
            ...
        ]
    """

    def __init__(self):
        self.cases: Dict[str, List[str]] = {}  # list_name -> [case_id, ...]

    def visit_Assign(self, node: ast.Assign):
        self._process_assignment(node.targets, node.value)
        self.generic_visit(node)

    def visit_AnnAssign(self, node: ast.AnnAssign):
        if node.target and node.value:
            self._process_assignment([node.target], node.value)
        self.generic_visit(node)

    def visit_AugAssign(self, node: ast.AugAssign):
        self._process_aug_assignment(node.target, node.op, node.value)
        self.generic_visit(node)

    def _process_assignment(self, targets: List[ast.AST], value: ast.AST):
        """Process an assignment to extract case IDs."""
        for target in targets:
            if isinstance(target, ast.Name):
                list_name = target.id
                case_ids = self._extract_case_ids(value)
                if case_ids is not None:
                    self.cases[list_name] = case_ids

    def _process_aug_assignment(self, target: ast.AST, op: ast.AST, value: ast.AST):
        """Process `+=` style assignment to merge case lists."""
        if not isinstance(target, ast.Name) or not isinstance(op, ast.Add):
            return

        if isinstance(value, ast.Name):
            target_suite = CASE_LIST_TO_SUITE.get(target.id)
            value_suite = CASE_LIST_TO_SUITE.get(value.id)
            if target_suite and value_suite and target_suite != value_suite:
                return

        rhs_case_ids = self._extract_case_ids(value)
        if rhs_case_ids is None:
            return

        lhs_case_ids = self.cases.get(target.id, [])
        self.cases[target.id] = [*lhs_case_ids, *rhs_case_ids]

    def _extract_case_ids(self, node: ast.AST) -> Optional[List[str]]:
        """Extract case IDs from a supported expression."""
        if isinstance(node, ast.List):
            return self._extract_case_ids_from_list(node)

        if isinstance(node, ast.Name):
            # Reference to a previously parsed list variable.
            if node.id not in self.cases:
                return None
            return list(self.cases[node.id])

        if isinstance(node, ast.BinOp) and isinstance(node.op, ast.Add):
            left_ids = self._extract_case_ids(node.left)
            right_ids = self._extract_case_ids(node.right)
            if left_ids is None or right_ids is None:
                return None
            return [*left_ids, *right_ids]

        return None

    def _extract_case_ids_from_list(self, node: ast.List) -> List[str]:
        """Extract case IDs from a literal list of DiffusionTestCase calls."""
        case_ids = []
        for elt in node.elts:
            if isinstance(elt, ast.Starred):
                starred_case_ids = self._extract_case_ids(elt.value)
                if starred_case_ids:
                    case_ids.extend(starred_case_ids)
                continue
            case_id = self._extract_case_id_from_call(elt)
            if case_id:
                case_ids.append(case_id)
        return case_ids

    def _extract_case_id_from_call(self, node: ast.AST) -> Optional[str]:
        """Extract case_id from DiffusionTestCase(...) call."""
        if not isinstance(node, ast.Call):
            return None

        # First positional argument is the case_id.
        if isinstance(node.func, ast.Name) and node.func.id in {
            "DiffusionTestCase",
            "_make_modelopt_ci_case",
        }:
            if node.args and isinstance(node.args[0], ast.Constant):
                return node.args[0].value

        return None
```
**EN:** This block declares the `DiffusionTestCaseVisitor` class. It extends ast.NodeVisitor. The class groups 9 method(s), including __init__, visit_Assign, visit_AnnAssign, visit_AugAssign, _process_assignment.
**CN:** 该代码块声明了 `DiffusionTestCaseVisitor` 类。它继承自 ast.NodeVisitor。该类集中定义了 9 个方法，包括 __init__, visit_Assign, visit_AnnAssign, visit_AugAssign, _process_assignment。

### Lines 174-215: Defines the `resolve_case_config_path` routine / 定义 `resolve_case_config_path` 例程
```python
def resolve_case_config_path(repo_root: Path, run_suite_path: Path) -> Path:
    """
    Resolve the diffusion case config path from run_suite imports.

    run_suite.py must import BOTH ONE_GPU_CASES and TWO_GPU_CASES from the same
    module. That imported module is treated as the single source of truth.
    """
    with open(run_suite_path, "r", encoding="utf-8") as f:
        content = f.read()

    tree = ast.parse(content, filename=str(run_suite_path))
    one_gpu_module: Optional[str] = None
    two_gpu_module: Optional[str] = None

    for node in ast.walk(tree):
        if not isinstance(node, ast.ImportFrom) or not node.module:
            continue
        imported_names = {alias.name for alias in node.names}
        if "ONE_GPU_CASES" in imported_names:
            one_gpu_module = node.module
        if "TWO_GPU_CASES" in imported_names:
            two_gpu_module = node.module

    if one_gpu_module is None or two_gpu_module is None:
        raise RuntimeError(
            "run_suite.py must import BOTH ONE_GPU_CASES and TWO_GPU_CASES."
        )
    if one_gpu_module != two_gpu_module:
        raise RuntimeError(
            "run_suite.py imports ONE_GPU_CASES and TWO_GPU_CASES from different "
            f"modules: {one_gpu_module} vs {two_gpu_module}"
        )

    rel_path = Path(*one_gpu_module.split(".")).with_suffix(".py")
    candidates = [repo_root / rel_path, repo_root / "python" / rel_path]
    case_config_path = next((path for path in candidates if path.exists()), None)
    if case_config_path is None:
        raise FileNotFoundError(
            "Resolved case config from run_suite does not exist. Checked: "
            + ", ".join(str(path) for path in candidates)
        )
    return case_config_path
```
**EN:** This block defines `resolve_case_config_path`. Resolve the diffusion case config path from run_suite imports. It accepts 2 parameter(s): repo_root, run_suite_path. Internally it touches filesystem paths, reads or writes files, checks file existence.
**CN:** 该代码块定义了 `resolve_case_config_path`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：repo_root, run_suite_path。内部会处理文件系统路径、读写文件、检查文件是否存在。

### Lines 218-278: Declares the `RunSuiteVisitor` class / 声明 `RunSuiteVisitor` 类
```python
class RunSuiteVisitor(ast.NodeVisitor):
    """
    AST visitor to extract standalone metadata from run_suite.py.

    Parses:
        STANDALONE_FILES = {
            "1-gpu": ["test_lora_format_adapter.py"],
            "2-gpu": [],
        }
    """

    def __init__(self):
        self.standalone_files: Dict[str, List[str]] = {}
        self.standalone_est_times: Dict[str, Dict[str, float]] = {}

    def visit_Assign(self, node: ast.Assign):
        for target in node.targets:
            if isinstance(target, ast.Name) and target.id == "STANDALONE_FILES":
                self.standalone_files = self._extract_file_dict(node.value)
            if (
                isinstance(target, ast.Name)
                and target.id == "STANDALONE_FILE_EST_TIMES"
            ):
                self.standalone_est_times = self._extract_est_time_dict(node.value)
        self.generic_visit(node)

    def _extract_file_dict(self, node: ast.AST) -> Dict[str, List[str]]:
        """Extract dictionary of suite -> file list."""
        result = {}
        if isinstance(node, ast.Dict):
            for key, value in zip(node.keys, node.values):
                if isinstance(key, ast.Constant) and isinstance(value, ast.List):
                    suite = key.value
                    files = [
                        elt.value for elt in value.elts if isinstance(elt, ast.Constant)
                    ]
                    result[suite] = files
        return result

    def _extract_est_time_dict(self, node: ast.AST) -> Dict[str, Dict[str, float]]:
        """Extract dictionary of suite -> standalone file -> estimated seconds."""
        result = {}
        if not isinstance(node, ast.Dict):
            return result

        for key, value in zip(node.keys, node.values):
            if not isinstance(key, ast.Constant) or not isinstance(value, ast.Dict):
                continue

            suite = key.value
            suite_est_times = {}
            for inner_key, inner_value in zip(value.keys, value.values):
                if not (
                    isinstance(inner_key, ast.Constant)
                    and isinstance(inner_value, ast.Constant)
                ):
                    continue
                suite_est_times[inner_key.value] = float(inner_value.value)
            result[suite] = suite_est_times

        return result
```
**EN:** This block declares the `RunSuiteVisitor` class. It extends ast.NodeVisitor. The class groups 4 method(s), including __init__, visit_Assign, _extract_file_dict, _extract_est_time_dict.
**CN:** 该代码块声明了 `RunSuiteVisitor` 类。它继承自 ast.NodeVisitor。该类集中定义了 4 个方法，包括 __init__, visit_Assign, _extract_file_dict, _extract_est_time_dict。

### Lines 281-304: Defines the `load_baselines` routine / 定义 `load_baselines` 例程
```python
def load_baselines(baseline_path: Path) -> Dict[str, float]:
    """
    Load performance baselines from JSON file.

    Returns:
        Dictionary mapping case_id to estimated time in seconds.
    """
    if not baseline_path.exists():
        return {}

    with open(baseline_path, "r", encoding="utf-8") as f:
        data = json.load(f)

    baselines = {}
    scenarios = data.get("scenarios", {})

    for case_id, scenario in scenarios.items():
        if scenario.get("estimated_full_test_time_s") is not None:
            baselines[case_id] = scenario["estimated_full_test_time_s"]
        else:
            expected_e2e_ms = scenario.get("expected_e2e_ms", 0)
            baselines[case_id] = expected_e2e_ms / 1000.0 + STARTUP_OVERHEAD_SECONDS

    return baselines
```
**EN:** This block defines `load_baselines`. Load performance baselines from JSON file. It accepts 1 parameter(s): baseline_path. Internally it checks file existence, loads structured data, reads or writes files.
**CN:** 该代码块定义了 `load_baselines`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：baseline_path。内部会检查文件是否存在、加载结构化数据、读写文件。

### Lines 307-309: Defines the `get_case_est_time` routine / 定义 `get_case_est_time` 例程
```python
def get_case_est_time(case_id: str, baselines: Dict[str, float]) -> float:
    """Get estimated time for a case, with fallback to default."""
    return baselines.get(case_id, DEFAULT_EST_TIME_SECONDS)
```
**EN:** This block defines `get_case_est_time`. Get estimated time for a case, with fallback to default. It accepts 2 parameter(s): case_id, baselines. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `get_case_est_time`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：case_id, baselines。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 312-327: Defines the `parse_testcase_configs` routine / 定义 `parse_testcase_configs` 例程
```python
def parse_testcase_configs(config_path: Path) -> Dict[str, List[str]]:
    """
    Parse a diffusion case config file to extract case IDs.

    Returns:
        Dictionary mapping list name to case IDs.
        e.g., {"ONE_GPU_CASES_A": ["qwen_image_t2i", ...], ...}
    """
    with open(config_path, "r", encoding="utf-8") as f:
        content = f.read()

    tree = ast.parse(content, filename=str(config_path))
    visitor = DiffusionTestCaseVisitor()
    visitor.visit(tree)

    return visitor.cases
```
**EN:** This block defines `parse_testcase_configs`. Parse a diffusion case config file to extract case IDs. It accepts 1 parameter(s): config_path. Internally it reads or writes files.
**CN:** 该代码块定义了 `parse_testcase_configs`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：config_path。内部会读写文件。

### Lines 330-348: Defines the `parse_run_suite_standalone_data` routine / 定义 `parse_run_suite_standalone_data` 例程
```python
def parse_run_suite_standalone_data(
    run_suite_path: Path,
) -> tuple[Dict[str, List[str]], Dict[str, Dict[str, float]]]:
    """
    Parse run_suite.py to extract standalone file metadata.

    Returns:
        Tuple of:
          - suite -> standalone file list
          - suite -> standalone file -> estimated seconds
    """
    with open(run_suite_path, "r", encoding="utf-8") as f:
        content = f.read()

    tree = ast.parse(content, filename=str(run_suite_path))
    visitor = RunSuiteVisitor()
    visitor.visit(tree)

    return visitor.standalone_files, visitor.standalone_est_times
```
**EN:** This block defines `parse_run_suite_standalone_data`. Parse run_suite.py to extract standalone file metadata. It accepts 1 parameter(s): run_suite_path. Internally it reads or writes files.
**CN:** 该代码块定义了 `parse_run_suite_standalone_data`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：run_suite_path。内部会读写文件。

### Lines 351-365: Defines the `validate_standalone_est_times` routine / 定义 `validate_standalone_est_times` 例程
```python
def validate_standalone_est_times(
    standalone_files: Dict[str, List[str]],
    standalone_est_times: Dict[str, Dict[str, float]],
) -> Dict[str, List[str]]:
    missing_by_suite = {}
    for suite, files in standalone_files.items():
        suite_est_times = standalone_est_times.get(suite, {})
        missing = [
            standalone_file
            for standalone_file in files
            if standalone_file not in suite_est_times
        ]
        if missing:
            missing_by_suite[suite] = missing
    return missing_by_suite
```
**EN:** This block defines `validate_standalone_est_times`. It accepts 2 parameter(s): standalone_files, standalone_est_times. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `validate_standalone_est_times`。它接收 2 个参数：standalone_files, standalone_est_times。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 368-434: Defines the `collect_diffusion_suites` routine / 定义 `collect_diffusion_suites` 例程
```python
def collect_diffusion_suites(
    case_config_path: Path,
    run_suite_path: Path,
    baseline_path: Path,
) -> Dict[str, DiffusionSuiteInfo]:
    """
    Collect all diffusion test suite information using AST parsing.

    Args:
        case_config_path: Path to case config (resolved from run_suite.py)
        run_suite_path: Path to run_suite.py
        baseline_path: Path to perf_baselines.json

    Returns:
        Dictionary mapping suite name to DiffusionSuiteInfo.
    """
    # Parse case IDs from the single source case config.
    case_lists = parse_testcase_configs(case_config_path)

    # Parse standalone files from run_suite.py
    standalone_files, standalone_est_times = parse_run_suite_standalone_data(
        run_suite_path
    )
    missing_standalone_estimates = validate_standalone_est_times(
        standalone_files, standalone_est_times
    )

    # Load baselines for time estimation
    baselines = load_baselines(baseline_path)

    # Build suite info
    suites = {}
    for list_name, suite in CASE_LIST_TO_SUITE.items():
        case_ids = case_lists.get(list_name, [])
        cases = [
            DiffusionCaseInfo(
                case_id=cid,
                suite=suite,
                est_time=get_case_est_time(cid, baselines),
            )
            for cid in case_ids
        ]

        if suite not in suites:
            suites[suite] = DiffusionSuiteInfo(
                suite=suite,
                cases=[],
                standalone_files=standalone_files.get(suite, []),
                standalone_est_times=dict(standalone_est_times.get(suite, {})),
                missing_standalone_estimates=list(
                    missing_standalone_estimates.get(suite, [])
                ),
            )
        suites[suite].cases.extend(cases)

    # Dedupe duplicated case IDs while preserving first-seen order.
    for suite_info in suites.values():
        seen_case_ids = set()
        deduped_cases = []
        for case in suite_info.cases:
            if case.case_id in seen_case_ids:
                continue
            seen_case_ids.add(case.case_id)
            deduped_cases.append(case)
        suite_info.cases = deduped_cases

    return suites
```
**EN:** This block defines `collect_diffusion_suites`. Collect all diffusion test suite information using AST parsing. It accepts 3 parameter(s): case_config_path, run_suite_path, baseline_path. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `collect_diffusion_suites`。其文档字符串说明了该逻辑的预期职责。它接收 3 个参数：case_config_path, run_suite_path, baseline_path。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

## Key Concepts / 关键概念
- **Filesystem coordination** / 文件系统协同
- **Environment management** / 环境管理
- **Structured data handling** / 结构化数据处理

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ast`, `dataclasses`, `json`, `pathlib`, `typing`
- **External commands inferred from code / 从代码推断的外部命令**: `python`
