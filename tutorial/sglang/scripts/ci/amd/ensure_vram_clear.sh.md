# ensure_vram_clear.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/ci/amd/ensure_vram_clear.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: Source the VRAM checking function This shell script is part of SGLang's `amd` automation flow. / 该Shell 脚本用于支撑 SGLang 中的 `ensure_vram_clear` 流程，主要负责CI 编排、运行器选择与准备。它属于 `amd` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Shell entrypoint and interpreter selection / Shell 入口与解释器选择
```bash
#!/bin/bash
```
**EN:** This opening block selects Bash as the interpreter and usually also documents expected inputs or outputs for the rest of the script.
**CN:** 该起始代码块指定 Bash 作为解释器，并通常顺带说明脚本后续步骤所需的输入与输出。

### Lines 3-5: Comment or metadata block / 注释或元数据块
```bash
# Source the VRAM checking function
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
source "$SCRIPT_DIR/check_vram_clear.sh"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 7-14: Defines the `stop_all_gpu_containers` shell function / 定义 `stop_all_gpu_containers` shell 函数
```bash
# Stop and remove every container that holds any /dev/kfd or /dev/dri device.
# Some failing CI runs leave behind containers other than `ci_sglang` (e.g.
# from previous AMD jobs that were force-killed mid-run); those still hold
# VRAM via KFD even though the host pgrep finds nothing.
stop_all_gpu_containers() {
    if ! command -v docker >/dev/null 2>&1; then
        return 0
    fi
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 16-21: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
    local all_ids
    all_ids=$(docker ps -aq 2>/dev/null || true)
    if [ -z "$all_ids" ]; then
        echo "No docker containers found on host"
        return 0
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It manages containerized steps, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会管理容器化步骤、打印状态信息。

### Lines 23-33: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
    local gpu_ids=""
    local cid
    for cid in $all_ids; do
        # A container is "GPU-attached" if its inspect output mentions any
        # GPU device or NVIDIA/ROCm GPU capability. Inspecting the raw JSON
        # (instead of a specific field) survives docker version differences.
        if docker inspect "$cid" 2>/dev/null \
            | grep -qE '"PathOnHost":"/dev/(kfd|dri)|"Capabilities":\[\["gpu"\]\]'; then
            gpu_ids+=" $cid"
        fi
    done
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It manages containerized steps.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会管理容器化步骤。

### Lines 35-39: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
    gpu_ids=$(echo "$gpu_ids" | tr ' ' '\n' | grep -E '^[a-f0-9]+$' || true)
    if [ -z "$gpu_ids" ]; then
        echo "No GPU-attached docker containers found on host"
        return 0
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It manages containerized steps, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会管理容器化步骤、打印状态信息。

### Lines 41-47: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
    echo "Found GPU-attached containers, stopping them:"
    for cid in $gpu_ids; do
        docker ps -a --filter "id=$cid" --format '  {{.ID}} {{.Image}} {{.Status}} {{.Names}}' 2>/dev/null || true
    done
    echo "$gpu_ids" | xargs -r docker stop --time 5 2>/dev/null || true
    echo "$gpu_ids" | xargs -r docker rm -f 2>/dev/null || true
}
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It manages containerized steps, cleans generated files, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会管理容器化步骤、清理生成文件、打印状态信息。

### Lines 49-55: Defines the `kill_processes_holding_gpu_devices` shell function / 定义 `kill_processes_holding_gpu_devices` shell 函数
```bash
# Find and kill any host process that holds an open handle to /dev/kfd or
# /dev/dri/renderD*. This is far more reliable than `rocm-smi --showpids`,
# which only sees processes that registered a HSA queue (zombies and
# processes that crashed mid-init are invisible to it).
kill_processes_holding_gpu_devices() {
    local signal=${1:-TERM}
    local pids=""
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 57-67: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    # If neither tool is present we silently degrade to a no-op, which used
    # to look identical in the log to "no holders found" and made the
    # script's failures very confusing. Emit a loud warning so the runner
    # owner knows why GPU device cleanup isn't happening.
    if ! command -v fuser >/dev/null 2>&1 && ! command -v lsof >/dev/null 2>&1; then
        echo "WARNING: neither fuser nor lsof installed on the host;" \
             "cannot detect processes holding /dev/kfd or /dev/dri/renderD*."
        echo "         Install psmisc (for fuser) or lsof on the runner host" \
             "to enable device-fd-based cleanup."
        return 0
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 69-77: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if command -v fuser >/dev/null 2>&1; then
        # `fuser` prints PIDs to stdout, names to stderr; collect everything
        # that has any handle on KFD or render nodes.
        pids+=" $(fuser /dev/kfd 2>/dev/null || true)"
        for dev in /dev/dri/renderD*; do
            [ -e "$dev" ] || continue
            pids+=" $(fuser "$dev" 2>/dev/null || true)"
        done
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

### Lines 79-85: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if command -v lsof >/dev/null 2>&1; then
        pids+=" $(lsof -t /dev/kfd 2>/dev/null || true)"
        for dev in /dev/dri/renderD*; do
            [ -e "$dev" ] || continue
            pids+=" $(lsof -t "$dev" 2>/dev/null || true)"
        done
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

### Lines 87-90: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    pids=$(echo "$pids" | tr ' ' '\n' | grep -E '^[0-9]+$' | sort -u || true)
    if [ -z "$pids" ]; then
        return 0
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 92-120: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    local self_pid=$$
    echo "Processes holding /dev/kfd or /dev/dri/renderD*:"
    for pid in $pids; do
        # Skip our own PID and any of our ancestors so we don't suicide.
        if [ "$pid" = "$self_pid" ]; then
            continue
        fi
        local cmd
        cmd=$(ps -p "$pid" -o pid,ppid,stat,cmd --no-headers 2>/dev/null || true)
        if [ -z "$cmd" ]; then
            continue
        fi
        echo "  $cmd"
        # If it's a zombie, kill the parent instead — kill -9 on a zombie is
        # a no-op, the only way to reap it is to make its parent reap it.
        local stat
        stat=$(ps -p "$pid" -o stat= 2>/dev/null | tr -d ' ')
        if [[ "$stat" == Z* ]]; then
            local ppid
            ppid=$(ps -p "$pid" -o ppid= 2>/dev/null | tr -d ' ')
            if [ -n "$ppid" ] && [ "$ppid" != "1" ] && [ "$ppid" != "$self_pid" ]; then
                echo "    -> $pid is a zombie, sending SIG$signal to parent $ppid"
                kill "-$signal" "$ppid" 2>/dev/null || true
            fi
            continue
        fi
        kill "-$signal" "$pid" 2>/dev/null || true
    done
}
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It builds native artifacts, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会构建原生产物、打印状态信息。

### Lines 122-134: Defines the `dump_gpu_diagnostics` shell function / 定义 `dump_gpu_diagnostics` shell 函数
```bash
# Print rich diagnostics that explain *why* VRAM is still allocated when no
# obvious owner exists. Helpful when zombies / other namespaces hold memory.
dump_gpu_diagnostics() {
    echo "=== GPU device file holders (fuser) ==="
    if command -v fuser >/dev/null 2>&1; then
        fuser -v /dev/kfd 2>&1 || true
        for dev in /dev/dri/renderD* /dev/dri/card*; do
            [ -e "$dev" ] || continue
            fuser -v "$dev" 2>&1 || true
        done
    else
        echo "fuser not installed"
    fi
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 136-142: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    echo "=== GPU device file holders (lsof) ==="
    if command -v lsof >/dev/null 2>&1; then
        lsof /dev/kfd 2>/dev/null || true
        lsof /dev/dri/renderD* 2>/dev/null || true
    else
        echo "lsof not installed"
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 144-145: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
    echo "=== Zombie processes on host ==="
    ps -eo pid,ppid,stat,etime,cmd 2>/dev/null | awk 'NR==1 || $3 ~ /^Z/ {print}'
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 147-150: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
    echo "=== Docker containers on host ==="
    if command -v docker >/dev/null 2>&1; then
        docker ps -a --format 'table {{.ID}}\t{{.Image}}\t{{.Status}}\t{{.Names}}' 2>/dev/null || true
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It manages containerized steps, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会管理容器化步骤、打印状态信息。

### Lines 152-153: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
    echo "=== rocm-smi --showpids ==="
    timeout 30 rocm-smi --showpids 2>&1 || echo "rocm-smi --showpids timed out"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 155-157: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
    echo "=== rocm-smi --showmemuse ==="
    timeout 30 rocm-smi --showmemuse 2>&1 || echo "rocm-smi --showmemuse timed out"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 159-161: Defines the `ensure_vram_clear` shell function / 定义 `ensure_vram_clear` shell 函数
```bash
ensure_vram_clear() {
    local max_retries=3
    local retry_count=0
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 163-170: Prints diagnostic status information / 打印诊断状态信息
```bash
    # Log host information for debugging
    echo "=== Host Information ==="
    echo "Hostname: $(hostname)"
    echo "Host IP: $(hostname -I 2>/dev/null || echo 'N/A')"
    echo "Date: $(date)"
    echo "Mode: rocm"
    echo "========================"
    echo "Running in ROCm mode"
```
**EN:** This block prints resolved values so operators can confirm the runtime context before or after heavier actions run.
**CN:** 该代码块会打印解析后的关键值，便于操作者在执行重型步骤前后确认运行上下文。

### Lines 172-175: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
    # Always stop the well-known CI container first (best-effort).
    echo "Stopping any existing ci_sglang container..."
    docker stop ci_sglang 2>/dev/null || true
    docker rm -f ci_sglang 2>/dev/null || true
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, cleans generated files, prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、清理生成文件、打印状态信息。

### Lines 177-180: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
    # Show initial GPU status
    echo "=== Initial GPU Memory Status ==="
    rocm-smi --showmemuse
    echo "=================================="
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 182-187: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    # Fast path: if the runner is already clean, skip the cleanup loop
    # entirely so healthy jobs don't pay the ~35s/attempt cleanup cost.
    if check_vram_clear; then
        echo "✓ VRAM is already clear; skipping cleanup."
        return 0
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 189-190: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    while [ $retry_count -lt $max_retries ]; do
        echo "=== Cleanup Attempt $((retry_count + 1))/$max_retries ==="
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 192-199: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
        # Step 1: kill SGLang-named processes on the host (cheap, fast).
        # NOTE: host pgrep cannot see PIDs inside a container's PID
        # namespace, so in CI this almost never matches anything; the
        # heavy lifting is done by step 2 below. Kept as a fast early
        # cleanup for the rare case where something runs on the host.
        echo "Killing SGLang processes..."
        pgrep -f 'sglang::|sglang\.launch_server|sglang\.bench|sglang\.data_parallel|sglang\.srt' \
            | xargs -r kill -9 2>/dev/null || true
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 201-205: Prints diagnostic status information / 打印诊断状态信息
```bash
        # Step 2: aggressive cleanup. Run on EVERY attempt — the previous
        # version skipped this on attempt 1, which made attempt 1 a near
        # no-op for the most common failure mode (a leftover container
        # holding VRAM, invisible to host pgrep).
        echo "Performing aggressive cleanup..."
```
**EN:** This block prints resolved values so operators can confirm the runtime context before or after heavier actions run.
**CN:** 该代码块会打印解析后的关键值，便于操作者在执行重型步骤前后确认运行上下文。

### Lines 207-210: Comment or metadata block / 注释或元数据块
```bash
        # 2a. Stop ALL GPU-attached containers, not just ci_sglang. A
        # leftover container from a previous job will keep VRAM held even
        # though `pgrep` on the host shows nothing.
        stop_all_gpu_containers
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 212-217: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
        # 2b. SIGTERM anything that has /dev/kfd or /dev/dri/renderD* open.
        # `lsof`/`fuser` see processes that `rocm-smi --showpids` misses
        # (notably zombies and processes outside our PID namespace).
        echo "Sending SIGTERM to processes holding GPU device files..."
        kill_processes_holding_gpu_devices TERM
        sleep 5
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 219-221: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
        # 2c. SIGKILL anything still holding GPU device files.
        echo "Sending SIGKILL to remaining holders..."
        kill_processes_holding_gpu_devices KILL
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 223-229: Executes `grep`-based shell logic / 执行基于 `grep` 的 shell 逻辑
```bash
        # 2d. Best-effort: also kill anything `rocm-smi --showpids` reports.
        # Handles both the legacy "PID: <n>" line format and the modern
        # tabular format (`<pid>\t<name>\t<gpus>\t...`); the previous
        # `grep 'PID:'` matched nothing on ROCm 5+ tabular output.
        rocm-smi --showpids 2>/dev/null \
            | awk '/^PID:[[:space:]]*[0-9]+/ {print $2} /^[0-9]+/ {print $1}' \
            | xargs -r kill -9 2>/dev/null || true
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 231-232: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
        echo "Waiting 30 seconds for VRAM to clear..."
        sleep 30
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 234-250: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
        # Step 3: re-check.
        echo "Checking VRAM status..."
        if check_vram_clear; then
            echo "✓ VRAM cleanup successful after $((retry_count + 1)) attempts"
            return 0
        else
            echo "✗ VRAM still not clear after attempt $((retry_count + 1))"
            # Step 4: dump diagnostics on every failed attempt so the next
            # attempt's logs already explain WHY cleanup didn't work.
            # Without this we'd only see what's holding the GPU at the very
            # end, which makes triage much harder.
            echo "--- Diagnostics for failed attempt $((retry_count + 1)) ---"
            dump_gpu_diagnostics
            echo "--- End of diagnostics for attempt $((retry_count + 1)) ---"
            retry_count=$((retry_count + 1))
        fi
    done
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 252-263: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
    # Failed after all retries — diagnostics for the last cleanup attempt
    # were already dumped above; just print the actionable hint.
    echo "=== FAILED: VRAM cleanup unsuccessful after $max_retries attempts ==="
    echo "(See diagnostics above for the final attempt.)"
    echo "=================================================================="
    echo "Hint: if no host process / container holds the GPU but VRAM is"
    echo "still allocated, this is almost certainly a zombie KFD context"
    echo "(see ROCm/aiter#2061). The node will need to be rebooted before"
    echo "subsequent jobs can succeed."
    echo "=================================================================="
    return 1
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 265-269: Enables strict shell execution flags / 启用严格的 shell 执行选项
```bash
# If this script is run directly (not sourced), run the ensure function
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    set -e
    ensure_vram_clear "$@"
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **Structured data handling** / 结构化数据处理
- **Container execution** / 容器执行
- **Runner provisioning** / 运行器准备

## Dependencies / 依赖关系
- **Commands / 外部命令**: `docker`, `make`, `rm`, `echo`, `grep`
- **Environment variables / 环境变量**: `BASH_SOURCE`, `SCRIPT_DIR`
- **Referenced files / 引用文件**: `SCRIPT_DIR/check_vram_clear.sh`
