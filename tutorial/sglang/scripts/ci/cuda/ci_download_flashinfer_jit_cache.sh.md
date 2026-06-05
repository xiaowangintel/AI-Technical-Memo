# ci_download_flashinfer_jit_cache.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/ci/cuda/ci_download_flashinfer_jit_cache.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: Install flashinfer-jit-cache with caching and retry logic (flashinfer.ai can have transient DNS issues). This shell script is part of SGLang's `cuda` automation flow. / 该Shell 脚本用于支撑 SGLang 中的 `ci_download_flashinfer_jit_cache` 流程，主要负责CI 编排、运行器选择与准备。它属于 `cuda` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Shell entrypoint and interpreter selection / Shell 入口与解释器选择
```bash
#!/bin/bash
# Install flashinfer-jit-cache with caching and retry logic (flashinfer.ai can have transient DNS issues).
# The jit-cache wheel is 1.2+ GB, so we skip the download entirely if already installed.
#
# Required environment (caller must export or set):
#   UNINSTALL_JIT_CACHE          — literal true/false (skip download when false)
#   FLASHINFER_PYTHON_REQUIRED   — e.g. from python/pyproject.toml (flashinfer_python)
#   CU_VERSION                   — e.g. cu130
#   PIP_CMD                      — e.g. "pip" or "uv pip"
#   PIP_INSTALL_SUFFIX           — extra pip args for this runner
set -euxo pipefail
```
**EN:** This opening block selects Bash as the interpreter and usually also documents expected inputs or outputs for the rest of the script.
**CN:** 该起始代码块指定 Bash 作为解释器，并通常顺带说明脚本后续步骤所需的输入与输出。

### Lines 13-16: Comment or metadata block / 注释或元数据块
```bash
: "${UNINSTALL_JIT_CACHE:?must be set}"
: "${FLASHINFER_PYTHON_REQUIRED:?must be set}"
: "${CU_VERSION:?must be set}"
: "${PIP_CMD:?must be set}"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 18-22: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
FLASHINFER_JIT_CACHE_INSTALLED=false
if [ "$UNINSTALL_JIT_CACHE" = false ]; then
    FLASHINFER_JIT_CACHE_INSTALLED=true
    echo "flashinfer-jit-cache already at correct version, skipping download"
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 24-26: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if [ "$FLASHINFER_JIT_CACHE_INSTALLED" = false ]; then
    FLASHINFER_CACHE_DIR="${HOME}/.cache/flashinfer-wheels"
    mkdir -p "${FLASHINFER_CACHE_DIR}"
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prepares directories.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会准备目录。

### Lines 28-29: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
    FLASHINFER_WHEEL_PATTERN="flashinfer_jit_cache-${FLASHINFER_PYTHON_REQUIRED}+${CU_VERSION}*.whl"
    CACHED_WHEEL=$(find "${FLASHINFER_CACHE_DIR}" -name "${FLASHINFER_WHEEL_PATTERN}" -type f 2>/dev/null | head -n 1)
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 31-40: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ -n "$CACHED_WHEEL" ] && [ -f "$CACHED_WHEEL" ]; then
        echo "Found cached flashinfer wheel: $CACHED_WHEEL"
        if $PIP_CMD install "$CACHED_WHEEL" $PIP_INSTALL_SUFFIX; then
            FLASHINFER_JIT_CACHE_INSTALLED=true
            echo "Successfully installed flashinfer-jit-cache from cache"
        else
            echo "Failed to install from cache, will try downloading..."
            rm -f "$CACHED_WHEEL"
        fi
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It cleans generated files, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会清理生成文件、打印状态信息。

### Lines 42-47: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ "$FLASHINFER_JIT_CACHE_INSTALLED" = false ]; then
        for i in {1..5}; do
            # Download wheel to cache directory (use pip directly as uv pip doesn't support download)
            if timeout 600 pip download "flashinfer-jit-cache==${FLASHINFER_PYTHON_REQUIRED}" \
                --index-url "https://flashinfer.ai/whl/${CU_VERSION}" \
                -d "${FLASHINFER_CACHE_DIR}"; then
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It installs Python packages, coordinates Python tooling.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会安装 Python 包、协调 Python 工具链。

### Lines 49-64: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
                CACHED_WHEEL=$(find "${FLASHINFER_CACHE_DIR}" -name "${FLASHINFER_WHEEL_PATTERN}" -type f 2>/dev/null | head -n 1)
                if [ -n "$CACHED_WHEEL" ] && [ -f "$CACHED_WHEEL" ]; then
                    if $PIP_CMD install "$CACHED_WHEEL" $PIP_INSTALL_SUFFIX; then
                        FLASHINFER_JIT_CACHE_INSTALLED=true
                        echo "Successfully downloaded and installed flashinfer-jit-cache"
                        break
                    fi
                else
                    echo "Warning: Download succeeded but wheel file not found"
                fi
            fi
            echo "Attempt $i to download flashinfer-jit-cache failed, retrying in 10 seconds..."
            sleep 10
        done
    fi
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 66-69: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if [ "$FLASHINFER_JIT_CACHE_INSTALLED" = false ]; then
    echo "ERROR: Failed to install flashinfer-jit-cache after 5 attempts"
    exit 1
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

## Key Concepts / 关键概念
- **Runner provisioning** / 运行器准备

## Dependencies / 依赖关系
- **Commands / 外部命令**: `pip`, `uv`, `mkdir`, `rm`, `echo`, `find`, `head`
- **Environment variables / 环境变量**: `CACHED_WHEEL`, `CU_VERSION`, `FLASHINFER_CACHE_DIR`, `FLASHINFER_JIT_CACHE_INSTALLED`, `FLASHINFER_PYTHON_REQUIRED`, `FLASHINFER_WHEEL_PATTERN`, `HOME`, `PIP_CMD`, `PIP_INSTALL_SUFFIX`, `UNINSTALL_JIT_CACHE`
