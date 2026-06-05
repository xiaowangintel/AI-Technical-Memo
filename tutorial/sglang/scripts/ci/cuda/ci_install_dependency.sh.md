# ci_install_dependency.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/ci/cuda/ci_install_dependency.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: Install dependencies for CUDA CI jobs. This shell script is part of SGLang's `cuda` automation flow. / 该Shell 脚本用于支撑 SGLang 中的 `ci_install_dependency` 流程，主要负责CI 编排、测试执行。它属于 `cuda` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Shell entrypoint and interpreter selection / Shell 入口与解释器选择
```bash
#!/bin/bash
# Install dependencies for CUDA CI jobs.
#
# CU_VERSION (default: cu130) controls PyTorch index URL, FlashInfer JIT cache
# index, and nvrtc variant selection.
set -euxo pipefail
```
**EN:** This opening block selects Bash as the interpreter and usually also documents expected inputs or outputs for the rest of the script.
**CN:** 该起始代码块指定 Bash 作为解释器，并通常顺带说明脚本后续步骤所需的输入与输出。

### Lines 8-9: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
REPO_ROOT="$(cd "${SCRIPT_DIR}/../../.." && pwd)"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 11-15: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
# ---------------------------------------------------------------------------
# Timing helper
# ---------------------------------------------------------------------------
SECONDS=0
_CI_MARK_PREV=${SECONDS}
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 17-24: Defines the `mark_step_done` shell function / 定义 `mark_step_done` shell 函数
```bash
mark_step_done() {
    local label=$1
    local now=${SECONDS}
    local step=$((now - _CI_MARK_PREV))
    printf '\n[STEP DONE] %s,  step: %ss,  total: %ss,  date: %s\n' \
        "${label}" "${step}" "${now}" "$(date -u '+%Y-%m-%dT%H:%M:%SZ')"
    _CI_MARK_PREV=${now}
}
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 26-28: Comment or metadata block / 注释或元数据块
```bash
# ---------------------------------------------------------------------------
# Functions
# ---------------------------------------------------------------------------
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 30-35: Defines the `configure_environment` shell function / 定义 `configure_environment` shell 函数
```bash
configure_environment() {
    # CU_VERSION controls PyTorch index URL, FlashInfer JIT cache index, and
    # nvrtc variant selection (cu12 vs cu13).
    CU_VERSION="${CU_VERSION:-cu130}"
    CU_STRIP="${CU_VERSION#cu}"
    CU_MAJOR="${CU_STRIP:0:2}"
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 37-37: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
    OPTIONAL_DEPS="${1:-}"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 39-41: Invokes Python-based tooling / 调用基于 Python 的工具
```bash
    # Whether to create a uv venv (set USE_VENV=1). Default: 0.
    USE_VENV="${USE_VENV:-0}"
    echo "USE_VENV=${USE_VENV}"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It coordinates Python tooling, prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会协调 Python 工具链、打印状态信息。

### Lines 43-46: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
    python3 -m pip install --upgrade pip
    if ! command -v uv >/dev/null 2>&1; then
        pip install uv
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It invokes Python helpers, installs Python packages, coordinates Python tooling.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会调用 Python 辅助脚本、安装 Python 包、协调 Python 工具链。

### Lines 48-48: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
    SYS_PYTHON_VER=$(python3 -c "import sys; print(f'{sys.version_info.major}.{sys.version_info.minor}')")
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 50-56: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
    if [ "$USE_VENV" = "1" ]; then
        UV_VENV="/tmp/sglang-ci-${GITHUB_RUN_ID:-norun}-${GITHUB_JOB:-nojob}-$$"
        uv venv "$UV_VENV" --python "python${SYS_PYTHON_VER}" --seed
        # shellcheck disable=SC1091
        source "$UV_VENV/bin/activate"
        [ "${VIRTUAL_ENV:-}" = "$UV_VENV" ] || { echo "FATAL: venv activation did not set VIRTUAL_ENV correctly"; exit 1; }
        [ "$(command -v python3)" = "$UV_VENV/bin/python3" ] || { echo "FATAL: python3 still resolves outside venv (got $(command -v python3))"; exit 1; }
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It invokes Python helpers, coordinates Python tooling, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会调用 Python 辅助脚本、协调 Python 工具链、打印状态信息。

### Lines 58-70: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
        if [ -n "${GITHUB_ENV:-}" ]; then
            echo "VIRTUAL_ENV=$UV_VENV" >> "$GITHUB_ENV"
            echo "SGLANG_CI_VENV_PATH=$UV_VENV" >> "$GITHUB_ENV"
            echo "BASH_ENV=$UV_VENV/env.sh" >> "$GITHUB_ENV"
            touch "$UV_VENV/env.sh"
        fi
        if [ -n "${GITHUB_PATH:-}" ]; then
            echo "$UV_VENV/bin" >> "$GITHUB_PATH"
        fi
    else
        echo "USE_VENV=0: skipping uv venv creation, installing into system Python"
        UV_VENV=""
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It coordinates Python tooling, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会协调 Python 工具链、打印状态信息。

### Lines 72-73: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 75-77: Defines the `detect_host` shell function / 定义 `detect_host` shell 函数
```bash
detect_host() {
    ARCH=$(uname -m)
    echo "Detected architecture: ${ARCH}"
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 79-94: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ "${IS_BLACKWELL+set}" = set ]; then
        case "$IS_BLACKWELL" in 1 | true | yes) IS_BLACKWELL=1 ;; *) IS_BLACKWELL=0 ;; esac
        echo "IS_BLACKWELL=${IS_BLACKWELL} (manually set via environment)"
    else
        IS_BLACKWELL=0
        if command -v nvidia-smi >/dev/null 2>&1; then
            while IFS= read -r cap; do
                major="${cap%%.*}"
                if [ "${major:-0}" -ge 10 ] 2>/dev/null; then
                    IS_BLACKWELL=1
                    break
                fi
            done <<< "$(nvidia-smi --query-gpu=compute_cap --format=csv,noheader 2>/dev/null || true)"
        fi
        echo "IS_BLACKWELL=${IS_BLACKWELL} (auto-detected via nvidia-smi)"
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 96-104: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ "${USE_UV+set}" != set ]; then
        if [ "$IS_BLACKWELL" = "1" ]; then
            USE_UV=false
        else
            USE_UV=true
        fi
    fi
    case "$(printf '%s' "$USE_UV" | tr '[:upper:]' '[:lower:]')" in 1 | true | yes) USE_UV=1 ;; *) USE_UV=0 ;; esac
    echo "USE_UV=${USE_UV}"
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 106-107: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 109-112: Defines the `kill_existing_processes` shell function / 定义 `kill_existing_processes` shell 函数
```bash
kill_existing_processes() {
    python3 "${REPO_ROOT}/python/sglang/cli/killall.py"
    KILLALL_EXIT=$?
    echo "CUDA_VISIBLE_DEVICES=${CUDA_VISIBLE_DEVICES:-}"
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 114-117: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ $KILLALL_EXIT -ne 0 ]; then
        echo "ERROR: killall.py detected uncleanable GPU memory. Aborting CI."
        exit 1
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 119-120: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 122-138: Defines the `install_apt_packages` shell function / 定义 `install_apt_packages` shell 函数
```bash
install_apt_packages() {
    apt-get update || true
    CI_APT_PACKAGES=(
        python3 python3-pip python3-venv python3-dev git libnuma-dev libssl-dev pkg-config
        libibverbs-dev libibverbs1 ibverbs-providers ibverbs-utils
        ffmpeg libavcodec-dev libavformat-dev libavutil-dev libswscale-dev
    )
    apt-get install -y --no-install-recommends "${CI_APT_PACKAGES[@]}" || {
        echo "Warning: apt-get install failed, checking if required packages are available..."
        for pkg in "${CI_APT_PACKAGES[@]}"; do
            if ! dpkg -l "$pkg" 2>/dev/null | grep -q "^ii"; then
                echo "ERROR: Required package $pkg is not installed and apt-get failed"
                exit 1
            fi
        done
        echo "All required packages are already installed, continuing..."
    }
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 140-141: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 143-145: Defines the `clean_site_packages` shell function / 定义 `clean_site_packages` shell 函数
```bash
clean_site_packages() {
    # Clear torch compilation cache
    python3 -c 'import os, shutil, tempfile, getpass; cache_dir = os.environ.get("TORCHINDUCTOR_CACHE_DIR") or os.path.join(tempfile.gettempdir(), "torchinductor_" + getpass.getuser()); shutil.rmtree(cache_dir, ignore_errors=True)'
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 147-158: Enables strict shell execution flags / 启用严格的 shell 执行选项
```bash
    # Remove broken dist-info directories (missing METADATA per PEP 376)
    SITE_PACKAGES=$(python3 -c "import site; print(site.getsitepackages()[0])")
    if [ -d "$SITE_PACKAGES" ]; then
        { set +x; } 2>/dev/null
        find "$SITE_PACKAGES" -maxdepth 1 -name "*.dist-info" -type d | while read -r d; do
            if [ ! -f "$d/METADATA" ]; then
                echo "Removing broken dist-info: $d"
                rm -rf "$d"
            fi
        done
        set -x
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It invokes Python helpers, cleans generated files, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会调用 Python 辅助脚本、清理生成文件、打印状态信息。

### Lines 160-162: Executes `bash`-based shell logic / 执行基于 `bash` 的 shell 逻辑
```bash
    # Install protoc + Rust toolchain (needed by setuptools-rust, e.g. the native gRPC extension)
    bash "${SCRIPT_DIR}/../utils/install_rust_protoc.sh"
    export PATH="${CARGO_HOME:-$HOME/.cargo}/bin:${PATH}"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 164-165: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 167-168: Defines the `setup_pip_toolchain` shell function / 定义 `setup_pip_toolchain` shell 函数
```bash
setup_pip_toolchain() {
    python3 -m pip install --upgrade pip
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 170-172: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ "$USE_VENV" != "1" ]; then
        export UV_SYSTEM_PYTHON=1
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

### Lines 174-178: Invokes Python-based tooling / 调用基于 Python 的工具
```bash
    export UV_LINK_MODE=copy
    PIP_CMD="uv pip"
    PIP_INSTALL_SUFFIX="--index-strategy unsafe-best-match"
    PIP_UNINSTALL_CMD="uv pip uninstall"
    PIP_UNINSTALL_SUFFIX=""
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It installs Python packages, coordinates Python tooling.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会安装 Python 包、协调 Python 工具链。

### Lines 180-180: Comment or metadata block / 注释或元数据块
```bash
    $PIP_UNINSTALL_CMD sgl-kernel sglang-kernel sglang sgl-fa4 flash-attn-4 $PIP_UNINSTALL_SUFFIX || true
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 182-183: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 185-193: Defines the `uninstall_stale_flashinfer` shell function / 定义 `uninstall_stale_flashinfer` shell 函数
```bash
uninstall_stale_flashinfer() {
    # Keep flashinfer packages if version matches to avoid re-downloading:
    # - flashinfer-cubin: 150+ MB
    # - flashinfer-jit-cache: 1.2+ GB
    FLASHINFER_PYTHON_REQUIRED=$(grep -Po -m1 '(?<=flashinfer_python==)[0-9A-Za-z\.\-]+' python/pyproject.toml || echo "")
    FLASHINFER_CUBIN_REQUIRED=$(grep -Po -m1 '(?<=flashinfer_cubin==)[0-9A-Za-z\.\-]+' python/pyproject.toml || echo "")
    FLASHINFER_CUBIN_INSTALLED=$(pip show flashinfer-cubin 2>/dev/null | grep "^Version:" | awk '{print $2}' || echo "")
    FLASHINFER_JIT_INSTALLED=$(pip show flashinfer-jit-cache 2>/dev/null | grep "^Version:" | awk '{print $2}' | sed 's/+.*//' || echo "")
    FLASHINFER_JIT_CU_VERSION=$(pip show flashinfer-jit-cache 2>/dev/null | grep "^Version:" | awk '{print $2}' | sed -n 's/.*+//p' || echo "")
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 195-196: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
    UNINSTALL_CUBIN=true
    UNINSTALL_JIT_CACHE=true
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 198-203: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ "$FLASHINFER_CUBIN_INSTALLED" = "$FLASHINFER_CUBIN_REQUIRED" ] && [ -n "$FLASHINFER_CUBIN_REQUIRED" ]; then
        echo "flashinfer-cubin==${FLASHINFER_CUBIN_REQUIRED} already installed, keeping it"
        UNINSTALL_CUBIN=false
    else
        echo "flashinfer-cubin version mismatch (installed: ${FLASHINFER_CUBIN_INSTALLED:-none}, required: ${FLASHINFER_CUBIN_REQUIRED}), reinstalling"
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 205-210: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ "$FLASHINFER_JIT_INSTALLED" = "$FLASHINFER_PYTHON_REQUIRED" ] && [ -n "$FLASHINFER_PYTHON_REQUIRED" ]; then
        echo "flashinfer-jit-cache==${FLASHINFER_PYTHON_REQUIRED} already installed, keeping it"
        UNINSTALL_JIT_CACHE=false
    else
        echo "flashinfer-jit-cache version mismatch (installed: ${FLASHINFER_JIT_INSTALLED:-none}, required: ${FLASHINFER_PYTHON_REQUIRED}), will reinstall"
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 212-215: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ "$UNINSTALL_JIT_CACHE" = false ] && [ "$FLASHINFER_JIT_CU_VERSION" != "$CU_VERSION" ]; then
        echo "flashinfer-jit-cache CUDA version mismatch (installed: ${FLASHINFER_JIT_CU_VERSION:-none}, required: ${CU_VERSION}), will reinstall"
        UNINSTALL_JIT_CACHE=true
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 217-221: Comment or metadata block / 注释或元数据块
```bash
    FLASHINFER_UNINSTALL="flashinfer-python"
    [ "$UNINSTALL_CUBIN" = true ] && FLASHINFER_UNINSTALL="$FLASHINFER_UNINSTALL flashinfer-cubin"
    [ "$UNINSTALL_JIT_CACHE" = true ] && FLASHINFER_UNINSTALL="$FLASHINFER_UNINSTALL flashinfer-jit-cache"
    $PIP_UNINSTALL_CMD $FLASHINFER_UNINSTALL $PIP_UNINSTALL_SUFFIX || true
    $PIP_UNINSTALL_CMD opencv-python opencv-python-headless $PIP_UNINSTALL_SUFFIX || true
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 223-224: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 226-232: Defines the `install_sglang` shell function / 定义 `install_sglang` shell 函数
```bash
install_sglang() {
    EXTRAS="dev,runai,tracing"
    if [ -n "$OPTIONAL_DEPS" ]; then
        EXTRAS="dev,runai,tracing,${OPTIONAL_DEPS}"
    fi
    echo "Installing python extras: [${EXTRAS}]"
    $PIP_CMD install -e "python[${EXTRAS}]" $PIP_INSTALL_SUFFIX
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 234-242: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
    # Defensive: some runners ended up with nvidia-cusparselt-cu13 metadata
    # present but libcusparseLt.so.0 missing on disk, breaking any torch import.
    # If the file is missing, force-reinstall the wheel before downstream steps.
    SITE_PACKAGES=$(python3 -c "import site; print(site.getsitepackages()[0])")
    if [ ! -f "$SITE_PACKAGES/nvidia/cusparselt/lib/libcusparseLt.so.0" ] \
       && pip show nvidia-cusparselt-cu13 >/dev/null 2>&1; then
        echo "WARNING: nvidia-cusparselt-cu13 metadata present but libcusparseLt.so.0 missing — reinstalling"
        $PIP_CMD install --reinstall nvidia-cusparselt-cu13 $PIP_INSTALL_SUFFIX
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It invokes Python helpers, installs Python packages, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会调用 Python 辅助脚本、安装 Python 包、打印状态信息。

### Lines 244-245: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 247-250: Defines the `install_sglang_kernel` shell function / 定义 `install_sglang_kernel` shell 函数
```bash
install_sglang_kernel() {
    SGL_KERNEL_VERSION_FROM_KERNEL=$(grep -Po '(?<=^version = ")[^"]*' sgl-kernel/pyproject.toml)
    SGL_KERNEL_VERSION_FROM_SRT=$(grep -Po -m1 '(?<=sglang-kernel==)[0-9A-Za-z\.\-]+' python/pyproject.toml)
    echo "SGL_KERNEL_VERSION_FROM_KERNEL=${SGL_KERNEL_VERSION_FROM_KERNEL} SGL_KERNEL_VERSION_FROM_SRT=${SGL_KERNEL_VERSION_FROM_SRT}"
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 252-274: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ "${CUSTOM_BUILD_SGL_KERNEL:-}" = "true" ] && [ -d "sgl-kernel/dist" ]; then
        ls -alh sgl-kernel/dist
        if [ "$ARCH" = "aarch64" ] || [ "$ARCH" = "arm64" ]; then
            WHEEL_ARCH="aarch64"
        else
            WHEEL_ARCH="x86_64"
        fi
        KERNEL_WHL=$(ls sgl-kernel/dist/sglang_kernel-${SGL_KERNEL_VERSION_FROM_KERNEL}+${CU_VERSION}-cp310-abi3-manylinux2014_${WHEEL_ARCH}.whl 2>/dev/null | head -1 || true)
        if [ -z "$KERNEL_WHL" ]; then
            echo "ERROR: No matching sgl-kernel wheel found in sgl-kernel/dist/ for version ${SGL_KERNEL_VERSION_FROM_KERNEL} arch ${WHEEL_ARCH} cuda ${CU_VERSION}"
            ls -alh sgl-kernel/dist/
            exit 1
        fi
        echo "Installing sgl-kernel wheel: $KERNEL_WHL"
        $PIP_CMD install "$KERNEL_WHL" --force-reinstall $PIP_INSTALL_SUFFIX
    else
        if [ "${CUSTOM_BUILD_SGL_KERNEL:-}" = "true" ] && [ ! -d "sgl-kernel/dist" ]; then
            echo "ERROR: CUSTOM_BUILD_SGL_KERNEL=true but sgl-kernel/dist not found."
            echo "This usually happens when rerunning a stage without the sgl-kernel-build-wheels job."
            echo "Please re-run the full workflow using /tag-and-rerun-ci to rebuild the kernel."
            exit 1
        fi
    fi
```
**EN:** This block summarizes the produced artifacts, making the final output set easy to inspect in CI logs or local runs.
**CN:** 该代码块会汇总最终产物，便于在 CI 日志或本地执行时快速核对输出结果。

### Lines 276-299: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
    # Reinstall torch with matching CUDA version if needed
    # TODO: Remove after torch 2.11 where cu13 is enabled by default
    TORCH_CUDA_VER=$(python3 -c "import torch; v=torch.version.cuda; parts=v.split('.'); print(f'cu{parts[0]}{parts[1]}')")
    echo "Detected torch CUDA version: ${TORCH_CUDA_VER}"
    TORCHAUDIO_CUDA_VER=$(pip show torchaudio 2>/dev/null | grep "^Version:" | awk '{print $2}' | sed -n 's/.*+\(cu[0-9][0-9]*\)$/\1/p' || true)
    TORCHVISION_CUDA_VER=$(pip show torchvision 2>/dev/null | grep "^Version:" | awk '{print $2}' | sed -n 's/.*+\(cu[0-9][0-9]*\)$/\1/p' || true)
    REINSTALL_TORCH=false
    if [ "${TORCH_CUDA_VER}" != "${CU_VERSION}" ]; then
        REINSTALL_TORCH=true
    else
        for cuda_ver in "${TORCHAUDIO_CUDA_VER}" "${TORCHVISION_CUDA_VER}"; do
            if [ -n "${cuda_ver}" ] && [ "${cuda_ver}" != "${CU_VERSION}" ]; then
                REINSTALL_TORCH=true
                break
            fi
        done
    fi
    if [ "${REINSTALL_TORCH}" = true ]; then
        TORCH_VER=$(pip show torch 2>/dev/null | grep "^Version:" | awk '{print $2}' | sed 's/+.*//')
        TORCHAUDIO_VER=$(pip show torchaudio 2>/dev/null | grep "^Version:" | awk '{print $2}' | sed 's/+.*//')
        TORCHVISION_VER=$(pip show torchvision 2>/dev/null | grep "^Version:" | awk '{print $2}' | sed 's/+.*//')
        echo "Reinstalling torch==${TORCH_VER} torchaudio==${TORCHAUDIO_VER} torchvision==${TORCHVISION_VER} from ${CU_VERSION} index to match torch..."
        $PIP_CMD install "torch==${TORCH_VER}" "torchaudio==${TORCHAUDIO_VER}" "torchvision==${TORCHVISION_VER}" --index-url "https://download.pytorch.org/whl/${CU_VERSION}" --force-reinstall --no-deps $PIP_INSTALL_SUFFIX
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It invokes Python helpers, installs Python packages, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会调用 Python 辅助脚本、安装 Python 包、打印状态信息。

### Lines 301-315: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ "${CUSTOM_BUILD_SGL_KERNEL:-}" != "true" ]; then
        # install_sglang above pulls sglang-kernel from PyPI, whose default wheel
        # tracks one CUDA version (currently cu130). Force-reinstall from the
        # CU_VERSION-matched sglang wheel index so runners on a different CUDA
        # (e.g. h20 / cu129) get a wheel linked against the right libnvrtc.
        $PIP_CMD install "sglang-kernel==${SGL_KERNEL_VERSION_FROM_SRT}" --index-url "https://docs.sglang.ai/whl/${CU_VERSION}/" --force-reinstall --no-deps $PIP_INSTALL_SUFFIX
    else
        echo "CUSTOM_BUILD_SGL_KERNEL=true: keeping freshly built sgl-kernel wheel."
    fi
    SGL_DEEP_GEMM_VERSION=$(grep -Po -m1 '(?<=sgl-deep-gemm==)[0-9A-Za-z\.\-]+' python/pyproject.toml)
    if [ "$CU_MAJOR" = "13" ]; then
        $PIP_CMD install "sgl-deep-gemm==${SGL_DEEP_GEMM_VERSION}" --force-reinstall $PIP_INSTALL_SUFFIX
    else
        $PIP_CMD install "https://github.com/sgl-project/whl/releases/download/v${SGL_DEEP_GEMM_VERSION}/sgl_deep_gemm-${SGL_DEEP_GEMM_VERSION}+cu129-py3-none-manylinux2014_$(uname -m).whl" --force-reinstall $PIP_INSTALL_SUFFIX
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 317-318: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 320-322: Defines the `install_sglang_router` shell function / 定义 `install_sglang_router` shell 函数
```bash
install_sglang_router() {
    $PIP_CMD install sglang-router $PIP_INSTALL_SUFFIX
    $PIP_CMD list
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 324-325: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 327-333: Defines the `download_flashinfer_cache` shell function / 定义 `download_flashinfer_cache` shell 函数
```bash
download_flashinfer_cache() {
    UNINSTALL_JIT_CACHE="$UNINSTALL_JIT_CACHE" \
        FLASHINFER_PYTHON_REQUIRED="$FLASHINFER_PYTHON_REQUIRED" \
        CU_VERSION="$CU_VERSION" \
        PIP_CMD="$PIP_CMD" \
        PIP_INSTALL_SUFFIX="$PIP_INSTALL_SUFFIX" \
        bash "${SCRIPT_DIR}/ci_download_flashinfer_jit_cache.sh"
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 335-336: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 338-344: Defines the `stabilize_flashinfer_jit_paths` shell function / 定义 `stabilize_flashinfer_jit_paths` shell 函数
```bash
stabilize_flashinfer_jit_paths() {
    # In venv mode, FlashInfer JIT writes build.ninja with hardcoded -isystem
    # paths. Per-job venvs get unique paths, but the JIT cache is shared on the
    # host mount. Fix by symlinking venv copies to a stable host-mounted path.
    if [ "$USE_VENV" != "1" ]; then
        return
    fi
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 346-346: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
    STABLE_FI_DIR="${HOME}/.cache/flashinfer/_stable_src"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 348-361: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    # Clear stale cached_ops (keep valid compiled kernels)
    if [ -d "${HOME}/.cache/flashinfer" ]; then
        STALE_COUNT=0
        while IFS= read -r ninja_file; do
            STALE_PATH=$(grep -o '/tmp/sglang-ci-[^ ]*\|flashinfer-src' "$ninja_file" 2>/dev/null | head -1 || true)
            if [ -n "$STALE_PATH" ]; then
                if echo "$STALE_PATH" | grep -q "flashinfer-src" || [ ! -d "$STALE_PATH" ]; then
                    rm -rf "$(dirname "$ninja_file")"
                    STALE_COUNT=$((STALE_COUNT + 1))
                fi
            fi
        done < <(find "${HOME}/.cache/flashinfer" -name "build.ninja" -type f 2>/dev/null)
        echo "Cleaned $STALE_COUNT stale FlashInfer cached_ops (kept valid ones)"
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It cleans generated files, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会清理生成文件、打印状态信息。

### Lines 363-365: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
    # Copy source files to stable path and symlink venv copies there
    FI_DATA=$(python3 -c "import flashinfer, os; print(os.path.join(os.path.dirname(flashinfer.__file__), 'data'))")
    TVM_INC=$(python3 -c "import tvm_ffi, os; print(os.path.join(os.path.dirname(tvm_ffi.__file__), 'include'))")
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 367-377: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    FI_VERSION="${FLASHINFER_PYTHON_REQUIRED}"
    if [ ! -d "$STABLE_FI_DIR/flashinfer-data" ] || [ "$(cat "$STABLE_FI_DIR/.version" 2>/dev/null)" != "$FI_VERSION" ]; then
        rm -rf "$STABLE_FI_DIR"
        mkdir -p "$STABLE_FI_DIR"
        cp -a "$FI_DATA" "$STABLE_FI_DIR/flashinfer-data"
        cp -a "$TVM_INC" "$STABLE_FI_DIR/tvm-ffi-include"
        echo "$FI_VERSION" > "$STABLE_FI_DIR/.version"
        echo "Copied flashinfer source files to stable path: $STABLE_FI_DIR (version=$FI_VERSION)"
    else
        echo "Stable flashinfer source path up to date (version=$FI_VERSION)"
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prepares directories, copies artifacts, cleans generated files, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会准备目录、复制产物、清理生成文件、打印状态信息。

### Lines 379-384: Prepares directories and generated artifacts / 准备目录与生成产物
```bash
    rm -rf "$FI_DATA"
    ln -s "$STABLE_FI_DIR/flashinfer-data" "$FI_DATA"
    TVM_INC_PARENT=$(dirname "$TVM_INC")
    rm -rf "$TVM_INC_PARENT/include"
    ln -s "$STABLE_FI_DIR/tvm-ffi-include" "$TVM_INC_PARENT/include"
    echo "Symlinked venv flashinfer/tvm_ffi -> $STABLE_FI_DIR"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It cleans generated files, prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会清理生成文件、打印状态信息。

### Lines 386-387: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 389-408: Defines the `install_extra_deps` shell function / 定义 `install_extra_deps` shell 函数
```bash
install_extra_deps() {
    if [ "$CU_MAJOR" = "13" ]; then
        MOONCAKE_PKG="mooncake-transfer-engine-cuda13==0.3.10.post2"
        MOONCAKE_STALE_PKG="mooncake-transfer-engine"
        EXTRA_NVIDIA_SPECS="nvidia-cuda-nvrtc"
    else
        MOONCAKE_PKG="mooncake-transfer-engine==0.3.10.post2"
        MOONCAKE_STALE_PKG="mooncake-transfer-engine-cuda13"
        EXTRA_NVIDIA_SPECS="nvidia-cuda-nvrtc-cu12"
    fi
    # Both variants own the same mooncake/ package files and bin/ scripts
    # (mooncake_master, etc.). Uninstalling the stale variant deletes shared
    # files that the live variant's RECORD still references, so we force a
    # reinstall to restore them — pip would otherwise see "already satisfied"
    # and skip.
    if pip show ${MOONCAKE_STALE_PKG} >/dev/null 2>&1; then
        $PIP_UNINSTALL_CMD ${MOONCAKE_STALE_PKG} $PIP_UNINSTALL_SUFFIX || true
        $PIP_CMD install ${MOONCAKE_PKG} --force-reinstall --no-deps $PIP_INSTALL_SUFFIX
    fi
    $PIP_CMD install ${MOONCAKE_PKG} ${EXTRA_NVIDIA_SPECS} py-spy scipy huggingface_hub[hf_xet] pytest $PIP_INSTALL_SUFFIX
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 410-411: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
    # Best-effort NIXL install for decode-radix disaggregation coverage.
    $PIP_CMD install nixl $PIP_INSTALL_SUFFIX || echo "Warning: nixl install failed; continuing without nixl"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 413-417: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ "$IS_BLACKWELL" != "1" ]; then
        git clone --branch v0.5 --depth 1 https://github.com/EvolvingLMMs-Lab/lmms-eval.git
        $PIP_CMD install -e lmms-eval/ $PIP_INSTALL_SUFFIX
    fi
    $PIP_CMD uninstall xformers || true
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It queries repository state.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会查询仓库状态。

### Lines 419-420: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 422-428: Defines the `install_test_tools` shell function / 定义 `install_test_tools` shell 函数
```bash
install_test_tools() {
    # Download kernels from kernels community
    kernels download python || true
    kernels lock python || true
    [ -e "${HOME}/.cache/sglang" ] && [ ! -d "${HOME}/.cache/sglang" ] && rm -f "${HOME}/.cache/sglang"
    mkdir -p "${HOME}/.cache/sglang/"
    mv python/kernels.lock "${HOME}/.cache/sglang/" || true
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 430-436: Executes `git`-based shell logic / 执行基于 `git` 的 shell 逻辑
```bash
    # Install human-eval (subshell keeps cd local)
    $PIP_CMD install "setuptools==70.0.0" $PIP_INSTALL_SUFFIX
    [ -d human-eval ] || git clone https://github.com/merrymercy/human-eval.git
    (
        cd human-eval
        $PIP_CMD install -e . --no-build-isolation $PIP_INSTALL_SUFFIX
    )
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It queries repository state.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会查询仓库状态。

### Lines 438-439: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 441-442: Defines the `prepare_runner` shell function / 定义 `prepare_runner` shell 函数
```bash
prepare_runner() {
    bash "${SCRIPT_DIR}/prepare_runner.sh"
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 444-445: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 447-454: Defines the `setup_ld_library_path` shell function / 定义 `setup_ld_library_path` shell 函数
```bash
setup_ld_library_path() {
    # NVIDIA pip packages and torch ship .so files under site-packages that are
    # not on the default LD_LIBRARY_PATH.
    SITE_PACKAGES=$(python3 -c "import site, sys; print(site.getsitepackages()[0])")
    NVIDIA_LIBS=$(find "$SITE_PACKAGES" -path "*/nvidia/*/lib" -type d 2>/dev/null | tr '\n' ':')
    TORCH_LIB="$SITE_PACKAGES/torch/lib"
    VENV_LD="${NVIDIA_LIBS}${TORCH_LIB}"
    export LD_LIBRARY_PATH="${VENV_LD}${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 456-462: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ "$USE_VENV" = "1" ] && [ -n "$UV_VENV" ]; then
        echo "export LD_LIBRARY_PATH=\"$LD_LIBRARY_PATH\"" >> "$UV_VENV/env.sh"
    fi
    if [ -n "${GITHUB_ENV:-}" ]; then
        echo "LD_LIBRARY_PATH=$LD_LIBRARY_PATH" >> "$GITHUB_ENV" || echo "WARNING: GITHUB_ENV write failed; LD_LIBRARY_PATH will be set via BASH_ENV instead"
    fi
    echo "LD_LIBRARY_PATH=$LD_LIBRARY_PATH"
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 464-465: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 467-470: Defines the `verify_imports` shell function / 定义 `verify_imports` shell 函数
```bash
verify_imports() {
    $PIP_CMD list
    python3 -c "import torch; print(torch.version.cuda)"
    python3 -c "import cutlass; import cutlass.cute;"
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 472-473: Comment or metadata block / 注释或元数据块
```bash
    mark_step_done "${FUNCNAME[0]}"
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 475-477: Comment or metadata block / 注释或元数据块
```bash
# ---------------------------------------------------------------------------
# Main
# ---------------------------------------------------------------------------
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 479-497: Defines the `main` shell function / 定义 `main` shell 函数
```bash
main() {
    configure_environment "$@"
    detect_host
    kill_existing_processes
    install_apt_packages
    clean_site_packages
    setup_pip_toolchain
    uninstall_stale_flashinfer
    install_sglang
    install_sglang_kernel
    install_sglang_router
    download_flashinfer_cache
    stabilize_flashinfer_jit_paths
    install_extra_deps
    install_test_tools
    prepare_runner
    setup_ld_library_path
    verify_imports
}
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 499-499: Comment or metadata block / 注释或元数据块
```bash
main "$@"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Git state inspection** / Git 状态检查
- **Automated testing** / 自动化测试
- **Runner provisioning** / 运行器准备

## Dependencies / 依赖关系
- **Commands / 外部命令**: `git`, `python`, `python3`, `pytest`, `pip`, `uv`, `mkdir`, `cp`, `mv`, `rm`, `echo`, `grep`, `sed`, `find`, `head`
- **Environment variables / 环境变量**: `ARCH`, `BASH_SOURCE`, `CARGO_HOME`, `CI_APT_PACKAGES`, `CUDA_VISIBLE_DEVICES`, `CUSTOM_BUILD_SGL_KERNEL`, `CU_MAJOR`, `CU_STRIP`, `CU_VERSION`, `EXTRAS`, `EXTRA_NVIDIA_SPECS`, `FI_DATA`, `FI_VERSION`, `FLASHINFER_CUBIN_INSTALLED`, `FLASHINFER_CUBIN_REQUIRED`
- **Referenced files / 引用文件**: `/../utils/install_rust_protoc.sh`, `//download.py`, `/ci_download_flashinfer_jit_cache.sh`, `/prepare_runner.sh`, `/python/sglang/cli/killall.py`, `UV_VENV/env.sh`, `killall.py`
