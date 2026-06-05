# npu_log_print.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/ci/npu/npu_log_print.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: This shell script supports the `npu_log_print` workflow in SGLang. It mainly handles CI orchestration. / 该Shell 脚本用于支撑 SGLang 中的 `npu_log_print` 流程，主要负责CI 编排。它属于 `npu` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: Shell entrypoint and interpreter selection / Shell 入口与解释器选择
```bash
#!/bin/bash
set -euo pipefail
```
**EN:** This opening block selects Bash as the interpreter and usually also documents expected inputs or outputs for the rest of the script.
**CN:** 该起始代码块指定 Bash 作为解释器，并通常顺带说明脚本后续步骤所需的输入与输出。

### Lines 4-26: Defines the `get_version` shell function / 定义 `get_version` shell 函数
```bash
# Print log information(sglang version, commit sha, sgl-kernel-npu version, sgl-kernel-npu commit sha, npu-smi info and pip list.
npu-smi info
pip list
get_version() {
    [ -f "$1" ] && python3 -c 'import re, sys; print(sys.argv[2] + " version: v" + re.search(r"__version__\s*=\s*[\"'"'"'](.*?)[\"'"'"']", open(sys.argv[1]).read()).group(1))' "$1" "$2" 2>/dev/null || echo "$2 version: unknown"
}
get_version "./python/sglang/version.py" "sglang"
get_version "./sgl-kernel/python/sgl_kernel/version.py" "sgl_kernel"
SGLANG_URL="https://github.com/sgl-project/sglang.git"
SGL_KERNEL_URL="https://github.com/sgl-project/sgl-kernel-npu.git"
SGLANG_BRANCH="main"
SGL_KERNEL_BRANCH="main"
get_sha() {
    local name="$1"
    local url="$2"
    local branch="$3"
    local sha
    sha=$(git ls-remote "$url" "refs/heads/$branch" | cut -f1)
    echo "$name SHA for branch $branch: ${sha:-"Not Found"}"
}
get_sha "sglang" "$SGLANG_URL" "$SGLANG_BRANCH"
get_sha "sgl-kernel" "$SGL_KERNEL_URL" "$SGL_KERNEL_BRANCH"
chmod +x scripts/ci/npu/npu_log_print.sh
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

## Key Concepts / 关键概念
- **Process control** / 进程控制
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Commands / 外部命令**: `git`, `python3`, `pip`, `echo`
- **Environment variables / 环境变量**: `SGLANG_BRANCH`, `SGLANG_URL`, `SGL_KERNEL_BRANCH`, `SGL_KERNEL_URL`
- **Referenced files / 引用文件**: `./python/sglang/version.py`, `./sgl-kernel/python/sgl_kernel/version.py`, `scripts/ci/npu/npu_log_print.sh`
