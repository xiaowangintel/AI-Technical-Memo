# passrate.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `scripts/compile_tests/passrate.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Provides Python helpers that download, aggregate, and summarize compile-test reporting artifacts.
- **用途 (CN)**: 提供 Python 辅助工具，用于下载、聚合并汇总编译测试报告产物。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10 / 第 1-10 行

````python
import argparse

from common import (
    get_excluded_testcases,
    get_passed_testcases,
    get_testcases,
    key,
    open_test_results,
)
from download_reports import download_reports
````

- EN: This block imports dependencies such as `argparse`, `common`, `download_reports`; parses command-line arguments for script entry points.
- CN: 该代码块导入当前模块运行所需的依赖；为脚本入口解析命令行参数。

### Lines 13-24 / 第 13-24 行

````python
"""
Usage: passrate.py commit_sha

Parses test reports to measure the passrate. The passrate is defined as:

A) Take the number of tests that pass under eager mode, excluding
CUDA, OpInfo, and ModuleInfo tests
B) Of those tests, count the number of tests that pass under Dynamo
C) Take B/A.

You'll need to provide the commit_sha for a commit on the main branch,
from which we will pull CI test results.
````

- EN: This block imports dependencies such as `which we will pull CI test results.`; iterates through collections, records, or generated items.
- CN: 该代码块导入当前模块运行所需的依赖；遍历集合、记录或生成项。

### Lines 26-35 / 第 26-35 行

````python
This script requires the `gh` cli. You'll need to install it and then
authenticate with it via `gh auth login` before using this script.
https://docs.github.com/en/github-cli/github-cli/quickstart
"""


def testcases_by_time(xmls):
    testcases = get_testcases(xmls)
    testcases.sort(reverse=True, key=lambda x: float(x.attrib["time"]))
    return testcases
````

- EN: This block defines callable units such as `testcases_by_time`; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 38-50 / 第 38-50 行

````python
def should_exclude(key):
    test_file = key.split("::")[0]
    # C++ tests
    if test_file == "UNKNOWN":
        return True
    # Policy: "pass rate" does not include inductor, export, or dynamo tests.
    return test_file.startswith(("inductor/", "export/", "dynamo/"))


def compute_pass_rate(eager_dir, dynamo_dir):
    print("parsing xmls")
    eager_xmls = open_test_results(eager_dir)
    dynamo_xmls = open_test_results(dynamo_dir)
````

- EN: This block defines callable units such as `should_exclude`, `compute_pass_rate`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 52-62 / 第 52-62 行

````python
    print("computing pass rate")
    eager_passed = get_passed_testcases(eager_xmls)
    dynamo_passed = get_passed_testcases(dynamo_xmls)
    dynamo_pass_keys = {key(testcase) for testcase in dynamo_passed}
    dynamo_pass_keys = {key_ for key_ in dynamo_pass_keys if not should_exclude(key_)}
    tmp_eager_pass_keys = {key(testcase) for testcase in eager_passed}
    tmp_eager_pass_keys = {
        key_ for key_ in tmp_eager_pass_keys if not should_exclude(key_)
    }
    excluded = [key(t) for t in get_excluded_testcases(dynamo_xmls)]
    eager_pass_keys = tmp_eager_pass_keys - set(excluded)
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 64-76 / 第 64-76 行

````python
    subset = eager_pass_keys.intersection(dynamo_pass_keys)
    total_subset = len(subset)
    total_tests = len(eager_pass_keys)
    print("pass rate", total_subset / total_tests, total_subset, total_tests)

    dynamo_testcases = get_testcases(dynamo_xmls)
    tc = {key(t): t for t in dynamo_testcases}

    # Useful for debugging
    not_there_keys = set()
    for key_ in eager_pass_keys:
        if key_ not in tc:
            not_there_keys.add(key_)
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 78-91 / 第 78-91 行

````python
    fail_keys = eager_pass_keys - subset
    return fail_keys


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        prog="passrate", description="Computes the Dynamo unittest pass rate"
    )
    parser.add_argument(
        "commit",
        help=(
            "The commit sha for the latest commit on a PR from which we will "
            "pull CI test results, e.g. 7e5f597aeeba30c390c05f7d316829b3798064a5"
        ),
````

- EN: This block parses command-line arguments for script entry points; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块为脚本入口解析命令行参数；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 92-95 / 第 92-95 行

````python
    )
    args = parser.parse_args()
    dynamo311, eager311 = download_reports(args.commit, ("dynamo311", "eager311"))
    compute_pass_rate(eager311, dynamo311)
````

- EN: This block implements local helper logic for passrate.
- CN: 该代码块实现与 passrate 相关的局部辅助逻辑。


## Key Concepts / 关键概念
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Configures native build inputs, optional backends, and dependency discovery at configure time.
  CN: 在配置阶段设置原生构建输入、可选后端以及依赖发现逻辑。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。
- EN: CLI entry points — the module exposes command-line arguments so developers or CI jobs can drive automation predictably.
  CN: 命令行入口——该模块暴露命令行参数，使开发者或 CI 作业能够以可预测方式驱动自动化逻辑。
- EN: Test coverage — the code validates neighboring automation behavior with assertions and expected fixtures.
  CN: 测试覆盖——代码通过断言和预期样例验证邻近自动化逻辑。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `argparse`
- Internal imports / 内部导入: none
- External imports / 外部导入: `common`, `download_reports`
- Classes / 类: none
- Functions / 函数: `testcases_by_time`, `should_exclude`, `compute_pass_rate`
- Test entry points / 测试入口: none
