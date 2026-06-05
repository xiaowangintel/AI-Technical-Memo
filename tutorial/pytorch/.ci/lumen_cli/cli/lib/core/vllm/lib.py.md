# lib.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `.ci/lumen_cli/cli/lib/core/vllm/lib.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python-side logic used for benchmarking, tooling, validation, or repository automation.
- **Purpose (CN)**: 实现用于基准、工具链、校验或仓库自动化的 Python 侧逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11 / 第 1-11 行

```python
from __future__ import annotations

import json
import logging
import os
import re
import textwrap
import urllib.request
from pathlib import Path
from typing import Any

```

- **EN:** This range mainly pulls in the imports/includes that the later benchmark or integration logic depends on.
- **CN:** 这一段主要引入后续基准或集成逻辑依赖的 import/include。

### Lines 12-19 / 第 12-19 行

```python
import yaml
from cli.lib.common.gh_summary import write_gh_step_summary
from cli.lib.common.git_helper import clone_external_repo
from cli.lib.common.pip_helper import pip_install_packages
from cli.lib.common.utils import run_command, temp_environ, working_directory
from jinja2 import Template


```

- **EN:** This range mainly pulls in the imports/includes that the later benchmark or integration logic depends on.
- **CN:** 这一段主要引入后续基准或集成逻辑依赖的 import/include。

### Lines 20-29 / 第 20-29 行

```python
VLLM_DEFAULT_RERUN_FAILURES_COUNT = 2
VLLM_DEFAULT_RERUN_FAILURES_DELAY = 10


logger = logging.getLogger(__name__)

_VLLM_TEST_LIBRARY_PATH = Path(__file__).parent / "vllm_test_library.yaml"
_DISABLED_VLLM_TESTS_PATH = Path(__file__).parent / "disabled_vllm_tests.yaml"
_DISABLED_VLLM_TESTS_ISSUE = 175899

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 30-38 / 第 30-38 行

```python

def _load_vllm_test_library_yaml() -> dict[str, Any]:
    """
    Load the VLLM test library configuration from YAML file.

    Returns:
        Dictionary containing the test library configuration.
    """
    if not _VLLM_TEST_LIBRARY_PATH.exists():
```

- **EN:** Important local symbols in this block include _load_vllm_test_library_yaml.
- **CN:** 该代码块中的重要局部符号包括 _load_vllm_test_library_yaml。

### Lines 39-46 / 第 39-46 行

```python
        raise FileNotFoundError(
            f"VLLM test library YAML file not found: {_VLLM_TEST_LIBRARY_PATH}"
        )

    with open(_VLLM_TEST_LIBRARY_PATH, encoding="utf-8") as f:
        _vllm_test_library_cache = yaml.safe_load(f)

    return _vllm_test_library_cache
```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 47-58 / 第 47-58 行

```python


_TPL_VLLM_INFO = Template(
    textwrap.dedent("""\
    ##  Vllm against Pytorch CI Test Summary
    **Vllm Commit**: [{{ vllm_commit }}](https://github.com/vllm-project/vllm/commit/{{ vllm_commit }})
    {%- if torch_sha %}
    **Pytorch Commit**: [{{ torch_sha }}](https://github.com/pytorch/pytorch/commit/{{ torch_sha }})
    {%- endif %}
""")
)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 59-67 / 第 59-67 行

```python

def sample_vllm_test_library() -> dict[str, Any]:
    """
    Load the VLLM test library configuration from YAML file.

    This is a simple sample to unblock the vllm ci development, which mimics
    https://github.com/vllm-project/vllm/blob/main/.buildkite/test-pipeline.yaml
    See run_test_plan for more details.

```

- **EN:** Important local symbols in this block include sample_vllm_test_library.
- **CN:** 该代码块中的重要局部符号包括 sample_vllm_test_library。

### Lines 68-80 / 第 68-80 行

```python
    Returns:
        Dictionary containing test configurations loaded from vllm_test_library.yaml
    """
    return _load_vllm_test_library_yaml()


def check_parallelism(tests: Any, title: str, shard_id: int = 0, num_shards: int = 0):
    """
    a method to check if the test plan is parallelism or not.
    """
    parallelism = int(tests.get("parallelism", "0"))
    is_parallel = parallelism and parallelism > 1

```

- **EN:** Important local symbols in this block include check_parallelism.
- **CN:** 该代码块中的重要局部符号包括 check_parallelism。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 81-88 / 第 81-88 行

```python
    if not is_parallel:
        return False

    if shard_id > num_shards:
        raise RuntimeError(
            f"Test {title} expects {num_shards} shards, but invalid {shard_id} is provided"
        )

```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 89-96 / 第 89-96 行

```python
    if num_shards != parallelism:
        raise RuntimeError(
            f"Test {title} expects {parallelism} shards, but invalid {num_shards} is provided"
        )

    return True


```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 97-105 / 第 97-105 行

```python
def _load_disabled_vllm_tests_from_yaml() -> list[dict[str, Any]]:
    if not _DISABLED_VLLM_TESTS_PATH.exists():
        return []
    with open(_DISABLED_VLLM_TESTS_PATH, encoding="utf-8") as f:
        data = yaml.safe_load(f)
    if not data or "disabled_tests" not in data:
        return []
    entries = data["disabled_tests"]
    if not entries:
```

- **EN:** Important local symbols in this block include _load_disabled_vllm_tests_from_yaml.
- **CN:** 该代码块中的重要局部符号包括 _load_disabled_vllm_tests_from_yaml。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 106-113 / 第 106-113 行

```python
        return []
    for entry in entries:
        if "test" not in entry or "issue" not in entry:
            raise ValueError(
                f"disabled_vllm_tests.yaml: each entry must have 'test' and 'issue' keys, got {entry}"
            )
    return entries

```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 114-121 / 第 114-121 行

```python

def _parse_disabled_tests_from_issue_body(body: str) -> list[dict[str, Any]]:
    match = re.search(r"```yaml\s*\n(.*?)```", body, re.DOTALL)
    if not match:
        return []
    block = match.group(1)
    data = yaml.safe_load(block)
    if not data or "disabled_tests" not in data:
```

- **EN:** Important local symbols in this block include _parse_disabled_tests_from_issue_body.
- **CN:** 该代码块中的重要局部符号包括 _parse_disabled_tests_from_issue_body。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 122-132 / 第 122-132 行

```python
        return []
    return data["disabled_tests"] or []


def _load_disabled_vllm_tests_from_github() -> list[dict[str, Any]]:
    if not _DISABLED_VLLM_TESTS_ISSUE:
        return []
    url = f"https://api.github.com/repos/pytorch/pytorch/issues/{_DISABLED_VLLM_TESTS_ISSUE}"
    headers = {"Accept": "application/vnd.github.v3+json"}
    token = os.environ.get("GITHUB_TOKEN")
    if token:
```

- **EN:** Important local symbols in this block include _load_disabled_vllm_tests_from_github.
- **CN:** 该代码块中的重要局部符号包括 _load_disabled_vllm_tests_from_github。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 133-143 / 第 133-143 行

```python
        headers["Authorization"] = f"token {token}"
    try:
        req = urllib.request.Request(url, headers=headers)
        with urllib.request.urlopen(req, timeout=30) as resp:
            issue = json.loads(resp.read())
        body = issue.get("body", "") or ""
        entries = _parse_disabled_tests_from_issue_body(body)
        # Filter out malformed entries — the issue body is user-editable
        entries = [e for e in entries if "test" in e]
        issue_url = issue.get("html_url", url)
        for entry in entries:
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 144-152 / 第 144-152 行

```python
            entry.setdefault("issue", issue_url)
        return entries
    except Exception:
        logger.warning(
            "Failed to fetch disabled vLLM tests from GitHub issue #%d",
            _DISABLED_VLLM_TESTS_ISSUE,
            exc_info=True,
        )
        return []
```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 153-160 / 第 153-160 行

```python


def _load_disabled_vllm_tests() -> list[dict[str, Any]]:
    yaml_entries = _load_disabled_vllm_tests_from_yaml()
    github_entries = _load_disabled_vllm_tests_from_github()
    seen = {e["test"] for e in yaml_entries}
    merged = list(yaml_entries)
    for entry in github_entries:
```

- **EN:** Important local symbols in this block include _load_disabled_vllm_tests.
- **CN:** 该代码块中的重要局部符号包括 _load_disabled_vllm_tests。

### Lines 161-171 / 第 161-171 行

```python
        if entry["test"] not in seen:
            seen.add(entry["test"])
            merged.append(entry)
    return merged


def _build_disabled_test_flags(
    disabled_tests: list[dict[str, Any]], test_plan: str
) -> str:
    flags = []
    for entry in disabled_tests:
```

- **EN:** Important local symbols in this block include _build_disabled_test_flags.
- **CN:** 该代码块中的重要局部符号包括 _build_disabled_test_flags。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 172-180 / 第 172-180 行

```python
        configs = entry.get("configs")
        if configs and test_plan not in configs:
            continue
        node_id = entry["test"]
        if "::" in node_id:
            flags.append(f"--deselect={node_id}")
        else:
            flags.append(f"--ignore={node_id}")
    return " ".join(flags)
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 181-194 / 第 181-194 行

```python


def run_test_plan(
    test_plan: str,
    test_target: str,
    tests_map: dict[str, Any],
    shard_id: int = 0,
    num_shards: int = 0,
):
    """
    a method to run list of tests based on the test plan.
    """
    logger.info("run %s tests.....", test_target)
    if test_plan not in tests_map:
```

- **EN:** Important local symbols in this block include run_test_plan.
- **CN:** 该代码块中的重要局部符号包括 run_test_plan。

### Lines 195-203 / 第 195-203 行

```python
        raise RuntimeError(
            f"test {test_plan} not found, please add it to test plan pool"
        )
    tests = tests_map[test_plan]
    pkgs = tests.get("package_install", [])
    title = tests.get("title", "unknown test")

    is_parallel = check_parallelism(tests, title, shard_id, num_shards)
    if is_parallel:
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 204-212 / 第 204-212 行

```python
        title = title.replace("%N", f"{shard_id}/{num_shards}")

    disabled_tests = _load_disabled_vllm_tests()
    disabled_flags = _build_disabled_test_flags(disabled_tests, test_plan)
    if disabled_flags:
        logger.info("Disabled test flags for %s: %s", test_plan, disabled_flags)

    logger.info("Running tests: %s", title)
    if pkgs:
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 213-220 / 第 213-220 行

```python
        logger.info("Installing packages: %s", pkgs)
        pip_install_packages(packages=pkgs, prefer_uv=True)
    with (
        working_directory(tests.get("working_directory", "tests")),
        temp_environ(tests.get("env_vars", {})),
    ):
        failures = []
        for step in tests["steps"]:
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 221-236 / 第 221-236 行

```python
            logger.info("Running step: %s", step)
            if is_parallel:
                step = replace_buildkite_placeholders(step, shard_id, num_shards)
                logger.info("Running parallel step: %s", step)
            if "pytest" in step:
                # Inject disabled test flags before rerun flags
                if disabled_flags:
                    step = step.replace("pytest", f"pytest {disabled_flags}", 1)
                # Support retry with delay for all pytest commands, pytest-rerunfailures
                # is already a dependency of vLLM. This is needed as a stop gap to reduce
                # the number of requests to HF until #172300 can be landed to enable
                # HF offline mode.
                # Use a low retry count and a high delay value to lower the risk of
                # having a retry storm and make thing worse
                rerun_count = os.getenv(
                    "VLLM_RERUN_FAILURES_COUNT", VLLM_DEFAULT_RERUN_FAILURES_COUNT
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 237-252 / 第 237-252 行

```python
                )
                rerun_delay = os.getenv(
                    "VLLM_RERUN_FAILURES_DELAY", VLLM_DEFAULT_RERUN_FAILURES_DELAY
                )
                if rerun_delay:
                    step = step.replace(
                        "pytest",
                        f"pytest --reruns {rerun_count} --reruns-delay {rerun_delay}",
                        1,
                    )
                else:
                    step = step.replace(
                        "pytest",
                        f"pytest --reruns {rerun_count}",
                        1,
                    )
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 253-262 / 第 253-262 行

```python

            code = run_command(cmd=step, check=False, use_shell=True)
            if code != 0:
                failures.append(step)
            logger.info("Finish running step: %s", step)
        if failures:
            logger.error("Failed tests: %s", failures)
            raise RuntimeError(f"{len(failures)} pytest runs failed: {failures}")
        logger.info("Done. All tests passed")

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 263-271 / 第 263-271 行

```python

def clone_vllm(dst: str = "vllm"):
    _, commit = clone_external_repo(
        target="vllm",
        repo="https://github.com/vllm-project/vllm.git",
        dst=dst,
        update_submodules=True,
    )
    return commit
```

- **EN:** Important local symbols in this block include clone_vllm.
- **CN:** 该代码块中的重要局部符号包括 clone_vllm。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 272-279 / 第 272-279 行

```python


def replace_buildkite_placeholders(step: str, shard_id: int, num_shards: int) -> str:
    mapping = {
        "$$BUILDKITE_PARALLEL_JOB_COUNT": str(num_shards),
        "$$BUILDKITE_PARALLEL_JOB": str(shard_id),
    }
    for k in sorted(mapping, key=len, reverse=True):
```

- **EN:** Important local symbols in this block include replace_buildkite_placeholders.
- **CN:** 该代码块中的重要局部符号包括 replace_buildkite_placeholders。

### Lines 280-290 / 第 280-290 行

```python
        step = step.replace(k, mapping[k])
    return step


def summarize_build_info(vllm_commit: str) -> bool:
    torch_sha = os.getenv("GITHUB_SHA")
    md = (
        _TPL_VLLM_INFO.render(vllm_commit=vllm_commit, torch_sha=torch_sha).strip()
        + "\n"
    )
    return write_gh_step_summary(md)
```

- **EN:** Important local symbols in this block include summarize_build_info.
- **CN:** 该代码块中的重要局部符号包括 summarize_build_info。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **Repository bootstrap** — 与仓库状态、子模块或源码检出流程交互。
- **Declarative configuration** — 以声明式格式表示构建或工作流设置。
- **Structured metadata** — 存放机器可读的设置或清单式元数据。
- **Representative symbols: _load_vllm_test_library_yaml, sample_vllm_test_library, check_parallelism, _load_disabled_vllm_tests_from_yaml, _parse_disabled_tests_from_issue_body, _load_disabled_vllm_tests_from_github, _load_disabled_vllm_tests, _build_disabled_test_flags** — 代表性符号：_load_vllm_test_library_yaml、sample_vllm_test_library、check_parallelism、_load_disabled_vllm_tests_from_yaml、_parse_disabled_tests_from_issue_body、_load_disabled_vllm_tests_from_github、_load_disabled_vllm_tests、_build_disabled_test_flags

## Dependencies / 依赖关系

- `__future__`
- `json`
- `logging`
- `os`
- `re`
- `textwrap`
- `urllib.request`
- `pathlib`
- `typing`
- `yaml`
- `cli.lib.common.gh_summary`
- `cli.lib.common.git_helper`
- `cli.lib.common.pip_helper`
- `cli.lib.common.utils`
- `jinja2`
