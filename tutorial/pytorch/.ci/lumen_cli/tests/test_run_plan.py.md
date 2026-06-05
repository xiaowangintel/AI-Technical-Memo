# test_run_plan.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `.ci/lumen_cli/tests/test_run_plan.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python-side logic used for benchmarking, tooling, validation, or repository automation. Backend or dtype coverage is a visible axis of variation in the implementation. The file header summarizes the intent as: "tests/test_run_test_plan.py."
- **Purpose (CN)**: 实现用于基准、工具链、校验或仓库自动化的 Python 侧逻辑。 后端或 dtype 覆盖是该实现中明显的变化维度。 文件头部将其意图概括为：“tests/test_run_test_plan.py”。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11 / 第 1-11 行

```python
# tests/test_run_test_plan.py
from __future__ import annotations

import importlib
import json
from contextlib import nullcontext
from types import SimpleNamespace
from unittest.mock import MagicMock, patch as mock_patch

import pytest

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 12-22 / 第 12-22 行

```python

MOD = "cli.lib.core.vllm.lib"

# We import inside tests so the MOD override above applies everywhere
run_test_plan_import_path = f"{MOD}.run_test_plan"


def _get_cmd(c):
    # Support both kwargs and positional args
    return c.kwargs.get("cmd", c.args[0] if c.args else None)

```

- **EN:** Important local symbols in this block include _get_cmd.
- **CN:** 该代码块中的重要局部符号包括 _get_cmd。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 23-39 / 第 23-39 行

```python

def _get_check(c):
    if "check" in c.kwargs:
        return c.kwargs["check"]
    # If positional, assume second arg is 'check' when present; default False
    return c.args[1] if len(c.args) > 1 else False


@pytest.fixture
def patch_module(monkeypatch):
    """
    Patch helpers ('pip_install_packages', 'temp_environ', 'working_directory',
    'run_command', 'logger', '_load_disabled_vllm_tests') inside the target
    module and expose them.
    """
    module = importlib.import_module(MOD)

```

- **EN:** Important local symbols in this block include _get_check, patch_module.
- **CN:** 该代码块中的重要局部符号包括 _get_check、patch_module。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 40-51 / 第 40-51 行

```python
    # Create fakes/mocks
    pip_install_packages = MagicMock(name="pip_install_packages")
    run_command = MagicMock(name="run_command", return_value=0)
    disabled_mock = MagicMock(name="_load_disabled_vllm_tests", return_value=[])

    # temp_environ / working_directory: record calls but act as context managers
    temp_calls: list[dict] = []
    workdir_calls: list[str] = []

    def fake_working_directory(path: str):
        workdir_calls.append(path)
        return nullcontext()
```

- **EN:** Important local symbols in this block include fake_working_directory.
- **CN:** 该代码块中的重要局部符号包括 fake_working_directory。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 52-62 / 第 52-62 行

```python

    def fake_temp_env(map: dict[str, str]):
        temp_calls.append(map)
        return nullcontext()

    logger = SimpleNamespace(
        info=MagicMock(name="logger.info"),
        warning=MagicMock(name="logger.warning"),
        error=MagicMock(name="logger.error"),
    )

```

- **EN:** Important local symbols in this block include fake_temp_env.
- **CN:** 该代码块中的重要局部符号包括 fake_temp_env。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 63-76 / 第 63-76 行

```python
    # Apply patches (raise if attribute doesn't exist)
    monkeypatch.setattr(
        module, "pip_install_packages", pip_install_packages, raising=True
    )
    monkeypatch.setattr(module, "run_command", run_command, raising=True)
    monkeypatch.setattr(
        module, "working_directory", fake_working_directory, raising=True
    )
    monkeypatch.setattr(module, "temp_environ", fake_temp_env, raising=True)
    monkeypatch.setattr(module, "logger", logger, raising=True)
    monkeypatch.setattr(
        module, "_load_disabled_vllm_tests", disabled_mock, raising=True
    )

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 77-87 / 第 77-87 行

```python
    return SimpleNamespace(
        module=module,
        run_test_plan=module.run_test_plan,  # expose to avoid getattr("constant") (Ruff B009)
        pip_install_packages=pip_install_packages,
        run_command=run_command,
        temp_calls=temp_calls,
        workdir_calls=workdir_calls,
        logger=logger,
        disabled_mock=disabled_mock,
    )

```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 88-104 / 第 88-104 行

```python

def test_success_runs_all_steps_and_uses_env_and_workdir(monkeypatch, patch_module):
    run_test_plan = patch_module.run_test_plan

    tests_map = {
        "basic": {
            "title": "Basic suite",
            "package_install": [],
            "working_directory": "tests",
            "env_vars": {"GLOBAL_FLAG": "1"},
            "steps": [
                "export A=x && pytest -q",
                "export B=y && pytest -q tests/unit",
            ],
        }
    }

```

- **EN:** Important local symbols in this block include test_success_runs_all_steps_and_uses_env_and_workdir.
- **CN:** 该代码块中的重要局部符号包括 test_success_runs_all_steps_and_uses_env_and_workdir。

### Lines 105-116 / 第 105-116 行

```python
    # One exit code per step (export + two pytest)
    patch_module.run_command.side_effect = [0, 0, 0]

    run_test_plan("basic", "cpu", tests_map)

    calls = patch_module.run_command.call_args_list
    cmds = [_get_cmd(c) for c in calls]
    checks = [_get_check(c) for c in calls]

    if len(cmds) != 2:
        raise AssertionError(f"Expected 2 commands, got {len(cmds)}: {cmds}")
    if "pytest" not in cmds[0] or "pytest" not in cmds[1]:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 117-129 / 第 117-129 行

```python
        raise AssertionError(f"Expected pytest in both commands, got {cmds}")
    if not all(chk is False for chk in checks):
        raise AssertionError(f"Expected all checks to be False, got checks={checks}")

    if patch_module.workdir_calls != ["tests"]:
        raise AssertionError(
            f"Expected workdir_calls=['tests'], got {patch_module.workdir_calls}"
        )
    if patch_module.temp_calls != [{"GLOBAL_FLAG": "1"}]:
        raise AssertionError(
            f"Expected temp_calls=[{{'GLOBAL_FLAG': '1'}}], got {patch_module.temp_calls}"
        )

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 130-141 / 第 130-141 行

```python

def test_installs_packages_when_present(monkeypatch, patch_module):
    run_test_plan = patch_module.module.run_test_plan

    tests_map = {
        "with_pkgs": {
            "title": "Needs deps",
            "package_install": ["timm==1.0.0", "flash-attn"],
            "steps": ["pytest -q"],
        }
    }

```

- **EN:** Important local symbols in this block include test_installs_packages_when_present.
- **CN:** 该代码块中的重要局部符号包括 test_installs_packages_when_present。

### Lines 142-152 / 第 142-152 行

```python
    patch_module.run_command.return_value = 0

    run_test_plan("with_pkgs", "gpu", tests_map)

    patch_module.pip_install_packages.assert_called_once_with(
        packages=["timm==1.0.0", "flash-attn"],
        prefer_uv=True,
    )


def test_raises_on_missing_plan(patch_module):
```

- **EN:** Important local symbols in this block include test_raises_on_missing_plan.
- **CN:** 该代码块中的重要局部符号包括 test_raises_on_missing_plan。

### Lines 153-163 / 第 153-163 行

```python
    run_test_plan = patch_module.module.run_test_plan
    with pytest.raises(RuntimeError) as ei:
        run_test_plan("nope", "cpu", tests_map={})

    if "test nope not found" not in str(ei.value):
        raise AssertionError(
            f"Expected 'test nope not found' in error, got: {ei.value}"
        )


def test_aggregates_failures_and_raises(monkeypatch, patch_module):
```

- **EN:** Important local symbols in this block include test_aggregates_failures_and_raises.
- **CN:** 该代码块中的重要局部符号包括 test_aggregates_failures_and_raises。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 164-176 / 第 164-176 行

```python
    run_test_plan = patch_module.module.run_test_plan

    tests_map = {
        "mix": {
            "title": "Some pass some fail",
            "steps": [
                "pytest test_a.py",  # 0 → pass
                "pytest test_b.py",  # 1 → fail
                "pytest test_c.py",  # 2 → fail
            ],
        }
    }

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 177-189 / 第 177-189 行

```python
    # Simulate pass, fail, fail
    patch_module.run_command.side_effect = [0, 1, 2]

    with pytest.raises(RuntimeError) as ei:
        run_test_plan("mix", "cpu", tests_map)

    msg = str(ei.value)
    if "2 pytest runs failed" not in msg:
        raise AssertionError(f"Expected '2 pytest runs failed' in error, got: {msg}")
    # Ensure logger captured failed tests list
    patch_module.logger.error.assert_called_once()
    # And we attempted all three commands
    if patch_module.run_command.call_count != 3:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 190-205 / 第 190-205 行

```python
        raise AssertionError(
            f"Expected run_command.call_count=3, got {patch_module.run_command.call_count}"
        )


def test_custom_working_directory_used(patch_module):
    run_test_plan = patch_module.module.run_test_plan

    tests_map = {
        "customwd": {
            "title": "Custom wd",
            "working_directory": "examples/ci",
            "steps": ["pytest -q"],
        }
    }

```

- **EN:** Important local symbols in this block include test_custom_working_directory_used.
- **CN:** 该代码块中的重要局部符号包括 test_custom_working_directory_used。

### Lines 206-216 / 第 206-216 行

```python
    patch_module.run_command.return_value = 0
    run_test_plan("customwd", "cpu", tests_map)

    if patch_module.workdir_calls != ["examples/ci"]:
        raise AssertionError(
            f"Expected workdir_calls=['examples/ci'], got {patch_module.workdir_calls}"
        )


# -- Disabled vLLM test injection (integration tests) -------------------------

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 217-227 / 第 217-227 行

```python
_SIMPLE_TESTS_MAP = {
    "plan_a": {
        "title": "Plan A",
        "steps": ["pytest -v -s test_foo.py", "pytest -v -s test_bar.py"],
    }
}


def _cmds(patch_module):
    return [_get_cmd(c) for c in patch_module.run_command.call_args_list]

```

- **EN:** Important local symbols in this block include _cmds.
- **CN:** 该代码块中的重要局部符号包括 _cmds。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 228-239 / 第 228-239 行

```python

def test_deselect_injected_for_test_level(patch_module):
    """Test-level node IDs (with ::) produce --deselect."""
    patch_module.disabled_mock.return_value = [
        {
            "test": "test_foo.py::test_x",
            "issue": "https://github.com/pytorch/pytorch/issues/175899",
        }
    ]
    patch_module.run_test_plan("plan_a", "cpu", _SIMPLE_TESTS_MAP)
    cmds = _cmds(patch_module)
    if not any("--deselect=test_foo.py::test_x" in c for c in cmds):
```

- **EN:** Important local symbols in this block include test_deselect_injected_for_test_level.
- **CN:** 该代码块中的重要局部符号包括 test_deselect_injected_for_test_level。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 240-255 / 第 240-255 行

```python
        raise AssertionError(
            f"Expected --deselect=test_foo.py::test_x in commands, got {cmds}"
        )


def test_ignore_injected_for_file_level(patch_module):
    """File-level node IDs (no ::) produce --ignore."""
    patch_module.disabled_mock.return_value = [
        {
            "test": "test_foo.py",
            "issue": "https://github.com/pytorch/pytorch/issues/175899",
        }
    ]
    patch_module.run_test_plan("plan_a", "cpu", _SIMPLE_TESTS_MAP)
    cmds = _cmds(patch_module)
    if not any("--ignore=test_foo.py" in c for c in cmds):
```

- **EN:** Important local symbols in this block include test_ignore_injected_for_file_level.
- **CN:** 该代码块中的重要局部符号包括 test_ignore_injected_for_file_level。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 256-266 / 第 256-266 行

```python
        raise AssertionError(f"Expected --ignore=test_foo.py in commands, got {cmds}")


def test_empty_disabled_list_no_modification(patch_module):
    """No flags injected when disabled list is empty."""
    patch_module.run_test_plan("plan_a", "cpu", _SIMPLE_TESTS_MAP)
    for cmd in _cmds(patch_module):
        if "--ignore" in cmd or "--deselect" in cmd:
            raise AssertionError(f"Expected no --ignore/--deselect flags, got {cmd}")


```

- **EN:** Important local symbols in this block include test_empty_disabled_list_no_modification.
- **CN:** 该代码块中的重要局部符号包括 test_empty_disabled_list_no_modification。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 267-283 / 第 267-283 行

```python
def test_non_pytest_steps_not_modified(patch_module):
    """Only pytest steps get disabled flags; other commands are left alone."""
    tests_map = {
        "mixed_steps": {
            "title": "Mixed",
            "steps": ["echo hello", "pytest -v test.py"],
        }
    }
    patch_module.disabled_mock.return_value = [
        {
            "test": "test.py::test_z",
            "issue": "https://github.com/pytorch/pytorch/issues/175899",
        }
    ]
    patch_module.run_test_plan("mixed_steps", "cpu", tests_map)
    cmds = _cmds(patch_module)
    if "--deselect" in cmds[0]:
```

- **EN:** Important local symbols in this block include test_non_pytest_steps_not_modified.
- **CN:** 该代码块中的重要局部符号包括 test_non_pytest_steps_not_modified。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 284-302 / 第 284-302 行

```python
        raise AssertionError(
            f"Non-pytest step should not have --deselect, got {cmds[0]}"
        )
    if "--deselect=test.py::test_z" not in cmds[1]:
        raise AssertionError(
            f"Expected --deselect=test.py::test_z in pytest step, got {cmds[1]}"
        )


def test_disabled_and_rerun_flags_both_present(patch_module):
    """Disabled flags and rerun flags compose into a single pytest invocation."""
    patch_module.disabled_mock.return_value = [
        {
            "test": "skip.py::test_x",
            "issue": "https://github.com/pytorch/pytorch/issues/175899",
        }
    ]
    patch_module.run_test_plan("plan_a", "cpu", _SIMPLE_TESTS_MAP)
    for cmd in _cmds(patch_module):
```

- **EN:** Important local symbols in this block include test_disabled_and_rerun_flags_both_present.
- **CN:** 该代码块中的重要局部符号包括 test_disabled_and_rerun_flags_both_present。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 303-313 / 第 303-313 行

```python
        if "--deselect=skip.py::test_x" not in cmd:
            raise AssertionError(f"Expected --deselect=skip.py::test_x, got {cmd}")
        if "--reruns" not in cmd:
            raise AssertionError(f"Expected --reruns in command, got {cmd}")
        if cmd.count("pytest") != 1:
            raise AssertionError(f"Expected exactly one 'pytest' token, got {cmd}")


# -- Disabled vLLM test helpers (unit tests) -----------------------------------


```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 314-324 / 第 314-324 行

```python
class FakeResponse:
    """Minimal fake for urllib.request.urlopen return value."""

    def __init__(self, body):
        self._data = json.dumps(body).encode()

    def read(self):
        return self._data

    def __enter__(self):
        return self
```

- **EN:** Important local symbols in this block include FakeResponse, __init__, read, __enter__.
- **CN:** 该代码块中的重要局部符号包括 FakeResponse、__init__、read、__enter__。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 325-342 / 第 325-342 行

```python

    def __exit__(self, *a):
        pass


@pytest.fixture
def vllm_module():
    return importlib.import_module(MOD)


@pytest.mark.parametrize(
    ("node_id", "expected_flag"),
    (
        ("a.py::test_1", "--deselect=a.py::test_1"),
        ("a.py", "--ignore=a.py"),
    ),
)
def test_build_flags_single_entry(vllm_module, node_id, expected_flag):
```

- **EN:** Important local symbols in this block include __exit__, vllm_module, test_build_flags_single_entry.
- **CN:** 该代码块中的重要局部符号包括 __exit__、vllm_module、test_build_flags_single_entry。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 343-358 / 第 343-358 行

```python
    """:: in node_id produces --deselect, otherwise --ignore."""
    flags = vllm_module._build_disabled_test_flags(
        [{"test": node_id, "issue": "url"}], "plan"
    )
    if flags != expected_flag:
        raise AssertionError(f"Expected '{expected_flag}', got '{flags}'")


def test_build_flags_config_filter(vllm_module):
    """Entries with matching configs are included; entries without configs apply to all."""
    entries = [
        {"test": "a.py", "issue": "url", "configs": ["plan_x"]},
        {"test": "b.py::test_1", "issue": "url"},
    ]
    flags = vllm_module._build_disabled_test_flags(entries, "plan_x")
    if "--ignore=a.py" not in flags:
```

- **EN:** Important local symbols in this block include test_build_flags_config_filter.
- **CN:** 该代码块中的重要局部符号包括 test_build_flags_config_filter。

### Lines 359-370 / 第 359-370 行

```python
        raise AssertionError(f"Expected '--ignore=a.py' in '{flags}'")
    if "--deselect=b.py::test_1" not in flags:
        raise AssertionError(f"Expected '--deselect=b.py::test_1' in '{flags}'")


def test_build_flags_config_excludes(vllm_module):
    """Entries with non-matching configs are excluded."""
    entries = [{"test": "a.py", "issue": "url", "configs": ["other"]}]
    flags = vllm_module._build_disabled_test_flags(entries, "plan_x")
    if flags != "":
        raise AssertionError(f"Expected empty flags, got '{flags}'")

```

- **EN:** Important local symbols in this block include test_build_flags_config_excludes.
- **CN:** 该代码块中的重要局部符号包括 test_build_flags_config_excludes。

### Lines 371-385 / 第 371-385 行

```python

def test_parse_issue_body(vllm_module):
    """Extracts disabled_tests from a ```yaml code block in issue body."""
    body = (
        "## Disabled vLLM Tests\n"
        "```yaml\n"
        "disabled_tests:\n"
        "  - test: foo.py::test_bar\n"
        "  - test: baz.py\n"
        "    configs:\n"
        "      - plan_a\n"
        "```\n"
    )
    entries = vllm_module._parse_disabled_tests_from_issue_body(body)
    if len(entries) != 2:
```

- **EN:** Important local symbols in this block include test_parse_issue_body.
- **CN:** 该代码块中的重要局部符号包括 test_parse_issue_body。

### Lines 386-398 / 第 386-398 行

```python
        raise AssertionError(f"Expected 2 entries, got {len(entries)}")
    if entries[0]["test"] != "foo.py::test_bar":
        raise AssertionError(f"Expected 'foo.py::test_bar', got '{entries[0]['test']}'")
    if entries[1]["test"] != "baz.py":
        raise AssertionError(f"Expected 'baz.py', got '{entries[1]['test']}'")
    if entries[1]["configs"] != ["plan_a"]:
        raise AssertionError(f"Expected ['plan_a'], got {entries[1]['configs']}")


def test_parse_issue_body_no_yaml(vllm_module):
    """Returns [] when issue body has no yaml code block."""
    entries = vllm_module._parse_disabled_tests_from_issue_body("no yaml here")
    if entries != []:
```

- **EN:** Important local symbols in this block include test_parse_issue_body_no_yaml.
- **CN:** 该代码块中的重要局部符号包括 test_parse_issue_body_no_yaml。

### Lines 399-416 / 第 399-416 行

```python
        raise AssertionError(f"Expected [], got {entries}")


def test_load_yaml_valid_entries(tmp_path, vllm_module):
    """Loads and validates entries from a well-formed YAML file."""
    yaml_file = tmp_path / "disabled.yaml"
    yaml_file.write_text(
        "disabled_tests:\n"
        "  - test: a.py\n"
        "    issue: https://github.com/pytorch/pytorch/issues/1\n"
        "  - test: b.py::test_x\n"
        "    issue: https://github.com/pytorch/pytorch/issues/2\n"
        "    configs:\n"
        "      - plan_a\n"
    )
    with mock_patch.object(vllm_module, "_DISABLED_VLLM_TESTS_PATH", yaml_file):
        entries = vllm_module._load_disabled_vllm_tests_from_yaml()

```

- **EN:** Important local symbols in this block include test_load_yaml_valid_entries.
- **CN:** 该代码块中的重要局部符号包括 test_load_yaml_valid_entries。

### Lines 417-428 / 第 417-428 行

```python
    if len(entries) != 2:
        raise AssertionError(f"Expected 2 entries, got {len(entries)}")
    if entries[0] != {
        "test": "a.py",
        "issue": "https://github.com/pytorch/pytorch/issues/175899",
    }:
        raise AssertionError(f"Unexpected first entry: {entries[0]}")
    if entries[1]["test"] != "b.py::test_x":
        raise AssertionError(f"Expected 'b.py::test_x', got '{entries[1]['test']}'")
    if entries[1]["configs"] != ["plan_a"]:
        raise AssertionError(f"Expected ['plan_a'], got {entries[1]['configs']}")

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 429-439 / 第 429-439 行

```python

def test_load_yaml_missing_keys(tmp_path, vllm_module):
    """Raises ValueError when YAML entry is missing required 'issue' key."""
    yaml_file = tmp_path / "disabled.yaml"
    yaml_file.write_text("disabled_tests:\n  - test: foo.py\n")
    with mock_patch.object(vllm_module, "_DISABLED_VLLM_TESTS_PATH", yaml_file):
        with pytest.raises(ValueError, match="must have 'test' and 'issue' keys"):
            vllm_module._load_disabled_vllm_tests_from_yaml()


def test_load_yaml_missing_file(tmp_path, vllm_module):
```

- **EN:** Important local symbols in this block include test_load_yaml_missing_keys, test_load_yaml_missing_file.
- **CN:** 该代码块中的重要局部符号包括 test_load_yaml_missing_keys、test_load_yaml_missing_file。

### Lines 440-453 / 第 440-453 行

```python
    """Returns [] when YAML file doesn't exist."""
    with mock_patch.object(
        vllm_module, "_DISABLED_VLLM_TESTS_PATH", tmp_path / "nope.yaml"
    ):
        result = vllm_module._load_disabled_vllm_tests_from_yaml()
    if result != []:
        raise AssertionError(f"Expected [], got {result}")


def test_load_github_skipped_when_issue_unset(vllm_module):
    """Returns [] immediately when _DISABLED_VLLM_TESTS_ISSUE is 0 (not configured)."""
    with mock_patch.object(vllm_module, "_DISABLED_VLLM_TESTS_ISSUE", 0):
        result = vllm_module._load_disabled_vllm_tests_from_github()
    if result != []:
```

- **EN:** Important local symbols in this block include test_load_github_skipped_when_issue_unset.
- **CN:** 该代码块中的重要局部符号包括 test_load_github_skipped_when_issue_unset。

### Lines 454-468 / 第 454-468 行

```python
        raise AssertionError(f"Expected [], got {result}")


def test_load_github_failure_returns_empty(vllm_module):
    """Network errors are swallowed and return []."""

    def _raise(*args, **kwargs):
        raise OSError("network error")

    with (
        mock_patch.object(vllm_module, "_DISABLED_VLLM_TESTS_ISSUE", 175899),
        mock_patch.object(vllm_module.urllib.request, "urlopen", _raise),
    ):
        result = vllm_module._load_disabled_vllm_tests_from_github()
    if result != []:
```

- **EN:** Important local symbols in this block include test_load_github_failure_returns_empty, _raise.
- **CN:** 该代码块中的重要局部符号包括 test_load_github_failure_returns_empty、_raise。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 469-487 / 第 469-487 行

```python
        raise AssertionError(f"Expected [], got {result}")


def test_load_github_filters_malformed_entries(vllm_module):
    """Entries without 'test' key are filtered out; issue URL is auto-filled."""
    fake_resp = FakeResponse(
        {
            "body": "```yaml\ndisabled_tests:\n  - bad_key: oops\n  - test: good.py\n```",
            "html_url": "https://github.com/pytorch/pytorch/issues/175899",
        }
    )
    with (
        mock_patch.object(vllm_module, "_DISABLED_VLLM_TESTS_ISSUE", 175899),
        mock_patch.object(
            vllm_module.urllib.request, "urlopen", lambda *a, **kw: fake_resp
        ),
    ):
        entries = vllm_module._load_disabled_vllm_tests_from_github()

```

- **EN:** Important local symbols in this block include test_load_github_filters_malformed_entries.
- **CN:** 该代码块中的重要局部符号包括 test_load_github_filters_malformed_entries。

### Lines 488-509 / 第 488-509 行

```python
    if len(entries) != 1:
        raise AssertionError(f"Expected 1 entry, got {len(entries)}")
    if entries[0]["test"] != "good.py":
        raise AssertionError(f"Expected 'good.py', got '{entries[0]['test']}'")
    if entries[0]["issue"] != "https://github.com/pytorch/pytorch/issues/175899":
        raise AssertionError(f"Expected issue URL, got '{entries[0]['issue']}'")


def test_deduplication_yaml_wins(vllm_module):
    """YAML entries take precedence over GitHub entries with the same test key."""
    yaml_entries = [{"test": "a.py", "issue": "yaml-url"}]
    github_entries = [
        {"test": "a.py", "issue": "github-url"},
        {"test": "b.py::test_1", "issue": "github-url"},
    ]
    with (
        mock_patch.object(
            vllm_module,
            "_load_disabled_vllm_tests_from_yaml",
            return_value=yaml_entries,
        ),
        mock_patch.object(
```

- **EN:** Important local symbols in this block include test_deduplication_yaml_wins.
- **CN:** 该代码块中的重要局部符号包括 test_deduplication_yaml_wins。

### Lines 510-521 / 第 510-521 行

```python
            vllm_module,
            "_load_disabled_vllm_tests_from_github",
            return_value=github_entries,
        ),
    ):
        merged = vllm_module._load_disabled_vllm_tests()

    if len(merged) != 2:
        raise AssertionError(f"Expected 2 merged entries, got {len(merged)}")
    if merged[0] != {"test": "a.py", "issue": "yaml-url"}:
        raise AssertionError(f"Expected yaml entry to win, got {merged[0]}")
    if merged[1] != {"test": "b.py::test_1", "issue": "github-url"}:
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 522-522 / 第 522-522 行

```python
        raise AssertionError(f"Unexpected second entry: {merged[1]}")
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **Declarative configuration** — 以声明式格式表示构建或工作流设置。
- **Structured metadata** — 存放机器可读的设置或清单式元数据。
- **Representative symbols: FakeResponse, _get_cmd, _get_check, patch_module, fake_working_directory, fake_temp_env, test_success_runs_all_steps_and_uses_env_and_workdir, test_installs_packages_when_present** — 代表性符号：FakeResponse、_get_cmd、_get_check、patch_module、fake_working_directory、fake_temp_env、test_success_runs_all_steps_and_uses_env_and_workdir、test_installs_packages_when_present

## Dependencies / 依赖关系

- `__future__`
- `importlib`
- `json`
- `contextlib`
- `types`
- `unittest.mock`
- `pytest`
