# failures_histogram.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `scripts/compile_tests/failures_histogram.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Provides Python helpers that download, aggregate, and summarize compile-test reporting artifacts.
- **用途 (CN)**: 提供 Python 辅助工具，用于下载、聚合并汇总编译测试报告产物。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13 / 第 1-13 行

````python
import argparse
import re

from common import download_reports, get_testcases, key, open_test_results, skipped_test
from passrate import compute_pass_rate


"""
python failures_histogram.py commit_sha

Analyzes skip reasons for Dynamo tests and prints a histogram with repro
commands. You'll need to provide the commit_sha for a commit on the main branch,
from which we will pull CI test results.
````

- EN: This block imports dependencies such as `argparse`, `re`, `common`, `passrate`; parses command-line arguments for script entry points; iterates through collections, records, or generated items.
- CN: 该代码块导入当前模块运行所需的依赖；为脚本入口解析命令行参数；遍历集合、记录或生成项。

### Lines 15-26 / 第 15-26 行

````python
This script requires the `gh` cli. You'll need to install it and then
authenticate with it via `gh auth login` before using this script.
https://docs.github.com/en/github-cli/github-cli/quickstart
"""


def skip_reason(testcase):
    for child in testcase.iter():
        if child.tag != "skipped":
            continue
        return child.attrib["message"]
    raise AssertionError("no message?")
````

- EN: This block defines callable units such as `skip_reason`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互。

### Lines 29-42 / 第 29-42 行

````python
def skip_reason_normalized(testcase):
    for child in testcase.iter():
        if child.tag != "skipped":
            continue
        result = child.attrib["message"].split("\n")[0]
        result = result.split(">")[0]
        result = re.sub(r"0x\w+", "0xDEADBEEF", result)
        result = re.sub(r"MagicMock id='\d+'", "MagicMock id='0000000000'", result)
        result = re.sub(r"issues/\d+", "issues/XXX", result)
        result = re.sub(r"torch.Size\(\[.*\]\)", "torch.Size([...])", result)
        result = re.sub(
            r"Could not get qualified name for class '.*'",
            "Could not get qualified name for class",
            result,
````

- EN: This block defines callable units such as `skip_reason_normalized`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 43-56 / 第 43-56 行

````python
        )
        return result
    raise AssertionError("no message?")


def get_failures(testcases):
    skipped = [t for t in testcases if skipped_test(t)]
    skipped_dict = {}
    for s in skipped:
        reason = skip_reason_normalized(s)
        if reason not in skipped_dict:
            skipped_dict[reason] = []
        skipped_dict[reason].append(s)
    result = []
````

- EN: This block defines callable units such as `get_failures`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; validates assumptions and surfaces failures explicitly.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；校验前提条件并显式暴露失败。

### Lines 57-68 / 第 57-68 行

````python
    for s, v in skipped_dict.items():
        result.append((len(v), s, v))
    result.sort(reverse=True)
    return result


def repro(testcase):
    return f"PYTORCH_TEST_WITH_DYNAMO=1 pytest {testcase.attrib['file']} -v -k {testcase.attrib['name']}"


def all_tests(testcase):
    return f"{testcase.attrib['file']}::{testcase.attrib['classname']}.{testcase.attrib['name']}"
````

- EN: This block defines callable units such as `repro`, `all_tests`; iterates through collections, records, or generated items; returns computed values to callers; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；向调用方返回计算结果；作为测试代码块校验预期行为。

### Lines 71-84 / 第 71-84 行

````python
# e.g. "17c5f69852/eager", "17c5f69852/dynamo"
def failures_histogram(eager_dir, dynamo_dir, verbose=False, format_issues=False):
    fail_keys = compute_pass_rate(eager_dir, dynamo_dir)
    xmls = open_test_results(dynamo_dir)

    testcases = get_testcases(xmls)
    testcases = [t for t in testcases if key(t) in fail_keys]
    dct = get_failures(testcases)

    result = []
    for count, reason, testcases in dct:
        if verbose:
            row = (
                count,
````

- EN: This block defines callable units such as `failures_histogram`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 85-98 / 第 85-98 行

````python
                reason,
                repro(testcases[0]),
                [all_tests(t) for t in testcases],
            )
        else:
            row = (count, reason, repro(testcases[0]))
        result.append(row)

    header = (
        "(num_failed_tests, error_msg, sample_test, all_tests)"
        if verbose
        else "(num_failed_tests, error_msg, sample_test)"
    )
    print(header)
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 99-105 / 第 99-105 行

````python
    sum_counts = sum(r[0] for r in result)
    for row in result:
        if format_issues:
            print(as_issue(*row))
        else:
            print(row)
    print("[counts]", sum_counts)
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 108-116 / 第 108-116 行

````python
def as_issue(count, msg, repro, tests):
    tests = "\n".join(tests)
    result = f"""
{"-" * 50}
{count} Dynamo test are failing with \"{msg}\".

## Repro

`{repro}`
````

- EN: This block defines callable units such as `as_issue`.
- CN: 该代码块定义可调用函数来执行具体自动化步骤。

### Lines 118-131 / 第 118-131 行

````python
You will need to remove the skip or expectedFailure before running the repro command.
This may be just removing a sentinel file from in
[dynamo_expected_failures](https://github.com/pytorch/pytorch/blob/main/test/dynamo_expected_failures)
or [dynamo_skips](https://github.com/pytorch/pytorch/blob/main/test/dynamo_skips).


## Failing tests

Here's a comprehensive list of tests that fail (as of this issue) with the above message:
<details>
<summary>Click me</summary>
```
{tests}
```
````

- EN: This block touches GitHub-related data or remote service interactions.
- CN: 该代码块处理 GitHub 相关数据或远程服务交互。

### Lines 132-145 / 第 132-145 行

````python
</details>
"""
    return result


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        prog="failures_histogram",
        description="See statistics about skipped Dynamo tests",
    )
    parser.add_argument(
        "commit",
        help=(
            "The commit sha for the latest commit on a PR from which we will "
````

- EN: This block parses command-line arguments for script entry points; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块为脚本入口解析命令行参数；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 146-157 / 第 146-157 行

````python
            "pull CI test results, e.g. 7e5f597aeeba30c390c05f7d316829b3798064a5"
        ),
    )
    parser.add_argument(
        "-v", "--verbose", help="Prints all failing test names", action="store_true"
    )
    parser.add_argument(
        "--format-issues",
        help="Prints histogram in a way that they can be copy-pasted as a github issues",
        action="store_true",
    )
    args = parser.parse_args()
````

- EN: This block parses command-line arguments for script entry points; touches GitHub-related data or remote service interactions.
- CN: 该代码块为脚本入口解析命令行参数；处理 GitHub 相关数据或远程服务交互。

### Lines 159-162 / 第 159-162 行

````python
    # args.format_issues implies verbose=True
    verbose = args.verbose
    if args.format_issues:
        verbose = True
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。

### Lines 164-165 / 第 164-165 行

````python
    dynamo311, eager311 = download_reports(args.commit, ("dynamo311", "eager311"))
    failures_histogram(eager311, dynamo311, verbose, args.format_issues)
````

- EN: This block implements local helper logic for failures histogram.
- CN: 该代码块实现与 failures histogram 相关的局部辅助逻辑。


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
- EN: Test coverage — the code validates neighboring automation behavior with assertions and expected fixtures.
  CN: 测试覆盖——代码通过断言和预期样例验证邻近自动化逻辑。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `argparse`, `re`
- Internal imports / 内部导入: none
- External imports / 外部导入: `common`, `passrate`
- Classes / 类: none
- Functions / 函数: `skip_reason`, `skip_reason_normalized`, `get_failures`, `repro`, `all_tests`, `failures_histogram`, `as_issue`
- Test entry points / 测试入口: none
