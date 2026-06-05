# test_map_ec2_to_arc.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `.github/scripts/test_map_ec2_to_arc.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements repository automation helpers used by CI, release engineering, or GitHub maintenance workflows. This particular file is a focused test module for adjacent automation helpers.
- **用途 (CN)**: 实现仓库自动化辅助脚本，供 CI、发布工程或 GitHub 维护流程使用。 该文件还是一个针对相邻自动化辅助逻辑的聚焦测试模块。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11 / 第 1-11 行

````python
#!/usr/bin/env python3

import json
import os
import subprocess
import sys
import tempfile
from pathlib import Path


SCRIPT = Path(__file__).resolve().parent / "map_ec2_to_arc.py"
````

- EN: This block imports dependencies such as `json`, `os`, `subprocess`, `sys`; serializes or parses structured data files.
- CN: 该代码块导入当前模块运行所需的依赖；序列化或解析结构化数据文件。

### Lines 14-28 / 第 14-28 行

````python
def run(
    matrix: str, prefix: str = "", github_output: str | None = None
) -> subprocess.CompletedProcess:
    cmd = [sys.executable, str(SCRIPT)]
    if prefix:
        cmd += ["--prefix", prefix]
    cmd.append(matrix)

    env = os.environ.copy()
    if github_output is not None:
        env["GITHUB_OUTPUT"] = github_output
    else:
        env.pop("GITHUB_OUTPUT", None)

    return subprocess.run(cmd, capture_output=True, text=True, env=env)
````

- EN: This block defines callable units such as `run`; branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 31-42 / 第 31-42 行

````python
def parse_output(stdout: str) -> dict:
    """Extract the JSON matrix from the 'Setting test-matrix=...' line."""
    prefix = "Setting test-matrix="
    for line in stdout.splitlines():
        if line.startswith(prefix):
            return json.loads(line[len(prefix) :])
    raise ValueError(f"no test-matrix output found in: {stdout}")


def check(condition: bool, msg: str = "") -> None:
    if not condition:
        raise AssertionError(msg)
````

- EN: This block defines callable units such as `parse_output`, `check`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; validates assumptions and surfaces failures explicitly; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；校验前提条件并显式暴露失败；序列化或解析结构化数据文件。

### Lines 45-54 / 第 45-54 行

````python
def test_basic_matrix():
    matrix = """{ include: [
      { config: "default", shard: 1, num_shards: 1, runner: "linux.4xlarge" },
      { config: "openreg", shard: 1, num_shards: 1, runner: "linux.2xlarge" },
    ]}"""
    result = run(matrix)
    check(result.returncode == 0, result.stderr)
    output = parse_output(result.stdout)
    runners = [e["runner"] for e in output["include"]]
    check(runners == ["l-x86iavx512-16-128", "l-x86iavx512-8-64"])
````

- EN: This block defines callable units such as `test_basic_matrix`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 57-74 / 第 57-74 行

````python
def test_matrix_with_prefix():
    matrix = """{ include: [
      { config: "default", shard: 1, num_shards: 7, runner: "mt-linux.4xlarge" },
      { config: "default", shard: 2, num_shards: 7, runner: "mt-linux.4xlarge" },
      { config: "openreg", shard: 1, num_shards: 1, runner: "mt-linux.2xlarge" },
    ]}"""
    result = run(matrix, prefix="mt-")
    check(result.returncode == 0, result.stderr)
    output = parse_output(result.stdout)
    runners = [e["runner"] for e in output["include"]]
    check(
        runners
        == [
            "mt-l-x86iavx512-16-128",
            "mt-l-x86iavx512-16-128",
            "mt-l-x86iavx512-8-64",
        ]
    )
````

- EN: This block defines callable units such as `test_matrix_with_prefix`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 77-93 / 第 77-93 行

````python
def test_matrix_without_prefix_when_none_present():
    matrix = """{ include: [
      { config: "default", shard: 1, num_shards: 1, runner: "linux.g5.4xlarge.nvidia.gpu" },
    ]}"""
    result = run(matrix)
    check(result.returncode == 0, result.stderr)
    output = parse_output(result.stdout)
    check(output["include"][0]["runner"] == "l-x86aavx2-29-113-a10g")


def test_unknown_runner_fails():
    matrix = """{ include: [
      { config: "default", shard: 1, num_shards: 1, runner: "bogus.runner" },
    ]}"""
    result = run(matrix)
    check(result.returncode == 1)
    check("no ARC runner found for 'bogus.runner'" in result.stderr)
````

- EN: This block defines callable units such as `test_matrix_without_prefix_when_none_present`, `test_unknown_runner_fails`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 96-104 / 第 96-104 行

````python
def test_prefix_not_present_on_runner():
    """When --prefix is given but a runner doesn't have it, the raw label is used."""
    matrix = """{ include: [
      { config: "default", shard: 1, num_shards: 1, runner: "linux.4xlarge" },
    ]}"""
    result = run(matrix, prefix="mt-")
    check(result.returncode == 0, result.stderr)
    output = parse_output(result.stdout)
    check(output["include"][0]["runner"] == "mt-l-x86iavx512-16-128")
````

- EN: This block defines callable units such as `test_prefix_not_present_on_runner`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 107-117 / 第 107-117 行

````python
def test_preserves_non_runner_fields():
    matrix = """{ include: [
      { config: "default", shard: 3, num_shards: 7, runner: "linux.large" },
    ]}"""
    result = run(matrix)
    check(result.returncode == 0, result.stderr)
    entry = parse_output(result.stdout)["include"][0]
    check(entry["config"] == "default")
    check(entry["shard"] == 3)
    check(entry["num_shards"] == 7)
    check(entry["runner"] == "l-x86iavx512-2-4")
````

- EN: This block defines callable units such as `test_preserves_non_runner_fields`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 120-130 / 第 120-130 行

````python
def test_empty_include_passes_through():
    matrix = """{ include: [] }"""
    result = run(matrix)
    check(result.returncode == 0, result.stderr)
    output = parse_output(result.stdout)
    check(output == {"include": []}, f"expected empty include, got {output}")


def test_empty_string_passes_through():
    result = run("")
    check(result.returncode == 0, result.stderr)
````

- EN: This block defines callable units such as `test_empty_include_passes_through`, `test_empty_string_passes_through`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 133-150 / 第 133-150 行

````python
def test_mixed_runners():
    matrix = """{ include: [
      { config: "default", shard: 1, num_shards: 1, runner: "linux.4xlarge" },
      { config: "gpu", shard: 1, num_shards: 1, runner: "linux.g5.4xlarge.nvidia.gpu" },
      { config: "arm", shard: 1, num_shards: 1, runner: "linux.arm64.2xlarge" },
    ]}"""
    result = run(matrix)
    check(result.returncode == 0, result.stderr)
    output = parse_output(result.stdout)
    runners = [e["runner"] for e in output["include"]]
    check(
        runners
        == [
            "l-x86iavx512-16-128",
            "l-x86aavx2-29-113-a10g",
            "l-arm64g2-6-32",
        ]
    )
````

- EN: This block defines callable units such as `test_mixed_runners`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 153-166 / 第 153-166 行

````python
def test_passthrough_runner_no_prefix():
    """Passthrough runners (ROCm, XPU) should not get the OSDC prefix."""
    matrix = """{ include: [
      { config: "default", shard: 1, num_shards: 3, runner: "linux.rocm.gpu.2" },
      { config: "default", shard: 1, num_shards: 1, runner: "linux.idc.xpu" },
    ]}"""
    result = run(matrix, prefix="mt-")
    check(result.returncode == 0, result.stderr)
    output = parse_output(result.stdout)
    runners = [e["runner"] for e in output["include"]]
    check(
        runners == ["linux.rocm.gpu.2", "linux.idc.xpu"],
        f"passthrough runners should not get prefix, got {runners}",
    )
````

- EN: This block defines callable units such as `test_passthrough_runner_no_prefix`; iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 169-179 / 第 169-179 行

````python
def test_github_output_file():
    """When GITHUB_OUTPUT is set, the script writes test-matrix to that file."""
    matrix = """{ include: [
      { config: "default", shard: 1, num_shards: 1, runner: "linux.2xlarge" },
    ]}"""
    with tempfile.NamedTemporaryFile(mode="w", suffix=".txt", delete=False) as f:
        tmp_path = f.name

    try:
        result = run(matrix, github_output=tmp_path)
        check(result.returncode == 0, result.stderr)
````

- EN: This block defines callable units such as `test_github_output_file`; touches GitHub-related data or remote service interactions; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互；作为测试代码块校验预期行为。

### Lines 181-198 / 第 181-198 行

````python
        contents = Path(tmp_path).read_text()
        check(
            contents.startswith("test-matrix="), f"unexpected file contents: {contents}"
        )
        written = json.loads(contents[len("test-matrix=") :].strip())
        check(written["include"][0]["runner"] == "l-x86iavx512-8-64")
    finally:
        os.unlink(tmp_path)


if __name__ == "__main__":
    tests = [v for k, v in sorted(globals().items()) if k.startswith("test_")]
    failed = 0
    for t in tests:
        try:
            t()
            print(f"  PASS  {t.__name__}")
        except AssertionError as e:
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; serializes or parses structured data files.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；序列化或解析结构化数据文件。

### Lines 199-204 / 第 199-204 行

````python
            print(f"  FAIL  {t.__name__}: {e}")
            failed += 1
    if failed:
        print(f"\n{failed}/{len(tests)} tests failed")
        sys.exit(1)
    print(f"\nAll {len(tests)} tests passed")
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。


## Key Concepts / 关键概念
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `json`, `os`, `subprocess`, `sys`, `tempfile`, `pathlib`
- Internal imports / 内部导入: none
- External imports / 外部导入: none
- Classes / 类: none
- Functions / 函数: `run`, `parse_output`, `check`, `test_basic_matrix`, `test_matrix_with_prefix`, `test_matrix_without_prefix_when_none_present`, `test_unknown_runner_fails`, `test_prefix_not_present_on_runner`, `test_preserves_non_runner_fields`, `test_empty_include_passes_through`, ...
- Test entry points / 测试入口: `test_basic_matrix`, `test_matrix_with_prefix`, `test_matrix_without_prefix_when_none_present`, `test_unknown_runner_fails`, `test_prefix_not_present_on_runner`, `test_preserves_non_runner_fields`, `test_empty_include_passes_through`, `test_empty_string_passes_through`, ...
