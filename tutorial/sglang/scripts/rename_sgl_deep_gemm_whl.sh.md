# rename_sgl_deep_gemm_whl.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/rename_sgl_deep_gemm_whl.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: Rename a freshly-built sgl-deep-gemm wheel so its filename, METADATA Version, and WHEEL platform tag carry the +cuXXX local version and manylinux2014_<arch> tag expected by the sgl-whl index and PyPI upload step. This shell script is part of SGLang's `scripts` automation flow. / 该Shell 脚本用于支撑 SGLang 中的 `rename_sgl_deep_gemm_whl` 流程，主要负责CI 编排、依赖安装。它属于 `scripts` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Shell entrypoint and interpreter selection / Shell 入口与解释器选择
```bash
#!/usr/bin/env bash
# Rename a freshly-built sgl-deep-gemm wheel so its filename, METADATA Version,
# and WHEEL platform tag carry the +cuXXX local version and manylinux2014_<arch>
# tag expected by the sgl-whl index and PyPI upload step.
#
# Input:  dist/sgl_deep_gemm-<VERSION>-py3-none-any.whl
# Output: dist/sgl_deep_gemm-<VERSION>+<CU_TAG>-py3-none-manylinux2014_<ARCH>.whl
#
# Usage: rename_wheels.sh <WHEEL_DIR> <CU_TAG> <ARCH>
#   WHEEL_DIR: directory containing the *.whl file (e.g. DeepGEMM/dist)
#   CU_TAG:    cu129 | cu130
#   ARCH:      x86_64 | aarch64
set -ex
```
**EN:** This opening block selects Bash as the interpreter and usually also documents expected inputs or outputs for the rest of the script.
**CN:** 该起始代码块指定 Bash 作为解释器，并通常顺带说明脚本后续步骤所需的输入与输出。

### Lines 15-18: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if [ $# -lt 3 ]; then
  echo "Usage: $0 <WHEEL_DIR> <CU_TAG> <ARCH>"
  exit 1
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 20-23: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
WHEEL_DIR="$1"
CU_TAG="$2"
ARCH="$3"
PLAT_TAG="manylinux2014_${ARCH}"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 25-26: Executes `pip`-based shell logic / 执行基于 `pip` 的 shell 逻辑
```bash
PYTHON="${PYTHON:-python3}"
"${PYTHON}" -m pip install --quiet wheel
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It installs Python packages.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会安装 Python 包。

### Lines 28-33: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
shopt -s nullglob
wheel_files=("${WHEEL_DIR}"/sgl_deep_gemm-*.whl)
if [ ${#wheel_files[@]} -eq 0 ]; then
  echo "No sgl_deep_gemm wheel found under ${WHEEL_DIR}" >&2
  exit 1
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 35-37: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
for wheel in "${wheel_files[@]}"; do
  TMPDIR=$(mktemp -d)
  trap 'rm -rf -- "$TMPDIR"' ERR
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It cleans generated files.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会清理生成文件。

### Lines 39-43: Executes `find`-based shell logic / 执行基于 `find` 的 shell 逻辑
```bash
  "${PYTHON}" -m wheel unpack "$wheel" --dest "$TMPDIR"
  UNPACKED=$(find "$TMPDIR" -mindepth 1 -maxdepth 1 -type d | head -1)
  DIST_INFO=$(find "$UNPACKED" -maxdepth 1 -type d -name "*.dist-info" | head -1)
  WHEEL_META="${DIST_INFO}/WHEEL"
  METADATA_FILE="${DIST_INFO}/METADATA"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 45-46: Executes `sed`-based shell logic / 执行基于 `sed` 的 shell 逻辑
```bash
  # Replace the py3-none-any tag with a platform-specific one.
  sed -i "s/^Tag: py3-none-any$/Tag: py3-none-${PLAT_TAG}/" "$WHEEL_META"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 48-57: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
  ORIG_VERSION=$(grep '^Version:' "$METADATA_FILE" | head -1 | sed 's/^Version:[[:space:]]*//')
  if [[ "$ORIG_VERSION" == *"+${CU_TAG}"* ]]; then
    NEW_VERSION="$ORIG_VERSION"
  else
    NEW_VERSION="${ORIG_VERSION}+${CU_TAG}"
    sed -i "s/^Version:.*/Version: ${NEW_VERSION}/" "$METADATA_FILE"
    OLD_BASE=$(basename "$DIST_INFO")
    NEW_BASE="${OLD_BASE/${ORIG_VERSION}/${NEW_VERSION}}"
    mv "$DIST_INFO" "${UNPACKED}/${NEW_BASE}"
  fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It moves artifacts.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会移动产物。

### Lines 59-63: Prepares directories and generated artifacts / 准备目录与生成产物
```bash
  rm -f "$wheel"
  "${PYTHON}" -m wheel pack "$UNPACKED" --dest-dir "$WHEEL_DIR"
  rm -rf "$TMPDIR"
  trap - ERR
done
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It cleans generated files.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会清理生成文件。

### Lines 65-66: Prints final artifact summary / 打印最终产物摘要
```bash
echo "Renamed wheels in ${WHEEL_DIR}:"
ls -lh "${WHEEL_DIR}"/*.whl
```
**EN:** This block summarizes the produced artifacts, making the final output set easy to inspect in CI logs or local runs.
**CN:** 该代码块会汇总最终产物，便于在 CI 日志或本地执行时快速核对输出结果。

## Key Concepts / 关键概念
- **Shell automation flow** / Shell 自动化流程

## Dependencies / 依赖关系
- **Commands / 外部命令**: `pip`, `mv`, `rm`, `echo`, `grep`, `sed`, `find`, `head`, `ls`, `mktemp`, `bash`
- **Environment variables / 环境变量**: `ARCH`, `CU_TAG`, `DIST_INFO`, `METADATA_FILE`, `NEW_BASE`, `NEW_VERSION`, `OLD_BASE`, `ORIG_VERSION`, `PLAT_TAG`, `PYTHON`, `TMPDIR`, `UNPACKED`, `WHEEL_DIR`, `WHEEL_META`
- **Referenced files / 引用文件**: `rename_wheels.sh`
