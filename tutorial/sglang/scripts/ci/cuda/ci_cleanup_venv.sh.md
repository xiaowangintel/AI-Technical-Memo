# ci_cleanup_venv.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/ci/cuda/ci_cleanup_venv.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: Remove the per-job uv venv created by ci_install_dependency.sh. This shell script is part of SGLang's `cuda` automation flow. / 该Shell 脚本用于支撑 SGLang 中的 `ci_cleanup_venv` 流程，主要负责CI 编排、运行器选择与准备。它属于 `cuda` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Shell entrypoint and interpreter selection / Shell 入口与解释器选择
```bash
#!/bin/bash
# Remove the per-job uv venv created by ci_install_dependency.sh.
#
# Meant to run in a post-job workflow step with `if: always()` so the venv is
# destroyed even on job failure/cancel. Runner-level safety net: a cron or
# startup task should also purge stale /tmp/sglang-ci-* directories to catch
# cancelled or crashed jobs that never reached this cleanup.
```
**EN:** This opening block selects Bash as the interpreter and usually also documents expected inputs or outputs for the rest of the script.
**CN:** 该起始代码块指定 Bash 作为解释器，并通常顺带说明脚本后续步骤所需的输入与输出。

### Lines 9-11: Enables strict shell execution flags / 启用严格的 shell 执行选项
```bash
# Best-effort cleanup: never fail the job.
set +e
set -u
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 13-23: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# Skip entirely when venv mode is disabled — no /tmp/sglang-ci-* dir exists
# and there's nothing to sweep. Matches the USE_VENV parsing in
# ci_install_dependency.sh (accepts 1/true/yes, case-insensitive).
USE_VENV_RAW="${USE_VENV:-true}"
case "$(printf '%s' "$USE_VENV_RAW" | tr '[:upper:]' '[:lower:]')" in
    1 | true | yes) ;;
    *)
        echo "USE_VENV=${USE_VENV_RAW}: skipping venv cleanup"
        exit 0
        ;;
esac
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 25-45: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# Prefer the path propagated via GITHUB_ENV. Fallback: glob for any venv from
# this run+job (covers the case where install crashed before exporting the path).
if [ -n "${SGLANG_CI_VENV_PATH:-}" ] && [ -d "$SGLANG_CI_VENV_PATH" ]; then
    if rm -rf "$SGLANG_CI_VENV_PATH"; then
        echo "Cleaned up venv: $SGLANG_CI_VENV_PATH"
    else
        echo "::warning::Failed to remove $SGLANG_CI_VENV_PATH — runner cron should sweep /tmp/sglang-ci-*"
    fi
else
    matched=0
    for venv in /tmp/sglang-ci-${GITHUB_RUN_ID:-unknownrun}-${GITHUB_JOB:-unknownjob}-*; do
        [ -d "$venv" ] || continue
        matched=1
        if rm -rf "$venv"; then
            echo "Cleaned up venv (via glob): $venv"
        else
            echo "::warning::Failed to remove $venv — runner cron should sweep /tmp/sglang-ci-*"
        fi
    done
    [ "$matched" -eq 0 ] && echo "No venv to clean for run=${GITHUB_RUN_ID:-?} job=${GITHUB_JOB:-?}"
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It cleans generated files, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会清理生成文件、打印状态信息。

### Lines 47-56: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# Sweep stale venvs from cancelled/crashed jobs that never reached cleanup.
# Any /tmp/sglang-ci-* dir older than 4 hours is considered orphaned.
stale_count=0
for venv in /tmp/sglang-ci-*; do
    [ -d "$venv" ] || continue
    if find "$venv" -maxdepth 0 -mmin +240 -print -quit | grep -q .; then
        rm -rf "$venv" && stale_count=$((stale_count + 1))
    fi
done
[ "$stale_count" -gt 0 ] && echo "Swept $stale_count stale venv(s) older than 4h"
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It cleans generated files, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会清理生成文件、打印状态信息。

### Lines 58-58: Comment or metadata block / 注释或元数据块
```bash
exit 0
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **Git state inspection** / Git 状态检查
- **Runner provisioning** / 运行器准备

## Dependencies / 依赖关系
- **Commands / 外部命令**: `uv`, `rm`, `echo`, `grep`, `find`
- **Environment variables / 环境变量**: `GITHUB_JOB`, `GITHUB_RUN_ID`, `SGLANG_CI_VENV_PATH`, `USE_VENV`, `USE_VENV_RAW`
- **Referenced files / 引用文件**: `ci_install_dependency.sh`
