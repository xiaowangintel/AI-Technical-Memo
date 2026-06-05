# build_sgl_deep_gemm.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/build_sgl_deep_gemm.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: Build sgl-deep-gemm wheel inside a CUDA-versioned container. This shell script is part of SGLang's `scripts` automation flow. / 该Shell 脚本用于支撑 SGLang 中的 `build_sgl_deep_gemm` 流程，主要负责容器构建步骤、构建自动化。它属于 `scripts` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Shell entrypoint and interpreter selection / Shell 入口与解释器选择
```bash
#!/bin/bash
# Build sgl-deep-gemm wheel inside a CUDA-versioned container.
#
# Usage: build_sgl_deep_gemm.sh <PYTHON_VERSION> <CUDA_VERSION> <DEEPGEMM_SRC> [ARCH]
#   PYTHON_VERSION: e.g. 3.10
#   CUDA_VERSION:   e.g. 12.9 or 13.0
#   DEEPGEMM_SRC:   path to a checkout of sgl-project/DeepGEMM
#   ARCH:           x86_64 (default) or aarch64
#
# Writes:
#   <DEEPGEMM_SRC>/dist/      — wheel(s) tagged +cu129 / +cu130 and manylinux
#   <DEEPGEMM_SRC>/dist-pypi/ — cu130 only: same wheel(s) with +cu130 stripped
#                              (PyPI rejects local-version segments)
set -ex
```
**EN:** This opening block selects Bash as the interpreter and usually also documents expected inputs or outputs for the rest of the script.
**CN:** 该起始代码块指定 Bash 作为解释器，并通常顺带说明脚本后续步骤所需的输入与输出。

### Lines 16-19: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if [ $# -lt 3 ]; then
  echo "Usage: $0 <PYTHON_VERSION> <CUDA_VERSION> <DEEPGEMM_SRC> [ARCH]"
  exit 1
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 21-24: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
PYTHON_VERSION="$1"
CUDA_VERSION="$2"
DEEPGEMM_SRC="$(cd "$3" && pwd)"
ARCH="${4:-$(uname -i)}"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 26-33: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
case "${CUDA_VERSION}" in
  13.0) CU_TAG=cu130 ;;
  12.9) CU_TAG=cu129 ;;
  *)
    echo "Unsupported CUDA_VERSION: ${CUDA_VERSION}" >&2
    exit 1
    ;;
esac
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 35-39: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if [ "${ARCH}" = "aarch64" ]; then
  BASE_IMG="pytorch/manylinuxaarch64-builder"
else
  BASE_IMG="pytorch/manylinux2_28-builder"
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

### Lines 41-41: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
PY_TAG="cp${PYTHON_VERSION//.}-cp${PYTHON_VERSION//.}"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 43-46: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
REPO_ROOT="$(cd "${SCRIPT_DIR}/.." && pwd)"
DOCKERFILE="${REPO_ROOT}/docker/sgl-deep-gemm.Dockerfile"
RENAME_SCRIPT="${SCRIPT_DIR}/rename_sgl_deep_gemm_whl.sh"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 48-48: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
DEPS_TAG="sgl-deep-gemm-deps:cuda${CUDA_VERSION}-${PY_TAG}-${ARCH}"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 50-58: Prints diagnostic status information / 打印诊断状态信息
```bash
echo "----------------------------------------"
echo "PYTHON_VERSION: ${PYTHON_VERSION}"
echo "CUDA_VERSION:   ${CUDA_VERSION}"
echo "CU_TAG:         ${CU_TAG}"
echo "ARCH:           ${ARCH}"
echo "BASE_IMG:       ${BASE_IMG}"
echo "DEEPGEMM_SRC:   ${DEEPGEMM_SRC}"
echo "DEPS_TAG:       ${DEPS_TAG}"
echo "----------------------------------------"
```
**EN:** This block prints resolved values so operators can confirm the runtime context before or after heavier actions run.
**CN:** 该代码块会打印解析后的关键值，便于操作者在执行重型步骤前后确认运行上下文。

### Lines 60-68: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
docker build \
  -f "${DOCKERFILE}" "$(dirname "${DOCKERFILE}")" \
  --build-arg BASE_IMG="${BASE_IMG}" \
  --build-arg CUDA_VERSION="${CUDA_VERSION}" \
  --build-arg ARCH="${ARCH}" \
  --build-arg PYTHON_VERSION="${PYTHON_VERSION}" \
  --build-arg PYTHON_TAG="${PY_TAG}" \
  -t "${DEPS_TAG}" \
  --network=host
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤。

### Lines 70-70: Prepares directories and generated artifacts / 准备目录与生成产物
```bash
mkdir -p "${DEEPGEMM_SRC}/dist"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prepares directories.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会准备目录。

### Lines 72-78: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
# 1) Build the wheel inside the deps container.
docker run --rm \
  --network=host \
  -v "${DEEPGEMM_SRC}:/deepgemm" \
  -w /deepgemm \
  "${DEPS_TAG}" \
  bash build_sgl_deep_gemm.sh
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤。

### Lines 80-87: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
# 2) Rename inside the same image so we have a working pip / wheel CLI and can
#    rewrite the root-owned wheel files written by the build container above.
docker run --rm \
  -v "${DEEPGEMM_SRC}:/deepgemm" \
  -v "${RENAME_SCRIPT}:/rename_sgl_deep_gemm_whl.sh:ro" \
  -w /deepgemm \
  "${DEPS_TAG}" \
  bash /rename_sgl_deep_gemm_whl.sh dist "${CU_TAG}" "${ARCH}"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, installs Python packages.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、安装 Python 包。

### Lines 89-118: Enables strict shell execution flags / 启用严格的 shell 执行选项
```bash
# 3) cu130 only: produce a sibling dist-pypi/ with the +cu130 local-version
#    stripped (PyPI rejects local versions).
if [ "${CU_TAG}" = "cu130" ]; then
  docker run --rm \
    -v "${DEEPGEMM_SRC}:/deepgemm" \
    -w /deepgemm \
    "${DEPS_TAG}" \
    bash -c '
set -eux
mkdir -p dist-pypi
for w in dist/*.whl; do
  tmp=$(mktemp -d)
  python3 -m wheel unpack "$w" --dest "$tmp"
  unpacked=$(find "$tmp" -mindepth 1 -maxdepth 1 -type d | head -1)
  info=$(find "$unpacked" -maxdepth 1 -type d -name "*.dist-info" | head -1)
  meta="$info/METADATA"
  orig=$(grep "^Version:" "$meta" | head -1 | sed "s/^Version:[[:space:]]*//")
  new=$(echo "$orig" | sed "s/+cu[0-9]\+$//")
  if [ "$orig" != "$new" ]; then
    sed -i "s/^Version:.*/Version: ${new}/" "$meta"
    old_base=$(basename "$info")
    new_base="${old_base/${orig}/${new}}"
    mv "$info" "$(dirname "$info")/${new_base}"
  fi
  python3 -m wheel pack "$unpacked" --dest-dir dist-pypi
  rm -rf "$tmp"
done
ls -lh dist-pypi/
'
fi
```
**EN:** This block summarizes the produced artifacts, making the final output set easy to inspect in CI logs or local runs.
**CN:** 该代码块会汇总最终产物，便于在 CI 日志或本地执行时快速核对输出结果。

### Lines 120-125: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
echo "Wheels in ${DEEPGEMM_SRC}/dist:"
ls -lh "${DEEPGEMM_SRC}/dist"/*.whl 2>/dev/null || true
if [ "${CU_TAG}" = "cu130" ]; then
  echo "PyPI-ready wheels in ${DEEPGEMM_SRC}/dist-pypi:"
  ls -lh "${DEEPGEMM_SRC}/dist-pypi"/*.whl 2>/dev/null || true
fi
```
**EN:** This block summarizes the produced artifacts, making the final output set easy to inspect in CI logs or local runs.
**CN:** 该代码块会汇总最终产物，便于在 CI 日志或本地执行时快速核对输出结果。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **Container execution** / 容器执行

## Dependencies / 依赖关系
- **Commands / 外部命令**: `docker`, `python3`, `pip`, `mkdir`, `cp`, `mv`, `rm`, `echo`, `grep`, `sed`, `find`, `head`, `ls`, `uname`, `mktemp`
- **Environment variables / 环境变量**: `ARCH`, `BASE_IMG`, `BASH_SOURCE`, `CUDA_VERSION`, `CU_TAG`, `DEEPGEMM_SRC`, `DEPS_TAG`, `DOCKERFILE`, `PYTHON_VERSION`, `PY_TAG`, `RENAME_SCRIPT`, `REPO_ROOT`, `SCRIPT_DIR`
- **Referenced files / 引用文件**: `/rename_sgl_deep_gemm_whl.sh`, `build_sgl_deep_gemm.sh`
