# ensure_actions_will_cancel.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `.github/scripts/ensure_actions_will_cancel.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements repository automation helpers used by CI, release engineering, or GitHub maintenance workflows.
- **用途 (CN)**: 实现仓库自动化辅助脚本，供 CI、发布工程或 GitHub 维护流程使用。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行

````python
#!/usr/bin/env python3

import sys
from pathlib import Path

import yaml


REPO_ROOT = Path(__file__).resolve().parents[2]
WORKFLOWS = REPO_ROOT / ".github" / "workflows"
EXPECTED_GROUP_PREFIX = (
    "${{ github.workflow }}-${{ github.event.pull_request.number || github.sha }}"
)
# Standard pattern - dispatches in same concurrency group will cancel each other
````

- EN: This block imports dependencies such as `sys`, `pathlib`, `yaml`; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块导入当前模块运行所需的依赖；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 15-27 / 第 15-27 行

````python
EXPECTED_GROUP_STANDARD = (
    EXPECTED_GROUP_PREFIX + "-${{ github.event_name == 'workflow_dispatch' }}"
)
# Concurrent dispatch pattern - uses run_id to allow concurrent dispatches (e.g., from autorevert bot)
EXPECTED_GROUP_CONCURRENT = (
    EXPECTED_GROUP_PREFIX
    + "-${{ github.event_name == 'workflow_dispatch' && github.run_id }}"
)


def should_check(filename: Path) -> bool:
    with open(filename) as f:
        content = f.read()
````

- EN: This block defines callable units such as `should_check`; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互。

### Lines 29-40 / 第 29-40 行

````python
    data = yaml.safe_load(content)
    on = data.get("on", data.get(True, {}))
    return "pull_request" in on


if __name__ == "__main__":
    errors_found = False
    files = [f for f in WORKFLOWS.glob("*.yml") if should_check(f)]
    names = set()
    for filename in files:
        with open(filename) as f:
            data = yaml.safe_load(f)
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; serializes or parses structured data files.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；序列化或解析结构化数据文件。

### Lines 42-55 / 第 42-55 行

````python
        name = data.get("name")
        if name is not None and name in names:
            print("ERROR: duplicate workflow name:", name, file=sys.stderr)
            errors_found = True
        names.add(name)
        actual = data.get("concurrency", {})
        if filename.name == "create_release.yml":
            if not actual.get("group", "").startswith(EXPECTED_GROUP_PREFIX):
                print(
                    f"'concurrency' incorrect or not found in '{filename.relative_to(REPO_ROOT)}'",
                    file=sys.stderr,
                )
                print(
                    f"concurrency group should start with {EXPECTED_GROUP_PREFIX} but found {actual.get('group', None)}",
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。

### Lines 56-69 / 第 56-69 行

````python
                    file=sys.stderr,
                )
                errors_found = True
        elif not (
            actual.get("group", "").startswith(EXPECTED_GROUP_STANDARD)
            or actual.get("group", "").startswith(EXPECTED_GROUP_CONCURRENT)
        ):
            print(
                f"'concurrency' incorrect or not found in '{filename.relative_to(REPO_ROOT)}'",
                file=sys.stderr,
            )
            print(
                f"concurrency group should start with {EXPECTED_GROUP_STANDARD} "
                f"or {EXPECTED_GROUP_CONCURRENT} but found {actual.get('group', None)}",
````

- EN: This block implements local helper logic for ensure actions will cancel.
- CN: 该代码块实现与 ensure actions will cancel 相关的局部辅助逻辑。

### Lines 70-81 / 第 70-81 行

````python
                file=sys.stderr,
            )
            errors_found = True
        if not actual.get("cancel-in-progress", False):
            print(
                f"'concurrency' incorrect or not found in '{filename.relative_to(REPO_ROOT)}'",
                file=sys.stderr,
            )
            print(
                f"concurrency cancel-in-progress should be True but found {actual.get('cancel-in-progress', None)}",
                file=sys.stderr,
            )
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。

### Lines 83-84 / 第 83-84 行

````python
    if errors_found:
        sys.exit(1)
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `sys`, `pathlib`
- Internal imports / 内部导入: none
- External imports / 外部导入: `yaml`
- Classes / 类: none
- Functions / 函数: `should_check`
- Test entry points / 测试入口: none
