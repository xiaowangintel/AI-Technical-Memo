# analyze_logs_with_modal.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/slurm/analyze_logs_with_modal.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `analyze_logs_with_modal` workflow in SGLang. It mainly handles CI orchestration, runner selection and provisioning. / 该Python 模块用于支撑 SGLang 中的 `analyze_logs_with_modal` 流程，主要负责CI 编排、运行器选择与准备。它属于 `slurm` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-15: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""Analyze srtslurm logs with opencode inside a Modal sandbox.

This script accepts either:
- a local log directory
- a `.tar.gz` bundle such as `multinode_server_logs.tar.gz`

It uploads the logs into an ephemeral Modal sandbox, installs and runs
opencode with an analysis prompt, and prints the resulting markdown.

Example:
  uv run --with modal python scripts/ci/slurm/analyze_logs_with_modal.py \
    --tarball /tmp/multinode_server_logs.tar.gz \
    --job-id 4645
"""
```
**EN:** Analyze srtslurm logs with opencode inside a Modal sandbox.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 17-27: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
from __future__ import annotations

import argparse
import logging
import os
import re
import shlex
import shutil
import tarfile
import tempfile
from pathlib import Path
```
**EN:** This block loads __future__, argparse, logging, os, re, shlex, shutil, tarfile. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 __future__, argparse, logging, os, re, shlex, shutil, tarfile。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 29-32: Top-level try logic / 顶层 try 逻辑
```python
try:
    import modal
except ImportError:  # pragma: no cover - runtime guard for local usage
    modal = None
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 35-54: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
logger = logging.getLogger("slurm_log_analysis")

SANDBOX_TIMEOUT = 600
DEFAULT_MODAL_SECRET_NAME = "or"
DEFAULT_MODEL = "openrouter/minimax/minimax-m2.7"
DEFAULT_REPOS = [
    "https://github.com/sgl-project/sglang.git",
]
PROMPT_PATH = Path(__file__).with_name("log_analysis_prompt.md")

# Matches common API key / token patterns (sk-..., ak-..., as-..., key-..., etc.)
_SECRET_PATTERN = re.compile(
    r"""(?:"""
    r"""(?:sk|ak|as|key|token|secret|bearer)[-_][A-Za-z0-9_\-]{16,}"""
    r"""|"""
    r"""(?:OPENROUTER_API_KEY|MODAL_TOKEN_ID|MODAL_TOKEN_SECRET|ANTHROPIC_API_KEY)"""
    r"""[=:]\s*\S+"""
    r""")""",
    re.IGNORECASE,
)
```
**EN:** This section defines SANDBOX_TIMEOUT, DEFAULT_MODAL_SECRET_NAME, DEFAULT_MODEL, DEFAULT_REPOS, PROMPT_PATH, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 SANDBOX_TIMEOUT, DEFAULT_MODAL_SECRET_NAME, DEFAULT_MODEL, DEFAULT_REPOS, PROMPT_PATH，用于把可复用的默认值集中在模块顶部。

### Lines 57-67: Defines the `sanitize` routine / 定义 `sanitize` 例程
```python
def sanitize(text: str) -> str:
    """Redact strings that look like API keys or secrets."""
    if not text:
        return text
    sanitized = _SECRET_PATTERN.sub("[REDACTED]", text)
    # Also redact any env var values we know are secrets
    for var in ("OPENROUTER_API_KEY", "MODAL_TOKEN_ID", "MODAL_TOKEN_SECRET"):
        val = os.environ.get(var)
        if val and len(val) > 8:
            sanitized = sanitized.replace(val, "[REDACTED]")
    return sanitized
```
**EN:** This block defines `sanitize`. Redact strings that look like API keys or secrets. It accepts 1 parameter(s): text. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `sanitize`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：text。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 70-75: Defines the `configure_logging` routine / 定义 `configure_logging` 例程
```python
def configure_logging(verbose: bool) -> None:
    logging.basicConfig(
        level=logging.INFO,
        format="%(levelname)s: %(message)s",
    )
    logger.setLevel(logging.DEBUG if verbose else logging.INFO)
```
**EN:** This block defines `configure_logging`. It accepts 1 parameter(s): verbose. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `configure_logging`。它接收 1 个参数：verbose。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 78-85: Defines the `extract_tarball` routine / 定义 `extract_tarball` 例程
```python
def extract_tarball(tarball: Path, destination: Path) -> None:
    with tarfile.open(tarball, "r:gz") as archive:
        # Python 3.14 changes the default extraction behavior. Use the
        # data filter when available so extraction remains explicit.
        if "data" in tarfile._NAMED_FILTERS:  # type: ignore[attr-defined]
            archive.extractall(destination, filter="data")
        else:
            archive.extractall(destination)
```
**EN:** This block defines `extract_tarball`. It accepts 2 parameter(s): tarball, destination. Internally it reads or writes files.
**CN:** 该代码块定义了 `extract_tarball`。它接收 2 个参数：tarball, destination。内部会读写文件。

### Lines 88-143: Defines the `parse_args` routine / 定义 `parse_args` 例程
```python
def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(
        description="Analyze a srtslurm log bundle with opencode in Modal."
    )
    source = parser.add_mutually_exclusive_group(required=True)
    source.add_argument(
        "--tarball",
        type=Path,
        help="Path to a local multinode_server_logs.tar.gz bundle.",
    )
    source.add_argument(
        "--log-dir",
        type=Path,
        help="Path to an unpacked log directory.",
    )
    parser.add_argument(
        "--job-id",
        default="unknown",
        help="Job identifier used in the report header and logs.",
    )
    parser.add_argument(
        "--model",
        default=DEFAULT_MODEL,
        help="Model selector to pass to opencode run.",
    )
    parser.add_argument(
        "--output",
        type=Path,
        help="Optional path to write the markdown analysis.",
    )
    parser.add_argument(
        "--repo-url",
        action="append",
        dest="repo_urls",
        help=(
            "Optional extra repo URL to clone into the sandbox for context. "
            "Can be specified multiple times."
        ),
    )
    parser.add_argument(
        "--timeout-seconds",
        type=int,
        default=SANDBOX_TIMEOUT,
        help="Sandbox lifetime in seconds.",
    )
    parser.add_argument(
        "--modal-secret-name",
        default=DEFAULT_MODAL_SECRET_NAME,
        help="Modal secret name that provides OPENROUTER_API_KEY to the sandbox.",
    )
    parser.add_argument(
        "--verbose",
        action="store_true",
        help="Enable debug logging.",
    )
    return parser.parse_args()
```
**EN:** This block defines `parse_args`. It takes no explicit parameters at the top level. Internally it parses CLI arguments.
**CN:** 该代码块定义了 `parse_args`。它在顶层定义中不接收显式参数。内部会解析命令行参数。

### Lines 146-164: Defines the `build_sandbox_image` routine / 定义 `build_sandbox_image` 例程
```python
def build_sandbox_image() -> "modal.Image":
    if modal is None:
        raise RuntimeError(
            "The 'modal' package is required. Run this script with "
            "`uv run --with modal python ...` or install modal locally."
        )

    return (
        modal.Image.debian_slim(python_version="3.12")
        .apt_install("bash", "curl", "git", "gh")
        .run_commands(
            "curl -fsSL https://opencode.ai/install | bash",
        )
        .env(
            {
                "PATH": "/root/.opencode/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
            }
        )
    )
```
**EN:** This block defines `build_sandbox_image`. It takes no explicit parameters at the top level. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `build_sandbox_image`。它在顶层定义中不接收显式参数。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 167-179: Defines the `prepare_log_dir` routine / 定义 `prepare_log_dir` 例程
```python
def prepare_log_dir(args: argparse.Namespace) -> tuple[Path, Path | None]:
    if args.log_dir:
        if not args.log_dir.is_dir():
            raise FileNotFoundError(f"log directory not found: {args.log_dir}")
        return args.log_dir.resolve(), None

    assert args.tarball is not None
    if not args.tarball.is_file():
        raise FileNotFoundError(f"tarball not found: {args.tarball}")

    temp_dir = Path(tempfile.mkdtemp(prefix="sglang_logs_"))
    extract_tarball(args.tarball, temp_dir)
    return temp_dir.resolve(), temp_dir
```
**EN:** This block defines `prepare_log_dir`. It accepts 1 parameter(s): args. Internally it touches filesystem paths.
**CN:** 该代码块定义了 `prepare_log_dir`。它接收 1 个参数：args。内部会处理文件系统路径。

### Lines 182-213: Defines the `build_prompt` routine / 定义 `build_prompt` 例程
```python
def build_prompt(job_id: str, repo_urls: list[str]) -> str:
    skill_content = PROMPT_PATH.read_text()
    repo_lines = []
    for repo_url in repo_urls:
        repo_name = repo_url.rsplit("/", 1)[-1].removesuffix(".git")
        repo_lines.append(f"- **{repo_name} repo**: `/workspace/repos/{repo_name}/`")
    repo_section = (
        "\n".join(repo_lines) if repo_lines else "- No extra repos were requested."
    )

    return f"""{skill_content}

---

## Your Environment

- **Logs**: `/workspace/logs/`
- **GitHub CLI**: `gh` is installed and authenticated.
{repo_section}

## Job

You are analyzing job `{job_id}`. Follow Steps 1–5 in the prompt above.

**You MUST write the final markdown report to `/workspace/logs/ai_analysis.md`.**
This is a hard requirement. Do not just print the report to stdout. Use your
file-writing tool to create `/workspace/logs/ai_analysis.md` with the full
analysis. The downstream pipeline reads this file.

**You MUST file GitHub issues when the root cause is clear (Category A or B).**
Do not skip issue filing. The whole point of this system is automated triage.
"""
```
**EN:** This block defines `build_prompt`. It accepts 2 parameter(s): job_id, repo_urls. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `build_prompt`。它接收 2 个参数：job_id, repo_urls。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 216-225: Defines the `upload_tree` routine / 定义 `upload_tree` 例程
```python
def upload_tree(sandbox: "modal.Sandbox", log_dir: Path) -> None:
    log_files = [path for path in log_dir.rglob("*") if path.is_file()]
    logger.info("Uploading %d log files into the sandbox", len(log_files))
    for index, log_file in enumerate(log_files, start=1):
        rel_path = log_file.relative_to(log_dir)
        remote_path = str(Path("/workspace/logs") / rel_path)
        sandbox.mkdir(str(Path(remote_path).parent), parents=True)
        sandbox.filesystem.write_bytes(log_file.read_bytes(), remote_path)
        if index % 10 == 0 or index == len(log_files):
            logger.info("Uploaded %d/%d files", index, len(log_files))
```
**EN:** This block defines `upload_tree`. It accepts 2 parameter(s): sandbox, log_dir. Internally it touches filesystem paths, creates directories.
**CN:** 该代码块定义了 `upload_tree`。它接收 2 个参数：sandbox, log_dir。内部会处理文件系统路径、创建目录。

### Lines 228-244: Defines the `clone_context_repos` routine / 定义 `clone_context_repos` 例程
```python
def clone_context_repos(sandbox: "modal.Sandbox", repo_urls: list[str]) -> None:
    if not repo_urls:
        return

    sandbox.mkdir("/workspace/repos", parents=True)

    for repo_url in repo_urls:
        repo_name = repo_url.rsplit("/", 1)[-1].removesuffix(".git")
        logger.info("Cloning %s into the sandbox", repo_name)
        sandbox.exec(
            "git",
            "clone",
            "--depth",
            "100",
            repo_url,
            f"/workspace/repos/{repo_name}",
        ).wait()
```
**EN:** This block defines `clone_context_repos`. It accepts 2 parameter(s): sandbox, repo_urls. Internally it creates directories.
**CN:** 该代码块定义了 `clone_context_repos`。它接收 2 个参数：sandbox, repo_urls。内部会创建目录。

### Lines 247-251: Defines the `read_optional_file` routine / 定义 `read_optional_file` 例程
```python
def read_optional_file(sandbox: "modal.Sandbox", path: str) -> str | None:
    try:
        return sandbox.filesystem.read_text(path)
    except Exception:
        return None
```
**EN:** This block defines `read_optional_file`. It accepts 2 parameter(s): sandbox, path. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `read_optional_file`。它接收 2 个参数：sandbox, path。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 254-343: Defines the `run_opencode_analysis` routine / 定义 `run_opencode_analysis` 例程
```python
def run_opencode_analysis(
    *,
    log_dir: Path,
    job_id: str,
    model: str,
    repo_urls: list[str],
    timeout_seconds: int,
    modal_secret_name: str,
) -> str:
    prompt = build_prompt(job_id, repo_urls)
    app = modal.App.lookup("sglang-log-analyzer", create_if_missing=True)
    sandbox = modal.Sandbox.create(
        app=app,
        image=build_sandbox_image(),
        timeout=timeout_seconds,
        secrets=[modal.Secret.from_name(modal_secret_name)],
    )
    logger.info("Created Modal sandbox %s", sandbox.object_id)

    try:
        sandbox.mkdir("/workspace/logs", parents=True)
        sandbox.mkdir("/workspace/repos", parents=True)

        clone_context_repos(sandbox, repo_urls)
        upload_tree(sandbox, log_dir)

        sandbox.filesystem.write_text(prompt, "/workspace/prompt.txt")

        runner_script = f"""#!/bin/bash
set -uo pipefail
cd /workspace
if opencode run \\
  --dangerously-skip-permissions \\
  --dir /workspace/logs \\
  -m {shlex.quote(model)} \\
  "$(cat /workspace/prompt.txt)" \\
  < /dev/null \\
  > /workspace/logs/opencode.stdout \\
  2> /workspace/logs/opencode.stderr; then
  echo 0 > /workspace/logs/opencode.exitcode
else
  echo $? > /workspace/logs/opencode.exitcode
fi
ls -la /workspace/logs > /workspace/logs/log_dir_listing.txt
"""
        sandbox.filesystem.write_text(runner_script, "/workspace/run_opencode.sh")
        sandbox.exec("chmod", "+x", "/workspace/run_opencode.sh").wait()

        logger.info("Running opencode analysis")
        process = sandbox.exec(
            "bash",
            "/workspace/run_opencode.sh",
        )
        process.wait()

        stderr = process.stderr.read()
        if stderr:
            logger.warning("runner stderr: %s", sanitize(stderr[:500]))

        exitcode = read_optional_file(sandbox, "/workspace/logs/opencode.exitcode")
        opencode_stdout = (
            read_optional_file(sandbox, "/workspace/logs/opencode.stdout") or ""
        )
        opencode_stderr = read_optional_file(sandbox, "/workspace/logs/opencode.stderr")
        log_dir_listing = read_optional_file(
            sandbox, "/workspace/logs/log_dir_listing.txt"
        )
        try:
            ai_analysis = read_optional_file(sandbox, "/workspace/logs/ai_analysis.md")
            if ai_analysis and ai_analysis.strip():
                return sanitize(ai_analysis)

            if opencode_stdout.strip():
                return sanitize(opencode_stdout)

            raise RuntimeError("opencode completed without producing analysis output")
        except Exception as exc:
            stdout = process.stdout.read()
            if stdout:
                return sanitize(stdout)
            details = [
                f"opencode analysis did not produce a usable report: {exc}",
                f"exitcode={exitcode!r}",
                f"stdout_preview={sanitize(opencode_stdout[:500])!r}",
                f"stderr_preview={sanitize((opencode_stderr or '')[:500])!r}",
                f"log_dir_listing={sanitize((log_dir_listing or '')[:500])!r}",
            ]
            raise RuntimeError(" ".join(details)) from exc
    finally:
        sandbox.terminate()
```
**EN:** This block defines `run_opencode_analysis`. It accepts 6 parameter(s): log_dir, job_id, model, repo_urls, timeout_seconds, modal_secret_name. Internally it creates directories.
**CN:** 该代码块定义了 `run_opencode_analysis`。它接收 6 个参数：log_dir, job_id, model, repo_urls, timeout_seconds, modal_secret_name。内部会创建目录。

### Lines 346-372: Defines the `main` routine / 定义 `main` 例程
```python
def main() -> int:
    args = parse_args()
    configure_logging(args.verbose)

    repo_urls = list(DEFAULT_REPOS)
    if args.repo_urls:
        repo_urls.extend(args.repo_urls)

    log_dir, cleanup_dir = prepare_log_dir(args)
    try:
        analysis = run_opencode_analysis(
            log_dir=log_dir,
            job_id=args.job_id,
            model=args.model,
            repo_urls=repo_urls,
            timeout_seconds=args.timeout_seconds,
            modal_secret_name=args.modal_secret_name,
        )
    finally:
        if cleanup_dir is not None:
            shutil.rmtree(cleanup_dir, ignore_errors=True)

    print(analysis)
    if args.output:
        args.output.write_text(analysis)
        logger.info("Wrote analysis to %s", args.output)
    return 0
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it parses CLI arguments, emits status messages.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会解析命令行参数、输出状态信息。

### Lines 375-376: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    raise SystemExit(main())
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **CLI parsing** / 命令行解析
- **Filesystem coordination** / 文件系统协同
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `argparse`, `logging`, `os`, `pathlib`, `re`, `shlex`, `shutil`, `tarfile`, `tempfile`
- **Third-party modules / 第三方模块**: `modal`
- **External commands inferred from code / 从代码推断的外部命令**: `git`
