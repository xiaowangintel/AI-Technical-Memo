# amd_ci_install_dependency.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/ci/amd/amd_ci_install_dependency.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: This shell script supports the `amd_ci_install_dependency` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Shell 脚本用于支撑 SGLang 中的 `amd_ci_install_dependency` 流程，主要负责CI 编排、测试执行。它属于 `amd` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Shell entrypoint and interpreter selection / Shell 入口与解释器选择
```bash
#!/bin/bash
set -euo pipefail
HOSTNAME_VALUE=$(hostname)
GPU_ARCH="mi30x"   # default
SKIP_TT_DEPS=""
SKIP_SGLANG_BUILD=""
SKIP_AITER_BUILD=""
```
**EN:** This opening block selects Bash as the interpreter and usually also documents expected inputs or outputs for the rest of the script.
**CN:** 该起始代码块指定 Bash 作为解释器，并通常顺带说明脚本后续步骤所需的输入与输出。

### Lines 9-24: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
while [[ $# -gt 0 ]]; do
  case $1 in
    --skip-aiter-build) SKIP_AITER_BUILD="1"; shift;;
    --skip-sglang-build) SKIP_SGLANG_BUILD="1"; shift;;
    --skip-test-time-deps) SKIP_TT_DEPS="1"; shift;;
    -h|--help)
      echo "Usage: $0 [OPTIONS] [OPTIONAL_DEPS]"
      echo "Options:"
      echo "  --skip-sglang-build         Don't build checkout sglang, use what was shipped with the image"
      echo "  --skip-aiter-build          Don't build aiter, use what was shipped with the image"
      echo "  --skip-test-time-deps       Don't build miscellaneous dependencies"
      exit 0
      ;;
    *) break ;;
  esac
done
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 26-26: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
OPTIONAL_DEPS="${1:-}"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 28-33: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
# Build python extras
EXTRAS="dev_hip,tracing"
if [ -n "$OPTIONAL_DEPS" ]; then
    EXTRAS="dev_hip,tracing,${OPTIONAL_DEPS}"
fi
echo "Installing python extras: [${EXTRAS}]"
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It invokes Python helpers, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会调用 Python 辅助脚本、打印状态信息。

### Lines 35-41: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# Host names look like: linux-mi35x-gpu-1-xxxxx-runner-zzzzz
if [[ "${HOSTNAME_VALUE}" =~ ^linux-(mi[0-9]+[a-z]*)-gpu-[0-9]+ ]]; then
  GPU_ARCH="${BASH_REMATCH[1]}"
  echo "Detected GPU architecture from hostname: ${GPU_ARCH}"
else
  echo "Warning: could not parse GPU architecture from '${HOSTNAME_VALUE}', defaulting to ${GPU_ARCH}"
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 43-46: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
# Install the required dependencies in CI.
# Fix permissions on pip cache, ignore errors from concurrent access or missing temp files
docker exec ci_sglang chown -R root:root /sgl-data/pip-cache 2>/dev/null || true
docker exec ci_sglang pip install --cache-dir=/sgl-data/pip-cache --upgrade pip
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, installs Python packages.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、安装 Python 包。

### Lines 48-51: Defines the `install_with_retry` shell function / 定义 `install_with_retry` shell 函数
```bash
# Helper function to install with retries and fallback PyPI mirror
install_with_retry() {
  local max_attempts=3
  local cmd="$@"
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 53-58: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
  for attempt in $(seq 1 $max_attempts); do
    echo "Attempt $attempt/$max_attempts: $cmd"
    if eval "$cmd"; then
      echo "Success!"
      return 0
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 60-69: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ $attempt -lt $max_attempts ]; then
      echo "Failed, retrying in 5 seconds..."
      sleep 5
      # Try with alternative PyPI index on retry
      if [[ "$cmd" =~ "pip install" ]] && [ $attempt -eq 2 ]; then
        cmd="$cmd --index-url https://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com"
        echo "Using fallback PyPI mirror: $cmd"
      fi
    fi
  done
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It installs Python packages, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会安装 Python 包、打印状态信息。

### Lines 71-73: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
  echo "Failed after $max_attempts attempts"
  return 1
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 75-80: Defines the `git_clone_with_retry` shell function / 定义 `git_clone_with_retry` shell 函数
```bash
# Helper function to git clone with retries
git_clone_with_retry() {
  local repo_url="$1"
  local dest_dir="${2:-}"
  local branch_args="${3:-}"
  local max_attempts=3
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 82-83: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
  for attempt in $(seq 1 $max_attempts); do
    echo "Git clone attempt $attempt/$max_attempts: $repo_url"
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 85-88: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    # prevent from partial clone
    if [ -n "$dest_dir" ] && [ -d "$dest_dir" ]; then
      rm -rf "$dest_dir"
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It cleans generated files.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会清理生成文件。

### Lines 90-96: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if git \
      -c http.lowSpeedLimit=1000 \
      -c http.lowSpeedTime=30 \
      clone --depth 1 ${branch_args:+$branch_args} "$repo_url" "$dest_dir"; then
      echo "Git clone succeeded."
      return 0
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It queries repository state, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会查询仓库状态、打印状态信息。

### Lines 98-102: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ $attempt -lt $max_attempts ]; then
      echo "Git clone failed, retrying in 5 seconds..."
      sleep 5
    fi
  done
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 104-106: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
  echo "Git clone failed after $max_attempts attempts: $repo_url"
  return 1
}
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 108-121: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
# Install checkout sglang
if [ -n "$SKIP_SGLANG_BUILD" ]; then
  echo "Didn't build checkout SGLang"
else
  docker exec ci_sglang pip uninstall sgl-kernel -y || true
  docker exec ci_sglang pip uninstall sglang-kernel -y || true
  docker exec ci_sglang pip uninstall sglang -y || true
  # Clear Python cache to ensure latest code is used
  docker exec ci_sglang find /opt/venv -name "*.pyc" -delete || true
  docker exec ci_sglang find /opt/venv -name "__pycache__" -type d -exec rm -rf {} + || true
  # Also clear cache in sglang-checkout
  docker exec ci_sglang find /sglang-checkout -name "*.pyc" -delete || true
  docker exec ci_sglang find /sglang-checkout -name "__pycache__" -type d -exec rm -rf {} + || true
  docker exec -w /sglang-checkout/sgl-kernel ci_sglang bash -c "rm -f pyproject.toml && mv pyproject_rocm.toml pyproject.toml && python3 setup_rocm.py install"
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It manages containerized steps, invokes Python helpers, installs Python packages, moves artifacts, cleans generated files, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会管理容器化步骤、调用 Python 辅助脚本、安装 Python 包、移动产物、清理生成文件、打印状态信息。

### Lines 123-125: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
  docker exec ci_sglang bash -c 'rm -rf python/pyproject.toml && mv python/pyproject_other.toml python/pyproject.toml'
  install_with_retry docker exec ci_sglang pip install --cache-dir=/sgl-data/pip-cache -e "python[${EXTRAS}]"
fi
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, invokes Python helpers, installs Python packages, moves artifacts, cleans generated files.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、调用 Python 辅助脚本、安装 Python 包、移动产物、清理生成文件。

### Lines 127-137: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
if [[ -n "${SKIP_TT_DEPS}" ]]; then
  echo "Didn't build lmms_eval, human-eval, and others"
else
  # For lmms_evals evaluating MMMU
  # Clone on host (with retry), then copy into the container. The checkout is
  # owned by the runner (non-root); mark it safe so setuptools_scm /
  # vcs_versioning can run `git` introspection during pip install.
  git_clone_with_retry https://github.com/EvolvingLMMs-Lab/lmms-eval.git lmms-eval "--branch v0.4.1"
  docker cp lmms-eval ci_sglang:/
  docker exec ci_sglang git config --global --add safe.directory /lmms-eval
  install_with_retry docker exec -w /lmms-eval ci_sglang pip install --cache-dir=/sgl-data/pip-cache -e .
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It manages containerized steps, queries repository state, installs Python packages, copies artifacts, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会管理容器化步骤、查询仓库状态、安装 Python 包、复制产物、打印状态信息。

### Lines 139-141: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
  git_clone_with_retry https://github.com/akao-amd/human-eval.git human-eval
  docker cp human-eval ci_sglang:/
  install_with_retry docker exec -w /human-eval ci_sglang pip install --cache-dir=/sgl-data/pip-cache -e .
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, installs Python packages, copies artifacts.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、安装 Python 包、复制产物。

### Lines 143-171: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
  docker exec -w / ci_sglang mkdir -p /dummy-grok
  # Create dummy grok config inline (bypasses Azure blob storage which may have auth issues)
  mkdir -p dummy-grok
  cat > dummy-grok/config.json << 'EOF'
  {
    "architectures": [
      "Grok1ModelForCausalLM"
    ],
    "embedding_multiplier_scale": 78.38367176906169,
    "output_multiplier_scale": 0.5773502691896257,
    "vocab_size": 131072,
    "hidden_size": 6144,
    "intermediate_size": 32768,
    "max_position_embeddings": 8192,
    "num_experts_per_tok": 2,
    "num_local_experts": 8,
    "num_attention_heads": 48,
    "num_hidden_layers": 64,
    "num_key_value_heads": 8,
    "head_dim": 128,
    "rms_norm_eps": 1e-05,
    "rope_theta": 10000.0,
    "model_type": "mixtral",
    "torch_dtype": "bfloat16"
  }
EOF
  # docker exec -w / ci_sglang mkdir -p /dummy-grok
  # mkdir -p dummy-grok && wget https://sharkpublic.blob.core.windows.net/sharkpublic/sglang/dummy_grok.json -O dummy-grok/config.json
  # docker cp ./dummy-grok ci_sglang:/
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, prepares directories, copies artifacts.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、准备目录、复制产物。

### Lines 173-174: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
  docker exec ci_sglang pip install --cache-dir=/sgl-data/pip-cache huggingface_hub[hf_xet]
  docker exec ci_sglang pip install --cache-dir=/sgl-data/pip-cache pytest
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, runs tests, installs Python packages.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、运行测试、安装 Python 包。

### Lines 176-177: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
  # Install cache-dit for qwen_image_t2i_cache_dit_enabled test (added in PR 16204)
  docker exec ci_sglang pip install --cache-dir=/sgl-data/pip-cache --upgrade 'cache-dit==1.3.0' || echo "cache-dit installation failed"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, installs Python packages, prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、安装 Python 包、打印状态信息。

### Lines 179-181: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
  # Install accelerate for distributed training and inference support
  docker exec ci_sglang pip install --cache-dir=/sgl-data/pip-cache accelerate || echo "accelerate installation failed"
fi
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, installs Python packages, prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、安装 Python 包、打印状态信息。

### Lines 183-185: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if [[ -n "${SKIP_AITER_BUILD}" ]]; then
  exit 0
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

### Lines 187-192: Comment or metadata block / 注释或元数据块
```bash
# Detect AITER version
#############################################
# Detect correct AITER_COMMIT for this runner
# + Check mismatch
# + Rebuild AITER if needed
#############################################
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 194-194: Prints diagnostic status information / 打印诊断状态信息
```bash
echo "[CI-AITER-CHECK] === AITER VERSION CHECK START ==="
```
**EN:** This block prints resolved values so operators can confirm the runtime context before or after heavier actions run.
**CN:** 该代码块会打印解析后的关键值，便于操作者在执行重型步骤前后确认运行上下文。

### Lines 196-196: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
DOCKERFILE="docker/rocm.Dockerfile"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 198-200: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
# GPU_ARCH
GPU_ARCH="${GPU_ARCH:-mi30x}"
echo "[CI-AITER-CHECK] Runner GPU_ARCH=${GPU_ARCH}"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 202-217: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
#############################################
# 1. Extract AITER_COMMIT from correct Dockerfile block
#############################################
if [[ "${GPU_ARCH}" == "mi35x" ]]; then
    echo "[CI-AITER-CHECK] Using gfx950 block from Dockerfile..."
    REPO_AITER_COMMIT=$(grep -F -A20 'FROM $BASE_IMAGE_950 AS gfx950' docker/rocm.Dockerfile \
                        | grep 'AITER_COMMIT_DEFAULT=' \
                        | head -n1 \
                        | sed 's/.*AITER_COMMIT_DEFAULT="\([^"]*\)".*/\1/')
else
    echo "[CI-AITER-CHECK] Using gfx942 block from Dockerfile..."
    REPO_AITER_COMMIT=$(grep -F -A20 'FROM $BASE_IMAGE_942 AS gfx942' docker/rocm.Dockerfile \
                        | grep 'AITER_COMMIT_DEFAULT=' \
                        | head -n1 \
                        | sed 's/.*AITER_COMMIT_DEFAULT="\([^"]*\)".*/\1/')
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 220-223: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if [[ -z "${REPO_AITER_COMMIT}" ]]; then
    echo "[CI-AITER-CHECK] ERROR: Failed to extract AITER_COMMIT from Dockerfile."
    exit 1
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 225-225: Prints diagnostic status information / 打印诊断状态信息
```bash
echo "[CI-AITER-CHECK] Dockerfile expects AITER_COMMIT=${REPO_AITER_COMMIT}"
```
**EN:** This block prints resolved values so operators can confirm the runtime context before or after heavier actions run.
**CN:** 该代码块会打印解析后的关键值，便于操作者在执行重型步骤前后确认运行上下文。

### Lines 227-232: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
#############################################
# 2. Check container pre-installed AITER version
#############################################
IMAGE_AITER_VERSION=$(docker exec ci_sglang bash -c "pip show amd-aiter 2>/dev/null | grep '^Version:' | awk '{print \$2}'" || echo "none")
IMAGE_AITER_VERSION="v${IMAGE_AITER_VERSION}"
echo "[CI-AITER-CHECK] AITER version inside CI image: ${IMAGE_AITER_VERSION}"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, installs Python packages, prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、安装 Python 包、打印状态信息。

### Lines 234-237: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
#############################################
# 3. Decide rebuild
#############################################
NEED_REBUILD="false"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 239-254: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
if [[ -n "${AITER_COMMIT_OVERRIDE:-}" ]]; then
    echo "[CI-AITER-CHECK] AITER_COMMIT_OVERRIDE=${AITER_COMMIT_OVERRIDE} → forcing rebuild"
    REPO_AITER_COMMIT="${AITER_COMMIT_OVERRIDE}"
    NEED_REBUILD="true"
elif [[ "${IMAGE_AITER_VERSION}" == "vnone" || "${IMAGE_AITER_VERSION}" == "v" ]]; then
    echo "[CI-AITER-CHECK] No AITER found in image → rebuild needed"
    NEED_REBUILD="true"
elif [[ "${IMAGE_AITER_VERSION}" == "${REPO_AITER_COMMIT}" ]]; then
    echo "[CI-AITER-CHECK] AITER version matches"
elif [[ "${IMAGE_AITER_VERSION}" =~ (dev|\+g[0-9a-f]+) ]]; then
    # Dev/patched version (contains 'dev' or git hash) → preserve it
    echo "[CI-AITER-CHECK] Dev/patched version detected: ${IMAGE_AITER_VERSION} → skipping rebuild"
else
    echo "[CI-AITER-CHECK] Version mismatch: image=${IMAGE_AITER_VERSION}, repo=${REPO_AITER_COMMIT}"
    NEED_REBUILD="true"
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It queries repository state, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会查询仓库状态、打印状态信息。

### Lines 257-261: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
#############################################
# 4. Rebuild AITER if needed
#############################################
if [[ "${NEED_REBUILD}" == "true" ]]; then
    echo "[CI-AITER-CHECK] === AITER REBUILD START ==="
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 263-264: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
    # uninstall existing aiter
    docker exec ci_sglang pip uninstall -y amd-aiter || true
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, installs Python packages.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、安装 Python 包。

### Lines 266-267: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
    # delete old aiter directory
    docker exec ci_sglang rm -rf /sgl-workspace/aiter
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, cleans generated files.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、清理生成文件。

### Lines 269-270: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
    # clone a fresh copy to /sgl-workspace/aiter
    docker exec ci_sglang git clone https://github.com/ROCm/aiter.git /sgl-workspace/aiter
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, queries repository state.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、查询仓库状态。

### Lines 272-279: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
    # checkout correct version and install requirements
    docker exec ci_sglang bash -c "
        cd /sgl-workspace/aiter && \
        git fetch --all && \
        git checkout ${REPO_AITER_COMMIT} && \
        git submodule update --init --recursive && \
        pip install -r requirements.txt
    "
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, queries repository state, installs Python packages.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、查询仓库状态、安装 Python 包。

### Lines 281-286: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [[ "${GPU_ARCH}" == "mi35x" ]]; then
        GPU_ARCH_LIST="gfx950"
    else
        GPU_ARCH_LIST="gfx942"
    fi
    echo "[CI-AITER-CHECK] GPU_ARCH_LIST=${GPU_ARCH_LIST}"
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 288-304: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
    # Re-apply Dockerfile hotpatches for ROCm 7.2 (the fresh clone lost them, can be removed after triton fixed this problem)
    ROCM_VERSION=$(docker exec ci_sglang bash -c "cat /opt/rocm/.info/version 2>/dev/null || echo unknown")
    if [[ "${ROCM_VERSION}" == 7.2* ]]; then
        echo "[CI-AITER-CHECK] ROCm 7.2 detected (${ROCM_VERSION}), applying AITER hotpatches..."
        docker exec ci_sglang bash -c "
            cd /sgl-workspace/aiter && \
            TARGET_FILE='aiter/ops/triton/attention/pa_mqa_logits.py' && \
            if [ -f \"\${TARGET_FILE}\" ]; then \
                sed -i '459 s/if.*:/if False:/' \"\${TARGET_FILE}\" && \
                echo '[CI-AITER-CHECK] Hotpatch applied to pa_mqa_logits.py'; \
            else \
                echo '[CI-AITER-CHECK] pa_mqa_logits.py not found, skipping hotpatch'; \
            fi
        "
    else
        echo "[CI-AITER-CHECK] ROCm version=${ROCM_VERSION}, no hotpatch needed"
    fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It manages containerized steps, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会管理容器化步骤、打印状态信息。

### Lines 306-310: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
    # build AITER
    docker exec ci_sglang bash -c "
        cd /sgl-workspace/aiter && \
        AITER_USE_SYSTEM_TRITON=1 GPU_ARCHS=${GPU_ARCH_LIST} python3 setup.py develop
    "
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, invokes Python helpers.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、调用 Python 辅助脚本。

### Lines 312-313: Executes `echo`-based shell logic / 执行基于 `echo` 的 shell 逻辑
```bash
    echo "[CI-AITER-CHECK] === AITER REBUILD COMPLETE ==="
fi
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会打印状态信息。

### Lines 315-315: Prints diagnostic status information / 打印诊断状态信息
```bash
echo "[CI-AITER-CHECK] === AITER VERSION CHECK END ==="
```
**EN:** This block prints resolved values so operators can confirm the runtime context before or after heavier actions run.
**CN:** 该代码块会打印解析后的关键值，便于操作者在执行重型步骤前后确认运行上下文。

### Lines 318-322: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
# # Clear pre-built AITER kernels from Docker image to avoid segfaults
# # The Docker image may contain pre-compiled kernels incompatible with the current environment
# echo "Clearing pre-built AITER kernels from Docker image..."
# docker exec ci_sglang find /sgl-workspace/aiter/aiter/jit -name "*.so" -delete 2>/dev/null || true
# docker exec ci_sglang ls -la /sgl-workspace/aiter/aiter/jit/ 2>/dev/null || echo "jit dir empty or not found"
```
**EN:** This block summarizes the produced artifacts, making the final output set easy to inspect in CI logs or local runs.
**CN:** 该代码块会汇总最终产物，便于在 CI 日志或本地执行时快速核对输出结果。

### Lines 324-326: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
# # Pre-build AITER kernels to avoid timeout during tests
# echo "Warming up AITER JIT kernels..."
# docker exec -e SGLANG_USE_AITER=1 ci_sglang python3 /sglang-checkout/scripts/ci/amd/amd_ci_warmup_aiter.py || echo "AITER warmup completed (some kernels may not be available)"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, invokes Python helpers, prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、调用 Python 辅助脚本、打印状态信息。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Structured data handling** / 结构化数据处理
- **Git state inspection** / Git 状态检查
- **Container execution** / 容器执行

## Dependencies / 依赖关系
- **Commands / 外部命令**: `docker`, `git`, `python`, `python3`, `pytest`, `pip`, `mkdir`, `cp`, `mv`, `rm`, `echo`, `grep`, `sed`, `find`, `head`
- **Environment variables / 环境变量**: `AITER_COMMIT_OVERRIDE`, `BASE_IMAGE_942`, `BASE_IMAGE_950`, `BASH_REMATCH`, `EXTRAS`, `GPU_ARCH`, `GPU_ARCH_LIST`, `HOSTNAME_VALUE`, `IMAGE_AITER_VERSION`, `NEED_REBUILD`, `OPTIONAL_DEPS`, `REPO_AITER_COMMIT`, `ROCM_VERSION`, `SKIP_AITER_BUILD`, `SKIP_SGLANG_BUILD`
- **Referenced files / 引用文件**: `//sharkpublic.blob.core.windows.net/sharkpublic/sglang/dummy_grok.json`, `/sglang-checkout/scripts/ci/amd/amd_ci_warmup_aiter.py`, `aiter/ops/triton/attention/pa_mqa_logits.py`, `dummy-grok/config.json`, `pa_mqa_logits.py`, `requirements.txt`, `setup.py`, `setup_rocm.py`
