# launch_gb200.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/ci/slurm/launch_gb200.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: Launch a dynamo-sglang benchmark job on the GB200 cluster via srt-slurm. This shell script is part of SGLang's `slurm` automation flow. / 该Shell 脚本用于支撑 SGLang 中的 `launch_gb200` 流程，主要负责CI 编排、基准测试自动化。它属于 `slurm` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Shell entrypoint and interpreter selection / Shell 入口与解释器选择
```bash
#!/usr/bin/env bash
# Launch a dynamo-sglang benchmark job on the GB200 cluster via srt-slurm.
#
# Required environment variables (set by the GitHub Actions workflow):
#   FRAMEWORK         - must be "dynamo-sglang"
#   MODEL             - HuggingFace model ID (used as fallback if no local path)
#   MODEL_PREFIX      - short prefix: "dsr1"
#   PRECISION         - "fp8" or "fp4"
#   ISL               - input sequence length (e.g. "1024")
#   OSL               - output sequence length (e.g. "1024")
#   CONFIG_FILE       - path relative to srt-slurm repo root (e.g. recipes/gb200-fp8/1k1k/low-latency.yaml)
#   RESULT_FILENAME   - prefix for output JSON filenames
#   RUNNER_NAME       - GitHub Actions runner name (used to tag the Slurm job)
#   SQUASH_FILE       - path to pre-imported sglang enroot squash file on Lustre
#   NGINX_SQUASH_FILE - path to pre-imported nginx enroot squash file on Lustre
#   SLURM_PARTITION   - Slurm partition (default: batch)
#   SLURM_ACCOUNT     - Slurm account  (default: sglang)
#   SRT_SLURM_BRANCH  - branch of srt-slurm repo to check out
#   GITHUB_WORKSPACE  - set automatically by GitHub Actions
#   MATRIX_CONFIG_NAME- matrix entry name (e.g. dsr1-fp4-1k1k-mid-curve); used in S3 prefix
#   S3_BUCKET         - MinIO bucket for benchmark log uploads
#   S3_ENDPOINT_URL   - MinIO endpoint URL (e.g. https://minio.<host>.nip.io)
#   AWS_ACCESS_KEY_ID - writer access key for S3_BUCKET (via GH secrets)
#   AWS_SECRET_ACCESS_KEY - writer secret key for S3_BUCKET (via GH secrets)
```
**EN:** This opening block selects Bash as the interpreter and usually also documents expected inputs or outputs for the rest of the script.
**CN:** 该起始代码块指定 Bash 作为解释器，并通常顺带说明脚本后续步骤所需的输入与输出。

### Lines 26-27: Enables strict shell execution flags / 启用严格的 shell 执行选项
```bash
set -euo pipefail
set -x
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 29-42: Comment or metadata block / 注释或元数据块
```bash
# ---------------------------------------------------------------------------
# Validate required vars
# ---------------------------------------------------------------------------
: "${FRAMEWORK:?}"
: "${MODEL_PREFIX:?}"
: "${PRECISION:?}"
: "${ISL:?}"
: "${OSL:?}"
: "${CONFIG_FILE:?}"
: "${RESULT_FILENAME:?}"
: "${RUNNER_NAME:?}"
: "${SQUASH_FILE:?}"
: "${NGINX_SQUASH_FILE:?}"
: "${GITHUB_WORKSPACE:?}"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 44-46: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
SLURM_PARTITION="${SLURM_PARTITION:-batch}"
SLURM_ACCOUNT="${SLURM_ACCOUNT:-sglang}"
SRT_SLURM_BRANCH="${SRT_SLURM_BRANCH:-sglang-nightly-regression}"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 48-60: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# ---------------------------------------------------------------------------
# Resolve local model paths on Lustre (avoids re-downloading on each run)
# ---------------------------------------------------------------------------
if [[ "$MODEL_PREFIX" == "dsr1" && "$PRECISION" == "fp8" ]]; then
    MODEL_PATH="/mnt/lustre01/models/deepseek-r1-0528"
    SRT_SLURM_MODEL_PREFIX="dsr1-fp8"
elif [[ "$MODEL_PREFIX" == "dsr1" && "$PRECISION" == "fp4" ]]; then
    MODEL_PATH="/mnt/lustre01/models/deepseek-r1-0528-fp4-v2/"
    SRT_SLURM_MODEL_PREFIX="dsr1-fp4"
else
    MODEL_PATH="$MODEL"
    SRT_SLURM_MODEL_PREFIX="$MODEL_PREFIX"
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

### Lines 62-68: Prepares directories and generated artifacts / 准备目录与生成产物
```bash
# ---------------------------------------------------------------------------
# Set up per-runner Lustre workspace (cleaned before each run, accessible
# to both the runner and compute nodes)
# ---------------------------------------------------------------------------
LUSTRE_WORKSPACE="/mnt/lustre01/users-public/sglang-ci/workspace/${RUNNER_NAME}"
rm -rf "$LUSTRE_WORKSPACE"
mkdir -p "$LUSTRE_WORKSPACE"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prepares directories, cleans generated files.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会准备目录、清理生成文件。

### Lines 70-73: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
# ---------------------------------------------------------------------------
# Clone and set up srt-slurm
# ---------------------------------------------------------------------------
SRT_REPO_DIR="$LUSTRE_WORKSPACE/srt-slurm"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 75-79: Executes `git`-based shell logic / 执行基于 `git` 的 shell 逻辑
```bash
git clone https://github.com/NVIDIA/srt-slurm.git "$SRT_REPO_DIR"
cd "$SRT_REPO_DIR"
git checkout "$SRT_SLURM_BRANCH"
echo "--- srt-slurm last commit ---"
git log -1 --format="commit %H%nauthor %an%ndate   %ad%nsubject %s" --date=iso
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It queries repository state, prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会查询仓库状态、打印状态信息。

### Lines 81-82: Executes `sh`-based shell logic / 执行基于 `sh` 的 shell 逻辑
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source "$HOME/.local/bin/env"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 84-86: Invokes Python-based tooling / 调用基于 Python 的工具
```bash
uv venv
source .venv/bin/activate
uv pip install -e .
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It installs Python packages, coordinates Python tooling.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会安装 Python 包、协调 Python 工具链。

### Lines 88-91: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if ! command -v srtctl &>/dev/null; then
    echo "ERROR: srtctl installation failed"
    exit 1
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 93-96: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
# ---------------------------------------------------------------------------
# Generate srtslurm.yaml
# ---------------------------------------------------------------------------
SRTCTL_ROOT="$SRT_REPO_DIR"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 98-104: Comment or metadata block / 注释或元数据块
```bash
: "${S3_BUCKET:?S3_BUCKET must be set}"
: "${S3_ENDPOINT_URL:?S3_ENDPOINT_URL must be set}"
: "${AWS_ACCESS_KEY_ID:?AWS_ACCESS_KEY_ID must be set}"
: "${AWS_SECRET_ACCESS_KEY:?AWS_SECRET_ACCESS_KEY must be set}"
: "${MATRIX_CONFIG_NAME:?MATRIX_CONFIG_NAME must be set}"
: "${GITHUB_RUN_ID:?GITHUB_RUN_ID must be set}"
: "${GITHUB_RUN_ATTEMPT:?GITHUB_RUN_ATTEMPT must be set}"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 106-111: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# Map the GitHub trigger into a friendlier top-level prefix: cron/manual.
case "${GITHUB_EVENT_NAME:-}" in
    schedule)          TRIGGER=cron ;;
    workflow_dispatch) TRIGGER=manual ;;
    *)                 TRIGGER="${GITHUB_EVENT_NAME:-unknown}" ;;
esac
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

### Lines 113-119: Defines the `fmt_seq_len` shell function / 定义 `fmt_seq_len` shell 函数
```bash
# Format ISL/OSL as "1k1k" / "1k8k" / "8k1k" etc. for the S3 prefix, so logs
# group naturally by sequence-length bucket under each run.
fmt_seq_len() {
    local n=$1
    if (( n % 1024 == 0 )); then echo "$((n / 1024))k"; else echo "$n"; fi
}
SEQ_LEN="$(fmt_seq_len "$ISL")$(fmt_seq_len "$OSL")"
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 121-121: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
S3_PREFIX="${TRIGGER}/${GITHUB_RUN_ID}-${GITHUB_RUN_ATTEMPT}/${SEQ_LEN}/${MATRIX_CONFIG_NAME}"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 123-127: Comment or metadata block / 注释或元数据块
```bash
cat > srtslurm.yaml <<EOF
# SRT SLURM configuration for SGLang GB200 nightly CI
default_account: "${SLURM_ACCOUNT}"
default_partition: "${SLURM_PARTITION}"
default_time_limit: "6:00:00"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 129-130: Comment or metadata block / 注释或元数据块
```bash
gpus_per_node: 4
network_interface: ""
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 132-132: Comment or metadata block / 注释或元数据块
```bash
srtctl_root: "${SRTCTL_ROOT}"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 134-135: Comment or metadata block / 注释或元数据块
```bash
model_paths:
  "${SRT_SLURM_MODEL_PREFIX}": "${MODEL_PATH}"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 137-140: Comment or metadata block / 注释或元数据块
```bash
containers:
  dynamo-sglang: ${SQUASH_FILE}
  nginx: ${NGINX_SQUASH_FILE}
  nginx-sqsh: ${NGINX_SQUASH_FILE}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 142-150: Comment or metadata block / 注释或元数据块
```bash
# srt-slurm postprocess uploads /logs to this bucket after each Slurm job.
# Credentials are read from AWS_ACCESS_KEY_ID / AWS_SECRET_ACCESS_KEY env vars,
# not written to disk. srt-slurm appends /<date>/<slurm-job-id>/ after prefix.
reporting:
  s3:
    bucket: "${S3_BUCKET}"
    prefix: "${S3_PREFIX}"
    endpoint_url: "${S3_ENDPOINT_URL}"
EOF
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 152-154: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
echo "--- srtslurm.yaml ---"
cat srtslurm.yaml
echo "--- S3 log upload: s3://${S3_BUCKET}/${S3_PREFIX}/ ---"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 156-156: Executes `make`-based shell logic / 执行基于 `make` 的 shell 逻辑
```bash
make setup ARCH=aarch64
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It builds native artifacts.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会构建原生产物。

### Lines 158-161: Executes `sed`-based shell logic / 执行基于 `sed` 的 shell 逻辑
```bash
# ---------------------------------------------------------------------------
# Patch job name and submit via srtctl
# ---------------------------------------------------------------------------
sed -i "s/^name:.*/name: \"${RUNNER_NAME}\"/" "$CONFIG_FILE"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 163-166: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
SRTCTL_OUTPUT=$(srtctl apply -f "$CONFIG_FILE" \
    --tags "gb200,${MODEL_PREFIX},${PRECISION},${ISL}x${OSL},sglang-nightly-$(date +%Y%m%d)" \
    --setup-script install-torchao.sh 2>&1)
echo "$SRTCTL_OUTPUT"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 168-168: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
JOB_ID=$(echo "$SRTCTL_OUTPUT" | grep -oP '✅ Job \K[0-9]+' || echo "$SRTCTL_OUTPUT" | grep -oP 'Job \K[0-9]+' || true)
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 170-173: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if [ -z "$JOB_ID" ]; then
    echo "ERROR: Could not extract JOB_ID from srtctl output"
    exit 1
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 175-175: Prints diagnostic status information / 打印诊断状态信息
```bash
echo "Submitted Slurm job: $JOB_ID"
```
**EN:** This block prints resolved values so operators can confirm the runtime context before or after heavier actions run.
**CN:** 该代码块会打印解析后的关键值，便于操作者在执行重型步骤前后确认运行上下文。

### Lines 177-177: Comment or metadata block / 注释或元数据块
```bash
set +x
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 179-183: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
# ---------------------------------------------------------------------------
# Wait for job and stream logs
# ---------------------------------------------------------------------------
LOGS_DIR="outputs/$JOB_ID/logs"
LOG_FILE="$LOGS_DIR/sweep_${JOB_ID}.log"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 185-185: Prepares directories and generated artifacts / 准备目录与生成产物
```bash
mkdir -p "$LOGS_DIR"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prepares directories.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会准备目录。

### Lines 187-195: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
while ! ls "$LOG_FILE" &>/dev/null; do
    if ! squeue -j "$JOB_ID" --noheader 2>/dev/null | grep -q "$JOB_ID"; then
        echo "ERROR: Job $JOB_ID failed before creating log file"
        scontrol show job "$JOB_ID" || true
        exit 1
    fi
    echo "Waiting for job $JOB_ID to start and $LOG_FILE to appear..."
    sleep 5
done
```
**EN:** This block summarizes the produced artifacts, making the final output set easy to inspect in CI logs or local runs.
**CN:** 该代码块会汇总最终产物，便于在 CI 日志或本地执行时快速核对输出结果。

### Lines 197-202: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
(
    while squeue -j "$JOB_ID" --noheader 2>/dev/null | grep -q "$JOB_ID"; do
        sleep 10
    done
) &
POLL_PID=$!
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

### Lines 204-204: Comment or metadata block / 注释或元数据块
```bash
tail -F -s 2 -n+1 "$LOG_FILE" --pid=$POLL_PID 2>/dev/null
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 206-206: Comment or metadata block / 注释或元数据块
```bash
wait $POLL_PID
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 208-208: Enables strict shell execution flags / 启用严格的 shell 执行选项
```bash
set -x
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 210-210: Prints diagnostic status information / 打印诊断状态信息
```bash
echo "Job $JOB_ID completed. Collecting results..."
```
**EN:** This block prints resolved values so operators can confirm the runtime context before or after heavier actions run.
**CN:** 该代码块会打印解析后的关键值，便于操作者在执行重型步骤前后确认运行上下文。

### Lines 212-218: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# ---------------------------------------------------------------------------
# Collect results
# ---------------------------------------------------------------------------
if [ ! -d "$LOGS_DIR" ]; then
    echo "WARNING: Logs directory not found at $LOGS_DIR"
    exit 1
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 220-221: Prepares directories and generated artifacts / 准备目录与生成产物
```bash
cp -r "$LOGS_DIR" "$GITHUB_WORKSPACE/LOGS"
tar czf "$GITHUB_WORKSPACE/multinode_server_logs.tar.gz" -C "$LOGS_DIR" .
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It copies artifacts.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会复制产物。

### Lines 223-223: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
RESULT_SUBDIRS=$(find "$LOGS_DIR" -maxdepth 1 -type d -name "*isl*osl*" 2>/dev/null || true)
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 225-251: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if [ -z "$RESULT_SUBDIRS" ]; then
    echo "ERROR: No result subdirectories found in $LOGS_DIR — benchmark did not produce any output"
    exit 1
else
    RESULT_COUNT=0
    for result_subdir in $RESULT_SUBDIRS; do
        CONFIG_NAME=$(basename "$result_subdir")
        RESULT_FILES=$(find "$result_subdir" -name "results_concurrency_*.json" 2>/dev/null || true)
        for result_file in $RESULT_FILES; do
            if [ -f "$result_file" ]; then
                filename=$(basename "$result_file")
                concurrency=$(echo "$filename" | sed -n 's/results_concurrency_\([0-9]*\)_gpus_.*/\1/p')
                gpus=$(echo "$filename" | sed -n 's/results_concurrency_[0-9]*_gpus_\([0-9]*\)_ctx_.*/\1/p')
                ctx=$(echo "$filename" | sed -n 's/.*_ctx_\([0-9]*\)_gen_.*/\1/p')
                gen=$(echo "$filename" | sed -n 's/.*_gen_\([0-9]*\)\.json/\1/p')
                DEST="$GITHUB_WORKSPACE/${RESULT_FILENAME}_${CONFIG_NAME}_conc${concurrency}_gpus_${gpus}_ctx_${ctx}_gen_${gen}.json"
                cp "$result_file" "$DEST"
                echo "Saved: $DEST"
                RESULT_COUNT=$((RESULT_COUNT + 1))
            fi
        done
    done
    if [ "$RESULT_COUNT" -eq 0 ]; then
        echo "ERROR: Result subdirectories found but no result JSON files produced — benchmark failed"
        exit 1
    fi
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It copies artifacts, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会复制产物、打印状态信息。

### Lines 253-253: Prints diagnostic status information / 打印诊断状态信息
```bash
echo "Done."
```
**EN:** This block prints resolved values so operators can confirm the runtime context before or after heavier actions run.
**CN:** 该代码块会打印解析后的关键值，便于操作者在执行重型步骤前后确认运行上下文。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **YAML configuration** / YAML 配置
- **Structured data handling** / 结构化数据处理
- **Git state inspection** / Git 状态检查
- **Cluster scheduling** / 集群调度

## Dependencies / 依赖关系
- **Commands / 外部命令**: `git`, `pip`, `uv`, `make`, `mkdir`, `cp`, `rm`, `echo`, `grep`, `sed`, `find`, `ls`, `bash`, `sh`, `curl`
- **Environment variables / 环境变量**: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `CONFIG_FILE`, `CONFIG_NAME`, `DEST`, `FRAMEWORK`, `GITHUB_EVENT_NAME`, `GITHUB_RUN_ATTEMPT`, `GITHUB_RUN_ID`, `GITHUB_WORKSPACE`, `HOME`, `ISL`, `JOB_ID`, `LOGS_DIR`, `LOG_FILE`
- **Referenced files / 引用文件**: `//astral.sh/uv/install.sh`, `install-torchao.sh`, `recipes/gb200-fp8/1k1k/low-latency.yaml`, `srtslurm.yaml`
