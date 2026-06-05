# label_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `.github/scripts/label_utils.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements repository automation helpers used by CI, release engineering, or GitHub maintenance workflows.
- **用途 (CN)**: 实现仓库自动化辅助脚本，供 CI、发布工程或 GitHub 维护流程使用。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9 / 第 1-9 行

````python
"""GitHub Label Utilities."""

from __future__ import annotations

import json
from functools import lru_cache
from typing import Any, TYPE_CHECKING

from github_utils import gh_fetch_url_and_headers, GitHubComment
````

- EN: This block imports dependencies such as `__future__`, `json`, `functools`, `typing`; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块导入当前模块运行所需的依赖；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 12-23 / 第 12-23 行

````python
# TODO: this is a temp workaround to avoid circular dependencies,
#       and should be removed once GitHubPR is refactored out of trymerge script.
if TYPE_CHECKING:
    from trymerge import GitHubPR

BOT_AUTHORS = ["github-actions", "pytorchmergebot", "pytorch-bot"]

LABEL_ERR_MSG_TITLE = "This PR needs a `release notes:` label"
LABEL_ERR_MSG = f"""# {LABEL_ERR_MSG_TITLE}
If your changes are user facing and intended to be a part of release notes, please use a label starting with `release notes:`.

If not, please add the `topic: not user facing` label.
````

- EN: This block imports dependencies such as `trymerge`; branches on runtime conditions or configuration choices; touches GitHub-related data or remote service interactions.
- CN: 该代码块导入当前模块运行所需的依赖；依据运行时条件或配置选择进行分支；处理 GitHub 相关数据或远程服务交互。

### Lines 25-37 / 第 25-37 行

````python
To add a label, you can comment to pytorchbot, for example
`@pytorchbot label "topic: not user facing"`

For more information, see
https://github.com/pytorch/pytorch/wiki/PyTorch-AutoLabel-Bot#why-categorize-for-release-notes-and-how-does-it-work.
"""


def request_for_labels(url: str) -> tuple[Any, Any]:
    headers = {"Accept": "application/vnd.github.v3+json"}
    return gh_fetch_url_and_headers(
        url, headers=headers, reader=lambda x: x.read().decode("utf-8")
    )
````

- EN: This block defines callable units such as `request_for_labels`; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 40-53 / 第 40-53 行

````python
def update_labels(labels: list[str], info: str) -> None:
    labels_json = json.loads(info)
    labels.extend([x["name"] for x in labels_json])


def get_last_page_num_from_header(header: Any) -> int:
    # Link info looks like: <https://api.github.com/repositories/65600975/labels?per_page=100&page=2>;
    # rel="next", <https://api.github.com/repositories/65600975/labels?per_page=100&page=3>; rel="last"
    link_info = header["link"]
    # Docs does not specify that it should be present for projects with just few labels
    # And https://github.com/malfet/deleteme/actions/runs/7334565243/job/19971396887 it's not the case  # @lint-ignore
    if link_info is None:
        return 1
    prefix = "&page="
````

- EN: This block defines callable units such as `update_labels`, `get_last_page_num_from_header`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 54-65 / 第 54-65 行

````python
    suffix = ">;"
    return int(
        link_info[link_info.rindex(prefix) + len(prefix) : link_info.rindex(suffix)]
    )


@lru_cache
def gh_get_labels(org: str, repo: str) -> list[str]:
    prefix = f"https://api.github.com/repos/{org}/{repo}/labels?per_page=100"
    header, info = request_for_labels(prefix + "&page=1")
    labels: list[str] = []
    update_labels(labels, info)
````

- EN: This block defines callable units such as `gh_get_labels`; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 67-76 / 第 67-76 行

````python
    last_page = get_last_page_num_from_header(header)
    if last_page <= 0:
        raise AssertionError(
            f"Error reading header info to determine total number of pages of labels, got last_page={last_page}"
        )
    for page_number in range(2, last_page + 1):  # skip page 1
        _, info = request_for_labels(prefix + f"&page={page_number}")
        update_labels(labels, info)

    return labels
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; validates assumptions and surfaces failures explicitly.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；校验前提条件并显式暴露失败。

### Lines 79-88 / 第 79-88 行

````python
def gh_add_labels(
    org: str, repo: str, pr_num: int, labels: str | list[str], dry_run: bool
) -> None:
    if dry_run:
        print(f"Dryrun: Adding labels {labels} to PR {pr_num}")
        return
    gh_fetch_url_and_headers(
        url=f"https://api.github.com/repos/{org}/{repo}/issues/{pr_num}/labels",
        data={"labels": labels},
    )
````

- EN: This block defines callable units such as `gh_add_labels`; branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 91-100 / 第 91-100 行

````python
def gh_remove_label(
    org: str, repo: str, pr_num: int, label: str, dry_run: bool
) -> None:
    if dry_run:
        print(f"Dryrun: Removing {label} from PR {pr_num}")
        return
    gh_fetch_url_and_headers(
        url=f"https://api.github.com/repos/{org}/{repo}/issues/{pr_num}/labels/{label}",
        method="DELETE",
    )
````

- EN: This block defines callable units such as `gh_remove_label`; branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 103-116 / 第 103-116 行

````python
def get_release_notes_labels(org: str, repo: str) -> list[str]:
    return [
        label
        for label in gh_get_labels(org, repo)
        if label.lstrip().startswith("release notes:")
    ]


def has_required_labels(pr: GitHubPR) -> bool:
    pr_labels = pr.get_labels()
    # Check if PR is not user facing
    is_not_user_facing_pr = any(
        label.strip() == "topic: not user facing" for label in pr_labels
    )
````

- EN: This block defines callable units such as `get_release_notes_labels`, `has_required_labels`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 117-129 / 第 117-129 行

````python
    return is_not_user_facing_pr or any(
        label.strip() in get_release_notes_labels(pr.org, pr.project)
        for label in pr_labels
    )


def is_label_err_comment(comment: GitHubComment) -> bool:
    # comment.body_text returns text without markdown
    no_format_title = LABEL_ERR_MSG_TITLE.replace("`", "")
    return (
        comment.body_text.lstrip(" #").startswith(no_format_title)
        and comment.author_login in BOT_AUTHORS
    )
````

- EN: This block defines callable units such as `is_label_err_comment`; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。


## Key Concepts / 关键概念
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `__future__`, `json`, `functools`, `typing`
- Internal imports / 内部导入: none
- External imports / 外部导入: `github_utils`
- Classes / 类: none
- Functions / 函数: `request_for_labels`, `update_labels`, `get_last_page_num_from_header`, `gh_get_labels`, `gh_add_labels`, `gh_remove_label`, `get_release_notes_labels`, `has_required_labels`, `is_label_err_comment`
- Test entry points / 测试入口: none
