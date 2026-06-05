# amd_ci_exec.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/ci/amd/amd_ci_exec.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: This shell script supports the `amd_ci_exec` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Shell 脚本用于支撑 SGLang 中的 `amd_ci_exec` 流程，主要负责CI 编排、测试执行。它属于 `amd` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: Shell entrypoint and interpreter selection / Shell 入口与解释器选择
```bash
#!/bin/bash
set -euo pipefail
```
**EN:** This opening block selects Bash as the interpreter and usually also documents expected inputs or outputs for the rest of the script.
**CN:** 该起始代码块指定 Bash 作为解释器，并通常顺带说明脚本后续步骤所需的输入与输出。

### Lines 4-6: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
# Detect GPU family from hostname (e.g., linux-mi35x-gpu-1-xxxxx-runner-zzzzz)
HOSTNAME_VALUE=$(hostname)
GPU_FAMILY=""
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 8-14: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# Host names look like: linux-mi35x-gpu-1-xxxxx-runner-zzzzz
if [[ "${HOSTNAME_VALUE}" =~ ^linux-(mi[0-9]+[a-z]*)-gpu-[0-9]+ ]]; then
  GPU_FAMILY="${BASH_REMATCH[1]}"
  echo "Detected GPU family from hostname: ${GPU_FAMILY}"
else
  echo "Warning: could not parse GPU family from '${HOSTNAME_VALUE}'"
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 16-21: Comment or metadata block / 注释或元数据块
```bash
WORKDIR="/sglang-checkout/test/srt"
declare -A ENV_MAP=(
  [SGLANG_IS_IN_CI_AMD]=1
  [SGLANG_IS_IN_CI]=1
  [SGLANG_USE_AITER]=1
)
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 23-26: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# Conditionally add GPU_ARCHS only for mi35x
if [[ "${GPU_FAMILY}" == "mi35x" ]]; then
  ENV_MAP[GPU_ARCHS]="gfx950"
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

### Lines 28-48: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# Parse -w/--workdir and -e ENV=VAL
while [[ $# -gt 0 ]]; do
  case "$1" in
    -w|--workdir)
      WORKDIR="$2"
      shift 2
      ;;
    -e)
      IFS="=" read -r key val <<< "$2"
      ENV_MAP["$key"]="$val"
      shift 2
      ;;
    --)
      shift
      break
      ;;
    *)
      break
      ;;
  esac
done
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

### Lines 50-54: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# Build final ENV_ARGS
ENV_ARGS=()
for key in "${!ENV_MAP[@]}"; do
  ENV_ARGS+=("-e" "$key=${ENV_MAP[$key]}")
done
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

### Lines 56-68: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
# Run docker exec with retry logic for HuggingFace network/download issues
# When HF model downloads fail due to network timeouts or rate limits,
# retrying with HF_HUB_OFFLINE=1 uses cached models from previous downloads.
#
# First attempt: normal mode (allows HF downloads)
if docker exec \
  -w "$WORKDIR" \
  "${ENV_ARGS[@]}" \
  ci_sglang "$@"; then
  exit 0
else
  FIRST_EXIT_CODE=$?
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It manages containerized steps.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会管理容器化步骤。

### Lines 70-70: Prints diagnostic status information / 打印诊断状态信息
```bash
echo "First attempt failed with exit code $FIRST_EXIT_CODE"
```
**EN:** This block prints resolved values so operators can confirm the runtime context before or after heavier actions run.
**CN:** 该代码块会打印解析后的关键值，便于操作者在执行重型步骤前后确认运行上下文。

### Lines 72-80: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# Skip retry for test failures that won't be fixed by offline mode:
#   - Exit 1: Test assertion failures (accuracy below threshold)
#   - Exit 137 (128+9): Process killed by OOM
#   - Exit 255: Test suite completed with test errors
# Only retry for other exit codes (e.g., network timeouts, HF download failures)
if [[ "$FIRST_EXIT_CODE" -eq 1 || "$FIRST_EXIT_CODE" -eq 137 || "$FIRST_EXIT_CODE" -eq 255 ]]; then
  echo "Exit code $FIRST_EXIT_CODE indicates test failure (not network issue), not retrying"
  exit $FIRST_EXIT_CODE
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 82-82: Prints diagnostic status information / 打印诊断状态信息
```bash
echo "Retrying with HF_HUB_OFFLINE=1 (offline mode to use cached models)..."
```
**EN:** This block prints resolved values so operators can confirm the runtime context before or after heavier actions run.
**CN:** 该代码块会打印解析后的关键值，便于操作者在执行重型步骤前后确认运行上下文。

### Lines 84-89: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
# Second attempt: force HF offline mode to avoid network timeouts
docker exec \
  -w "$WORKDIR" \
  "${ENV_ARGS[@]}" \
  -e HF_HUB_OFFLINE=1 \
  ci_sglang "$@"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **Container execution** / 容器执行
- **Runner provisioning** / 运行器准备

## Dependencies / 依赖关系
- **Commands / 外部命令**: `docker`, `echo`
- **Environment variables / 环境变量**: `BASH_REMATCH`, `ENV_ARGS`, `ENV_MAP`, `FIRST_EXIT_CODE`, `GPU_FAMILY`, `HOSTNAME_VALUE`, `WORKDIR`
