# musa_install_dependency.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/ci/musa/musa_install_dependency.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: This shell script supports the `musa_install_dependency` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Shell 脚本用于支撑 SGLang 中的 `musa_install_dependency` 流程，主要负责CI 编排、测试执行。它属于 `musa` 自动化路径的一部分。

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
# Parse command line arguments
OPTIONAL_DEPS=""
SKIP_SGLANG_BUILD=""
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 8-22: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
while [[ $# -gt 0 ]]; do
  case $1 in
    --skip-sglang-build) SKIP_SGLANG_BUILD="1"; shift;;
    -h|--help)
      echo "Usage: $0 [OPTIONS] [OPTIONAL_DEPS]"
      echo "Options:"
      echo "  --skip-sglang-build         Don't build checkout sglang, use what was shipped with the image"
      exit 0
      ;;
    *)
      OPTIONAL_DEPS="$1"
      shift
      ;;
  esac
done
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 24-25: Invokes Python-based tooling / 调用基于 Python 的工具
```bash
PIP_INSTALL="python3 -m pip install --no-cache-dir"
${PIP_INSTALL} --upgrade pip setuptools torchada --user
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It invokes Python helpers, installs Python packages.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会调用 Python 辅助脚本、安装 Python 包。

### Lines 27-43: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
WHL_DIR="/sglang-checkout/whl"
if [ -d "$WHL_DIR" ] && compgen -G "${WHL_DIR}"/*.whl > /dev/null; then
    echo "Uninstall old packages based on wheel METADATA..."
    PKGS=$(
      for whl in "${WHL_DIR}"/*.whl; do
        meta_file=$(zipinfo -1 "$whl" | awk '/\.dist-info\/METADATA$/ {print; exit}')
        [ -n "$meta_file" ] || continue
        unzip -p "$whl" "$meta_file" 2>/dev/null | sed -n 's/^Name: //p' | head -n1
      done | sort -u
    )
    for pkg in $PKGS; do
      echo "Uninstalling $pkg"
      pip uninstall -y "$pkg" || true
    done
    echo "Installing wheel files without dependency resolution..."
    ${PIP_INSTALL} "${WHL_DIR}"/*.whl --user
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It installs Python packages, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会安装 Python 包、打印状态信息。

### Lines 45-54: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if [ -n "$SKIP_SGLANG_BUILD" ]; then
    echo "Didn't build checkout SGLang"
    exit 0
else
    pip uninstall sgl-kernel -y || true
    pip uninstall sglang -y || true
    # Clear Python cache to ensure latest code is used (works for any env: venv, system, conda)
    REPO_ROOT="${GITHUB_WORKSPACE:-$(pwd)}"
    find "$REPO_ROOT" -name "*.pyc" -delete 2>/dev/null || true
    find "$REPO_ROOT" -name "__pycache__" -type d -exec rm -rf {} + 2>/dev/null || true
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It installs Python packages, cleans generated files, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会安装 Python 包、清理生成文件、打印状态信息。

### Lines 56-57: Prepares directories and generated artifacts / 准备目录与生成产物
```bash
    rm -f "${REPO_ROOT}/python/pyproject.toml" && mv "${REPO_ROOT}/python/pyproject_other.toml" "${REPO_ROOT}/python/pyproject.toml"
    cd "${REPO_ROOT}" && ${PIP_INSTALL} -v -e "python[dev_musa]" --user
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It invokes Python helpers, moves artifacts, cleans generated files.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会调用 Python 辅助脚本、移动产物、清理生成文件。

### Lines 59-62: Prepares directories and generated artifacts / 准备目录与生成产物
```bash
    cd "${REPO_ROOT}/sgl-kernel"
    rm -f pyproject.toml && mv pyproject_musa.toml pyproject.toml && MTGPU_TARGET=mp_31 python3 setup_musa.py install --user
    echo "$HOME/.local/bin" >> "$GITHUB_PATH"
fi
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It invokes Python helpers, moves artifacts, cleans generated files, prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会调用 Python 辅助脚本、移动产物、清理生成文件、打印状态信息。

## Key Concepts / 关键概念
- **Process control** / 进程控制
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Commands / 外部命令**: `python`, `python3`, `pip`, `mv`, `rm`, `echo`, `sed`, `find`, `head`
- **Environment variables / 环境变量**: `GITHUB_PATH`, `GITHUB_WORKSPACE`, `HOME`, `PIP_INSTALL`, `PKGS`, `REPO_ROOT`, `SKIP_SGLANG_BUILD`, `WHL_DIR`
- **Referenced files / 引用文件**: `setup_musa.py`
