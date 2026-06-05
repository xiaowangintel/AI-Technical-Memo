# s3_init.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/linter/adapters/s3_init.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements lint adapters and validation helpers that enforce repository-wide code-quality rules.
- **Purpose (CN)**: 实现 lint 适配器与校验辅助逻辑，用于执行仓库范围的代码质量规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```python
import argparse
import hashlib
import json
import logging
import os
import platform
import stat
import subprocess
import sys
import urllib.error
import urllib.request
from pathlib import Path
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as argparse, hashlib, json, and 9 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 argparse、hashlib、json 等共 12 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 14-30
```python

# String representing the host platform (e.g. Linux, Darwin).
HOST_PLATFORM = platform.system()
HOST_PLATFORM_ARCH = platform.system() + "-" + platform.machine()

# PyTorch directory root
try:
    result = subprocess.run(
        ["git", "rev-parse", "--show-toplevel"],
        stdout=subprocess.PIPE,
        check=True,
    )
    PYTORCH_ROOT = result.stdout.decode("utf-8").strip()
except subprocess.CalledProcessError:
    # If git is not installed, compute repo root as 3 folders up from this file
    PYTORCH_ROOT = str(Path(__file__).absolute().parents[3])
```
- **EN**: Configuration constants such as HOST_PLATFORM, HOST_PLATFORM_ARCH, PYTORCH_ROOT, and 1 more centralize defaults so later functions share the same policy knobs. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: HOST_PLATFORM、HOST_PLATFORM_ARCH、PYTORCH_ROOT 等共 4 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 31-39
```python
DRY_RUN = False


def compute_file_sha256(path: str) -> str:
    """Compute the SHA256 hash of a file and return it as a hex string."""
    # If the file doesn't exist, return an empty string.
    if not os.path.exists(path):
        return ""
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `compute_file_sha256`, which implements a focused step inside the lint tooling pipeline. Configuration constants such as DRY_RUN centralize defaults so later functions share the same policy knobs. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `compute_file_sha256`，其作用是实现Lint 工具链流水线中的一个关键步骤。 DRY_RUN 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 40-49
```python
    hash = hashlib.sha256()

    # Open the file in binary mode and hash it.
    with open(path, "rb") as f:
        for b in f:
            hash.update(b)

    # Return the hash as a hexadecimal string.
    return hash.hexdigest()
```
- **EN**: This chunk continues `compute_file_sha256` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `compute_file_sha256`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 50-61
```python

def report_download_progress(
    chunk_number: int, chunk_size: int, file_size: int
) -> None:
    """
    Pretty printer for file download progress.
    """
    if file_size != -1:
        percent = min(1, (chunk_number * chunk_size) / file_size)
        bar = "#" * int(64 * percent)
        sys.stdout.write(f"\r0% |{bar:<64}| {int(percent * 100)}%")
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `report_download_progress`, which parses or loads structured input into tool-friendly data structures. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `report_download_progress`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 62-71
```python

def check(binary_path: Path, reference_hash: str) -> bool:
    """Check whether the binary exists and is the right one.

    If there is hash difference, delete the actual binary.
    """
    if not binary_path.exists():
        logging.info("%s does not exist.", binary_path)
        return False
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `check`, which validates invariants and reports policy violations early. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `check`，其作用是校验不变量，并尽早报告策略违规。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 72-82
```python
    existing_binary_hash = compute_file_sha256(str(binary_path))
    if existing_binary_hash == reference_hash:
        return True

    logging.warning(
        """\
Found binary hash does not match reference!

Found hash: %s
Reference hash: %s
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `check` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `check`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 83-94
```python
Deleting %s just to be safe.
""",
        existing_binary_hash,
        reference_hash,
        binary_path,
    )
    if DRY_RUN:
        logging.critical(
            "In dry run mode, so not actually deleting the binary. But consider deleting it ASAP!"
        )
        return False
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `check` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `check`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 95-104
```python
    try:
        binary_path.unlink()
    except OSError:
        logging.critical("Failed to delete binary", exc_info=True)
        logging.critical(
            "Delete this binary as soon as possible and do not execute it!"
        )

    return False
```
- **EN**: This chunk continues `check` and expands its internal control flow or data movement. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `check`，进一步展开其内部控制流或数据流转。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 105-118
```python

def download(
    name: str,
    output_dir: str,
    url: str,
    reference_bin_hash: str,
) -> bool:
    """
    Download a platform-appropriate binary if one doesn't already exist at the expected location and verifies
    that it is the right binary by checking its SHA256 hash against the expected hash.
    """
    # First check if we need to do anything
    binary_path = Path(output_dir, name)
    if check(binary_path, reference_bin_hash):
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `download`, which parses or loads structured input into tool-friendly data structures. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `download`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 119-127
```python
        logging.info("Correct binary already exists at %s. Exiting.", binary_path)
        return True

    # Create the output folder
    binary_path.parent.mkdir(parents=True, exist_ok=True)

    # Download the binary
    logging.info("Downloading %s to %s", url, binary_path)
```
- **EN**: This chunk continues `download` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `download`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 128-137
```python
    if DRY_RUN:
        logging.info("Exiting as there is nothing left to do in dry run mode")
        return True

    urllib.request.urlretrieve(
        url,
        binary_path,
        reporthook=report_download_progress if sys.stdout.isatty() else None,
    )
```
- **EN**: This chunk continues `download` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `download`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 138-149
```python
    logging.info("Downloaded %s successfully.", name)

    # Check the downloaded binary
    if not check(binary_path, reference_bin_hash):
        logging.critical("Downloaded binary %s failed its hash check", name)
        return False

    # Ensure that executable bits are set
    mode = os.stat(binary_path).st_mode
    mode |= stat.S_IXUSR
    os.chmod(binary_path, mode)
```
- **EN**: This chunk continues `download` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `download`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 150-167
```python
    logging.info("Using %s located at %s", name, binary_path)
    return True


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="downloads and checks binaries from s3",
    )
    parser.add_argument(
        "--config-json",
        required=True,
        help="Path to config json that describes where to find binaries and hashes",
    )
    parser.add_argument(
        "--linter",
        required=True,
        help="Which linter to initialize from the config json",
    )
```
- **EN**: This chunk continues `download` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `download`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 168-183
```python
    parser.add_argument(
        "--output-dir",
        required=True,
        help="place to put the binary",
    )
    parser.add_argument(
        "--output-name",
        required=True,
        help="name of binary",
    )
    parser.add_argument(
        "--dry-run",
        default=False,
        help="do not download, just print what would be done",
    )
```
- **EN**: This chunk continues `download` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段延续了 `download`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 184-192
```python
    args = parser.parse_args()
    if args.dry_run == "0":
        DRY_RUN = False
    else:
        DRY_RUN = True

    logging.basicConfig(
        format="[DRY_RUN] %(levelname)s: %(message)s"
        if DRY_RUN
```
- **EN**: This chunk continues `download` and expands its internal control flow or data movement. Configuration constants such as DRY_RUN, DRY_RUN centralize defaults so later functions share the same policy knobs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `download`，进一步展开其内部控制流或数据流转。 DRY_RUN、DRY_RUN 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 193-207
```python
        else "%(levelname)s: %(message)s",
        level=logging.INFO,
        stream=sys.stderr,
    )
    with open(args.config_json) as f:
        config = json.load(f)
    config = config[args.linter]

    # Allow processor specific binaries for platform (e.g. Intel/M1 for macOS, x86_64/aarch64 for Linux)
    # Try arch-specific first, then fall back to generic platform
    host_platform = (
        HOST_PLATFORM_ARCH if HOST_PLATFORM_ARCH in config else HOST_PLATFORM
    )
    # If the host platform is not in platform_to_hash, it is unsupported.
    if host_platform not in config:
```
- **EN**: This chunk continues `download` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `download`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 208-217
```python
        logging.error("Unsupported platform: %s/%s", HOST_PLATFORM, HOST_PLATFORM_ARCH)
        sys.exit(1)

    url = config[host_platform]["download_url"]
    hash = config[host_platform]["hash"]

    ok = download(args.output_name, args.output_dir, url, hash)
    if not ok:
        logging.critical("Unable to initialize %s", args.linter)
        sys.exit(1)
```
- **EN**: This chunk continues `download` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `download`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

## Key Concepts / 关键概念

- **Lint tooling**
  - EN: This file belongs to the lint tooling layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于Lint 工具链层，应结合同一子目录中的相邻脚本一起理解。
- **Lint enforcement**
  - EN: The code applies policy checks to source files and reports actionable diagnostics.
  - CN: 代码对源码施加策略检查，并产出可执行的诊断信息。
- **CLI interface**
  - EN: The implementation exposes a command-line entry point and converts arguments into tool actions.
  - CN: 该实现暴露命令行入口，并把参数转换为工具动作。
- **Process orchestration**
  - EN: The file launches external commands and translates their results back into Python control flow.
  - CN: 该文件负责启动外部命令，并把结果重新映射回 Python 控制流。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **HOST_PLATFORM**
  - EN: `HOST_PLATFORM` is one of the main local symbols exposed or implemented here.
  - CN: `HOST_PLATFORM` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `argparse`, `hashlib`, `json`, `logging`, `os`, `platform`, `stat`, `subprocess`, `sys`, `urllib.error`, `urllib.request`, `pathlib`
- **Primary symbols in this file / 本文件核心符号**: `HOST_PLATFORM`, `HOST_PLATFORM_ARCH`, `DRY_RUN`, `compute_file_sha256`, `report_download_progress`, `check`, `download`
