# ci_start_disaggregation_servers.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/ci/cuda/ci_start_disaggregation_servers.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: This shell script supports the `ci_start_disaggregation_servers` workflow in SGLang. It mainly handles CI orchestration, policy validation. / 该Shell 脚本用于支撑 SGLang 中的 `ci_start_disaggregation_servers` 流程，主要负责CI 编排、策略校验。它属于 `cuda` 自动化路径的一部分。

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
# Optional: set DISAGG_READY_FILE to a filepath; when all servers are healthy, the script will
# create this file as a readiness signal (useful for CI to proceed to next steps).
DISAGG_READY_FILE="${DISAGG_READY_FILE:-}"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 8-8: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
MODEL_PATH="/raid/models/meta-llama/Llama-3.1-8B-Instruct"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 10-23: Defines the `find_active_ib_device` shell function / 定义 `find_active_ib_device` shell 函数
```bash
# Function to find the first available active IB device
find_active_ib_device() {
    for device in mlx5_{0..11}; do
        if ibv_devinfo $device >/dev/null 2>&1; then
            state=$(ibv_devinfo $device | grep "state:" | head -1 | awk '{print $2}')
            if [[ "$state" == "PORT_ACTIVE" ]]; then
                echo "$device"
                return 0
            fi
        fi
    done
    echo "No active IB device found" >&2
    return 1
}
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 25-27: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
# Get the first available active IB device
DEVICE=$(find_active_ib_device)
echo "Using IB device: $DEVICE"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 29-43: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
# Launch prefill servers on GPU 0–3
for i in {0..3}; do
  PORT=$((30001 + i))
  BOOTSTRAP_PORT=$((9001 + i))
  HOST="127.0.0.$((i + 1))"
  echo "Launching PREFILL server on GPU $i at $HOST:$PORT (bootstrap: $BOOTSTRAP_PORT)"
  CUDA_VISIBLE_DEVICES=$i \
  python3 -m sglang.launch_server \
    --model-path "$MODEL_PATH" \
    --disaggregation-mode prefill \
    --host "$HOST" \
    --port "$PORT" \
    --disaggregation-ib-device "$DEVICE" \
    --disaggregation-bootstrap-port "$BOOTSTRAP_PORT" &
done
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It invokes Python helpers, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会调用 Python 辅助脚本、打印状态信息。

### Lines 45-58: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
# Launch decode servers on GPU 4–7
for i in {4..7}; do
  PORT=$((30001 + i))
  HOST="127.0.0.$((i + 1))"
  echo "Launching DECODE server on GPU $i at $HOST:$PORT"
  CUDA_VISIBLE_DEVICES=$i \
  python3 -m sglang.launch_server \
    --model-path "$MODEL_PATH" \
    --disaggregation-mode decode \
    --host "$HOST" \
    --port "$PORT" \
    --disaggregation-ib-device "$DEVICE" \
    --base-gpu-id 0 &
done
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It invokes Python helpers, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会调用 Python 辅助脚本、打印状态信息。

### Lines 60-61: Prints diagnostic status information / 打印诊断状态信息
```bash
# Wait for disaggregation servers to initialize
echo "Waiting for disaggregation servers to initialize..."
```
**EN:** This block prints resolved values so operators can confirm the runtime context before or after heavier actions run.
**CN:** 该代码块会打印解析后的关键值，便于操作者在执行重型步骤前后确认运行上下文。

### Lines 63-65: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
# Health check with 5-minute timeout
TIMEOUT=300
START_TIME=$(date +%s)
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 67-70: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
echo "Checking health of all 8 servers..."
while true; do
    CURRENT_TIME=$(date +%s)
    ELAPSED=$((CURRENT_TIME - START_TIME))
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 72-75: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ $ELAPSED -ge $TIMEOUT ]; then
        echo "❌ Timeout: Servers did not become healthy within 5 minutes"
        exit 1
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 77-83: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    HEALTHY_COUNT=0
    # Check all 8 servers (127.0.0.1-8:30001-30008)
    for i in {1..8}; do
        if curl -s -f "http://127.0.0.$i:$((30000 + i))/health" >/dev/null 2>&1; then
            HEALTHY_COUNT=$((HEALTHY_COUNT + 1))
        fi
    done
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

### Lines 85-85: Prints diagnostic status information / 打印诊断状态信息
```bash
    echo "Healthy servers: $HEALTHY_COUNT/8 (elapsed: ${ELAPSED}s)"
```
**EN:** This block prints resolved values so operators can confirm the runtime context before or after heavier actions run.
**CN:** 该代码块会打印解析后的关键值，便于操作者在执行重型步骤前后确认运行上下文。

### Lines 87-100: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ $HEALTHY_COUNT -eq 8 ]; then
        echo "✅ All 8 servers are healthy!"
        # Emit readiness signal file if requested
        if [ -n "$DISAGG_READY_FILE" ]; then
            echo "Creating readiness flag: $DISAGG_READY_FILE"
            # Ensure parent dir exists; ignore errors
            mkdir -p "$(dirname "$DISAGG_READY_FILE")" 2>/dev/null || true
            touch "$DISAGG_READY_FILE"
        fi
        break
    else
        sleep 10  # Wait 10 seconds before next check
    fi
done
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prepares directories, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会准备目录、打印状态信息。

### Lines 102-103: Prints diagnostic status information / 打印诊断状态信息
```bash
# Don't launch router here - just keep servers running
echo "✅ All disaggregation servers are ready and waiting for router connections"
```
**EN:** This block prints resolved values so operators can confirm the runtime context before or after heavier actions run.
**CN:** 该代码块会打印解析后的关键值，便于操作者在执行重型步骤前后确认运行上下文。

### Lines 105-106: Comment or metadata block / 注释或元数据块
```bash
# Keep the script running
wait
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

## Key Concepts / 关键概念
- **Environment management** / 环境管理

## Dependencies / 依赖关系
- **Commands / 外部命令**: `python3`, `mkdir`, `echo`, `grep`, `find`, `head`, `curl`
- **Environment variables / 环境变量**: `BOOTSTRAP_PORT`, `DEVICE`, `DISAGG_READY_FILE`, `ELAPSED`, `HEALTHY_COUNT`, `HOST`, `MODEL_PATH`, `PORT`, `TIMEOUT`
