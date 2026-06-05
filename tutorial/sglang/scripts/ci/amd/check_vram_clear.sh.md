# check_vram_clear.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/ci/amd/check_vram_clear.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: This shell script supports the `check_vram_clear` workflow in SGLang. It mainly handles CI orchestration, policy validation. / 该Shell 脚本用于支撑 SGLang 中的 `check_vram_clear` 流程，主要负责CI 编排、策略校验。它属于 `amd` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Shell entrypoint and interpreter selection / Shell 入口与解释器选择
```bash
#!/bin/bash
```
**EN:** This opening block selects Bash as the interpreter and usually also documents expected inputs or outputs for the rest of the script.
**CN:** 该起始代码块指定 Bash 作为解释器，并通常顺带说明脚本后续步骤所需的输入与输出。

### Lines 3-5: Defines the `check_vram_clear` shell function / 定义 `check_vram_clear` shell 函数
```bash
check_vram_clear() {
    local vram_threshold_percent=5  # Allow up to 5% VRAM usage
    local memory_threshold_mb=500   # Allow up to 500MB memory usage
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 7-21: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if command -v rocm-smi >/dev/null 2>&1; then
        echo "Checking ROCm GPU VRAM usage..."
        # Check if any GPU has more than threshold VRAM allocated
        local high_usage=$(rocm-smi --showmemuse | grep -E "GPU Memory Allocated \(VRAM%\): ([6-9]|[1-9][0-9]|100)")
        if [ -n "$high_usage" ]; then
            echo "ERROR: VRAM usage exceeds threshold (${vram_threshold_percent}%) on some GPUs:"
            echo "$high_usage"
            rocm-smi --showmemuse
            return 1
        else
            echo "✓ VRAM usage is within acceptable limits on all GPUs"
            return 0
        fi
   fi
}
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 23-27: Enables strict shell execution flags / 启用严格的 shell 执行选项
```bash
# If this script is run directly (not sourced), run the check
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    set -e
    check_vram_clear
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

## Key Concepts / 关键概念
- **Shell automation flow** / Shell 自动化流程

## Dependencies / 依赖关系
- **Commands / 外部命令**: `echo`, `grep`
- **Environment variables / 环境变量**: `BASH_SOURCE`
