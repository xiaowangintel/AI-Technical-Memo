# cherry_pick.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `.github/scripts/cherry_pick.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements repository automation helpers used by CI, release engineering, or GitHub maintenance workflows.
- **用途 (CN)**: 实现仓库自动化辅助脚本，供 CI、发布工程或 GitHub 维护流程使用。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13 / 第 1-13 行

````python
#!/usr/bin/env python3

from __future__ import annotations

import json
import os
import re
from typing import Any, cast
from urllib.error import HTTPError

from github_utils import gh_fetch_url, gh_post_pr_comment, gh_query_issues_by_labels
from gitutils import get_git_remote_name, get_git_repo_dir, GitRepo
from trymerge import get_pr_commit_sha, GitHubPR
````

- EN: This block imports dependencies such as `__future__`, `json`, `os`, `re`; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块导入当前模块运行所需的依赖；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 16-26 / 第 16-26 行

````python
# This is only a suggestion for now, not a strict requirement
REQUIRES_ISSUE = {
    "regression",
    "critical",
    "fixnewfeature",
}
RELEASE_BRANCH_REGEX = re.compile(r"release/(?P<version>.+)")


def parse_args() -> Any:
    from argparse import ArgumentParser
````

- EN: This block imports dependencies such as `argparse`; defines callable units such as `parse_args`; parses command-line arguments for script entry points; iterates through collections, records, or generated items.
- CN: 该代码块导入当前模块运行所需的依赖；定义可调用函数来执行具体自动化步骤；为脚本入口解析命令行参数；遍历集合、记录或生成项。

### Lines 28-45 / 第 28-45 行

````python
    parser = ArgumentParser("cherry pick a landed PR onto a release branch")
    parser.add_argument(
        "--onto-branch", type=str, required=True, help="the target release branch"
    )
    parser.add_argument(
        "--github-actor", type=str, required=True, help="all the world's a stage"
    )
    parser.add_argument(
        "--classification",
        choices=["regression", "critical", "fixnewfeature", "docs", "release"],
        required=True,
        help="the cherry pick category",
    )
    parser.add_argument("pr_num", type=int)
    parser.add_argument(
        "--fixes",
        type=str,
        default="",
````

- EN: This block parses command-line arguments for script entry points; touches GitHub-related data or remote service interactions.
- CN: 该代码块为脚本入口解析命令行参数；处理 GitHub 相关数据或远程服务交互。

### Lines 46-59 / 第 46-59 行

````python
        help="the GitHub issue that the cherry pick fixes",
    )
    parser.add_argument("--dry-run", action="store_true")

    return parser.parse_args()


def get_merge_commit_sha(repo: GitRepo, pr: GitHubPR) -> str | None:
    """
    Return the merge commit SHA iff the PR has been merged. For simplicity, we
    will only cherry pick PRs that have been merged into main
    """
    commit_sha = get_pr_commit_sha(repo, pr)
    return commit_sha if pr.is_closed() else None
````

- EN: This block defines callable units such as `get_merge_commit_sha`; parses command-line arguments for script entry points; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；为脚本入口解析命令行参数；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 62-79 / 第 62-79 行

````python
def get_release_version(onto_branch: str) -> str | None:
    """
    Return the release version if the target branch is a release branch
    """
    m = re.match(RELEASE_BRANCH_REGEX, onto_branch)
    return m.group("version") if m else ""


def get_tracker_issues(
    org: str, project: str, onto_branch: str
) -> list[dict[str, Any]]:
    """
    Find the tracker issue from the repo. The tracker issue needs to have the title
    like [VERSION] Release Tracker following the convention on PyTorch
    """
    version = get_release_version(onto_branch)
    if not version:
        return []
````

- EN: This block defines callable units such as `get_release_version`, `get_tracker_issues`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 81-98 / 第 81-98 行

````python
    tracker_issues = gh_query_issues_by_labels(org, project, labels=["release tracker"])
    if not tracker_issues:
        return []

    # Figure out the tracker issue from the list by looking at the title
    return [issue for issue in tracker_issues if version in issue.get("title", "")]


def cherry_pick(
    github_actor: str,
    repo: GitRepo,
    pr: GitHubPR,
    commit_sha: str,
    onto_branch: str,
    classification: str,
    fixes: str,
    dry_run: bool = False,
) -> None:
````

- EN: This block defines callable units such as `cherry_pick`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 99-112 / 第 99-112 行

````python
    """
    Create a local branch to cherry pick the commit and submit it as a pull request
    """
    current_branch = repo.current_branch()
    cherry_pick_branch = create_cherry_pick_branch(
        github_actor, repo, pr, commit_sha, onto_branch
    )

    try:
        org, project = repo.gh_owner_and_name()

        cherry_pick_pr = ""
        if not dry_run:
            cherry_pick_pr = submit_pr(repo, pr, cherry_pick_branch, onto_branch)
````

- EN: This block branches on runtime conditions or configuration choices; touches GitHub-related data or remote service interactions.
- CN: 该代码块依据运行时条件或配置选择进行分支；处理 GitHub 相关数据或远程服务交互。

### Lines 114-131 / 第 114-131 行

````python
        tracker_issues_comments = []
        tracker_issues = get_tracker_issues(org, project, onto_branch)
        for issue in tracker_issues:
            issue_number = int(str(issue.get("number", "0")))
            if not issue_number:
                continue

            res = cast(
                dict[str, Any],
                post_tracker_issue_comment(
                    org,
                    project,
                    issue_number,
                    pr.pr_num,
                    cherry_pick_pr,
                    classification,
                    fixes,
                    dry_run,
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 132-148 / 第 132-148 行

````python
                ),
            )

            comment_url = res.get("html_url", "")
            if comment_url:
                tracker_issues_comments.append(comment_url)

        msg = f"The cherry pick PR is at {cherry_pick_pr}"
        if fixes:
            msg += f" and it is linked with issue {fixes}."
        elif classification in REQUIRES_ISSUE:
            msg += f" and it is recommended to link a {classification} cherry pick PR with an issue."

        if tracker_issues_comments:
            msg += " The following tracker issues are updated:\n"
            for tracker_issues_comment in tracker_issues_comments:
                msg += f"* {tracker_issues_comment}\n"
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 150-165 / 第 150-165 行

````python
        post_pr_comment(org, project, pr.pr_num, msg, dry_run)

    finally:
        if current_branch:
            repo.checkout(branch=current_branch)


def create_cherry_pick_branch(
    github_actor: str, repo: GitRepo, pr: GitHubPR, commit_sha: str, onto_branch: str
) -> str:
    """
    Create a local branch and cherry pick the commit. Return the name of the local
    cherry picking branch.
    """
    repo.checkout(branch=onto_branch)
    repo._run_git("submodule", "update", "--init", "--recursive")
````

- EN: This block defines callable units such as `create_cherry_pick_branch`; branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 167-178 / 第 167-178 行

````python
    # Remove all special characters if we want to include the actor in the branch name
    github_actor = re.sub("[^0-9a-zA-Z]+", "_", github_actor)

    cherry_pick_branch = f"cherry-pick-{pr.pr_num}-by-{github_actor}"
    repo.create_branch_and_checkout(branch=cherry_pick_branch)

    # We might want to support ghstack later
    # We don't want to resolve conflicts here.
    repo._run_git("cherry-pick", "-x", commit_sha)
    repo.push(branch=cherry_pick_branch, dry_run=False)

    return cherry_pick_branch
````

- EN: This block branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 181-194 / 第 181-194 行

````python
def submit_pr(
    repo: GitRepo,
    pr: GitHubPR,
    cherry_pick_branch: str,
    onto_branch: str,
) -> str:
    """
    Submit the cherry pick PR and return the link to the PR
    """
    org, project = repo.gh_owner_and_name()

    default_msg = f"Cherry pick #{pr.pr_num} onto {onto_branch} branch"
    title = pr.info.get("title", default_msg)
    body = pr.info.get("body", default_msg)
````

- EN: This block defines callable units such as `submit_pr`; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 196-208 / 第 196-208 行

````python
    try:
        response = gh_fetch_url(
            f"https://api.github.com/repos/{org}/{project}/pulls",
            method="POST",
            data={
                "title": title,
                "body": body,
                "head": cherry_pick_branch,
                "base": onto_branch,
            },
            headers={"Accept": "application/vnd.github.v3+json"},
            reader=json.load,
        )
````

- EN: This block touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 210-220 / 第 210-220 行

````python
        cherry_pick_pr = response.get("html_url", "")
        if not cherry_pick_pr:
            raise RuntimeError(
                f"Fail to find the cherry pick PR: {json.dumps(response)}"
            )

        return str(cherry_pick_pr)

    except HTTPError as error:
        msg = f"Fail to submit the cherry pick PR: {error}"
        raise RuntimeError(msg) from error
````

- EN: This block branches on runtime conditions or configuration choices; returns computed values to callers; validates assumptions and surfaces failures explicitly; serializes or parses structured data files.
- CN: 该代码块依据运行时条件或配置选择进行分支；向调用方返回计算结果；校验前提条件并显式暴露失败；序列化或解析结构化数据文件。

### Lines 223-240 / 第 223-240 行

````python
def post_pr_comment(
    org: str, project: str, pr_num: int, msg: str, dry_run: bool = False
) -> list[dict[str, Any]]:
    """
    Post a comment on the PR itself to point to the cherry picking PR when success
    or print the error when failure
    """
    internal_debugging = ""

    run_url = os.getenv("GH_RUN_URL")
    # Post a comment to tell folks that the PR is being cherry picked
    if run_url is not None:
        internal_debugging = "\n".join(
            line
            for line in (
                "<details><summary>Details for Dev Infra team</summary>",
                f'Raised by <a href="{run_url}">workflow job</a>\n',
                "</details>",
````

- EN: This block defines callable units such as `post_pr_comment`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 241-258 / 第 241-258 行

````python
            )
            if line
        )

    comment = "\n".join(
        (f"### Cherry picking #{pr_num}", f"{msg}", "", f"{internal_debugging}")
    )
    return gh_post_pr_comment(org, project, pr_num, comment, dry_run)


def post_tracker_issue_comment(
    org: str,
    project: str,
    issue_num: int,
    pr_num: int,
    cherry_pick_pr: str,
    classification: str,
    fixes: str,
````

- EN: This block defines callable units such as `post_tracker_issue_comment`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 259-276 / 第 259-276 行

````python
    dry_run: bool = False,
) -> list[dict[str, Any]]:
    """
    Post a comment on the tracker issue (if any) to record the cherry pick
    """
    comment = "\n".join(
        (
            "Link to landed trunk PR (if applicable):",
            f"* https://github.com/{org}/{project}/pull/{pr_num}",
            "",
            "Link to release branch PR:",
            f"* {cherry_pick_pr}",
            "",
            "Criteria Category:",
            " - ".join((classification.capitalize(), fixes.capitalize())),
        )
    )
    return gh_post_pr_comment(org, project, issue_num, comment, dry_run)
````

- EN: This block branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 279-293 / 第 279-293 行

````python
def main() -> None:
    args = parse_args()
    pr_num = args.pr_num

    repo = GitRepo(get_git_repo_dir(), get_git_remote_name())
    org, project = repo.gh_owner_and_name()

    pr = GitHubPR(org, project, pr_num)

    try:
        commit_sha = get_merge_commit_sha(repo, pr)
        if not commit_sha:
            raise RuntimeError(
                f"Refuse to cherry pick #{pr_num} because it hasn't been merged yet"
            )
````

- EN: This block defines callable units such as `main`; branches on runtime conditions or configuration choices; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互。

### Lines 295-310 / 第 295-310 行

````python
        cherry_pick(
            args.github_actor,
            repo,
            pr,
            commit_sha,
            args.onto_branch,
            args.classification,
            args.fixes,
            args.dry_run,
        )

    except RuntimeError as error:
        if not args.dry_run:
            post_pr_comment(org, project, pr_num, str(error))
        else:
            raise error
````

- EN: This block branches on runtime conditions or configuration choices; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions.
- CN: 该代码块依据运行时条件或配置选择进行分支；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互。

### Lines 313-314 / 第 313-314 行

````python
if __name__ == "__main__":
    main()
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
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。
- EN: CLI entry points — the module exposes command-line arguments so developers or CI jobs can drive automation predictably.
  CN: 命令行入口——该模块暴露命令行参数，使开发者或 CI 作业能够以可预测方式驱动自动化逻辑。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `__future__`, `json`, `os`, `re`, `typing`, `urllib.error`
- Internal imports / 内部导入: none
- External imports / 外部导入: `github_utils`, `gitutils`, `trymerge`
- Classes / 类: none
- Functions / 函数: `parse_args`, `get_merge_commit_sha`, `get_release_version`, `get_tracker_issues`, `cherry_pick`, `create_cherry_pick_branch`, `submit_pr`, `post_pr_comment`, `post_tracker_issue_comment`, `main`
- Test entry points / 测试入口: none
