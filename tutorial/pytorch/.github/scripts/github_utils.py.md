# github_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `.github/scripts/github_utils.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements repository automation helpers used by CI, release engineering, or GitHub maintenance workflows.
- **用途 (CN)**: 实现仓库自动化辅助脚本，供 CI、发布工程或 GitHub 维护流程使用。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16 / 第 1-16 行

````python
"""GitHub Utilities"""

from __future__ import annotations

import json
import os
import warnings
from dataclasses import dataclass
from typing import Any, cast, TYPE_CHECKING
from urllib.error import HTTPError
from urllib.parse import quote
from urllib.request import Request, urlopen


if TYPE_CHECKING:
    from collections.abc import Callable
````

- EN: This block imports dependencies such as `__future__`, `json`, `os`, `warnings`; branches on runtime conditions or configuration choices; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块导入当前模块运行所需的依赖；依据运行时条件或配置选择进行分支；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 19-31 / 第 19-31 行

````python
GITHUB_API_URL = "https://api.github.com"


@dataclass
class GitHubComment:
    body_text: str
    created_at: str
    author_login: str
    author_url: str | None
    author_association: str
    editor_login: str | None
    database_id: int
    url: str
````

- EN: This block introduces classes like `GitHubComment`; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义新的类来封装状态与行为；处理 GitHub 相关数据或远程服务交互。

### Lines 34-50 / 第 34-50 行

````python
def gh_fetch_url_and_headers(
    url: str,
    *,
    headers: dict[str, str] | None = None,
    data: dict[str, Any] | None | str = None,
    method: str | None = None,
    reader: Callable[[Any], Any] = lambda x: x.read(),
) -> tuple[Any, Any]:
    if headers is None:
        headers = {}
    token = os.environ.get("GITHUB_TOKEN")
    if token is not None and url.startswith(f"{GITHUB_API_URL}/"):
        headers["Authorization"] = f"token {token}"

    data_ = None
    if data is not None:
        data_ = data.encode() if isinstance(data, str) else json.dumps(data).encode()
````

- EN: This block defines callable units such as `gh_fetch_url_and_headers`; branches on runtime conditions or configuration choices; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 52-69 / 第 52-69 行

````python
    try:
        with urlopen(Request(url, headers=headers, data=data_, method=method)) as conn:
            return conn.headers, reader(conn)
    except HTTPError as err:
        if (
            err.code == 403
            and all(
                key in err.headers
                for key in ["X-RateLimit-Limit", "X-RateLimit-Remaining"]
            )
            and int(err.headers["X-RateLimit-Remaining"]) == 0
        ):
            print(
                f"""{url}
                Rate limit exceeded:
                Used: {err.headers["X-RateLimit-Used"]}
                Limit: {err.headers["X-RateLimit-Limit"]}
                Remaining: {err.headers["X-RateLimit-Remaining"]}
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 70-87 / 第 70-87 行

````python
                Resets at: {err.headers["x-RateLimit-Reset"]}"""
            )
        else:
            print(f"Error fetching {url} {err}")
        raise


def gh_fetch_url(
    url: str,
    *,
    headers: dict[str, str] | None = None,
    data: dict[str, Any] | None | str = None,
    method: str | None = None,
    reader: Callable[[Any], Any] = json.load,
) -> Any:
    return gh_fetch_url_and_headers(
        url, headers=headers, data=data, reader=reader, method=method
    )[1]
````

- EN: This block defines callable units such as `gh_fetch_url`; returns computed values to callers; validates assumptions and surfaces failures explicitly; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果；校验前提条件并显式暴露失败；序列化或解析结构化数据文件。

### Lines 90-104 / 第 90-104 行

````python
def gh_fetch_json(
    url: str,
    params: dict[str, Any] | None = None,
    data: dict[str, Any] | None = None,
    method: str | None = None,
) -> list[dict[str, Any]]:
    headers = {"Accept": "application/vnd.github.v3+json"}
    if params is not None and len(params) > 0:
        url += "?" + "&".join(
            f"{name}={quote(str(val))}" for name, val in params.items()
        )
    return cast(
        list[dict[str, Any]],
        gh_fetch_url(url, headers=headers, data=data, reader=json.load, method=method),
    )
````

- EN: This block defines callable units such as `gh_fetch_json`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 107-117 / 第 107-117 行

````python
def _gh_fetch_json_any(
    url: str,
    params: dict[str, Any] | None = None,
    data: dict[str, Any] | None = None,
) -> Any:
    headers = {"Accept": "application/vnd.github.v3+json"}
    if params is not None and len(params) > 0:
        url += "?" + "&".join(
            f"{name}={quote(str(val))}" for name, val in params.items()
        )
    return gh_fetch_url(url, headers=headers, data=data, reader=json.load)
````

- EN: This block defines callable units such as `_gh_fetch_json_any`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 120-133 / 第 120-133 行

````python
def gh_fetch_json_list(
    url: str,
    params: dict[str, Any] | None = None,
    data: dict[str, Any] | None = None,
) -> list[dict[str, Any]]:
    return cast(list[dict[str, Any]], _gh_fetch_json_any(url, params, data))


def gh_fetch_json_dict(
    url: str,
    params: dict[str, Any] | None = None,
    data: dict[str, Any] | None = None,
) -> dict[str, Any]:
    return cast(dict[str, Any], _gh_fetch_json_any(url, params, data))
````

- EN: This block defines callable units such as `gh_fetch_json_list`, `gh_fetch_json_dict`; returns computed values to callers; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果；序列化或解析结构化数据文件。

### Lines 136-146 / 第 136-146 行

````python
def gh_graphql(query: str, **kwargs: Any) -> dict[str, Any]:
    rc = gh_fetch_url(
        "https://api.github.com/graphql",  # @lint-ignore
        data={"query": query, "variables": kwargs},
        reader=json.load,
    )
    if "errors" in rc:
        raise RuntimeError(
            f"GraphQL query {query}, args {kwargs} failed: {rc['errors']}"
        )
    return cast(dict[str, Any], rc)
````

- EN: This block defines callable units such as `gh_graphql`; branches on runtime conditions or configuration choices; returns computed values to callers; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 149-165 / 第 149-165 行

````python
def _gh_post_comment(
    url: str, comment: str, dry_run: bool = False
) -> list[dict[str, Any]]:
    if dry_run:
        print(comment)
        return []
    return gh_fetch_json_list(url, data={"body": comment})


def gh_post_pr_comment(
    org: str, repo: str, pr_num: int, comment: str, dry_run: bool = False
) -> list[dict[str, Any]]:
    return _gh_post_comment(
        f"{GITHUB_API_URL}/repos/{org}/{repo}/issues/{pr_num}/comments",
        comment,
        dry_run,
    )
````

- EN: This block defines callable units such as `_gh_post_comment`, `gh_post_pr_comment`; branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 168-183 / 第 168-183 行

````python
def gh_post_commit_comment(
    org: str, repo: str, sha: str, comment: str, dry_run: bool = False
) -> list[dict[str, Any]]:
    return _gh_post_comment(
        f"{GITHUB_API_URL}/repos/{org}/{repo}/commits/{sha}/comments",
        comment,
        dry_run,
    )


def gh_close_pr(org: str, repo: str, pr_num: int, dry_run: bool = False) -> None:
    url = f"{GITHUB_API_URL}/repos/{org}/{repo}/pulls/{pr_num}"
    if dry_run:
        print(f"Dry run closing PR {pr_num}")
    else:
        gh_fetch_url(url, method="PATCH", data={"state": "closed"})
````

- EN: This block defines callable units such as `gh_post_commit_comment`, `gh_close_pr`; branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 186-203 / 第 186-203 行

````python
def gh_delete_comment(org: str, repo: str, comment_id: int) -> None:
    url = f"{GITHUB_API_URL}/repos/{org}/{repo}/issues/comments/{comment_id}"
    gh_fetch_url(url, method="DELETE", reader=lambda x: x.read())


def gh_fetch_merge_base(org: str, repo: str, base: str, head: str) -> str:
    merge_base = ""
    # Get the merge base using the GitHub REST API. This is the same as using
    # git merge-base without the need to have git. The API doc can be found at
    # https://docs.github.com/en/rest/commits/commits?apiVersion=2022-11-28#compare-two-commits
    try:
        json_data = gh_fetch_url(
            f"{GITHUB_API_URL}/repos/{org}/{repo}/compare/{base}...{head}",
            headers={"Accept": "application/vnd.github.v3+json"},
            reader=json.load,
        )
        if json_data:
            merge_base = json_data.get("merge_base_commit", {}).get("sha", "")
````

- EN: This block defines callable units such as `gh_delete_comment`, `gh_fetch_merge_base`; branches on runtime conditions or configuration choices; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 204-221 / 第 204-221 行

````python
        else:
            warnings.warn(
                f"Failed to get merge base for {base}...{head}: Empty response"
            )
    except Exception as error:
        warnings.warn(f"Failed to get merge base for {base}...{head}: {error}")

    return merge_base


def gh_update_pr_state(org: str, repo: str, pr_num: int, state: str = "open") -> None:
    url = f"{GITHUB_API_URL}/repos/{org}/{repo}/pulls/{pr_num}"
    try:
        gh_fetch_url(url, method="PATCH", data={"state": state})
    except HTTPError as err:
        # When trying to open the pull request, error 422 means that the branch
        # has been deleted and the API couldn't re-open it
        if err.code == 422 and state == "open":
````

- EN: This block defines callable units such as `gh_update_pr_state`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 222-226 / 第 222-226 行

````python
            warnings.warn(
                f"Failed to open {pr_num} because its head branch has been deleted: {err}"
            )
        else:
            raise
````

- EN: This block validates assumptions and surfaces failures explicitly.
- CN: 该代码块校验前提条件并显式暴露失败。

### Lines 229-235 / 第 229-235 行

````python
def gh_query_issues_by_labels(
    org: str, repo: str, labels: list[str], state: str = "open"
) -> list[dict[str, Any]]:
    url = f"{GITHUB_API_URL}/repos/{org}/{repo}/issues"
    return gh_fetch_json(
        url, method="GET", params={"labels": ",".join(labels), "state": state}
    )
````

- EN: This block defines callable units such as `gh_query_issues_by_labels`; returns computed values to callers; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。


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
- Standard-library imports / 标准库导入: `__future__`, `json`, `os`, `warnings`, `dataclasses`, `typing`, `urllib.error`, `urllib.parse`, ...
- Internal imports / 内部导入: none
- External imports / 外部导入: none
- Classes / 类: `GitHubComment`
- Functions / 函数: `gh_fetch_url_and_headers`, `gh_fetch_url`, `gh_fetch_json`, `_gh_fetch_json_any`, `gh_fetch_json_list`, `gh_fetch_json_dict`, `gh_graphql`, `_gh_post_comment`, `gh_post_pr_comment`, `gh_post_commit_comment`, ...
- Test entry points / 测试入口: none
