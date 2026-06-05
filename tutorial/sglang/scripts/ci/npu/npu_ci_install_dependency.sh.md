# npu_ci_install_dependency.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/ci/npu/npu_ci_install_dependency.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: This shell script supports the `npu_ci_install_dependency` workflow in SGLang. It mainly handles CI orchestration, build automation. / 该Shell 脚本用于支撑 SGLang 中的 `npu_ci_install_dependency` 流程，主要负责CI 编排、构建自动化。它属于 `npu` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: Shell entrypoint and interpreter selection / Shell 入口与解释器选择
```bash
#!/bin/bash
set -euo pipefail
```
**EN:** This opening block selects Bash as the interpreter and usually also documents expected inputs or outputs for the rest of the script.
**CN:** 该起始代码块指定 Bash 作为解释器，并通常顺带说明脚本后续步骤所需的输入与输出。

### Lines 4-7: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
PIP_INSTALL="python3 -m pip install --no-cache-dir"
UV_PIP_INSTALL="uv pip install "
DEVICE_TYPE=$1
OPTIONAL_DEPS="${2:-}"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 10-33: Invokes Python-based tooling / 调用基于 Python 的工具
```bash
# Install the required dependencies in CI.
apt update -y && apt install -y \
    unzip \
    build-essential \
    cmake \
    wget \
    curl \
    net-tools \
    zlib1g-dev \
    lld \
    clang \
    locales \
    ccache \
    libgl1-mesa-glx \
    libgl1-mesa-dri \
    ca-certificates \
    libgl1 \
    libglib2.0-0
update-ca-certificates
${PIP_INSTALL} --upgrade pip
${PIP_INSTALL} uv
export UV_NO_CACHE=true
export UV_SYSTEM_PYTHON=true
export UV_INDEX_STRATEGY=unsafe-best-match
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It installs Python packages, coordinates Python tooling, configures native builds.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会安装 Python 包、协调 Python 工具链、配置原生构建。

### Lines 35-39: Executes `bash`-based shell logic / 执行基于 `bash` 的 shell 逻辑
```bash
# Install Rust toolchain (needed by crates built via setuptools-rust, e.g. the
# native gRPC extension bundled into the sglang wheel).
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
bash "${SCRIPT_DIR}/../utils/install_rustup.sh"
export PATH="${CARGO_HOME:-$HOME/.cargo}/bin:${PATH}"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 41-42: Comment or metadata block / 注释或元数据块
```bash
# Pin wheel to 0.45.1, REF: https://github.com/pypa/wheel/issues/662
${UV_PIP_INSTALL} wheel==0.45.1 pybind11 pyyaml decorator scipy attrs psutil
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 45-46: Comment or metadata block / 注释或元数据块
```bash
### Install MemFabric
${UV_PIP_INSTALL} memfabric-hybrid==1.0.5
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 49-63: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
### Install PyTorch and PTA
if [ -n "$OPTIONAL_DEPS" ]; then
    PYTORCH_VERSION="2.10.0"
    TORCHVISION_VERSION="0.25.0"
    ${UV_PIP_INSTALL} torch==${PYTORCH_VERSION} torchvision==${TORCHVISION_VERSION} --index-url ${TORCH_CACHE_URL:="https://download.pytorch.org/whl/cpu"} --extra-index-url ${PYPI_CACHE_URL:="https://pypi.org/simple/"}
    PTA_URL="https://gitcode.com/Ascend/pytorch/releases/download/7.3.0.alpha002/torch_npu-2.10.0rc2-cp311-cp311-manylinux_2_28_aarch64.whl"
    # GitCode does not allow UV downloads.
    ${PIP_INSTALL} ${PTA_URL}
else
    PYTORCH_VERSION="2.8.0"
    TORCHVISION_VERSION="0.23.0"
    ${UV_PIP_INSTALL} torch==${PYTORCH_VERSION} torchvision==${TORCHVISION_VERSION} --index-url ${TORCH_CACHE_URL:="https://download.pytorch.org/whl/cpu"} --extra-index-url ${PYPI_CACHE_URL:="https://pypi.org/simple/"}
    PTA_URL="https://gitcode.com/Ascend/pytorch/releases/download/v7.3.0-pytorch2.8.0/torch_npu-2.8.0.post2-cp311-cp311-manylinux_2_28_aarch64.whl"
    ${PIP_INSTALL} ${PTA_URL}
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

### Lines 66-67: Comment or metadata block / 注释或元数据块
```bash
### Install Triton-Ascend
${UV_PIP_INSTALL} triton-ascend
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 70-76: Prepares directories and generated artifacts / 准备目录与生成产物
```bash
### Install sgl-kernel-npu
SGLANG_KERNEL_NPU_TAG="2026.05.01"
mkdir sgl-kernel-npu
(cd sgl-kernel-npu && wget "${GITHUB_PROXY_URL:=""}https://github.com/sgl-project/sgl-kernel-npu/releases/download/${SGLANG_KERNEL_NPU_TAG}/sgl-kernel-npu-${SGLANG_KERNEL_NPU_TAG}-torch${PYTORCH_VERSION}-py311-cann8.5.0-${DEVICE_TYPE}-$(arch).zip" \
&& unzip ./sgl-kernel-npu-${SGLANG_KERNEL_NPU_TAG}-torch${PYTORCH_VERSION}-py311-cann8.5.0-${DEVICE_TYPE}-$(arch).zip \
&& ${UV_PIP_INSTALL} ./deep_ep*.whl ./sgl_kernel_npu*.whl \
&& (cd "$(python3 -m pip show deep-ep | grep -E '^Location:' | awk '{print $2}')" && ln -s deep_ep/deep_ep_cpp*.so))
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It invokes Python helpers, installs Python packages, prepares directories.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会调用 Python 辅助脚本、安装 Python 包、准备目录。

### Lines 79-81: Prepares directories and generated artifacts / 准备目录与生成产物
```bash
### Install SGLang
rm -rf python/pyproject.toml && mv python/pyproject_npu.toml python/pyproject.toml
${UV_PIP_INSTALL} -v -e "python[dev_npu]"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It invokes Python helpers, moves artifacts, cleans generated files.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会调用 Python 辅助脚本、移动产物、清理生成文件。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **YAML configuration** / YAML 配置
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Commands / 外部命令**: `python`, `python3`, `pip`, `uv`, `cmake`, `mkdir`, `mv`, `rm`, `grep`, `bash`, `curl`
- **Environment variables / 环境变量**: `BASH_SOURCE`, `CARGO_HOME`, `DEVICE_TYPE`, `GITHUB_PROXY_URL`, `HOME`, `OPTIONAL_DEPS`, `PATH`, `PIP_INSTALL`, `PTA_URL`, `PYPI_CACHE_URL`, `PYTORCH_VERSION`, `SCRIPT_DIR`, `SGLANG_KERNEL_NPU_TAG`, `TORCHVISION_VERSION`, `TORCH_CACHE_URL`
- **Referenced files / 引用文件**: `/../utils/install_rustup.sh`, `//download.py`
