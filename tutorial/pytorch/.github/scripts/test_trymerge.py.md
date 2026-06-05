# test_trymerge.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `.github/scripts/test_trymerge.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements repository automation helpers used by CI, release engineering, or GitHub maintenance workflows. This particular file is a focused test module for adjacent automation helpers.
- **用途 (CN)**: 实现仓库自动化辅助脚本，供 CI、发布工程或 GitHub 维护流程使用。 该文件还是一个针对相邻自动化辅助逻辑的聚焦测试模块。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行

````python
#!/usr/bin/env python3

# Tests implemented in this file are relying on GitHub GraphQL APIs
# In order to avoid test flakiness, results of the queries
# are cached in gql_mocks.json
# PyTorch Lint workflow does not have GITHUB_TOKEN defined to avoid
# flakiness, so if you are making changes to merge_rules or
# GraphQL queries in trymerge.py, please make sure to delete `gql_mocks.json`
# And re-run the test locally with ones PAT

from __future__ import annotations

import gzip
import json
import os
import warnings
from hashlib import sha256
from typing import Any
from unittest import main, mock, skip, TestCase
from urllib.error import HTTPError
````

- EN: This block imports dependencies such as `__future__`, `gzip`, `json`, `os`; branches on runtime conditions or configuration choices; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块导入当前模块运行所需的依赖；依据运行时条件或配置选择进行分支；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 22-45 / 第 22-45 行

````python
from github_utils import gh_graphql
from gitutils import get_git_remote_name, get_git_repo_dir, GitRepo
from trymerge import (
    _revlist_to_prs,
    categorize_checks,
    DRCI_CHECKRUN_NAME,
    find_matching_merge_rule,
    get_classifications,
    get_drci_classifications,
    gh_get_team_members,
    GitHubPR,
    iter_issue_timeline_until_comment,
    JobCheckState,
    main as trymerge_main,
    MandatoryChecksMissingError,
    MergeRule,
    PostCommentError,
    RE_GHSTACK_DESC,
    read_merge_rules,
    remove_job_name_suffix,
    sha_from_committed_event,
    sha_from_force_push_after,
    validate_revert,
)
````

- EN: This block imports dependencies such as `github_utils`, `gitutils`, `trymerge`; touches GitHub-related data or remote service interactions.
- CN: 该代码块导入当前模块运行所需的依赖；处理 GitHub 相关数据或远程服务交互。

### Lines 48-67 / 第 48-67 行

````python
if "GIT_REMOTE_URL" not in os.environ:
    os.environ["GIT_REMOTE_URL"] = "https://github.com/pytorch/pytorch"

GQL_MOCKS = "gql_mocks.json.gz"
DRCI_MOCKS = "drci_mocks.json.gz"


def mock_query(
    fallback_function: Any,
    file_name: str,
    key_function: Any,
    *args: Any,
) -> Any:
    gql_db_fname = os.path.join(os.path.dirname(__file__), file_name)

    def get_mocked_queries() -> Any:
        if not os.path.exists(gql_db_fname):
            return {}
        with gzip.open(gql_db_fname, encoding="utf-8", mode="rt") as f:
            return json.load(f)
````

- EN: This block defines callable units such as `mock_query`, `get_mocked_queries`; branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 69-92 / 第 69-92 行

````python
    def save_mocked_queries(obj: Any) -> None:
        with gzip.open(gql_db_fname, encoding="utf-8", mode="wt") as f:
            json.dump(obj, f, indent=2)
            f.write("\n")

    key = key_function(*args)
    mocked_queries = get_mocked_queries()

    if key in mocked_queries:
        return mocked_queries[key]

    # TODO: Remove me once https://github.com/pytorch/pytorch/issues/160489 is resolved
    raise ValueError(f"Key {key} could not be found in gql_mocks")

    try:
        rc = fallback_function(*args)
    except HTTPError as err:
        if err.code == 401 or err.code == 403:
            err_msg = f"If you are seeing this message during workflow run, please make sure to update {file_name}"
            err_msg += f" locally, by deleting it and running {os.path.basename(__file__)} with"
            err_msg += " GitHub Personal Access Token passed via GITHUB_TOKEN"
            err_msg += " and drci api key passed via DRCI_BOT_KEY environment variables"
            if os.getenv("GITHUB_TOKEN") is None or os.getenv("DRCI_BOT_KEY") is None:
                err_msg = (
````

- EN: This block defines callable units such as `save_mocked_queries`; branches on runtime conditions or configuration choices; returns computed values to callers; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 93-114 / 第 93-114 行

````python
                    "Failed to update cached queries as GITHUB_TOKEN or DRCI_BOT_KEY "
                    + "is not defined. "
                    + err_msg
                )
            raise RuntimeError(err_msg) from err
    mocked_queries[key] = rc

    save_mocked_queries(mocked_queries)

    return rc


def mocked_gh_graphql(query: str, **kwargs: Any) -> Any:
    def key_function(query: str, kwargs: Any) -> str:
        return f"query_sha={sha256(query.encode('utf-8')).hexdigest()} " + " ".join(
            [f"{k}={kwargs[k]}" for k in sorted(kwargs.keys())]
        )

    def gh_graphql_wrapper(query: str, kwargs: Any) -> Any:
        return gh_graphql(query, **kwargs)

    return mock_query(gh_graphql_wrapper, GQL_MOCKS, key_function, query, kwargs)
````

- EN: This block defines callable units such as `mocked_gh_graphql`, `key_function`, `gh_graphql_wrapper`; iterates through collections, records, or generated items; returns computed values to callers; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；向调用方返回计算结果；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互。

### Lines 117-139 / 第 117-139 行

````python
def mocked_drci_classifications(pr_num: int, project: str, num_retries: int = 3) -> Any:
    return mock_query(
        get_drci_classifications,
        DRCI_MOCKS,
        lambda x, y: f"{x} {y}",
        pr_num,
        project,
    )


def mock_parse_args(revert: bool = False, force: bool = False) -> Any:
    class Object:
        def __init__(self) -> None:
            self.revert = revert
            self.force = force
            self.pr_num = 76123
            self.dry_run = True
            self.comment_id = 12345  # Set to non-zero value
            self.reason = "this is for testing"
            self.ignore_current = False
            self.check_mergeability = False

    return Object()
````

- EN: This block introduces classes like `Object`; defines callable units such as `mocked_drci_classifications`, `mock_parse_args`, `__init__`; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块定义新的类来封装状态与行为；定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 142-156 / 第 142-156 行

````python
def mock_remove_label(
    org: str, repo: str, pr_num: str, label: str, dry_run: bool
) -> None:
    pass


def mock_revert(
    repo: GitRepo,
    pr: GitHubPR,
    *,
    dry_run: bool = False,
    comment_id: int | None = None,
    reason: str | None = None,
) -> None:
    pass
````

- EN: This block defines callable units such as `mock_remove_label`, `mock_revert`; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互。

### Lines 159-181 / 第 159-181 行

````python
def mock_merge(
    pr: GitHubPR,
    repo: GitRepo,
    comment_id: int,
    dry_run: bool = False,
    skip_mandatory_checks: bool = False,
    timeout_minutes: int = 400,
    stale_pr_days: int = 3,
    ignore_current: bool = False,
) -> None:
    pass


def mock_gh_get_info() -> Any:
    return {
        "closed": False,
        "isCrossRepository": False,
        "headRefName": "foo",
        "baseRefName": "bar",
        "baseRepository": {"defaultBranchRef": {"name": "bar"}},
        "files": {"nodes": [], "pageInfo": {"hasNextPage": False}},
        "changedFiles": 0,
    }
````

- EN: This block defines callable units such as `mock_merge`, `mock_gh_get_info`; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 184-207 / 第 184-207 行

````python
def mocked_read_merge_rules_NE(repo: Any, org: str, project: str) -> list[MergeRule]:
    return [
        MergeRule(
            name="mock with nonexistent check",
            patterns=["*"],
            approved_by=[],
            mandatory_checks_name=["Lint", "Facebook CLA Check", "nonexistent"],
            ignore_flaky_failures=True,
        ),
    ]


def mocked_read_merge_rules(repo: Any, org: str, project: str) -> list[MergeRule]:
    return [
        MergeRule(
            name="super",
            patterns=["*"],
            approved_by=["pytorch/metamates", "ngimel"],
            mandatory_checks_name=[
                "Lint",
                "pull / linux-xenial-cuda11.3-py3.7-gcc7 / build",
            ],
            ignore_flaky_failures=True,
        ),
````

- EN: This block defines callable units such as `mocked_read_merge_rules_NE`, `mocked_read_merge_rules`; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果。

### Lines 208-220 / 第 208-220 行

````python
        MergeRule(
            name="xla",
            patterns=[".github/ci_commit_pins/xla.txt"],
            approved_by=["pytorchbot"],
            mandatory_checks_name=[
                "Lint",
                "EasyCLA",
                "pull / linux-focal-py3_8-clang9-xla / build",
                "pull / linux-focal-py3_8-clang9-xla / test (xla, 1, 1, linux.12xlarge)",
            ],
            ignore_flaky_failures=True,
        ),
    ]
````

- EN: This block touches GitHub-related data or remote service interactions.
- CN: 该代码块处理 GitHub 相关数据或远程服务交互。

### Lines 223-245 / 第 223-245 行

````python
def mocked_read_merge_rules_approvers(
    repo: Any, org: str, project: str
) -> list[MergeRule]:
    return [
        MergeRule(
            name="Core Reviewers",
            patterns=["*"],
            approved_by=["1", "2", "3", "4", "5", "6"],
            mandatory_checks_name=[
                "Lint",
                "pull",
            ],
        ),
        MergeRule(
            name="Core Maintainers",
            patterns=["*"],
            approved_by=["1", "2", "malfet"],
            mandatory_checks_name=[
                "Lint",
                "pull",
            ],
        ),
    ]
````

- EN: This block defines callable units such as `mocked_read_merge_rules_approvers`; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果。

### Lines 248-270 / 第 248-270 行

````python
def mocked_read_merge_rules_raise(repo: Any, org: str, project: str) -> list[MergeRule]:
    raise RuntimeError("testing")


def xla_merge_rules(repo: Any, org: str, project: str) -> list[MergeRule]:
    return [
        MergeRule(
            name=" OSS CI / pytorchbot / XLA",
            patterns=[".github/ci_commit_pins/xla.txt"],
            approved_by=["pytorchbot"],
            mandatory_checks_name=[
                "Lint",
                "EasyCLA",
                "pull / linux-bionic-py3_8-clang8-xla / build",
                "pull / linux-bionic-py3_8-clang8-xla / test (xla, 1, 1, linux.4xlarge)",
                "inductor / cuda11.8-py3.10-gcc7-sm86 / test (inductor_torchbench_dynamic, 1, 1, linux.g5.4xlarge.nvidia.gpu)",
            ],
            ignore_flaky_failures=False,
        ),
    ]


class DummyGitRepo(GitRepo):
````

- EN: This block introduces classes like `DummyGitRepo`; defines callable units such as `mocked_read_merge_rules_raise`, `xla_merge_rules`; returns computed values to callers; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义新的类来封装状态与行为；定义可调用函数来执行具体自动化步骤；向调用方返回计算结果；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互。

### Lines 271-292 / 第 271-292 行

````python
    def __init__(self) -> None:
        super().__init__(get_git_repo_dir(), get_git_remote_name())

    def commits_resolving_gh_pr(self, pr_num: int) -> list[str]:
        return ["FakeCommitSha"]

    def commit_message(self, ref: str) -> str:
        return "super awesome commit message"


@mock.patch("trymerge.gh_graphql", side_effect=mocked_gh_graphql)
@mock.patch(
    "trymerge.get_drci_classifications", side_effect=mocked_drci_classifications
)
class TestTryMerge(TestCase):
    def test_merge_rules_valid(self, *args: Any) -> None:
        "Test that merge_rules.yaml can be parsed"
        repo = DummyGitRepo()
        merge_rules = read_merge_rules(repo, "pytorch", "pytorch")
        self.assertGreater(len(merge_rules), 1)

    @mock.patch("trymerge.read_merge_rules", side_effect=mocked_read_merge_rules)
````

- EN: This block introduces classes like `TestTryMerge`; defines callable units such as `__init__`, `commits_resolving_gh_pr`, `commit_message`, `test_merge_rules_valid`; returns computed values to callers; serializes or parses structured data files; acts as a test block that checks expected behavior.
- CN: 该代码块定义新的类来封装状态与行为；定义可调用函数来执行具体自动化步骤；向调用方返回计算结果；序列化或解析结构化数据文件；作为测试代码块校验预期行为。

### Lines 293-313 / 第 293-313 行

````python
    def test_match_rules(self, *args: Any) -> None:
        "Tests that PR passes merge rules"
        pr = GitHubPR("pytorch", "pytorch", 109999)
        repo = DummyGitRepo()
        self.assertTrue(find_matching_merge_rule(pr, repo) is not None)

    @mock.patch("trymerge.read_merge_rules", side_effect=mocked_read_merge_rules_raise)
    def test_read_merge_rules_fails(self, *args: Any) -> None:
        "Tests that PR fails to read the merge rules"
        pr = GitHubPR("pytorch", "pytorch", 77700)
        repo = DummyGitRepo()
        self.assertRaisesRegex(
            RuntimeError, "testing", lambda: find_matching_merge_rule(pr, repo)
        )

    @mock.patch(
        "trymerge.read_merge_rules", side_effect=mocked_read_merge_rules_approvers
    )
    def test_match_rules_approvers(self, *args: Any) -> None:
        "Tests that PR has the necessary approvers"
        repo = DummyGitRepo()
````

- EN: This block defines callable units such as `test_match_rules`, `test_read_merge_rules_fails`, `test_match_rules_approvers`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 315-334 / 第 315-334 行

````python
        pr = GitHubPR("pytorch", "pytorch", 115329)
        # Test that all potential approvers across all rules are listed if the
        # PR doesn't have one of them
        for mock_rule in ["Core Reviewers", "Core Maintainers"]:
            self.assertRaisesRegex(
                RuntimeError,
                mock_rule,
                lambda: find_matching_merge_rule(pr, repo),
            )

        pr = GitHubPR("pytorch", "pytorch", 115495)
        # Test that PR with the correct approvers doesn't raise any exception
        self.assertTrue(find_matching_merge_rule(pr, repo) is not None)

    @mock.patch("trymerge.read_merge_rules", side_effect=mocked_read_merge_rules)
    def test_lint_fails(self, *args: Any) -> None:
        "Tests that PR fails mandatory lint check"
        pr = GitHubPR("pytorch", "pytorch", 90791)
        repo = DummyGitRepo()
        self.assertRaises(RuntimeError, lambda: find_matching_merge_rule(pr, repo))
````

- EN: This block defines callable units such as `test_lint_fails`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 336-358 / 第 336-358 行

````python
    def test_get_last_comment(self, *args: Any) -> None:
        "Tests that last comment can be fetched"
        pr = GitHubPR("pytorch", "pytorch", 71759)
        comment = pr.get_last_comment()
        self.assertEqual(comment.author_login, "github-actions")
        self.assertIsNone(comment.editor_login)
        self.assertTrue("You've committed this PR" in comment.body_text)

    def test_get_author_null(self, *args: Any) -> None:
        """Tests that PR author can be computed
        If reply contains NULL
        """
        pr = GitHubPR("pytorch", "pytorch", 71759)
        author = pr.get_author()
        self.assertTrue(author is not None)
        self.assertTrue("@" in author)
        self.assertTrue(pr.get_diff_revision() is None)

        # PR with multiple contributors, but creator id is not among authors
        pr = GitHubPR("pytorch", "pytorch", 75095)
        self.assertEqual(pr.get_pr_creator_login(), "mruberry")
        author = pr.get_author()
        self.assertTrue(author is not None)
````

- EN: This block defines callable units such as `test_get_last_comment`, `test_get_author_null`; branches on runtime conditions or configuration choices; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 360-375 / 第 360-375 行

````python
    def test_large_diff(self, *args: Any) -> None:
        "Tests that PR with 100+ files can be fetched"
        pr = GitHubPR("pytorch", "pytorch", 73099)
        self.assertTrue(pr.get_changed_files_count() > 100)
        flist = pr.get_changed_files()
        self.assertEqual(len(flist), pr.get_changed_files_count())

    def test_internal_changes(self, *args: Any) -> None:
        "Tests that PR with internal changes is detected"
        pr = GitHubPR("pytorch", "pytorch", 110140)
        self.assertTrue(pr.has_internal_changes())

    def test_comments_pagination(self, *args: Any) -> None:
        "Tests that PR with 50+ comments can be fetched"
        pr = GitHubPR("pytorch", "pytorch", 31093)
        self.assertGreater(len(pr.get_comments()), 50)
````

- EN: This block defines callable units such as `test_large_diff`, `test_internal_changes`, `test_comments_pagination`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 377-392 / 第 377-392 行

````python
    def test_gql_complexity(self, *args: Any) -> None:
        "Fetch comments and conclusions for PR with 60 commits"
        # Previous version of GrapQL query used to cause HTTP/502 error
        # see https://gist.github.com/malfet/9b93bc7eeddeaf1d84546efc4f0c577f
        pr = GitHubPR("pytorch", "pytorch", 68111)
        self.assertGreater(len(pr.get_comments()), 20)
        # NS(09/27/2023): GitHub seems to recycle older checkruns
        # https://github.com/pytorch/pytorch/pull/68111/checks shows 0 runs
        # self.assertGreater(len(pr.get_checkrun_conclusions()), 3)
        self.assertGreater(pr.get_commit_count(), 60)

    @skip("GitHub doesn't keep this data anymore")
    def test_gql_retrieve_checksuites(self, *args: Any) -> None:
        "Fetch comments and conclusions for PR with 60 commits"
        pr = GitHubPR("pytorch", "pytorch", 94787)
        self.assertEqual(len(pr.get_checkrun_conclusions()), 182)
````

- EN: This block defines callable units such as `test_gql_complexity`, `test_gql_retrieve_checksuites`; iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 394-410 / 第 394-410 行

````python
    def test_team_members(self, *args: Any) -> None:
        "Test fetching team members works"
        dev_infra_team = gh_get_team_members("pytorch", "pytorch-dev-infra")
        self.assertGreater(len(dev_infra_team), 2)
        with self.assertWarns(Warning):
            non_existing_team = gh_get_team_members("pytorch", "qwertyuiop")
            self.assertEqual(len(non_existing_team), 0)

    def test_get_author_many_commits(self, *args: Any) -> None:
        """Tests that authors for all commits can be fetched"""
        pr = GitHubPR("pytorch", "pytorch", 76118)
        authors = pr.get_authors()
        self.assertGreater(pr.get_commit_count(), 100)
        self.assertGreater(len(authors), 50)
        self.assertTrue("@" in pr.get_author())

    @mock.patch("trymerge.read_merge_rules", side_effect=mocked_read_merge_rules_NE)
````

- EN: This block defines callable units such as `test_team_members`, `test_get_author_many_commits`; iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 411-428 / 第 411-428 行

````python
    def test_pending_status_check(self, *args: Any) -> None:
        """Tests that PR with nonexistent/pending status checks fails with the right reason."""
        pr = GitHubPR("pytorch", "pytorch", 76118)
        repo = DummyGitRepo()
        self.assertRaisesRegex(
            MandatoryChecksMissingError,
            ".*are pending/not yet run.*",
            lambda: find_matching_merge_rule(pr, repo),
        )

    def test_get_author_many_reviews(self, *args: Any) -> None:
        """Tests that all reviews can be fetched"""
        pr = GitHubPR("pytorch", "pytorch", 76123)
        approved_by = pr.get_approved_by()
        self.assertGreater(len(approved_by), 0)
        if pr._reviews is None:  # to pacify mypy
            raise AssertionError("pr._reviews is None")
        self.assertGreater(len(pr._reviews), 100)
````

- EN: This block defines callable units such as `test_pending_status_check`, `test_get_author_many_reviews`; branches on runtime conditions or configuration choices; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 430-452 / 第 430-452 行

````python
    def get_co_authors(self, *args: Any) -> None:
        """Tests that co-authors are recognized"""
        pr = GitHubPR("pytorch", "pytorch", 118347)
        authors = pr.get_authors()
        self.assertIn("kit1980", authors)
        self.assertIn("Co-authored-by:", pr.gen_commit_message())

    def test_get_checkruns_many_runs(self, *args: Any) -> None:
        """Tests that all checkruns can be fetched"""
        pr = GitHubPR("pytorch", "pytorch", 105260)
        conclusions = pr.get_checkrun_conclusions()
        self.assertEqual(len(conclusions), 221)
        self.assertTrue("pull / linux-docs / build-docs-cpp-false" in conclusions)

    def test_cancelled_gets_ignored(self, *args: Any) -> None:
        """Tests that cancelled workflow does not override existing successful status"""
        pr = GitHubPR("pytorch", "pytorch", 110367)
        conclusions = pr.get_checkrun_conclusions()
        lint_checks = [name for name in conclusions if "Lint" in name]
        self.assertTrue(len(lint_checks) > 0)
        self.assertTrue(
            all(conclusions[name].status == "SUCCESS" for name in lint_checks)
        )
````

- EN: This block defines callable units such as `get_co_authors`, `test_get_checkruns_many_runs`, `test_cancelled_gets_ignored`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 454-471 / 第 454-471 行

````python
    def test_get_review_comment_by_id(self, *args: Any) -> None:
        """Tests that even if the comment requested was actually a review instead of a simple comment, we can still find it"""
        pr = GitHubPR("pytorch", "pytorch", 107070)
        review_comment_id = 1582767635
        comment = pr.get_comment_by_id(review_comment_id)
        self.assertIsNotNone(comment)

    @mock.patch("trymerge.gh_get_pr_info", return_value=mock_gh_get_info())
    @mock.patch("trymerge.parse_args", return_value=mock_parse_args(True, False))
    @mock.patch("trymerge.try_revert", side_effect=mock_revert)
    def test_main_revert(self, mock_revert: Any, *args: Any) -> None:
        trymerge_main()
        mock_revert.assert_called_once()

    @mock.patch("trymerge.gh_get_pr_info", return_value=mock_gh_get_info())
    @mock.patch("trymerge.parse_args", return_value=mock_parse_args(False, True))
    @mock.patch("trymerge.gh_remove_label", side_effect=mock_remove_label)
    @mock.patch("trymerge.merge", side_effect=mock_merge)
````

- EN: This block defines callable units such as `test_get_review_comment_by_id`, `test_main_revert`; branches on runtime conditions or configuration choices; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 472-488 / 第 472-488 行

````python
    def test_main_force(
        self, mock_merge: Any, mock_parse_args: Any, *args: Any
    ) -> None:
        trymerge_main()
        mock_merge.assert_called_once_with(
            mock.ANY,
            mock.ANY,
            comment_id=mock.ANY,
            dry_run=mock.ANY,
            skip_mandatory_checks=True,
            ignore_current=False,
        )

    @mock.patch("trymerge.gh_get_pr_info", return_value=mock_gh_get_info())
    @mock.patch("trymerge.parse_args", return_value=mock_parse_args(False, False))
    @mock.patch("trymerge.gh_remove_label", side_effect=mock_remove_label)
    @mock.patch("trymerge.merge", side_effect=mock_merge)
````

- EN: This block defines callable units such as `test_main_force`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 489-505 / 第 489-505 行

````python
    def test_main_merge(self, mock_merge: Any, *args: Any) -> None:
        trymerge_main()
        mock_merge.assert_called_once_with(
            mock.ANY,
            mock.ANY,
            comment_id=mock.ANY,
            dry_run=mock.ANY,
            skip_mandatory_checks=False,
            ignore_current=False,
        )

    @mock.patch("trymerge.read_merge_rules", side_effect=mocked_read_merge_rules)
    def test_revert_rules(self, *args: Any) -> None:
        """Tests that reverts from collaborators are allowed"""
        pr = GitHubPR("pytorch", "pytorch", 79694)
        repo = DummyGitRepo()
        self.assertIsNotNone(validate_revert(repo, pr, comment_id=1189459845))
````

- EN: This block defines callable units such as `test_main_merge`, `test_revert_rules`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 507-527 / 第 507-527 行

````python
    def test_get_changed_files(self, *args: Any) -> None:
        """
        Tests that the list changed files in a PR doesn't include duplicates
        """
        pr = GitHubPR("pytorch", "pytorch", 95233)
        try:
            changed_files = pr.get_changed_files()
        except RuntimeError as error:
            self.fail(f"get_changed_files throws an exception: {error}")

        self.assertEqual(len(changed_files), pr.get_changed_files_count())

    def test_revert_codev_abandoned_diff_succeeds(self, *args: Any) -> None:
        pr = GitHubPR("pytorch", "pytorch", 100652)

        class GitRepoCoDev(DummyGitRepo):
            def commit_message(self, ref: str) -> str:
                return pr.get_body()

        repo = GitRepoCoDev()
        validate_revert(repo, pr, comment_id=1588195237)
````

- EN: This block introduces classes like `GitRepoCoDev`; defines callable units such as `test_get_changed_files`, `test_revert_codev_abandoned_diff_succeeds`, `commit_message`; returns computed values to callers; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义新的类来封装状态与行为；定义可调用函数来执行具体自动化步骤；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 529-543 / 第 529-543 行

````python
    def test_pr_changed_submodule_detection(self, *args: Any) -> None:
        # Updates submodule during dev-cycle but reverts it later
        pr = GitHubPR("pytorch", "pytorch", 95045)
        self.assertEqual(pr.get_changed_submodules(), [])
        self.assertFalse(pr.has_invalid_submodule_updates())

        # PR updates ideep
        pr = GitHubPR("pytorch", "pytorch", 94939)
        self.assertEqual(pr.get_changed_submodules(), ["third_party/ideep"])
        self.assertTrue(pr.has_invalid_submodule_updates())

        # Automated submodule update
        pr = GitHubPR("pytorch", "pytorch", 91051)
        self.assertEqual(pr.get_changed_submodules(), ["third_party/kineto"])
        self.assertFalse(pr.has_invalid_submodule_updates())
````

- EN: This block defines callable units such as `test_pr_changed_submodule_detection`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 545-568 / 第 545-568 行

````python
    def test_remove_job_name_suffix(self, *args: Any) -> None:
        test_cases = [
            {
                "name": "linux-bionic-cuda12.6-py3.10-gcc9-sm86 / test (default, 1, 5, linux.g5.4xlarge.nvidia.gpu)",
                "expected": "linux-bionic-cuda12.6-py3.10-gcc9-sm86 / test (default)",
            },
            {
                "name": "android-emulator-build-test / build-and-test (default, 1, 1, ubuntu-20.04-16x)",
                "expected": "android-emulator-build-test / build-and-test (default)",
            },
            {
                "name": "linux-focal-rocm5.4.2-py3.8 / build",
                "expected": "linux-focal-rocm5.4.2-py3.8 / build",
            },
            {
                "name": "libtorch-cpu-shared-with-deps-release-build",
                "expected": "libtorch-cpu-shared-with-deps-release-build",
            },
            {
                "name": "manywheel-py3_8-cuda11_8-test / test",
                "expected": "manywheel-py3_8-cuda11_8-test / test",
            },
            {
                "name": "lintrunner / linux-job",
````

- EN: This block defines callable units such as `test_remove_job_name_suffix`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 569-587 / 第 569-587 行

````python
                "expected": "lintrunner / linux-job",
            },
            {
                "name": "Test `run_test.py` is usable without boto3",
                "expected": "Test `run_test.py` is usable without boto3",
            },
        ]

        for case in test_cases:
            self.assertEqual(case["expected"], remove_job_name_suffix(case["name"]))

    def test_get_merge_base(self, *args: Any) -> None:
        pr = GitHubPR("pytorch", "pytorch", 104121)

        mock_merge_base = "mocked-sha"
        with mock.patch(
            "trymerge.gh_fetch_merge_base", return_value=mock_merge_base
        ) as mocked_gh_fetch_merge_base:
            self.assertEqual(mock_merge_base, pr.get_merge_base())
````

- EN: This block defines callable units such as `test_get_merge_base`; iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 589-609 / 第 589-609 行

````python
            # Make sure that consecutive calls will use the same merge base instead of
            # making another query
            self.assertEqual(mock_merge_base, pr.get_merge_base())
            mocked_gh_fetch_merge_base.assert_called_once()

    def test_app_can_revert(self, *args: Any) -> None:
        pr = GitHubPR("pytorch", "pytorch", 164660)
        repo = DummyGitRepo()
        app_comment_id, impostor_comment_id = 3375785595, 3377647892
        # Check that app can revert
        self.assertIsNotNone(validate_revert(repo, pr, comment_id=app_comment_id))
        # But impostor can not
        self.assertRaises(
            PostCommentError,
            lambda: validate_revert(repo, pr, comment_id=impostor_comment_id),
        )
        # Despite it's name being the name of the bot
        self.assertEqual(
            pr.get_comment_by_id(impostor_comment_id).author_login,
            "pytorch-auto-revert",
        )
````

- EN: This block defines callable units such as `test_app_can_revert`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 612-635 / 第 612-635 行

````python
@mock.patch("trymerge.gh_graphql", side_effect=mocked_gh_graphql)
@mock.patch("trymerge.gh_fetch_merge_base", return_value="")
@mock.patch(
    "trymerge.get_drci_classifications", side_effect=mocked_drci_classifications
)
class TestBypassFailures(TestCase):
    def test_get_classifications(self, *args: Any) -> None:
        pr = GitHubPR("pytorch", "pytorch", 109584)
        checks = pr.get_checkrun_conclusions()
        checks = get_classifications(
            pr.pr_num,
            pr.project,
            checks,
            [],
        )
        self.assertTrue(
            checks[
                "pull / linux-focal-py3.11-clang10 / test (dynamo, 1, 2, linux.2xlarge)"
            ].classification
            == "BROKEN_TRUNK"
        )
        self.assertTrue(
            checks[
                "trunk / win-vs2019-cpu-py3 / test (default, 2, 3, windows.4xlarge.nonephemeral)"
````

- EN: This block introduces classes like `TestBypassFailures`; defines callable units such as `test_get_classifications`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义新的类来封装状态与行为；定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 636-650 / 第 636-650 行

````python
            ].classification
            == "FLAKY"
        )
        self.assertTrue(
            checks[
                "pull / linux-jammy-py3.8-gcc11 / test (distributed, 1, 2, linux.2xlarge)"
            ].classification
            == "FLAKY"
        )
        self.assertTrue(
            checks[
                "pull / linux-focal-cuda11.8-py3.10-gcc9 / test (distributed, 1, 3, linux.8xlarge.nvidia.gpu)"
            ].classification
            == "FLAKY"
        )
````

- EN: This block implements local helper logic for test trymerge.
- CN: 该代码块实现与 test trymerge 相关的局部辅助逻辑。

### Lines 652-666 / 第 652-666 行

````python
        # Set the threshold larger or equal to the number of ok failures
        pending, failed, ignorable = categorize_checks(
            checks, list(checks.keys()), ok_failed_checks_threshold=6
        )
        self.assertTrue(len(pending) == 0)
        self.assertTrue(len(failed) == 0)
        self.assertTrue(len(ignorable["FLAKY"]) == 4)
        self.assertTrue(len(ignorable["BROKEN_TRUNK"]) == 2)

        # Not set any threshold, defaults to -1 to ignore all flaky and broken trunk failures
        pending, failed, ignorable = categorize_checks(checks, list(checks.keys()))
        self.assertTrue(len(pending) == 0)
        self.assertTrue(len(failed) == 0)
        self.assertTrue(len(ignorable["FLAKY"]) == 4)
        self.assertTrue(len(ignorable["BROKEN_TRUNK"]) == 2)
````

- EN: This block implements local helper logic for test trymerge.
- CN: 该代码块实现与 test trymerge 相关的局部辅助逻辑。

### Lines 668-684 / 第 668-684 行

````python
        # Set the threshold lower than the number of ok failures
        pending, failed, ignorable = categorize_checks(
            checks, list(checks.keys()), ok_failed_checks_threshold=1
        )
        self.assertTrue(len(pending) == 0)
        self.assertTrue(len(failed) == 6)
        self.assertTrue(len(ignorable["FLAKY"]) == 4)
        self.assertTrue(len(ignorable["BROKEN_TRUNK"]) == 2)

        # Set the threshold to 0 like when ignore_flaky_failures is on
        pending, failed, ignorable = categorize_checks(
            checks, list(checks.keys()), ok_failed_checks_threshold=1
        )
        self.assertTrue(len(pending) == 0)
        self.assertTrue(len(failed) == 6)
        self.assertTrue(len(ignorable["FLAKY"]) == 4)
        self.assertTrue(len(ignorable["BROKEN_TRUNK"]) == 2)
````

- EN: This block implements local helper logic for test trymerge.
- CN: 该代码块实现与 test trymerge 相关的局部辅助逻辑。

### Lines 686-709 / 第 686-709 行

````python
    def test_get_classifications_flaky_fullname(self, *args: Any) -> None:
        pr = GitHubPR("pytorch", "pytorch", 110362)
        checks = pr.get_checkrun_conclusions()
        checks = get_classifications(
            pr.pr_num,
            pr.project,
            checks,
            [],
        )
        pending, failed, ignorable = categorize_checks(checks, list(checks.keys()))
        self.assertTrue(len(pending) == 0)
        self.assertTrue(len(failed) == 0)
        self.assertTrue(len(ignorable["FLAKY"]) == 1)

    def test_get_classifications_invalid_cancel(self, *args: Any) -> None:
        pr = GitHubPR("pytorch", "pytorch", 110367)
        checks = pr.get_checkrun_conclusions()
        checks = get_classifications(
            pr.pr_num,
            pr.project,
            checks,
            [],
        )
        pending, failed, ignorable = categorize_checks(checks, list(checks.keys()))
````

- EN: This block defines callable units such as `test_get_classifications_flaky_fullname`, `test_get_classifications_invalid_cancel`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 710-728 / 第 710-728 行

````python
        self.assertTrue(len(pending) == 0)
        self.assertTrue(len(failed) == 0)
        self.assertTrue(len(ignorable["FLAKY"]) == 0)
        self.assertTrue(len(ignorable["BROKEN_TRUNK"]) == 0)
        self.assertTrue(len(ignorable["UNSTABLE"]) == 3)

    def test_get_classifications_similar_failures(self, *args: Any) -> None:
        pr = GitHubPR("pytorch", "pytorch", 109750)
        checks = pr.get_checkrun_conclusions()
        checks = get_classifications(
            pr.pr_num,
            pr.project,
            checks,
            [],
        )
        pending, failed, ignorable = categorize_checks(checks, list(checks.keys()))
        self.assertTrue(len(pending) == 0)
        self.assertTrue(len(failed) == 0)
        self.assertTrue(len(ignorable["FLAKY"]) == 1)
````

- EN: This block defines callable units such as `test_get_classifications_similar_failures`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 730-749 / 第 730-749 行

````python
    def test_get_classifications_unstable(self, *args: Any) -> None:
        pr = GitHubPR("pytorch", "pytorch", 104312)
        checks = pr.get_checkrun_conclusions()
        checks = get_classifications(
            pr.pr_num,
            pr.project,
            checks,
            [],
        )
        workflow_name = "linux-bionic-cuda12.1-py3.10-gcc9-bazel-test"
        job_name = "build-and-test (default, 1, 1, linux.4xlarge.nvidia.gpu, unstable)"
        self.assertTrue(
            checks[f"pull / {workflow_name} / {job_name}"].classification == "UNSTABLE"
        )
        pending, failed, ignorable = categorize_checks(
            checks, list(checks.keys()), ok_failed_checks_threshold=1
        )
        self.assertTrue(len(pending) == 0)
        self.assertTrue(len(failed) == 0)
        self.assertTrue(len(ignorable["UNSTABLE"]) == 1)
````

- EN: This block defines callable units such as `test_get_classifications_unstable`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 751-773 / 第 751-773 行

````python
        # Add another test case where there is no unstable keyword in the job name, but
        # the job has already been marked as unstable
        pr = GitHubPR("pytorch", "executorch", 3318)
        checks = pr.get_checkrun_conclusions()
        checks = get_classifications(
            pr.pr_num,
            pr.project,
            checks,
            [],
        )
        print(checks)
        workflow_name = "test-llama-app"
        job_name = "mobile-job (android)"
        self.assertTrue(
            checks[f"Android / {workflow_name} / {job_name}"].classification
            == "UNSTABLE"
        )
        pending, failed, ignorable = categorize_checks(
            checks, list(checks.keys()), ok_failed_checks_threshold=1
        )
        self.assertTrue(len(pending) == 0)
        self.assertTrue(len(failed) == 0)
        self.assertTrue(len(ignorable["UNSTABLE"]) == 1)
````

- EN: This block touches GitHub-related data or remote service interactions.
- CN: 该代码块处理 GitHub 相关数据或远程服务交互。

### Lines 775-798 / 第 775-798 行

````python
    def test_get_classifications_broken_trunk(self, *args: Any) -> None:
        # The mock merge base is the actual value returned by gh_fetch_merge_base
        test_cases = [
            {
                # This PR had one broken trunk failure but it was run on a different shard
                # than the one on the base commit. This should still count as broken trunk
                "pr_num": 104214,
                "related_failure_count": 0,
                "flaky_or_broken_trunk": 1,
            },
            {
                # This PR had one broken trunk failure and it used ghstack
                "pr_num": 105145,
                "related_failure_count": 0,
                "flaky_or_broken_trunk": 1,
            },
            {
                # The failure on the merge base was retried successfully and
                # its conclusion changed from failure to success. We want to
                # keep the failure record from the merge base so that it can
                # be used to detect broken trunk
                "pr_num": 107160,
                "related_failure_count": 0,
                "flaky_or_broken_trunk": 1,
````

- EN: This block defines callable units such as `test_get_classifications_broken_trunk`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 799-820 / 第 799-820 行

````python
            },
            {
                # This PR used Dr.CI broken trunk classification
                "pr_num": 111253,
                "related_failure_count": 1,
                "flaky_or_broken_trunk": 1,
            },
        ]

        for case in test_cases:
            pr_num = case["pr_num"]
            related_failure_count = case["related_failure_count"]
            flaky_or_broken_trunk = case["flaky_or_broken_trunk"]

            pr = GitHubPR("pytorch", "pytorch", pr_num)
            checks = pr.get_checkrun_conclusions()
            checks = get_classifications(
                pr.pr_num,
                pr.project,
                checks,
                [],
            )
````

- EN: This block iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions.
- CN: 该代码块遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互。

### Lines 822-844 / 第 822-844 行

````python
            pending, failed, _ = categorize_checks(checks, list(checks.keys()))
            self.assertTrue(len(pending) == 0)
            self.assertTrue(len(failed) == related_failure_count)

            # When the ok_failed_checks_threshold is set to 0, the broken trunk failure
            # won't be ignored
            pending, failed, _ = categorize_checks(
                checks, list(checks.keys()), ok_failed_checks_threshold=0
            )
            self.assertTrue(len(pending) == 0)
            self.assertTrue(
                len(failed) == flaky_or_broken_trunk + related_failure_count
            )

    def test_ignore_current(self, *args: Any) -> None:
        # Test various interactions of the failure classifier to ensure that ignore
        # current checks takes place after other classifications: flaky, unstable,
        # or broken trunk. Only actual new failures should be kept in the list of
        # ignore current checks to use to record force merge with actual failures
        flaky = "pull / linux-focal-cuda11.8-py3.10-gcc9 / test (distributed, 1, 3, linux.8xlarge.nvidia.gpu)"
        broken_trunk = (
            "pull / linux-focal-py3.11-clang10 / test (dynamo, 1, 2, linux.2xlarge)"
        )
````

- EN: This block defines callable units such as `test_ignore_current`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 846-868 / 第 846-868 行

````python
        pr = GitHubPR("pytorch", "pytorch", 109584)
        checks = pr.get_checkrun_conclusions()

        # Known flaky failure takes precedence over ignore current (need to set the
        # merge base here to get the results from Dr. CI, and that categorize the
        # broken trunk failure too
        checks = get_classifications(
            pr.pr_num,
            pr.project,
            checks,
            [broken_trunk, flaky],
        )
        self.assertTrue(checks[flaky].classification == "FLAKY")
        self.assertTrue(checks[broken_trunk].classification == "BROKEN_TRUNK")
        _, failed, ignorable = categorize_checks(checks, list(checks.keys()))
        self.assertTrue(len(failed) == 0)
        self.assertTrue(len(ignorable["IGNORE_CURRENT_CHECK"]) == 0)
        self.assertTrue(len(ignorable["FLAKY"]) == 4)
        self.assertTrue(len(ignorable["BROKEN_TRUNK"]) == 2)

    def test_get_classifications_wrong_workflow_name(self, *args: Any) -> None:
        pr = GitHubPR("pytorch", "pytorch", 123104)
        checks = pr.get_checkrun_conclusions()
````

- EN: This block defines callable units such as `test_get_classifications_wrong_workflow_name`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 870-893 / 第 870-893 行

````python
        check_name = "linux-binary-conda / conda-py3_8-cuda11_8-build / build"
        check_name_workflow_path = ".github/workflows/generated-linux-binary-conda-nightly.yml / conda-py3_8-cuda11_8-build / build"

        # Mock a check where the workflow name uses the full path
        checks[check_name_workflow_path] = JobCheckState(
            check_name_workflow_path,
            checks[check_name].url,
            checks[check_name].status,
            checks[check_name].classification,
            checks[check_name].job_id,
            checks[check_name].title,
            checks[check_name].summary,
        )
        del checks[check_name]

        checks = get_classifications(
            pr.pr_num,
            pr.project,
            checks,
            [],
        )
        pending, failed, ignorable = categorize_checks(
            checks,
            list(checks.keys()),
````

- EN: This block touches GitHub-related data or remote service interactions.
- CN: 该代码块处理 GitHub 相关数据或远程服务交互。

### Lines 894-917 / 第 894-917 行

````python
        )

        self.assertTrue(len(pending) == 0)
        self.assertTrue(len(failed) == 0)
        self.assertTrue(len(ignorable["FLAKY"]) == 1)
        self.assertTrue(len(ignorable["BROKEN_TRUNK"]) == 0)

    def test_ignore_failures_older_run_same_workflow(self, *args: Any) -> None:
        pr = GitHubPR("pytorch", "pytorch", 129013)
        checks = pr.get_checkrun_conclusions()
        checks = get_classifications(
            pr.pr_num,
            pr.project,
            checks,
            [],
        )
        pending, failed, ignorable = categorize_checks(
            checks,
            list(checks.keys()),
        )
        self.assertTrue(len(pending) == 0)
        self.assertTrue(len(failed) == 0)
        self.assertTrue(len(ignorable["FLAKY"]) == 2)
        self.assertTrue(len(ignorable["UNSTABLE"]) == 13)
````

- EN: This block defines callable units such as `test_ignore_failures_older_run_same_workflow`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 919-939 / 第 919-939 行

````python
    @mock.patch("trymerge.read_merge_rules", side_effect=xla_merge_rules)
    def test_dont_ignore_flaky_failures(self, *args: Any) -> None:
        """
        Regression test for https://github.com/pytorch/test-infra/issues/4126
        """
        pr = GitHubPR("pytorch", "pytorch", 105312)
        repo = DummyGitRepo()
        # Check that failure is classified as flaky but still raises exception
        with warnings.catch_warnings(record=True) as w, self.assertRaises(RuntimeError):
            find_matching_merge_rule(pr, repo)
        self.assertEqual(len(w), 1)
        self.assertIn(
            "1 checks failed but were likely due flakiness or broken trunk",
            str(w[0].message),
        )


@mock.patch("trymerge.gh_graphql", side_effect=mocked_gh_graphql)
@mock.patch("trymerge.gh_fetch_merge_base", return_value="")
@mock.patch("trymerge.get_drci_classifications", return_value={})
class TestBypassFailuresOnSandCastle(TestCase):
````

- EN: This block introduces classes like `TestBypassFailuresOnSandCastle`; defines callable units such as `test_dont_ignore_flaky_failures`; iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义新的类来封装状态与行为；定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 940-956 / 第 940-956 行

````python
    def test_get_classifications(self, *args: Any) -> None:
        pr = GitHubPR("pytorch", "pytorch", 111467)
        checks = pr.get_checkrun_conclusions()
        checks = get_classifications(
            pr.pr_num,
            pr.project,
            checks,
            [],
        )
        pending, failed, ignorable = categorize_checks(checks, list(checks.keys()))
        self.assertTrue(len(pending) == 0)
        self.assertTrue(len(failed) == 0)
        self.assertTrue(len(ignorable["FLAKY"]) == 1)
        self.assertTrue(len(ignorable["BROKEN_TRUNK"]) == 1)

    def test_get_classifications_drci_checkrun_not_found(self, *args: Any) -> None:
        pr = GitHubPR("pytorch", "pytorch", 111467)
````

- EN: This block defines callable units such as `test_get_classifications`, `test_get_classifications_drci_checkrun_not_found`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 958-977 / 第 958-977 行

````python
        # No summary
        checks = pr.get_checkrun_conclusions()
        checks[DRCI_CHECKRUN_NAME] = JobCheckState(
            DRCI_CHECKRUN_NAME,
            "",
            "NEUTRAL",
            None,
            1,
            "",
            None,
        )
        checks = get_classifications(
            pr.pr_num,
            pr.project,
            checks,
            [],
        )
        pending, failed, ignorable = categorize_checks(checks, list(checks.keys()))
        self.assertTrue(len(pending) == 0)
        self.assertTrue(len(failed) == 2)
````

- EN: This block implements local helper logic for test trymerge.
- CN: 该代码块实现与 test trymerge 相关的局部辅助逻辑。

### Lines 979-998 / 第 979-998 行

````python
        # Empty summary
        checks = pr.get_checkrun_conclusions()
        checks[DRCI_CHECKRUN_NAME] = JobCheckState(
            DRCI_CHECKRUN_NAME,
            "",
            "NEUTRAL",
            None,
            1,
            "",
            "",
        )
        checks = get_classifications(
            pr.pr_num,
            pr.project,
            checks,
            [],
        )
        pending, failed, ignorable = categorize_checks(checks, list(checks.keys()))
        self.assertTrue(len(pending) == 0)
        self.assertTrue(len(failed) == 2)
````

- EN: This block implements local helper logic for test trymerge.
- CN: 该代码块实现与 test trymerge 相关的局部辅助逻辑。

### Lines 1000-1019 / 第 1000-1019 行

````python
        # No Dr.CI checkrun
        checks = pr.get_checkrun_conclusions()
        del checks[DRCI_CHECKRUN_NAME]
        checks = get_classifications(
            pr.pr_num,
            pr.project,
            checks,
            [],
        )
        pending, failed, ignorable = categorize_checks(checks, list(checks.keys()))
        self.assertTrue(len(pending) == 0)
        self.assertTrue(len(failed) == 2)


@mock.patch("trymerge.gh_graphql", side_effect=mocked_gh_graphql)
@mock.patch("trymerge.gh_fetch_merge_base", return_value="")
@mock.patch(
    "trymerge.get_drci_classifications", side_effect=mocked_drci_classifications
)
class TestGitHubPRGhstackDependencies(TestCase):
````

- EN: This block introduces classes like `TestGitHubPRGhstackDependencies`; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义新的类来封装状态与行为；处理 GitHub 相关数据或远程服务交互。

### Lines 1020-1042 / 第 1020-1042 行

````python
    def test_pr_dependencies(self, *args: Any) -> None:
        pr = GitHubPR("pytorch", "pytorch", 106068)
        msg = pr.gen_commit_message(filter_ghstack=True)
        self.assertEqual(
            msg,
            f"{pr.get_title()} (#106068)\n\n{RE_GHSTACK_DESC.sub('', pr.get_body())}\n"
            "Pull Request resolved: https://github.com/pytorch/pytorch/pull/106068\n"
            "Approved by: https://github.com/ezyang, https://github.com/fegin\n",
        )

    def test_pr_dependencies_ghstack(self, *args: Any) -> None:
        pr0 = GitHubPR("pytorch", "pytorch", 106032)
        pr1 = GitHubPR("pytorch", "pytorch", 106033)
        pr2 = GitHubPR("pytorch", "pytorch", 106034)
        pr = GitHubPR("pytorch", "pytorch", 106068)
        msg = pr.gen_commit_message(filter_ghstack=True, ghstack_deps=[pr0, pr1, pr2])
        self.assertEqual(
            msg,
            f"{pr.get_title()} (#106068)\n\n{RE_GHSTACK_DESC.sub('', pr.get_body())}\n"
            "Pull Request resolved: https://github.com/pytorch/pytorch/pull/106068\n"
            "Approved by: https://github.com/ezyang, https://github.com/fegin\n"
            "ghstack dependencies: #106032, #106033, #106034\n",
        )
````

- EN: This block defines callable units such as `test_pr_dependencies`, `test_pr_dependencies_ghstack`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 1044-1063 / 第 1044-1063 行

````python
    @skip(
        reason="This test is run against a mutable PR that has changed, so it no longer works. The test should be changed"
    )
    @mock.patch("trymerge.read_merge_rules")
    @mock.patch("trymerge.GitRepo")
    @mock.patch("trymerge.get_ghstack_prs")
    def test_merge_ghstack_into(
        self,
        mock_get_ghstack_prs: mock.MagicMock,
        mock_repo: mock.MagicMock,
        mock_merge_rules: mock.MagicMock,
        *args: Any,
    ) -> None:
        """
        Test that the merge_ghstack_into method works correctly
        """
        pr0 = GitHubPR("pytorch", "pytorch", 106032)
        pr1 = GitHubPR("pytorch", "pytorch", 106033)
        pr2 = GitHubPR("pytorch", "pytorch", 106034)
        pr = GitHubPR("pytorch", "pytorch", 106068)
````

- EN: This block defines callable units such as `test_merge_ghstack_into`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 1065-1085 / 第 1065-1085 行

````python
        # note: in reverse order (e.g. self.pr is the last commit, top of the stack)
        mock_get_ghstack_prs.return_value = [
            (pr0, "rev0"),
            (pr1, "rev1"),
            (pr2, "rev2"),
            (pr, "rev123"),
        ]

        mock_merge_rules.return_value = [
            MergeRule(
                "Mock title", patterns=["*"], approved_by=[], mandatory_checks_name=None
            )
        ]

        mock_repo.cherry_pick.return_value = None
        mock_repo.amend_commit_message.return_value = None

        # Call the method under test
        res = pr.merge_ghstack_into(mock_repo, True)

        self.assertEqual(res, [pr2, pr])
````

- EN: This block implements local helper logic for test trymerge.
- CN: 该代码块实现与 test trymerge 相关的局部辅助逻辑。

### Lines 1087-1104 / 第 1087-1104 行

````python
        mock_repo.cherry_pick.assert_any_call("rev2")
        mock_repo.cherry_pick.assert_any_call("rev123")

        self.assertTrue(mock.call("rev1") not in mock_repo.cherry_pick.call_args_list)

        # Verify the first call
        message = mock_repo.amend_commit_message.call_args_list[0].args[0]
        prefix = (
            "[FSDP] Optimize away intermediate `div_` for HSDP (#106034)\n\n\r\n"
            "### Background: Gradient Pre-Divide"
        )
        suffix = (
            "\nPull Request resolved: https://github.com/pytorch/pytorch/pull/106034\nApproved by: \nghstack "
            "dependencies: #106032, #106033\n"
        )

        self.assertTrue(message.startswith(prefix))
        self.assertTrue(message.endswith(suffix))
````

- EN: This block iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions.
- CN: 该代码块遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互。

### Lines 1106-1125 / 第 1106-1125 行

````python
        # Verify the second call
        mock_repo.amend_commit_message.assert_any_call(
            "[FSDP] Break up `_post_backward_hook` into smaller funcs (#106068)\n\n\n"
            "Differential Revision: ["
            "D47852461](https://our.internmc.facebook.com/intern/diff/D47852461)\n"
            "Pull Request resolved: "
            "https://github.com/pytorch/pytorch/pull/106068\n"
            "Approved by: \n"
            "ghstack dependencies: #106032, #106033, #106034\n"
        )


@mock.patch("trymerge.gh_graphql", side_effect=mocked_gh_graphql)
@mock.patch("trymerge.gh_fetch_merge_base", return_value="")
@mock.patch(
    "trymerge.get_drci_classifications", side_effect=mocked_drci_classifications
)
@mock.patch.object(DummyGitRepo, "commit_message")
class TestRevListToPR(TestCase):
    # Tests for _revlist_to_prs function
````

- EN: This block introduces classes like `TestRevListToPR`; iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义新的类来封装状态与行为；遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互。

### Lines 1126-1147 / 第 1126-1147 行

````python
    def test__revlist_to_prs_zero_matches(
        self, mock_commit_message: mock.MagicMock, *args: Any
    ) -> None:
        # If zero PRs are mentioned in the commit message, it should raise an error
        pr_num = 154098
        pr = GitHubPR("pytorch", "pytorch", pr_num)
        repo = DummyGitRepo()
        mock_commit_message.return_value = "no PRs"
        self.assertRaisesRegex(
            RuntimeError,
            "PRs mentioned in commit dummy: 0.",
            lambda: _revlist_to_prs(repo, pr, ["dummy"]),
        )

    def test__revlist_to_prs_two_prs(
        self, mock_commit_message: mock.MagicMock, *args: Any
    ) -> None:
        # If two PRs are mentioned in the commit message, it should raise an error
        pr_num = 154394
        pr = GitHubPR("pytorch", "pytorch", pr_num)
        repo = DummyGitRepo()
        # https://github.com/pytorch/pytorch/commit/343c56e7650f55fd030aca0b9275d6d73501d3f4
````

- EN: This block defines callable units such as `test__revlist_to_prs_zero_matches`, `test__revlist_to_prs_two_prs`; branches on runtime conditions or configuration choices; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 1149-1170 / 第 1149-1170 行

````python
        commit_message = """add sticky cache pgo

ghstack-source-id: 9bc6dee0b427819f978bfabccb72727ba8be2f81
Pull-Request-resolved: https://github.com/pytorch/pytorch/pull/154098

ghstack-source-id: 9bc6dee0b427819f978bfabccb72727ba8be2f81
Pull Request resolved: https://github.com/pytorch/pytorch/pull/154394"""
        mock_commit_message.return_value = commit_message
        self.assertRaisesRegex(
            RuntimeError,
            "PRs mentioned in commit dummy: 2.",
            lambda: _revlist_to_prs(repo, pr, ["dummy"]),
        )


@mock.patch("trymerge.gh_graphql", side_effect=mocked_gh_graphql)
@mock.patch("trymerge.gh_fetch_merge_base", return_value="")
@mock.patch(
    "trymerge.get_drci_classifications", side_effect=mocked_drci_classifications
)
class TestTimelineFunctions(TestCase):
    """Tests for the new timeline-related functions"""
````

- EN: This block introduces classes like `TestTimelineFunctions`; iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义新的类来封装状态与行为；遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互。

### Lines 1172-1191 / 第 1172-1191 行

````python
    def test_sha_from_committed_event(self, *args: Any) -> None:
        """Test extracting SHA from committed event"""
        # Based on actual GitHub API format - committed events have "sha" at top level
        event = {
            "event": "committed",
            "sha": "fb21ce932ded6670c918804a0d9151b773770a7c",
        }
        self.assertEqual(
            sha_from_committed_event(event), "fb21ce932ded6670c918804a0d9151b773770a7c"
        )

        # Test with missing SHA
        event_no_sha = {"event": "committed"}
        self.assertIsNone(sha_from_committed_event(event_no_sha))

    def test_sha_from_force_push_after(self, *args: Any) -> None:
        """Test extracting SHA from force push event"""
        # NOTE: The current function doesn't handle the actual GitHub API format
        # Real force push events have "commit_id" at top level, but this function
        # looks for "after", "after_commit", "after_sha", or "head_sha" fields
````

- EN: This block defines callable units such as `test_sha_from_committed_event`, `test_sha_from_force_push_after`; iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 1193-1215 / 第 1193-1215 行

````python
        # Test with the legacy format the current function handles
        event_legacy = {
            "event": "head_ref_force_pushed",
            "after": {"sha": "ef22bcbc54bb0f787e1e4ffd3d83df18fc407f5e"},
        }
        self.assertEqual(
            sha_from_force_push_after(event_legacy),
            "ef22bcbc54bb0f787e1e4ffd3d83df18fc407f5e",
        )

        # Test with current GitHub API format (should return None with current implementation)
        event_real_api = {
            "event": "head_ref_force_pushed",
            "commit_id": "ef22bcbc54bb0f787e1e4ffd3d83df18fc407f5e",
        }
        self.assertEqual(
            sha_from_force_push_after(event_real_api),
            "ef22bcbc54bb0f787e1e4ffd3d83df18fc407f5e",
        )  # Current function doesn't handle commit_id

        # Test with missing SHA
        event_no_sha = {"event": "head_ref_force_pushed"}
        self.assertIsNone(sha_from_force_push_after(event_no_sha))
````

- EN: This block returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 1217-1239 / 第 1217-1239 行

````python
    @mock.patch("trymerge.gh_fetch_json_list")
    def test_iter_issue_timeline_until_comment(
        self, mock_gh_fetch_json_list: Any, *args: Any
    ) -> None:
        """Test timeline iteration until target comment"""
        # Mock timeline data based on actual GitHub API format
        timeline_data = [
            {"event": "commented", "id": 100, "body": "first comment"},
            {"event": "committed", "sha": "fb21ce932ded6670c918804a0d9151b773770a7c"},
            {"event": "commented", "id": 200, "body": "target comment"},
            {"event": "commented", "id": 300, "body": "after target"},
        ]
        mock_gh_fetch_json_list.return_value = timeline_data

        # Test iteration stops at target comment
        events = list(iter_issue_timeline_until_comment("pytorch", "pytorch", 123, 200))
        self.assertEqual(len(events), 3)  # Should stop at target comment
        self.assertEqual(events[0]["event"], "commented")
        self.assertEqual(events[0]["id"], 100)
        self.assertEqual(events[1]["event"], "committed")
        self.assertEqual(events[1]["sha"], "fb21ce932ded6670c918804a0d9151b773770a7c")
        self.assertEqual(events[2]["event"], "commented")
        self.assertEqual(events[2]["id"], 200)
````

- EN: This block defines callable units such as `test_iter_issue_timeline_until_comment`; touches GitHub-related data or remote service interactions; serializes or parses structured data files; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件；作为测试代码块校验预期行为。

### Lines 1241-1264 / 第 1241-1264 行

````python
    @mock.patch("trymerge.gh_fetch_json_list")
    def test_iter_issue_timeline_until_comment_not_found(
        self, mock_gh_fetch_json_list: Any, *args: Any
    ) -> None:
        """Test timeline iteration when target comment is not found"""
        # Mock empty timeline
        mock_gh_fetch_json_list.return_value = []

        events = list(iter_issue_timeline_until_comment("pytorch", "pytorch", 123, 999))
        self.assertEqual(len(events), 0)

    @mock.patch("trymerge.iter_issue_timeline_until_comment")
    def test_get_commit_sha_at_comment_commit_after_comment(
        self, mock_iter_timeline: Any, *args: Any
    ) -> None:
        """Test get_commit_sha_at_comment returns correct SHA after comment"""
        mock_iter_timeline.return_value = [
            {"event": "committed", "sha": "commit1"},
            {"event": "committed", "sha": "commit2"},
            {"event": "commented", "id": 100},
            {"event": "head_ref_force_pushed", "after": {"sha": "commit3"}},
        ]
        pr = GitHubPR("pytorch", "pytorch", 77700)
        sha = pr.get_commit_sha_at_comment(100)
````

- EN: This block defines callable units such as `test_iter_issue_timeline_until_comment_not_found`, `test_get_commit_sha_at_comment_commit_after_comment`; touches GitHub-related data or remote service interactions; serializes or parses structured data files; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件；作为测试代码块校验预期行为。

### Lines 1265-1281 / 第 1265-1281 行

````python
        self.assertEqual(sha, "commit2")

    @mock.patch("trymerge.iter_issue_timeline_until_comment")
    def test_get_commit_sha_at_comment_force_push_before_comment(
        self, mock_iter_timeline: Any, *args: Any
    ) -> None:
        mock_iter_timeline.return_value = [
            {"event": "committed", "sha": "commit1"},
            {"event": "committed", "sha": "commit2"},
            {"event": "head_ref_force_pushed", "commit_id": "commit3"},
            {"event": "commented", "id": 100},
        ]
        pr = GitHubPR("pytorch", "pytorch", 77700)
        sha = pr.get_commit_sha_at_comment(100)
        self.assertEqual(sha, "commit3")

    @mock.patch("trymerge.iter_issue_timeline_until_comment")
````

- EN: This block defines callable units such as `test_get_commit_sha_at_comment_force_push_before_comment`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 1282-1305 / 第 1282-1305 行

````python
    def test_get_commit_sha_at_comment_force_push_before_comment_legacy_mode(
        self, mock_iter_timeline: Any, *args: Any
    ) -> None:
        mock_iter_timeline.return_value = [
            {"event": "committed", "sha": "commit1"},
            {"event": "committed", "sha": "commit2"},
            {"event": "head_ref_force_pushed", "after": {"sha": "commit3"}},
            {"event": "commented", "id": 100},
        ]
        pr = GitHubPR("pytorch", "pytorch", 77700)
        sha = pr.get_commit_sha_at_comment(100)
        self.assertEqual(sha, "commit3")

    @mock.patch("trymerge.iter_issue_timeline_until_comment")
    def test_get_commit_sha_at_comment_multiple_comments(
        self, mock_iter_timeline: Any, *args: Any
    ) -> None:
        mock_iter_timeline.return_value = [
            {"event": "committed", "sha": "commit1"},
            {"event": "commented", "id": 100},
            {"event": "committed", "sha": "commit2"},
            {"event": "commented", "id": 200},
            {"event": "head_ref_force_pushed", "after": {"sha": "commit3"}},
            {"event": "commented", "id": 300},
````

- EN: This block defines callable units such as `test_get_commit_sha_at_comment_force_push_before_comment_legacy_mode`, `test_get_commit_sha_at_comment_multiple_comments`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 1306-1325 / 第 1306-1325 行

````python
        ]
        pr = GitHubPR("pytorch", "pytorch", 77700)
        sha = pr.get_commit_sha_at_comment(200)
        self.assertEqual(sha, "commit2")
        sha = pr.get_commit_sha_at_comment(300)
        self.assertEqual(sha, "commit3")

    @mock.patch("trymerge.iter_issue_timeline_until_comment")
    def test_get_commit_sha_at_comment_no_events(
        self, mock_iter_timeline: Any, *args: Any
    ) -> None:
        mock_iter_timeline.return_value = [
            {"event": "commented", "id": 100},
            {"event": "labeled", "label": {"name": "test"}},
        ]
        pr = GitHubPR("pytorch", "pytorch", 77700)
        sha = pr.get_commit_sha_at_comment(100)
        self.assertIsNone(sha)

    @mock.patch("trymerge.iter_issue_timeline_until_comment")
````

- EN: This block defines callable units such as `test_get_commit_sha_at_comment_no_events`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 1326-1332 / 第 1326-1332 行

````python
    def test_get_commit_sha_at_comment_exception(
        self, mock_iter_timeline: Any, *args: Any
    ) -> None:
        mock_iter_timeline.side_effect = Exception("API error")
        pr = GitHubPR("pytorch", "pytorch", 77700)
        sha = pr.get_commit_sha_at_comment(100)
        self.assertIsNone(sha)
````

- EN: This block defines callable units such as `test_get_commit_sha_at_comment_exception`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 1335-1336 / 第 1335-1336 行

````python
if __name__ == "__main__":
    main()
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Configures native build inputs, optional backends, and dependency discovery at configure time.
  CN: 在配置阶段设置原生构建输入、可选后端以及依赖发现逻辑。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `__future__`, `gzip`, `json`, `os`, `warnings`, `hashlib`, `typing`, `unittest`, ...
- Internal imports / 内部导入: none
- External imports / 外部导入: `github_utils`, `gitutils`, `trymerge`
- Classes / 类: `DummyGitRepo`, `TestTryMerge`, `TestBypassFailures`, `TestBypassFailuresOnSandCastle`, `TestGitHubPRGhstackDependencies`, `TestRevListToPR`, `TestTimelineFunctions`
- Functions / 函数: `mock_query`, `mocked_gh_graphql`, `mocked_drci_classifications`, `mock_parse_args`, `mock_remove_label`, `mock_revert`, `mock_merge`, `mock_gh_get_info`, `mocked_read_merge_rules_NE`, `mocked_read_merge_rules`, ...
- Test entry points / 测试入口: none
