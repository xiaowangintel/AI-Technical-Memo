# gitutils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `.github/scripts/gitutils.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements repository automation helpers used by CI, release engineering, or GitHub maintenance workflows.
- **用途 (CN)**: 实现仓库自动化辅助脚本，供 CI、发布工程或 GitHub 维护流程使用。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17 / 第 1-17 行

````python
#!/usr/bin/env python3

from __future__ import annotations

import os
import re
import tempfile
from collections import defaultdict
from collections.abc import Callable, Iterator
from datetime import datetime
from functools import wraps
from typing import Any, cast, TypeVar


T = TypeVar("T")

RE_GITHUB_URL_MATCH = re.compile("^https://.*@?github.com/(.+)/(.+)$")
````

- EN: This block imports dependencies such as `__future__`, `os`, `re`, `tempfile`; touches GitHub-related data or remote service interactions.
- CN: 该代码块导入当前模块运行所需的依赖；处理 GitHub 相关数据或远程服务交互。

### Lines 20-37 / 第 20-37 行

````python
def get_git_remote_name() -> str:
    return os.getenv("GIT_REMOTE_NAME", "origin")


def get_git_repo_dir() -> str:
    from pathlib import Path

    return os.getenv("GIT_REPO_DIR", str(Path(__file__).resolve().parents[2]))


def fuzzy_list_to_dict(items: list[tuple[str, str]]) -> dict[str, list[str]]:
    """
    Converts list to dict preserving elements with duplicate keys
    """
    rc: dict[str, list[str]] = defaultdict(list)
    for key, val in items:
        rc[key].append(val)
    return dict(rc)
````

- EN: This block imports dependencies such as `pathlib`; defines callable units such as `get_git_remote_name`, `get_git_repo_dir`, `fuzzy_list_to_dict`; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块导入当前模块运行所需的依赖；定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 40-56 / 第 40-56 行

````python
def _check_output(items: list[str], encoding: str = "utf-8") -> str:
    from subprocess import CalledProcessError, check_output, STDOUT

    try:
        return check_output(items, stderr=STDOUT).decode(encoding)
    except CalledProcessError as e:
        msg = f"Command `{' '.join(e.cmd)}` returned non-zero exit code {e.returncode}"
        stdout = e.stdout.decode(encoding) if e.stdout is not None else ""
        stderr = e.stderr.decode(encoding) if e.stderr is not None else ""
        # These get swallowed up, so print them here for debugging
        print(f"stdout: \n{stdout}")
        print(f"stderr: \n{stderr}")
        if len(stderr) == 0:
            msg += f"\n```\n{stdout}```"
        else:
            msg += f"\nstdout:\n```\n{stdout}```\nstderr:\n```\n{stderr}```"
        raise RuntimeError(msg) from e
````

- EN: This block imports dependencies such as `subprocess`; defines callable units such as `_check_output`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; validates assumptions and surfaces failures explicitly.
- CN: 该代码块导入当前模块运行所需的依赖；定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；校验前提条件并显式暴露失败。

### Lines 59-76 / 第 59-76 行

````python
class GitCommit:
    commit_hash: str
    title: str
    body: str
    author: str
    author_date: datetime
    commit_date: datetime | None

    def __init__(
        self,
        commit_hash: str,
        author: str,
        author_date: datetime,
        title: str,
        body: str,
        commit_date: datetime | None = None,
    ) -> None:
        self.commit_hash = commit_hash
````

- EN: This block introduces classes like `GitCommit`; defines callable units such as `__init__`.
- CN: 该代码块定义新的类来封装状态与行为；定义可调用函数来执行具体自动化步骤。

### Lines 77-87 / 第 77-87 行

````python
        self.author = author
        self.author_date = author_date
        self.commit_date = commit_date
        self.title = title
        self.body = body

    def __repr__(self) -> str:
        return f"{self.title} ({self.commit_hash})"

    def __contains__(self, item: Any) -> bool:
        return item in self.body or item in self.title
````

- EN: This block defines callable units such as `__repr__`, `__contains__`; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果。

### Lines 90-101 / 第 90-101 行

````python
def parse_fuller_format(lines: str | list[str]) -> GitCommit:
    """
    Expect commit message generated using `--format=fuller --date=unix` format, i.e.:
        commit <sha1>
        Author:     <author>
        AuthorDate: <author date>
        Commit:     <committer>
        CommitDate: <committer date>

        <title line>

        <full commit message>
````

- EN: This block defines callable units such as `parse_fuller_format`.
- CN: 该代码块定义可调用函数来执行具体自动化步骤。

### Lines 103-120 / 第 103-120 行

````python
    """
    if isinstance(lines, str):
        lines = lines.split("\n")
    # TODO: Handle merge commits correctly
    if len(lines) > 1 and lines[1].startswith("Merge:"):
        del lines[1]
    if len(lines) <= 7:
        raise AssertionError(
            f"Expected at least 8 lines in git log fuller format, got {len(lines)}"
        )
    if not lines[0].startswith("commit"):
        raise AssertionError(f"Expected line 0 to start with 'commit', got: {lines[0]}")
    if not lines[1].startswith("Author: "):
        raise AssertionError(
            f"Expected line 1 to start with 'Author: ', got: {lines[1]}"
        )
    if not lines[2].startswith("AuthorDate: "):
        raise AssertionError(
````

- EN: This block branches on runtime conditions or configuration choices; validates assumptions and surfaces failures explicitly.
- CN: 该代码块依据运行时条件或配置选择进行分支；校验前提条件并显式暴露失败。

### Lines 121-138 / 第 121-138 行

````python
            f"Expected line 2 to start with 'AuthorDate: ', got: {lines[2]}"
        )
    if not lines[3].startswith("Commit: "):
        raise AssertionError(
            f"Expected line 3 to start with 'Commit: ', got: {lines[3]}"
        )
    if not lines[4].startswith("CommitDate: "):
        raise AssertionError(
            f"Expected line 4 to start with 'CommitDate: ', got: {lines[4]}"
        )
    if len(lines[5]) != 0:
        raise AssertionError(f"Expected line 5 to be empty, got: {lines[5]}")
    return GitCommit(
        commit_hash=lines[0].split()[1].strip(),
        author=lines[1].split(":", 1)[1].strip(),
        author_date=datetime.fromtimestamp(int(lines[2].split(":", 1)[1].strip())),
        commit_date=datetime.fromtimestamp(int(lines[4].split(":", 1)[1].strip())),
        title=lines[6].strip(),
````

- EN: This block branches on runtime conditions or configuration choices; returns computed values to callers; validates assumptions and surfaces failures explicitly.
- CN: 该代码块依据运行时条件或配置选择进行分支；向调用方返回计算结果；校验前提条件并显式暴露失败。

### Lines 139-152 / 第 139-152 行

````python
        body="\n".join(lines[7:]),
    )


class GitRepo:
    def __init__(self, path: str, remote: str = "origin", debug: bool = False) -> None:
        self.repo_dir = path
        self.remote = remote
        self.debug = debug

    def _run_git(self, *args: Any) -> str:
        if self.debug:
            print(f"+ git -C {self.repo_dir} {' '.join(args)}")
        return _check_output(["git", "-C", self.repo_dir] + list(args))
````

- EN: This block introduces classes like `GitRepo`; defines callable units such as `__init__`, `_run_git`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义新的类来封装状态与行为；定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 154-166 / 第 154-166 行

````python
    def revlist(self, revision_range: str) -> list[str]:
        rc = self._run_git("rev-list", revision_range, "--", ".").strip()
        return rc.split("\n") if len(rc) > 0 else []

    def branches_containing_ref(
        self, ref: str, *, include_remote: bool = True
    ) -> list[str]:
        rc = (
            self._run_git("branch", "--remote", "--contains", ref)
            if include_remote
            else self._run_git("branch", "--contains", ref)
        )
        return [x.strip() for x in rc.split("\n") if x.strip()] if len(rc) > 0 else []
````

- EN: This block defines callable units such as `revlist`, `branches_containing_ref`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 168-179 / 第 168-179 行

````python
    def current_branch(self) -> str | None:
        try:
            return self._run_git("symbolic-ref", "--short", "HEAD").strip()
        except RuntimeError:
            # we are in detached HEAD state
            return None

    def checkout(self, branch: str) -> None:
        self._run_git("checkout", branch)

    def create_branch_and_checkout(self, branch: str) -> None:
        self._run_git("checkout", "-b", branch)
````

- EN: This block defines callable units such as `current_branch`, `checkout`, `create_branch_and_checkout`; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果。

### Lines 181-196 / 第 181-196 行

````python
    def fetch(self, ref: str | None = None, branch: str | None = None) -> None:
        if branch is None and ref is None:
            self._run_git("fetch", self.remote)
        elif branch is None:
            self._run_git("fetch", self.remote, ref)
        else:
            self._run_git("fetch", self.remote, f"{ref}:{branch}")

    def show_ref(self, name: str) -> str:
        refs = self._run_git("show-ref", "-s", name).strip().split("\n")
        if not all(refs[i] == refs[0] for i in range(1, len(refs))):
            raise RuntimeError(f"reference {name} is ambiguous")
        return refs[0]

    def rev_parse(self, name: str) -> str:
        return self._run_git("rev-parse", "--verify", name).strip()
````

- EN: This block defines callable units such as `fetch`, `show_ref`, `rev_parse`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; validates assumptions and surfaces failures explicitly.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；校验前提条件并显式暴露失败。

### Lines 198-210 / 第 198-210 行

````python
    def get_merge_base(self, from_ref: str, to_ref: str) -> str:
        return self._run_git("merge-base", from_ref, to_ref).strip()

    def patch_id(self, ref: str | list[str]) -> list[tuple[str, str]]:
        is_list = isinstance(ref, list)
        if is_list:
            if len(ref) == 0:
                return []
            ref = " ".join(ref)
        rc = _check_output(
            ["sh", "-c", f"git -C {self.repo_dir} show {ref}|git patch-id --stable"]
        ).strip()
        return [cast(tuple[str, str], x.split(" ", 1)) for x in rc.split("\n")]
````

- EN: This block defines callable units such as `get_merge_base`, `patch_id`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 212-227 / 第 212-227 行

````python
    def commits_resolving_gh_pr(self, pr_num: int) -> list[str]:
        owner, name = self.gh_owner_and_name()
        msg = f"Pull Request resolved: https://github.com/{owner}/{name}/pull/{pr_num}"
        rc = self._run_git("log", "--format=%H", "--grep", msg).strip()
        return rc.split("\n") if len(rc) > 0 else []

    def get_commit(self, ref: str) -> GitCommit:
        return parse_fuller_format(
            self._run_git("show", "--format=fuller", "--date=unix", "--shortstat", ref)
        )

    def cherry_pick(self, ref: str) -> None:
        self._run_git("cherry-pick", "-x", ref)

    def revert(self, ref: str) -> None:
        self._run_git("revert", "--no-edit", ref)
````

- EN: This block defines callable units such as `commits_resolving_gh_pr`, `get_commit`, `cherry_pick`, `revert`; branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 229-246 / 第 229-246 行

````python
    def compute_branch_diffs(
        self, from_branch: str, to_branch: str
    ) -> tuple[list[str], list[str]]:
        """
        Returns list of commits that are missing in each other branch since their merge base
        Might be slow if merge base is between two branches is pretty far off
        """
        from_ref = self.rev_parse(from_branch)
        to_ref = self.rev_parse(to_branch)
        merge_base = self.get_merge_base(from_ref, to_ref)
        from_commits = self.revlist(f"{merge_base}..{from_ref}")
        to_commits = self.revlist(f"{merge_base}..{to_ref}")
        from_ids = fuzzy_list_to_dict(self.patch_id(from_commits))
        to_ids = fuzzy_list_to_dict(self.patch_id(to_commits))
        for patch_id in set(from_ids).intersection(set(to_ids)):
            from_values = from_ids[patch_id]
            to_values = to_ids[patch_id]
            if len(from_values) != len(to_values):
````

- EN: This block defines callable units such as `compute_branch_diffs`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 247-264 / 第 247-264 行

````python
                # Eliminate duplicate commits+reverts from the list
                while len(from_values) > 0 and len(to_values) > 0:
                    frc = self.get_commit(from_values.pop())
                    toc = self.get_commit(to_values.pop())
                    # FRC branch might have PR number added to the title
                    if frc.title != toc.title or frc.author_date != toc.author_date:
                        # HACK: Same commit were merged, reverted and landed again
                        # which creates a tracking problem
                        if (
                            "pytorch/pytorch" not in self.remote_url()
                            or frc.commit_hash
                            not in {
                                "0a6a1b27a464ba5be5f587cce2ee12ab8c504dbf",
                                "6d0f4a1d545a8f161df459e8d4ccafd4b9017dbe",
                                "edf909e58f06150f7be41da2f98a3b9de3167bca",
                                "a58c6aea5a0c9f8759a4154e46f544c8b03b8db1",
                                "7106d216c29ca16a3504aa2bedad948ebcf4abc2",
                            }
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 265-282 / 第 265-282 行

````python
                        ):
                            raise RuntimeError(
                                f"Unexpected differences between {frc} and {toc}"
                            )
                    from_commits.remove(frc.commit_hash)
                    to_commits.remove(toc.commit_hash)
                continue
            for commit in from_values:
                from_commits.remove(commit)
            for commit in to_values:
                to_commits.remove(commit)
        # Another HACK: Patch-id is not stable for commits with binary files or for big changes across commits
        # I.e. cherry-picking those from one branch into another will change patchid
        if "pytorch/pytorch" in self.remote_url():
            for excluded_commit in {
                "8e09e20c1dafcdbdb45c2d1574da68a32e54a3a5",
                "5f37e5c2a39c3acb776756a17730b865f0953432",
                "b5222584e6d6990c6585981a936defd1af14c0ba",
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; validates assumptions and surfaces failures explicitly.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；校验前提条件并显式暴露失败。

### Lines 283-300 / 第 283-300 行

````python
                "84d9a2e42d5ed30ec3b8b4140c38dd83abbce88d",
                "f211ec90a6cdc8a2a5795478b5b5c8d7d7896f7e",
            }:
                if excluded_commit in from_commits:
                    from_commits.remove(excluded_commit)

        return (from_commits, to_commits)

    def cherry_pick_commits(self, from_branch: str, to_branch: str) -> None:
        orig_branch = self.current_branch()
        if orig_branch is None:
            raise AssertionError("Must be on a branch to cherry pick commits")
        self.checkout(to_branch)
        from_commits, to_commits = self.compute_branch_diffs(from_branch, to_branch)
        if len(from_commits) == 0:
            print("Nothing to do")
            self.checkout(orig_branch)
            return
````

- EN: This block defines callable units such as `cherry_pick_commits`; branches on runtime conditions or configuration choices; returns computed values to callers; validates assumptions and surfaces failures explicitly.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；校验前提条件并显式暴露失败。

### Lines 301-316 / 第 301-316 行

````python
        for commit in reversed(from_commits):
            print(f"Cherry picking commit {commit}")
            self.cherry_pick(commit)
        self.checkout(orig_branch)

    def push(self, branch: str, dry_run: bool, retry: int = 3) -> None:
        for cnt in range(retry):
            try:
                if dry_run:
                    self._run_git("push", "--dry-run", self.remote, branch)
                else:
                    self._run_git("push", self.remote, branch)
            except RuntimeError as e:
                print(f"{cnt} push attempt failed with {e}")
                self.fetch()
                self._run_git("rebase", f"{self.remote}/{branch}")
````

- EN: This block defines callable units such as `push`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 318-334 / 第 318-334 行

````python
    def head_hash(self) -> str:
        return self._run_git("show-ref", "--hash", "HEAD").strip()

    def remote_url(self) -> str:
        return self._run_git("remote", "get-url", self.remote)

    def gh_owner_and_name(self) -> tuple[str, str]:
        url = os.getenv("GIT_REMOTE_URL", None)
        if url is None:
            url = self.remote_url()
        rc = RE_GITHUB_URL_MATCH.match(url)
        if rc is None:
            raise RuntimeError(f"Unexpected url format {url}")
        return cast(tuple[str, str], rc.groups())

    def commit_message(self, ref: str) -> str:
        return self._run_git("log", "-1", "--format=%B", ref)
````

- EN: This block defines callable units such as `head_hash`, `remote_url`, `gh_owner_and_name`, `commit_message`; branches on runtime conditions or configuration choices; returns computed values to callers; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互。

### Lines 336-353 / 第 336-353 行

````python
    def amend_commit_message(self, msg: str) -> None:
        self._run_git("commit", "--amend", "-m", msg)

    def diff(self, from_ref: str, to_ref: str | None = None) -> str:
        if to_ref is None:
            return self._run_git("diff", f"{from_ref}^!")
        return self._run_git("diff", f"{from_ref}..{to_ref}")


def clone_repo(username: str, password: str, org: str, project: str) -> GitRepo:
    path = tempfile.mkdtemp()
    _check_output(
        [
            "git",
            "clone",
            f"https://{username}:{password}@github.com/{org}/{project}",
            path,
        ]
````

- EN: This block defines callable units such as `amend_commit_message`, `diff`, `clone_repo`; branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 354-369 / 第 354-369 行

````python
    ).strip()
    return GitRepo(path=path)


class PeekableIterator(Iterator[str]):
    def __init__(self, val: str) -> None:
        self._val = val
        self._idx = -1

    def peek(self) -> str | None:
        if self._idx + 1 >= len(self._val):
            return None
        return self._val[self._idx + 1]

    def __iter__(self) -> PeekableIterator:
        return self
````

- EN: This block introduces classes like `PeekableIterator`; defines callable units such as `__init__`, `peek`, `__iter__`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义新的类来封装状态与行为；定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 371-388 / 第 371-388 行

````python
    def __next__(self) -> str:
        rc = self.peek()
        if rc is None:
            raise StopIteration
        self._idx += 1
        return rc


def patterns_to_regex(allowed_patterns: list[str]) -> Any:
    """
    pattern is glob-like, i.e. the only special sequences it has are:
      - ? - matches single character
      - * - matches any non-folder separator characters or no character
      - ** - matches any characters or no character
      Assuming that patterns are free of braces and backslashes
      the only character that needs to be escaped are dot and plus
    """
    rc = "("
````

- EN: This block defines callable units such as `__next__`, `patterns_to_regex`; branches on runtime conditions or configuration choices; returns computed values to callers; validates assumptions and surfaces failures explicitly.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；校验前提条件并显式暴露失败。

### Lines 389-406 / 第 389-406 行

````python
    for idx, pattern in enumerate(allowed_patterns):
        if idx > 0:
            rc += "|"
        pattern_ = PeekableIterator(pattern)
        if any(c in pattern for c in "{}()[]\\"):
            raise AssertionError(
                f"Pattern contains invalid characters (braces/parens/brackets/backslash): {pattern}"
            )
        for c in pattern_:
            if c == ".":
                rc += "\\."
            elif c == "+":
                rc += "\\+"
            elif c == "*":
                if pattern_.peek() == "*":
                    next(pattern_)
                    rc += ".*"
                else:
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; validates assumptions and surfaces failures explicitly.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；校验前提条件并显式暴露失败。

### Lines 407-419 / 第 407-419 行

````python
                    rc += "[^/]*"
            else:
                rc += c
    rc += ")"
    return re.compile(rc)


def _shasum(value: str) -> str:
    import hashlib

    m = hashlib.sha256()
    m.update(value.encode("utf-8"))
    return m.hexdigest()
````

- EN: This block imports dependencies such as `hashlib`; defines callable units such as `_shasum`; returns computed values to callers.
- CN: 该代码块导入当前模块运行所需的依赖；定义可调用函数来执行具体自动化步骤；向调用方返回计算结果。

### Lines 422-439 / 第 422-439 行

````python
def is_commit_hash(ref: str) -> bool:
    "True if ref is hexadecimal number, else false"
    try:
        int(ref, 16)
    except ValueError:
        return False
    return True


def are_ghstack_branches_in_sync(
    repo: GitRepo, head_ref: str, base_ref: str | None = None
) -> bool:
    """Checks that diff between base and head is the same as diff between orig and its parent"""
    orig_ref = re.sub(r"/head$", "/orig", head_ref)
    if base_ref is None:
        base_ref = re.sub(r"/head$", "/base", head_ref)
    orig_diff_sha = _shasum(repo.diff(f"{repo.remote}/{orig_ref}"))
    head_diff_sha = _shasum(
````

- EN: This block defines callable units such as `is_commit_hash`, `are_ghstack_branches_in_sync`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 440-452 / 第 440-452 行

````python
        repo.diff(
            base_ref if is_commit_hash(base_ref) else f"{repo.remote}/{base_ref}",
            f"{repo.remote}/{head_ref}",
        )
    )
    return orig_diff_sha == head_diff_sha


def retries_decorator(
    rc: Any = None, num_retries: int = 3
) -> Callable[[Callable[..., T]], Callable[..., T]]:
    def decorator(f: Callable[..., T]) -> Callable[..., T]:
        @wraps(f)
````

- EN: This block defines callable units such as `retries_decorator`, `decorator`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 453-463 / 第 453-463 行

````python
        def wrapper(*args: list[Any], **kwargs: dict[str, Any]) -> T:
            for idx in range(num_retries):
                try:
                    return f(*args, **kwargs)
                except Exception as e:
                    print(
                        f'Attempt {idx} of {num_retries} to call {f.__name__} failed with "{e}"'
                    )
            return cast(T, rc)

        return wrapper
````

- EN: This block defines callable units such as `wrapper`; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 465-465 / 第 465-465 行

````python
    return decorator
````

- EN: This block returns computed values to callers.
- CN: 该代码块向调用方返回计算结果。


## Key Concepts / 关键概念
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `__future__`, `os`, `re`, `tempfile`, `collections`, `collections.abc`, `datetime`, `functools`, ...
- Internal imports / 内部导入: none
- External imports / 外部导入: none
- Classes / 类: `GitCommit`, `GitRepo`, `PeekableIterator`
- Functions / 函数: `get_git_remote_name`, `get_git_repo_dir`, `fuzzy_list_to_dict`, `_check_output`, `parse_fuller_format`, `clone_repo`, `patterns_to_regex`, `_shasum`, `is_commit_hash`, `are_ghstack_branches_in_sync`, ...
- Test entry points / 测试入口: none
