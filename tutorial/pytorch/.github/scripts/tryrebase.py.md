# tryrebase.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `.github/scripts/tryrebase.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements repository automation helpers used by CI, release engineering, or GitHub maintenance workflows.
- **用途 (CN)**: 实现仓库自动化辅助脚本，供 CI、发布工程或 GitHub 维护流程使用。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13 / 第 1-13 行

````python
#!/usr/bin/env python3

import contextlib
import os
import re
import subprocess
import sys
from collections.abc import Generator
from typing import Any

from github_utils import gh_post_pr_comment as gh_post_comment
from gitutils import get_git_remote_name, get_git_repo_dir, GitRepo
from trymerge import GitHubPR
````

- EN: This block imports dependencies such as `contextlib`, `os`, `re`, `subprocess`; touches GitHub-related data or remote service interactions.
- CN: 该代码块导入当前模块运行所需的依赖；处理 GitHub 相关数据或远程服务交互。

### Lines 16-29 / 第 16-29 行

````python
SAME_SHA_ERROR = (
    "\n```\nAborting rebase because rebasing the branch resulted in the same sha as the target branch.\n"
    + "This usually happens because the PR has already been merged.  Please rebase locally and push.\n```"
)


def parse_args() -> Any:
    from argparse import ArgumentParser

    parser = ArgumentParser("Rebase PR into branch")
    parser.add_argument("--dry-run", action="store_true")
    parser.add_argument("--branch", type=str)
    parser.add_argument("pr_num", type=int)
    return parser.parse_args()
````

- EN: This block imports dependencies such as `argparse`; defines callable units such as `parse_args`; parses command-line arguments for script entry points; returns computed values to callers.
- CN: 该代码块导入当前模块运行所需的依赖；定义可调用函数来执行具体自动化步骤；为脚本入口解析命令行参数；向调用方返回计算结果。

### Lines 32-43 / 第 32-43 行

````python
def post_already_uptodate(
    pr: GitHubPR, repo: GitRepo, onto_branch: str, dry_run: bool
) -> None:
    msg = f"Tried to rebase and push PR #{pr.pr_num}, but it was already up to date."
    def_branch = pr.default_branch()
    def_branch_fcn = f"refs/remotes/{repo.remote}/{def_branch}"
    if onto_branch != def_branch_fcn and repo.rev_parse(
        def_branch_fcn
    ) != repo.rev_parse(onto_branch):
        def_branch_url = f"https://github.com/{pr.org}/{pr.project}/tree/{def_branch}"
        msg += f" Try rebasing against [{def_branch}]({def_branch_url}) by issuing:"
        msg += f"\n`@pytorchbot rebase -b {def_branch}`"
````

- EN: This block defines callable units such as `post_already_uptodate`; branches on runtime conditions or configuration choices; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；处理 GitHub 相关数据或远程服务交互。

### Lines 45-59 / 第 45-59 行

````python
    gh_post_comment(
        pr.org,
        pr.project,
        pr.pr_num,
        msg,
        dry_run=dry_run,
    )


def rebase_onto(
    pr: GitHubPR, repo: GitRepo, onto_branch: str, dry_run: bool = False
) -> bool:
    branch = f"pull/{pr.pr_num}/head"
    remote_url = f"https://github.com/{pr.info['headRepository']['nameWithOwner']}.git"
    refspec = f"{branch}:{pr.head_ref()}"
````

- EN: This block defines callable units such as `rebase_onto`; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互。

### Lines 61-78 / 第 61-78 行

````python
    repo.fetch(branch, branch)
    repo._run_git("rebase", onto_branch, branch)

    if repo.rev_parse(branch) == repo.rev_parse(onto_branch):
        raise Exception(SAME_SHA_ERROR)  # noqa: TRY002

    if dry_run:
        push_result = repo._run_git("push", "--dry-run", "-f", remote_url, refspec)
    else:
        push_result = repo._run_git("push", "-f", remote_url, refspec)
    if "Everything up-to-date" in push_result:
        post_already_uptodate(pr, repo, onto_branch, dry_run)
        return False
    else:
        gh_post_comment(
            pr.org,
            pr.project,
            pr.pr_num,
````

- EN: This block branches on runtime conditions or configuration choices; returns computed values to callers; validates assumptions and surfaces failures explicitly.
- CN: 该代码块依据运行时条件或配置选择进行分支；向调用方返回计算结果；校验前提条件并显式暴露失败。

### Lines 79-96 / 第 79-96 行

````python
            f"Successfully rebased `{pr.head_ref()}` onto `{onto_branch}`, please pull locally "
            + f"before adding more changes (for example, via `git checkout {pr.head_ref()} && "
            + "git pull --rebase`)",
            dry_run=dry_run,
        )
        return True


def rebase_ghstack_onto(
    pr: GitHubPR, repo: GitRepo, onto_branch: str, dry_run: bool = False
) -> bool:
    if (
        subprocess.run(
            [sys.executable, "-m", "ghstack", "--help"],
            capture_output=True,
            check=False,
        ).returncode
        != 0
````

- EN: This block defines callable units such as `rebase_ghstack_onto`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 97-111 / 第 97-111 行

````python
    ):
        subprocess.run([sys.executable, "-m", "pip", "install", "ghstack"], check=True)
    orig_ref = f"{re.sub(r'/head$', '/orig', pr.head_ref())}"

    repo.fetch(orig_ref, orig_ref)
    repo._run_git("rebase", onto_branch, orig_ref)

    if repo.rev_parse(orig_ref) == repo.rev_parse(onto_branch):
        raise Exception(SAME_SHA_ERROR)  # noqa: TRY002

    # steal the identity of the committer of the commit on the orig branch
    email = repo._run_git("log", orig_ref, "--pretty=format:%ae", "-1")
    name = repo._run_git("log", orig_ref, "--pretty=format:%an", "-1")
    repo._run_git("config", "--global", "user.email", email)
    repo._run_git("config", "--global", "user.name", name)
````

- EN: This block branches on runtime conditions or configuration choices; validates assumptions and surfaces failures explicitly.
- CN: 该代码块依据运行时条件或配置选择进行分支；校验前提条件并显式暴露失败。

### Lines 113-130 / 第 113-130 行

````python
    os.environ["OAUTH_TOKEN"] = os.environ["GITHUB_TOKEN"]
    with open(".ghstackrc", "w+") as f:
        f.write(
            "[ghstack]\n"
            + "github_url=github.com\n"
            + "github_username=pytorchmergebot\n"
            + "remote_name=origin"
        )

    if dry_run:
        print("Don't know how to dry-run ghstack")
        return False
    else:
        ghstack_result = subprocess.run(["ghstack"], capture_output=True, check=True)
        push_result = ghstack_result.stdout.decode("utf-8")
        print(push_result)
        if ghstack_result.returncode != 0:
            print(ghstack_result.stderr.decode("utf-8"))
````

- EN: This block branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 131-143 / 第 131-143 行

````python
            raise Exception(f"\n```{push_result}```")  # noqa: TRY002
        # The contents of a successful push result should look like:
        # Summary of changes (ghstack 0.6.0)

        #  - Updated https://github.com/clee2000/random-testing-public/pull/2
        #  - Updated https://github.com/clee2000/random-testing-public/pull/1

        # Facebook employees can import your changes by running
        # (on a Facebook machine):

        #     ghimport -s https://github.com/clee2000/random-testing-public/pull/2

        # If you want to work on this diff stack on another machine:
````

- EN: This block branches on runtime conditions or configuration choices; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions.
- CN: 该代码块依据运行时条件或配置选择进行分支；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互。

### Lines 145-162 / 第 145-162 行

````python
        #     ghstack checkout https://github.com/clee2000/random-testing-public/pull/2
        org, project = repo.gh_owner_and_name()
        for line in push_result.splitlines():
            if "Updated" in line:
                pr_num = int(line.split("/")[-1])
                if pr_num != pr.pr_num:
                    gh_post_comment(
                        pr.org,
                        pr.project,
                        pr_num,
                        f"Rebased `{orig_ref}` onto `{onto_branch}` because #{pr.pr_num} was rebased, "
                        "please pull locally before adding more changes (for example, via `ghstack "
                        + f"checkout https://github.com/{org}/{project}/pull/{pr_num}`)",
                        dry_run=dry_run,
                    )
                else:
                    gh_post_comment(
                        pr.org,
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互。

### Lines 163-177 / 第 163-177 行

````python
                        pr.project,
                        pr_num,
                        f"Successfully rebased `{orig_ref}` onto `{onto_branch}`, please pull locally "
                        + "before adding more changes (for example, via `ghstack "
                        + f"checkout https://github.com/{org}/{project}/pull/{pr.pr_num}`)",
                        dry_run=dry_run,
                    )

        if (
            f"Skipped https://github.com/{org}/{project}/pull/{pr.pr_num}"
            in push_result
        ):
            post_already_uptodate(pr, repo, onto_branch, dry_run)
            return False
        return True
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 180-191 / 第 180-191 行

````python
def additional_rebase_failure_info(e: Exception) -> str:
    if re.search(
        r"remote: Permission to .* denied to .*\.\nfatal: unable to access", str(e)
    ):
        return (
            "\nThis is likely because the author did not allow edits from maintainers on the PR or because the "
            "repo has additional permissions settings that mergebot does not qualify."
        )
    return ""


@contextlib.contextmanager
````

- EN: This block defines callable units such as `additional_rebase_failure_info`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 192-208 / 第 192-208 行

````python
def git_config_guard(repo: GitRepo) -> Generator[None, None, None]:
    """Restores user.name and user.email global properties after context is finished"""
    user_email = repo._run_git("config", "user.email")
    user_name = repo._run_git("config", "user.name")
    try:
        yield
    finally:
        if user_email:
            repo._run_git("config", "--global", "user.email", user_email)
        if user_name:
            repo._run_git("config", "--global", "user.name", user_name)


def main() -> None:
    args = parse_args()
    repo = GitRepo(get_git_repo_dir(), get_git_remote_name(), debug=True)
    org, project = repo.gh_owner_and_name()
````

- EN: This block defines callable units such as `git_config_guard`, `main`; branches on runtime conditions or configuration choices.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支。

### Lines 210-227 / 第 210-227 行

````python
    pr = GitHubPR(org, project, args.pr_num)
    onto_branch = args.branch if args.branch else pr.default_branch()
    onto_branch = f"refs/remotes/{repo.remote}/{onto_branch}"
    onto_branch_url = (
        f"https://github.com/{org}/{project}/commit/{repo.rev_parse(onto_branch)}"
    )

    msg = f"@pytorchbot started a rebase job onto [{onto_branch}]({onto_branch_url})."
    msg += f" Check the current status [here]({os.getenv('GH_RUN_URL')})"
    gh_post_comment(org, project, args.pr_num, msg, dry_run=args.dry_run)

    if pr.is_closed():
        gh_post_comment(
            org,
            project,
            args.pr_num,
            f"PR #{args.pr_num} is closed, won't rebase",
            dry_run=args.dry_run,
````

- EN: This block branches on runtime conditions or configuration choices; touches GitHub-related data or remote service interactions.
- CN: 该代码块依据运行时条件或配置选择进行分支；处理 GitHub 相关数据或远程服务交互。

### Lines 228-245 / 第 228-245 行

````python
        )
        return

    try:
        if pr.is_ghstack_pr():
            with git_config_guard(repo):
                rc = rebase_ghstack_onto(pr, repo, onto_branch, dry_run=args.dry_run)
        else:
            rc = rebase_onto(pr, repo, onto_branch, dry_run=args.dry_run)
        sys.exit(0 if rc else 1)

    except Exception as e:
        msg = f"Rebase failed due to {e}"
        msg += additional_rebase_failure_info(e)
        run_url = os.getenv("GH_RUN_URL")
        if run_url is not None:
            msg += f"\nRaised by {run_url}"
        gh_post_comment(org, project, args.pr_num, msg, dry_run=args.dry_run)
````

- EN: This block branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 248-249 / 第 248-249 行

````python
if __name__ == "__main__":
    main()
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。


## Key Concepts / 关键概念
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。
- EN: CLI entry points — the module exposes command-line arguments so developers or CI jobs can drive automation predictably.
  CN: 命令行入口——该模块暴露命令行参数，使开发者或 CI 作业能够以可预测方式驱动自动化逻辑。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `contextlib`, `os`, `re`, `subprocess`, `sys`, `collections.abc`, `typing`
- Internal imports / 内部导入: none
- External imports / 外部导入: `github_utils`, `gitutils`, `trymerge`
- Classes / 类: none
- Functions / 函数: `parse_args`, `post_already_uptodate`, `rebase_onto`, `rebase_ghstack_onto`, `additional_rebase_failure_info`, `git_config_guard`, `main`
- Test entry points / 测试入口: none
