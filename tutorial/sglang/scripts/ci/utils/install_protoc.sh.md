# install_protoc.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/ci/utils/install_protoc.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: Ensure protoc is installed for router build (gRPC protobuf compilation). This shell script is part of SGLang's `utils` automation flow. / 该Shell 脚本用于支撑 SGLang 中的 `install_protoc` 流程，主要负责CI 编排、构建自动化。它属于 `utils` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: Shell entrypoint and interpreter selection / Shell 入口与解释器选择
```bash
#!/bin/bash
# Ensure protoc is installed for router build (gRPC protobuf compilation).
set -euxo pipefail
```
**EN:** This opening block selects Bash as the interpreter and usually also documents expected inputs or outputs for the rest of the script.
**CN:** 该起始代码块指定 Bash 作为解释器，并通常顺带说明脚本后续步骤所需的输入与输出。

### Lines 5-8: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if command -v protoc >/dev/null 2>&1 && protoc --version >/dev/null 2>&1; then
    echo "protoc already installed: $(protoc --version)"
    exit 0
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 10-14: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if command -v protoc >/dev/null 2>&1; then
    echo "protoc found but not runnable, reinstalling..."
else
    echo "protoc not found, installing..."
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 16-16: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
ARCH=$(uname -m)
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 18-39: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if command -v apt-get &> /dev/null; then
    # Ubuntu/Debian
    apt-get update || true # May fail due to unrelated broken packages
    PROTOC_APT_PACKAGES=(wget unzip)
    apt-get install -y --no-install-recommends "${PROTOC_APT_PACKAGES[@]}" || {
        echo "Warning: apt-get install failed, checking if required packages are available..."
        for pkg in "${PROTOC_APT_PACKAGES[@]}"; do
            if ! dpkg -l "$pkg" 2>/dev/null | grep -q "^ii"; then
                echo "ERROR: Required package $pkg is not installed and apt-get failed"
                exit 1
            fi
        done
        echo "All required packages are already installed, continuing..."
    }
elif command -v yum &> /dev/null; then
    # RHEL/CentOS
    yum update -y
    yum install -y wget unzip
else
    echo "ERROR: Neither apt-get nor yum found; cannot install protoc"
    exit 1
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 41-53: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if [ "$ARCH" = "aarch64" ] || [ "$ARCH" = "arm64" ]; then
    PROTOC_ARCH="aarch_64"
else
    PROTOC_ARCH="x86_64"
fi
PROTOC_ZIP="protoc-32.0-linux-${PROTOC_ARCH}.zip"
(
    cd /tmp
    wget "https://github.com/protocolbuffers/protobuf/releases/download/v32.0/${PROTOC_ZIP}"
    unzip -o "${PROTOC_ZIP}" -d /usr/local
    rm -f "${PROTOC_ZIP}"
)
protoc --version
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It cleans generated files.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会清理生成文件。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Commands / 外部命令**: `rm`, `echo`, `grep`, `uname`
- **Environment variables / 环境变量**: `ARCH`, `PROTOC_APT_PACKAGES`, `PROTOC_ARCH`, `PROTOC_ZIP`
