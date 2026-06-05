# install_rustup.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/ci/utils/install_rustup.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: Ensure a Rust toolchain (rustc/cargo) is installed for crates built from source, e.g. This shell script is part of SGLang's `utils` automation flow. / 该Shell 脚本用于支撑 SGLang 中的 `install_rustup` 流程，主要负责CI 编排、运行器选择与准备。它属于 `utils` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Shell entrypoint and interpreter selection / Shell 入口与解释器选择
```bash
#!/bin/bash
# Ensure a Rust toolchain (rustc/cargo) is installed for crates built from
# source, e.g. the native gRPC extension bundled into the sglang wheel via
# setuptools-rust. Minimum supported version is 1.85 (edition 2024).
set -euxo pipefail
```
**EN:** This opening block selects Bash as the interpreter and usually also documents expected inputs or outputs for the rest of the script.
**CN:** 该起始代码块指定 Bash 作为解释器，并通常顺带说明脚本后续步骤所需的输入与输出。

### Lines 7-12: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# Make cargo/rustc visible to the rest of this shell and to subsequent
# GitHub Actions steps in the same job.
export PATH="${CARGO_HOME:-$HOME/.cargo}/bin:${PATH}"
if [ -n "${GITHUB_PATH:-}" ]; then
    echo "${CARGO_HOME:-$HOME/.cargo}/bin" >> "${GITHUB_PATH}"
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 14-17: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if command -v cargo >/dev/null 2>&1 && command -v rustc >/dev/null 2>&1; then
    echo "rust already installed: $(rustc --version), $(cargo --version)"
    exit 0
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 19-19: Prints diagnostic status information / 打印诊断状态信息
```bash
echo "rust not found, installing via rustup..."
```
**EN:** This block prints resolved values so operators can confirm the runtime context before or after heavier actions run.
**CN:** 该代码块会打印解析后的关键值，便于操作者在执行重型步骤前后确认运行上下文。

### Lines 21-32: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# rustup.rs requires curl — make sure it's present.
if ! command -v curl >/dev/null 2>&1; then
    if command -v apt-get &> /dev/null; then
        apt-get update || true
        apt-get install -y --no-install-recommends curl ca-certificates
    elif command -v yum &> /dev/null; then
        yum install -y curl ca-certificates
    else
        echo "ERROR: curl is required to install rustup, but no supported package manager was found"
        exit 1
    fi
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It builds native artifacts, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会构建原生产物、打印状态信息。

### Lines 34-53: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if [ -n "${RUSTUP_CACHE_URL:-}" ]; then
    # An in-cluster HTTP mirror is available (e.g. on NPU runners).
    export RUSTUP_DIST_SERVER="${RUSTUP_CACHE_URL}/rustup"
    export RUSTUP_UPDATE_ROOT="${RUSTUP_CACHE_URL}/rustup/rustup"
    case "$(uname -m)" in
        x86_64)  RUSTUP_ARCH="x86_64-unknown-linux-gnu" ;;
        aarch64) RUSTUP_ARCH="aarch64-unknown-linux-gnu" ;;
        *) echo "ERROR: unsupported arch $(uname -m)"; exit 1 ;;
    esac
    RUSTUP_TMP="$(mktemp -d)"
    trap 'rm -rf "${RUSTUP_TMP}"' EXIT
    curl --retry 3 --retry-delay 2 -sSfL \
        "${RUSTUP_UPDATE_ROOT}/dist/${RUSTUP_ARCH}/rustup-init" \
        -o "${RUSTUP_TMP}/rustup-init"
    chmod +x "${RUSTUP_TMP}/rustup-init"
    "${RUSTUP_TMP}/rustup-init" -y --no-modify-path
else
    curl --proto '=https' --tlsv1.2 --retry 3 --retry-delay 2 -sSf https://sh.rustup.rs \
        | sh -s -- -y --no-modify-path
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It cleans generated files, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会清理生成文件、打印状态信息。

### Lines 55-56: Comment or metadata block / 注释或元数据块
```bash
rustc --version
cargo --version
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

## Key Concepts / 关键概念
- **Git state inspection** / Git 状态检查
- **Runner provisioning** / 运行器准备

## Dependencies / 依赖关系
- **Commands / 外部命令**: `make`, `rm`, `echo`, `uname`, `mktemp`, `sh`, `curl`
- **Environment variables / 环境变量**: `CARGO_HOME`, `GITHUB_PATH`, `HOME`, `PATH`, `RUSTUP_ARCH`, `RUSTUP_CACHE_URL`, `RUSTUP_TMP`, `RUSTUP_UPDATE_ROOT`
