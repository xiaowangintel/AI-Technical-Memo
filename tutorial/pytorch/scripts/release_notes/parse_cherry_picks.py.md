# parse_cherry_picks.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `scripts/release_notes/parse_cherry_picks.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements release-note tooling that classifies commits, applies categories, and assembles changelog inputs.
- **用途 (CN)**: 实现发布说明工具，用于分类提交、应用类别并组装变更日志输入。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13 / 第 1-13 行

````python
#!/usr/bin/env python3
"""Parse cherry-pick comments from a PyTorch GitHub issue and extract trunk PR info.

Usage:
    python scripts/release_notes/parse_cherry_picks.py \\
        https://github.com/pytorch/pytorch/issues/170119 \\
        --commitlist scripts/release_notes/results/commitlist.csv

Outputs a CSV file to scripts/release_notes/results/cherry_picks_<issue_number>.csv.
Columns: comment_id, pr_number, pr_title, commit_sha
Validates that each commit_sha matches an entry in the commitlist and logs
warnings for any mismatches.
"""
````

- EN: This block iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 15-26 / 第 15-26 行

````python
import argparse
import csv
import json
import logging
import re
import subprocess
import sys
from pathlib import Path


logging.basicConfig(level=logging.INFO, format="%(levelname)s: %(message)s")
logger = logging.getLogger(__name__)
````

- EN: This block imports dependencies such as `argparse`, `csv`, `json`, `logging`; parses command-line arguments for script entry points; serializes or parses structured data files.
- CN: 该代码块导入当前模块运行所需的依赖；为脚本入口解析命令行参数；序列化或解析结构化数据文件。

### Lines 29-46 / 第 29-46 行

````python
def parse_issue_url(url: str) -> tuple[str, str]:
    """Extract owner/repo and issue number from a GitHub issue URL."""
    m = re.match(r"https://github\.com/([^/]+/[^/]+)/issues/(\d+)", url)  # @lint-ignore
    if not m:
        raise ValueError(f"Invalid GitHub issue URL: {url}")
    return m.group(1), m.group(2)


def gh_api(endpoint: str) -> list | dict:
    """Call gh api with pagination and return parsed JSON."""
    result = subprocess.run(
        ["gh", "api", endpoint, "--paginate"],
        capture_output=True,
        text=True,
        check=True,
    )
    # --paginate may return multiple JSON arrays concatenated; we need to handle that
    # gh api --paginate with --jq is cleaner, but let's parse raw output
````

- EN: This block defines callable units such as `parse_issue_url`, `gh_api`; branches on runtime conditions or configuration choices; returns computed values to callers; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 47-64 / 第 47-64 行

````python
    # When paginating, gh outputs one JSON array per page on separate "lines"
    output = result.stdout.strip()
    if not output:
        return []

    # Try parsing as a single JSON value first
    try:
        return json.loads(output)
    except json.JSONDecodeError:
        pass

    # If that fails, it's multiple JSON arrays concatenated
    # Split on ][ boundaries and merge
    all_items = []
    decoder = json.JSONDecoder()
    pos = 0
    while pos < len(output):
        # Skip whitespace
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 65-75 / 第 65-75 行

````python
        while pos < len(output) and output[pos] in " \t\n\r":
            pos += 1
        if pos >= len(output):
            break
        obj, end_pos = decoder.raw_decode(output, pos)
        if isinstance(obj, list):
            all_items.extend(obj)
        else:
            all_items.append(obj)
        pos = end_pos
    return all_items
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 78-95 / 第 78-95 行

````python
def fetch_comments(repo: str, issue_number: str) -> list[dict]:
    """Fetch all comments from a GitHub issue."""
    endpoint = f"repos/{repo}/issues/{issue_number}/comments?per_page=100"
    comments = gh_api(endpoint)
    logger.info(f"Fetched {len(comments)} comments from {repo}#{issue_number}")
    return comments


def fetch_pr_title(repo: str, pr_number: str) -> str:
    """Fetch the title of a PR."""
    try:
        result = subprocess.run(
            ["gh", "api", f"repos/{repo}/pulls/{pr_number}", "--jq", ".title"],
            capture_output=True,
            text=True,
            check=True,
        )
        return result.stdout.strip()
````

- EN: This block defines callable units such as `fetch_comments`, `fetch_pr_title`; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 96-113 / 第 96-113 行

````python
    except subprocess.CalledProcessError as e:
        logger.warning(f"Failed to fetch title for PR #{pr_number}: {e}")
        return ""


def fetch_landed_commit(repo: str, pr_number: str) -> str:
    """Fetch the actual landed commit SHA for a PR.

    PyTorch uses a merge bot that squash-merges outside of GitHub's standard
    merge mechanism, so the PR API's merge_commit_sha is unreliable.

    Strategy:
    1. Issue events API: look for the 'closed' event with a commit_id
    2. Commit search API: search for commits containing '(#NNNNN)' in message
    """
    # Strategy 1: Issue events API
    try:
        result = subprocess.run(
````

- EN: This block defines callable units such as `fetch_landed_commit`; iterates through collections, records, or generated items; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 114-130 / 第 114-130 行

````python
            [
                "gh",
                "api",
                f"repos/{repo}/issues/{pr_number}/events?per_page=100",
                "--paginate",
                "--jq",
                '.[] | select(.event == "closed" and .commit_id) | .commit_id',
            ],
            capture_output=True,
            text=True,
            check=True,
        )
        commit_ids = result.stdout.strip().splitlines()
        if commit_ids and commit_ids[0]:
            return commit_ids[0]
    except subprocess.CalledProcessError:
        pass
````

- EN: This block branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 132-149 / 第 132-149 行

````python
    # Strategy 2: Commit search API
    logger.info(f"  Events API had no commit for PR #{pr_number}, trying search API...")
    try:
        result = subprocess.run(
            [
                "gh",
                "api",
                f'search/commits?q=repo:{repo}+"(#{pr_number})"',
                "--jq",
                ".items[0].sha",
            ],
            capture_output=True,
            text=True,
            check=True,
        )
        sha = result.stdout.strip()
        if sha and sha != "null":
            return sha
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 150-167 / 第 150-167 行

````python
    except subprocess.CalledProcessError:
        pass

    logger.warning(f"Could not find landed commit for PR #{pr_number}")
    return ""


def extract_trunk_prs(comment_body: str) -> list[dict]:
    """Extract trunk PR numbers/commit hashes from a comment body.

    Returns a list of dicts with keys: pr_number (str or ""), raw_commit (str or "").
    Returns an empty list if the comment doesn't contain the trunk PR section.
    """
    # Find the "Link to landed trunk PR" section
    # Handle variations: "Link to landed trunk PR", "Link to the landed trunk PR"
    trunk_pattern = re.compile(
        r"Link to (?:the )?landed trunk PR[^:]*:\s*\n(.*?)(?=Li[nn][kt] to (?:the )?release branch PR|Criteria Category:|$)",
        re.DOTALL | re.IGNORECASE,
````

- EN: This block defines callable units such as `extract_trunk_prs`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 168-179 / 第 168-179 行

````python
    )
    match = trunk_pattern.search(comment_body)
    if not match:
        return []

    trunk_section = match.group(1).strip()

    # Check for NA/N/A
    if re.match(r"^\*?\s*-?\s*N/?A\s*$", trunk_section, re.IGNORECASE):
        return [{"pr_number": "", "raw_commit": ""}]

    results = []
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 181-194 / 第 181-194 行

````python
    # Extract PR URLs
    pr_urls = re.findall(
        r"https://github\.com/pytorch/pytorch/pull/(\d+)",  # @lint-ignore
        trunk_section,
    )
    for pr_num in pr_urls:
        results.append({"pr_number": pr_num, "raw_commit": ""})

    # Extract raw commit hashes (40-char hex) that aren't part of a URL
    # Remove URLs first to avoid matching hashes embedded in URLs
    section_no_urls = re.sub(r"https://\S+", "", trunk_section)
    raw_commits = re.findall(r"\b([0-9a-f]{40})\b", section_no_urls)
    for commit_hash in raw_commits:
        results.append({"pr_number": "", "raw_commit": commit_hash})
````

- EN: This block iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions.
- CN: 该代码块遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互。

### Lines 196-213 / 第 196-213 行

````python
    # If we found the section but nothing matched, treat as empty/NA
    if not results:
        return [{"pr_number": "", "raw_commit": ""}]

    return results


def main():
    parser = argparse.ArgumentParser(
        description="Parse cherry-pick comments from a PyTorch GitHub issue"
    )
    parser.add_argument("issue_url", help="GitHub issue URL to parse")
    parser.add_argument(
        "-o",
        "--output",
        default=None,
        help="Output CSV file path (default: results/cherry_picks_<issue_number>.csv relative to script)",
    )
````

- EN: This block defines callable units such as `main`; parses command-line arguments for script entry points; branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；为脚本入口解析命令行参数；依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 214-230 / 第 214-230 行

````python
    parser.add_argument(
        "--commitlist",
        required=True,
        help="Path to commitlist.csv to validate commit hashes against",
    )
    args = parser.parse_args()

    repo, issue_number = parse_issue_url(args.issue_url)
    if args.output:
        output_path = args.output
    else:
        script_dir = Path(__file__).resolve().parent
        results_dir = script_dir / "results"
        results_dir.mkdir(exist_ok=True)
        output_path = str(results_dir / f"cherry_picks_{issue_number}.csv")

    comments = fetch_comments(repo, issue_number)
````

- EN: This block parses command-line arguments for script entry points; branches on runtime conditions or configuration choices; serializes or parses structured data files.
- CN: 该代码块为脚本入口解析命令行参数；依据运行时条件或配置选择进行分支；序列化或解析结构化数据文件。

### Lines 232-248 / 第 232-248 行

````python
    rows = []
    for comment in comments:
        comment_id = comment["id"]
        body = comment["body"] or ""

        trunk_prs = extract_trunk_prs(body)
        if not trunk_prs:
            # Comment doesn't contain the trunk PR section — skip
            continue

        if len(trunk_prs) > 1:
            logger.warning(
                f"Comment {comment_id} has {len(trunk_prs)} trunk PRs: "
                + ", ".join(
                    p["pr_number"] or p["raw_commit"] or "N/A" for p in trunk_prs
                )
            )
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 250-267 / 第 250-267 行

````python
        for pr_info in trunk_prs:
            pr_number = pr_info["pr_number"]
            raw_commit = pr_info["raw_commit"]
            pr_title = ""
            commit_sha = raw_commit  # Use raw commit if provided

            if pr_number:
                logger.info(f"Fetching info for PR #{pr_number}...")
                pr_title = fetch_pr_title(repo, pr_number)
                commit_sha = fetch_landed_commit(repo, pr_number)

            rows.append(
                {
                    "comment_id": comment_id,
                    "pr_number": pr_number,
                    "pr_title": pr_title,
                    "commit_sha": commit_sha,
                }
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 268-279 / 第 268-279 行

````python
            )

    # Validate against commitlist
    commitlist_hashes = set()
    with open(args.commitlist, newline="") as f:
        reader = csv.reader(f)
        next(reader)  # skip header
        for row in reader:
            if row:
                commitlist_hashes.add(row[0])

    logger.info(f"Loaded {len(commitlist_hashes)} hashes from {args.commitlist}")
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; serializes or parses structured data files.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；序列化或解析结构化数据文件。

### Lines 281-298 / 第 281-298 行

````python
    matched = 0
    mismatched = 0
    skipped = 0
    for row in rows:
        sha = row["commit_sha"]
        if not sha:
            skipped += 1
            continue
        # commitlist uses abbreviated hashes; check if any is a prefix of
        # our full hash, or vice versa
        if any(
            sha.startswith(cl_hash) or cl_hash.startswith(sha)
            for cl_hash in commitlist_hashes
        ):
            matched += 1
        else:
            mismatched += 1
            logger.warning(
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 299-315 / 第 299-315 行

````python
                f"Commit {sha[:11]} (PR #{row['pr_number'] or 'N/A'}) "
                f"not found in commitlist"
            )

    logger.info(
        f"Commitlist validation: {matched} matched, {mismatched} not found, "
        f"{skipped} skipped (no hash)"
    )

    # Write CSV
    fieldnames = ["comment_id", "pr_number", "pr_title", "commit_sha"]
    with open(output_path, "w", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(rows)

    logger.info(f"Wrote {len(rows)} rows to {output_path}")
````

- EN: This block serializes or parses structured data files.
- CN: 该代码块序列化或解析结构化数据文件。

### Lines 318-319 / 第 318-319 行

````python
if __name__ == "__main__":
    main()
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。


## Key Concepts / 关键概念
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。
- EN: CLI entry points — the module exposes command-line arguments so developers or CI jobs can drive automation predictably.
  CN: 命令行入口——该模块暴露命令行参数，使开发者或 CI 作业能够以可预测方式驱动自动化逻辑。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `argparse`, `csv`, `json`, `logging`, `re`, `subprocess`, `sys`, `pathlib`
- Internal imports / 内部导入: none
- External imports / 外部导入: none
- Classes / 类: none
- Functions / 函数: `parse_issue_url`, `gh_api`, `fetch_comments`, `fetch_pr_title`, `fetch_landed_commit`, `extract_trunk_prs`, `main`
- Test entry points / 测试入口: none
