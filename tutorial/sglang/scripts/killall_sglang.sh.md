# killall_sglang.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/killall_sglang.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: DEPRECATED: This script will be migrated to python/sglang/cli/killall.py. This shell script is part of SGLang's `scripts` automation flow. / 该Shell 脚本用于支撑 SGLang 中的 `killall_sglang` 流程，主要负责CI 编排、构建自动化。它属于 `scripts` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Shell entrypoint and interpreter selection / Shell 入口与解释器选择
```bash
#!/bin/bash
```
**EN:** This opening block selects Bash as the interpreter and usually also documents expected inputs or outputs for the rest of the script.
**CN:** 该起始代码块指定 Bash 作为解释器，并通常顺带说明脚本后续步骤所需的输入与输出。

### Lines 3-12: Comment or metadata block / 注释或元数据块
```bash
# DEPRECATED: This script will be migrated to python/sglang/cli/killall.py.
# CI mode is already handled there. This script remains for local/non-CI usage.
#
# TODO: Migrate remaining modes (rocm, all, gpus) to killall.py and remove this file.
#
# Usage:
#   ./killall_sglang.sh              - Kill SGLang processes only (NVIDIA mode)
#   ./killall_sglang.sh rocm         - Kill SGLang processes only (ROCm mode)
#   ./killall_sglang.sh all          - Kill all GPU processes (NVIDIA mode)
#   ./killall_sglang.sh gpus 0,1,2,3 - Kill all processes on specific GPUs
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 14-15: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if [ "$1" = "rocm" ]; then
    echo "Running in ROCm mode"
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 17-18: Comment or metadata block / 注释或元数据块
```bash
    # Clean SGLang processes
    pgrep -f 'sglang::|sglang\.launch_server|sglang\.bench|sglang\.data_parallel|sglang\.srt|sgl_diffusion::' | xargs -r kill -9
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 20-22: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
elif [ "$1" = "gpus" ] && [ -n "$2" ]; then
    # Kill all processes on specific GPUs only
    echo "Killing all processes on GPUs: $2"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 24-25: Comment or metadata block / 注释或元数据块
```bash
    # Show current GPU status
    nvidia-smi
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 27-29: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
    # Build device file list from GPU IDs (e.g., "0,1,2,3" -> "/dev/nvidia0 /dev/nvidia1 ...")
    devices=$(echo "$2" | tr ',' '\n' | sed 's/^[[:space:]]*//;s/[[:space:]]*$//' | sed 's|^|/dev/nvidia|' | tr '\n' ' ')
    echo "Targeting devices: $devices"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 31-32: Comment or metadata block / 注释或元数据块
```bash
    # Kill all processes using specified GPU devices
    [ -n "$devices" ] && lsof $devices 2>/dev/null | awk 'NR>1 {print $2}' | sort -u | xargs -r kill -9 2>/dev/null
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 34-35: Comment or metadata block / 注释或元数据块
```bash
    # Show GPU status after clean up
    nvidia-smi
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 37-39: Comment or metadata block / 注释或元数据块
```bash
else
    # Show current GPU status
    nvidia-smi
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 41-42: Comment or metadata block / 注释或元数据块
```bash
    # Clean SGLang processes
    pgrep -f 'sglang::|sglang\.launch_server|sglang\.bench|sglang\.data_parallel|sglang\.srt|sgl_diffusion::' | xargs -r kill -9
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 44-56: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    # Clean all GPU processes if "all" argument is provided
    if [ "$1" = "all" ]; then
        # Check if sudo is available
        if command -v sudo >/dev/null 2>&1; then
            sudo apt-get update
            sudo apt-get install -y lsof
        else
            apt-get update
            apt-get install -y lsof
        fi
        kill -9 $(nvidia-smi | sed -n '/Processes:/,$p' | grep "   [0-9]" | awk '{print $5}') 2>/dev/null
        lsof /dev/nvidia* | awk '{print $2}' | xargs kill -9 2>/dev/null
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

### Lines 58-60: Comment or metadata block / 注释或元数据块
```bash
    # Show GPU status after clean up
    nvidia-smi
fi
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

## Key Concepts / 关键概念
- **Shell automation flow** / Shell 自动化流程

## Dependencies / 依赖关系
- **Commands / 外部命令**: `echo`, `grep`, `sed`
- **Referenced files / 引用文件**: `./killall_sglang.sh`, `killall.py`, `python/sglang/cli/killall.py`
