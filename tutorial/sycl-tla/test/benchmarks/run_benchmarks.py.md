# run_benchmarks.py — Code Analysis / 代码分析
## Source / 来源
- **Path / 路径:** `test/benchmarks/run_benchmarks.py`
- **EN:** Runs a fixed set of benchmark executables, parses their text logs into structured records, optionally pushes the metrics to InfluxDB/Grafana, and saves CSV reports.
- **CN:** 运行一组固定的 benchmark 可执行文件，将文本日志解析为结构化记录，可选地把指标推送到 InfluxDB/Grafana，并保存为 CSV 报告。

## Line-by-Line Analysis / 逐行分析
### Lines 32-62
```python
import csv
import re
import subprocess
from datetime import datetime
from pathlib import Path
import sys
import argparse
from utils.grafana_push import push_results

TEST_SUITES = [
    {
        "name": "gemm_sycl",
        "executable": "./benchmarks/gemm/cutlass_benchmarks_gemm_sycl",
        "config_file": "../benchmarks/device/bmg/input_files/bmg_small_input.in",
    },
    {
        "name": "flash_attention_prefill",
        "executable": "./benchmarks/flash_attention/cutlass_benchmarks_flash_attention_prefill_xe",
        "config_file": "../benchmarks/device/bmg/input_files/input_flash_attention_prefill_bf16.in",
    },
    {
        "name": "flash_attention_decode",
        "executable": "./benchmarks/flash_attention/cutlass_benchmarks_flash_attention_decode_xe",
        "config_file": "../benchmarks/device/bmg/input_files/input_flash_attention_decode_bf16.in",
    },
    {
        "name": "cutlass_benchmarks_gemm_sycl_legacy",
        "executable": "./benchmarks/gemm/legacy/cutlass_benchmarks_gemm_sycl_legacy",
        "config_file": "../benchmarks/device/bmg/input_files/all_in_one.in",
    }
]
```
**EN:** The script imports standard modules for CSV output, regex parsing, process launching, timestamps, filesystem paths, CLI parsing, and dashboard publishing. `TEST_SUITES` is the core manifest: each entry binds a logical suite name to one executable path and one input config file, so later orchestration can iterate without hard-coding command logic in multiple places.

**CN:** 脚本导入了用于 CSV 输出、正则解析、进程启动、时间戳、路径处理、命令行解析以及仪表盘上报的标准模块。`TEST_SUITES` 是核心清单：每个条目把逻辑测试名、可执行文件路径和输入配置文件绑定在一起，因此后续流程只需遍历该列表，而不必在多个位置重复硬编码命令。

### Lines 64-79
```python
def run_command(command, cwd, log_path=None):
    print(f"\n$ {' '.join(command)}")
    if log_path:
        with open(log_path, "w") as log_file:
            try:
                results = subprocess.run(command, cwd=cwd, text=True, check=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
                for line in results.stdout:
                    sys.stdout.write(line)
                    log_file.write(line)
            except subprocess.CalledProcessError as e:
                print(f"Error: Command failed with return code {e.returncode}")
                print("Stderr:", e.stderr)
        print(f"Log written to: {log_path}")
    else:
        subprocess.run(command, cwd=cwd, check=True)

```
**EN:** `run_command()` prints the command in shell-like form, then either executes it directly or captures combined stdout/stderr into a log file. When logging is enabled, it uses `subprocess.run(..., stdout=PIPE, stderr=STDOUT)` so the benchmark output is preserved in one stream and mirrored to both the terminal and the log file.

**CN:** `run_command()` 先以接近 shell 的形式打印命令，然后选择直接执行，或把合并后的标准输出/标准错误写入日志文件。启用日志时，它通过 `subprocess.run(..., stdout=PIPE, stderr=STDOUT)` 把 benchmark 输出合并为单一流，并同时回显到终端和日志文件。

### Lines 80-135
```python
def parse_benchmark_log(log_path):
    records = []
    total=0
    failed=0
    passed=0
    if not log_path.exists():
        return records

    with open(log_path, "r") as handle:
        for line in handle:
            if not re.search(r"(Gemm|manual_time)", line):
                continue

            parts = line.strip().split()
            if not parts:
                continue

            benchmark_token = parts[0]
            tokens = benchmark_token.split("/")
            if len(tokens) < 3:
                continue

            kernel_name = tokens[0]
            dimensions = tokens[2]
            result = "Fail"
            reason=""
            avg_tflops = ""
            avg_throughput = ""

            if any(sub in line for sub in ["ERROR OCCURRED", "ERROR"]):
                result = "Fail"
                reason=line.strip()
                failed+=1
            elif "avg_tflops" in line:
                result = "Pass"
                passed+=1
                tflops_match = re.search(r"avg_tflops=([0-9.]+[a-z]*)", line)
                throughput_match = re.search(r"avg_throughput=([0-9.]+)", line)
                if tflops_match:
                    avg_tflops = tflops_match.group(1)
                if throughput_match:
                    avg_throughput = throughput_match.group(1)
            total+=1
            records.append({
                "Kernel": kernel_name,
                "Shape": dimensions,
                "Result": result,
                "Tflops": avg_tflops,
                "Throughput": avg_throughput,
                "Reason": reason
            })
    print("failed: ", failed)
    print("passed: ", passed)
    print("total: ", total)
    return records

```
**EN:** `parse_benchmark_log()` converts raw benchmark text into dictionaries. It keeps only lines containing `Gemm` or `manual_time`, splits the first token on `/`, treats the first segment as the kernel name and the third as the shape descriptor, then classifies each record as pass or fail. For successful lines it extracts `avg_tflops` and `avg_throughput` with regexes; for failures it stores the whole error line in `Reason`. It also maintains total/passed/failed counters for quick summaries.

**CN:** `parse_benchmark_log()` 把原始 benchmark 文本转换为字典记录。它只保留包含 `Gemm` 或 `manual_time` 的行，对首个 token 按 `/` 分割，把第一段视为 kernel 名称、第三段视为形状描述，然后把每条记录归类为通过或失败。成功行会用正则提取 `avg_tflops` 和 `avg_throughput`；失败行则把完整错误文本保存到 `Reason`。同时它还维护 total/passed/failed 计数，方便快速汇总。

### Lines 136-158
```python
def write_report_csv(path, records):
    with open(path, "w", newline="") as csvfile:
        writer = csv.DictWriter(
            csvfile,
            fieldnames=["Kernel", "Shape", "Result", "Tflops", "Throughput","Reason"],
        )
        writer.writeheader()
        writer.writerows(records)
        print(f"csv written to: {path}")

def get_git_commit_id(repo_root):
    try:
        result = subprocess.run(
            ["git", "rev-parse", "HEAD"],
            cwd=repo_root,
            text=True,
            check=True,
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE,
        )
        return result.stdout.strip()
    except (subprocess.CalledProcessError, FileNotFoundError):
        return ""
```
**EN:** `write_report_csv()` serializes the parsed records into a stable schema with six columns. `get_git_commit_id()` is a small helper that runs `git rev-parse HEAD` in the repository root and returns an empty string if Git is unavailable or the command fails, allowing dashboard uploads to carry commit metadata without breaking local runs.

**CN:** `write_report_csv()` 将解析后的记录按固定的六列表头写成 CSV。`get_git_commit_id()` 是一个小辅助函数：它在仓库根目录执行 `git rev-parse HEAD`，如果 Git 不可用或命令失败则返回空字符串，从而让仪表盘上报可以携带提交信息，同时又不会影响本地运行。

### Lines 161-175
```python
def run_tests(logs_dir, branch, build_dir, repo_root, grafana, git_commit_id):
    for test_suite in TEST_SUITES:
        run_cmd = [
            test_suite["executable"],
            f"--config_file={test_suite['config_file']}",
        ]
        test_name = test_suite["name"]
        test_log = logs_dir / f"{test_name}_run_{branch}.log"
        test_report = logs_dir / f"{test_name}_report_{branch}.csv"
        run_command(run_cmd, cwd=Path(repo_root, build_dir), log_path=test_log)
        main_records = parse_benchmark_log(test_log)
        if grafana:
            push_results(main_records, test_name, git_commit_id=git_commit_id)
        write_report_csv(test_report, main_records)

```
**EN:** `run_tests()` is the high-level driver for one benchmark batch. For every suite in `TEST_SUITES`, it builds the executable command, constructs per-suite log and CSV filenames under the chosen log directory, runs the benchmark from `repo_root/build`, parses the log, optionally calls `push_results()`, and finally writes the CSV report.

**CN:** `run_tests()` 是单轮 benchmark 批处理的高层驱动函数。它遍历 `TEST_SUITES` 中的每个条目，拼接执行命令，在指定日志目录下生成对应的日志与 CSV 文件名，在 `repo_root/build` 中运行 benchmark，解析日志，可选调用 `push_results()`，最后写出 CSV 报告。

### Lines 176-200
```python
def main():
    parser = argparse.ArgumentParser(description="Benchmarking script for cutlass kernels.")
    parser.add_argument(
        "--push-to-dashboard",
        dest="grafana",
        action="store_true",
        help="Increase output verbosity"
    )
    args = parser.parse_args()

    build_dir = "build"
    branch = "main"
    # This file is expected to be run from the root of the repository,
    # so we can directly use relative paths to access logs and benchmarks.
    repo_root = Path.cwd()
    logs_root = repo_root / "logs"
    logs_root.mkdir(parents=True, exist_ok=True)
    workdir = f"{datetime.now().strftime('%Y%m%d%I%M')}_benchmarks_{branch}"
    logs_dir = logs_root / workdir
    logs_dir.mkdir(parents=True, exist_ok=True)
    git_commit_id = get_git_commit_id(repo_root)
    run_tests(logs_dir, branch, build_dir, repo_root, args.grafana, git_commit_id)

if __name__ == "__main__":
    main()
```
**EN:** `main()` exposes one CLI flag, `--push-to-dashboard`, then assumes the script is launched from the repository root. It creates `logs/` plus a timestamped subdirectory, captures the current commit id, and dispatches the whole batch through `run_tests()`. The `if __name__ == "__main__"` guard makes the module usable as both a script and an importable helper.

**CN:** `main()` 只暴露一个命令行开关 `--push-to-dashboard`，并假定脚本从仓库根目录启动。它创建 `logs/` 以及带时间戳的子目录，获取当前提交号，然后通过 `run_tests()` 启动整批任务。结尾的 `if __name__ == "__main__"` 保护使该模块既可作为脚本运行，也可被其他代码导入。

## Key Concepts / 关键概念
- **EN:** Manifest-driven execution: benchmark suites are described declaratively in `TEST_SUITES`.
  **CN:** 清单驱动执行：benchmark 套件通过 `TEST_SUITES` 声明式定义。
- **EN:** Text-to-structure parsing: raw benchmark lines are normalized into reusable dictionaries before export.
  **CN:** 文本到结构化数据的转换：原始 benchmark 日志先被标准化为可复用的字典，再进行导出。
- **EN:** Dual reporting path: the same parsed records can feed CSV artifacts and InfluxDB/Grafana uploads.
  **CN:** 双重上报路径：同一份解析结果既可生成 CSV，也可上传到 InfluxDB/Grafana。
- **EN:** Repository-root assumption: executable paths, config paths, and log layout all depend on `Path.cwd()` being the repo root.
  **CN:** 仓库根目录假设：可执行文件路径、配置文件路径和日志目录布局都依赖 `Path.cwd()` 指向仓库根目录。

## Dependencies / 依赖关系
- **EN:** Standard library modules `csv`, `re`, `subprocess`, `datetime`, `pathlib`, `sys`, and `argparse` support file output, parsing, process control, and CLI handling.
  **CN:** 标准库模块 `csv`、`re`、`subprocess`、`datetime`、`pathlib`、`sys` 和 `argparse` 提供文件输出、文本解析、进程控制与命令行处理能力。
- **EN:** Internal dependency `utils.grafana_push.push_results` transforms parsed benchmark records into InfluxDB write requests.
  **CN:** 内部依赖 `utils.grafana_push.push_results` 会把解析后的 benchmark 记录转换为 InfluxDB 写入请求。
- **EN:** External runtime dependencies include the benchmark executables under `build/benchmarks/...`, their input files under `benchmarks/device/bmg/input_files/`, and the `git` CLI for commit lookup.
  **CN:** 外部运行时依赖包括 `build/benchmarks/...` 下的 benchmark 可执行文件、`benchmarks/device/bmg/input_files/` 下的输入文件，以及用于查询提交号的 `git` 命令。
