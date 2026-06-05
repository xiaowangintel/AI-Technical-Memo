# amd_ci_start_container_disagg.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `scripts/ci/amd/amd_ci_start_container_disagg.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: This shell script supports the `amd_ci_start_container_disagg` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Shell 脚本用于支撑 SGLang 中的 `amd_ci_start_container_disagg` 流程，主要负责CI 编排、测试执行。它属于 `amd` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: Shell entrypoint and interpreter selection / Shell 入口与解释器选择
```bash
#!/bin/bash
set -euo pipefail
```
**EN:** This opening block selects Bash as the interpreter and usually also documents expected inputs or outputs for the rest of the script.
**CN:** 该起始代码块指定 Bash 作为解释器，并通常顺带说明脚本后续步骤所需的输入与输出。

### Lines 4-5: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
# Get version from git tags
SGLANG_VERSION="v0.5.5"   # Default version, will be overridden if git tags are found
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 7-19: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
# Fetch tags from origin to ensure we have the latest
if git fetch --tags origin; then
  # Use the shared helper so stable/post releases sort above rc tags.
  VERSION_FROM_TAG=$(python3 python/tools/get_version_tag.py --tag-only || true)
  if [ -n "$VERSION_FROM_TAG" ]; then
    SGLANG_VERSION="$VERSION_FROM_TAG"
    echo "Using SGLang version from git tags: $SGLANG_VERSION"
  else
    echo "Warning: No version tags found; using default $SGLANG_VERSION" >&2
  fi
else
  echo "Warning: Failed to fetch tags from origin; using default $SGLANG_VERSION" >&2
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It queries repository state, invokes Python helpers, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会查询仓库状态、调用 Python 辅助脚本、打印状态信息。

### Lines 22-26: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
# Default base tags (can be overridden by command line arguments)
ROCM_VERSION="rocm700"
DEFAULT_MI30X_BASE_TAG="${SGLANG_VERSION}-${ROCM_VERSION}-mi30x"
DEFAULT_MI35X_BASE_TAG="${SGLANG_VERSION}-${ROCM_VERSION}-mi35x"
LOCAL_DOCKER_REGISTRY="10.245.143.50:5000"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 28-30: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
# Parse command line arguments
MI30X_BASE_TAG="${DEFAULT_MI30X_BASE_TAG}"
MI35X_BASE_TAG="${DEFAULT_MI35X_BASE_TAG}"
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 32-48: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
while [[ $# -gt 0 ]]; do
  case $1 in
    --mi30x-base-tag) MI30X_BASE_TAG="$2"; shift 2;;
    --mi35x-base-tag) MI35X_BASE_TAG="$2"; shift 2;;
    --rocm-version)
      ROCM_VERSION="$2"
      MI30X_BASE_TAG="${SGLANG_VERSION}-${ROCM_VERSION}-mi30x"
      MI35X_BASE_TAG="${SGLANG_VERSION}-${ROCM_VERSION}-mi35x"
      echo "Using ROCm version override: ${ROCM_VERSION}"
      shift 2;;
    -h|--help)
      echo "Usage: $0 [--mi30x-base-tag TAG] [--mi35x-base-tag TAG] [--rocm-version VERSION]"
      exit 0
      ;;
    *) echo "Unknown option $1"; exit 1;;
  esac
done
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 52-54: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
# Detect GPU architecture from the Kubernetes runner hostname
HOSTNAME_VALUE=$(hostname)
GPU_ARCH="mi30x"   # default
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 56-62: Performs control-flow checks and command selection / 执行控制流判断与命令选择
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

### Lines 64-77: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# Normalise / collapse architectures we don’t yet build specifically for
case "${GPU_ARCH}" in
  mi35x)
    echo "Runner uses ${GPU_ARCH}; will fetch mi35x image."
    ;;
  mi30x|mi300|mi325)
    echo "Runner uses ${GPU_ARCH}; will fetch mi30x image."
    GPU_ARCH="mi30x"
    ;;
  *)
    echo "Runner architecture '${GPU_ARCH}' unrecognised; defaulting to mi30x image." >&2
    GPU_ARCH="mi30x"
    ;;
esac
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 80-85: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# Set up DEVICE_FLAG based on Kubernetes pod info
if [[ -f /etc/podinfo/gha-render-devices ]]; then
  DEVICE_FLAG=$(cat /etc/podinfo/gha-render-devices)
else
  DEVICE_FLAG="--device /dev/dri"
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

### Lines 87-109: Defines the `retry_with_backoff` shell function / 定义 `retry_with_backoff` shell 函数
```bash
# Retry a command with exponential backoff. Usage: retry_with_backoff <max_attempts> <cmd...>
retry_with_backoff() {
  local max_attempts=$1; shift
  local attempt=1
  local wait_secs=30
  # Add jitter (0-30s) so concurrent jobs don't all retry at the same instant
  local jitter=$(( RANDOM % 30 ))
  while true; do
    if "$@"; then
      return 0
    fi
    if (( attempt >= max_attempts )); then
      echo "Error: '$*' failed after ${max_attempts} attempts" >&2
      return 1
    fi
    local sleep_time=$(( wait_secs + jitter ))
    echo "Attempt ${attempt}/${max_attempts} failed. Retrying in ${sleep_time}s…" >&2
    sleep "${sleep_time}"
    (( attempt++ ))
    (( wait_secs = wait_secs * 2 > 300 ? 300 : wait_secs * 2 ))
    jitter=$(( RANDOM % 30 ))
  done
}
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 111-120: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
# Authenticate to Docker Hub to avoid anonymous pull rate limits.
# Credentials are optional; when absent we fall back to unauthenticated pulls.
if [[ -n "${DOCKERHUB_AMD_USERNAME:-}" && -n "${DOCKERHUB_AMD_TOKEN:-}" ]]; then
  echo "Logging in to Docker Hub…"
  if retry_with_backoff 6 sh -c 'echo "${DOCKERHUB_AMD_TOKEN}" | docker login -u "${DOCKERHUB_AMD_USERNAME}" --password-stdin >/dev/null 2>&1'; then
    echo "Docker Hub login successful"
  else
    echo "Warning: Docker Hub login failed after retries; continuing with unauthenticated pulls" >&2
  fi
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It manages containerized steps, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会管理容器化步骤、打印状态信息。

### Lines 122-125: Defines the `find_latest_image` shell function / 定义 `find_latest_image` shell 函数
```bash
# Find the latest image
find_latest_image() {
  local gpu_arch=$1
  local base_tag days_back image_tag image_id remote_tags
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 127-131: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
  case "${gpu_arch}" in
      mi30x) base_tag="${MI30X_BASE_TAG}" ;;
      mi35x) base_tag="${MI35X_BASE_TAG}" ;;
      *)     echo "Error: unsupported GPU architecture '${gpu_arch}'" >&2; return 1 ;;
  esac
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 133-142: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
  # First, check local cache on the runner.
  for days_back in {0..6}; do
    image_tag="${base_tag}-$(date -d "${days_back} days ago" +%Y%m%d)"
    image_id=$(docker images -q "rocm/sgl-dev:${image_tag}")
    if [[ -n "$image_id" ]]; then
      echo "Found cached image locally: rocm/sgl-dev:${image_tag}" >&2
      echo "rocm/sgl-dev:${image_tag}"
      return 0
    fi
  done
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It manages containerized steps, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会管理容器化步骤、打印状态信息。

### Lines 144-157: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
  # If not found locally, fall back to pulling from public registry.
  # See amd_ci_start_container.sh for why we don't probe
  # ${LOCAL_DOCKER_REGISTRY} with `docker manifest inspect --insecure` from
  # the runner pod's network namespace; the actual local-registry pull
  # happens at the call site below via the docker daemon on the host.
  for days_back in {0..6}; do
    image_tag="${base_tag}-$(date -d "${days_back} days ago" +%Y%m%d)"
    echo "Checking for image: rocm/sgl-dev:${image_tag}" >&2
    if docker manifest inspect "rocm/sgl-dev:${image_tag}" >/dev/null 2>&1; then
      echo "Found available image: rocm/sgl-dev:${image_tag}" >&2
      echo "rocm/sgl-dev:${image_tag}"
      return 0
    fi
  done
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It manages containerized steps, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会管理容器化步骤、打印状态信息。

### Lines 159-169: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
  # If still not found, try finding any image matching ROCm+arch from remote registry
  echo "Exact version not found. Searching remote registry for any ${ROCM_VERSION}-${gpu_arch} image…" >&2
  for days_back in {0..6}; do
    local target_date=$(date -d "${days_back} days ago" +%Y%m%d)
    remote_tags=$(curl -s "https://registry.hub.docker.com/v2/repositories/rocm/sgl-dev/tags?page_size=100&name=${ROCM_VERSION}-${gpu_arch}-${target_date}" 2>/dev/null | grep -o '"name":"[^"]*"' | cut -d'"' -f4 | head -n 1 || true)
    if [[ -n "$remote_tags" ]]; then
      echo "Found available image: rocm/sgl-dev:${remote_tags}" >&2
      echo "rocm/sgl-dev:${remote_tags}"
      return 0
    fi
  done
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 171-178: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
  echo "No recent images found. Searching any cached local images matching ROCm+arch…" >&2
  local any_local
  any_local=$(docker images --format '{{.Repository}}:{{.Tag}}' --filter "reference=rocm/sgl-dev:*${ROCM_VERSION}*${gpu_arch}*" | sort -r | head -n 1)
  if [[ -n "$any_local" ]]; then
      echo "Using cached fallback image: ${any_local}" >&2
      echo "${any_local}"
      return 0
  fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It manages containerized steps, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会管理容器化步骤、打印状态信息。

### Lines 180-202: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
  echo "Error: no ${gpu_arch} image found in the last 7 days for base ${base_tag}" >&2
  echo "Using hard-coded fallback for ${ROCM_VERSION}…" >&2
  case "${ROCM_VERSION}" in
    rocm720)
      if [[ "${gpu_arch}" == "mi35x" ]]; then
        echo "rocm/sgl-dev:v0.5.8.post1-rocm720-mi35x-20260211-preview"
      else
        echo "rocm/sgl-dev:v0.5.8.post1-rocm720-mi30x-20260211-preview"
      fi
      ;;
    rocm700)
      if [[ "${gpu_arch}" == "mi35x" ]]; then
        echo "rocm/sgl-dev:v0.5.8.post1-rocm700-mi35x-20260211"
      else
        echo "rocm/sgl-dev:v0.5.8.post1-rocm700-mi30x-20260211"
      fi
      ;;
    *)
      echo "Error: no hard-coded fallback available for ${ROCM_VERSION}" >&2
      return 1
      ;;
  esac
}
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 204-218: Uses control flow to gate optional execution paths / 使用控制流选择可选执行路径
```bash
# Pull and run the latest image
IMAGE=$(find_latest_image "${GPU_ARCH}")
# Try the local docker registry first (avoids Docker Hub rate limits and is
# faster on the LAN); if that fails for any reason, fall back to the
# public registry with exponential-backoff retries. Capture stderr so the
# real failure reason (TLS handshake, 404, connection refused, etc.) is
# visible in the job log instead of being silently swallowed.
if local_pull_output=$(docker pull "${LOCAL_DOCKER_REGISTRY}/${IMAGE}" 2>&1); then
  echo "Pulled from local docker registry: ${LOCAL_DOCKER_REGISTRY}/${IMAGE}"
  docker tag "${LOCAL_DOCKER_REGISTRY}/${IMAGE}" "${IMAGE}"
else
  echo "Local docker registry pull failed; falling back to public registry: ${IMAGE}" >&2
  printf '%s\n' "${local_pull_output}" | sed 's/^/  [local-pull] /' >&2
  retry_with_backoff 6 docker pull "${IMAGE}"
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It manages containerized steps, prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会管理容器化步骤、打印状态信息。

### Lines 220-226: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# CACHE_HOST=/home/runner/sgl-data
CACHE_HOST=/home/runner/sglang-data
if [[ -d "$CACHE_HOST" ]]; then
    CACHE_VOLUME="-v $CACHE_HOST:/sgl-data"
else
    CACHE_VOLUME=""
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。

### Lines 228-253: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
# Detect libionic library for RDMA support
LIBIONIC_MOUNT=""
IONIC_SYMLINK="/usr/lib/x86_64-linux-gnu/libibverbs/libionic-rdmav34.so"
if [[ -L "$IONIC_SYMLINK" ]]; then
    LIBIONIC_LIB=$(readlink -f "$IONIC_SYMLINK" 2>/dev/null)
    if [[ -f "$LIBIONIC_LIB" ]]; then
        echo "Found libionic library: $LIBIONIC_LIB (resolved from symlink)"
        LIBIONIC_MOUNT="-v ${LIBIONIC_LIB}:${LIBIONIC_LIB}:ro"
    else
        echo "Warning: libionic symlink exists but target does not: $LIBIONIC_LIB"
    fi
else
    # Fallback: try to find directly
    LIBIONIC_FOUND=$(find /usr/lib/x86_64-linux-gnu -maxdepth 1 -name "libionic.so.*" 2>/dev/null | head -1)
    if [[ -n "$LIBIONIC_FOUND" ]]; then
        LIBIONIC_LIB=$(readlink -f "$LIBIONIC_FOUND" 2>/dev/null)
        if [[ -f "$LIBIONIC_LIB" ]]; then
            echo "Found libionic library: $LIBIONIC_LIB"
            LIBIONIC_MOUNT="-v ${LIBIONIC_LIB}:${LIBIONIC_LIB}:ro"
        else
            echo "Warning: libionic found but cannot resolve real path: $LIBIONIC_FOUND"
        fi
    else
        echo "Warning: libionic library not found on host, RDMA may not work"
    fi
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 255-255: Captures inputs and derived variables / 记录输入参数与派生变量
```bash
MOUNT_ARGS=""
```
**EN:** This block normalizes incoming arguments and computes derived variables that later commands can reuse safely.
**CN:** 该代码块会规范化输入参数，并计算后续命令可复用的派生变量。

### Lines 257-260: Defines the `add_mount_if_exists` shell function / 定义 `add_mount_if_exists` shell 函数
```bash
add_mount_if_exists() {
    local name=$1
    local search_pattern=$2
    local path=$(find /lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu /lib64 /usr/lib64 -name "$search_pattern" -print -quit 2>/dev/null)
```
**EN:** This block packages reusable shell logic into a named function so later parts of the script can call it when needed.
**CN:** 该代码块把可复用的 shell 逻辑封装成具名函数，供脚本后续在需要时调用。

### Lines 262-268: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
    if [ -n "$path" ]; then
        echo "Found $name at: $path"
        MOUNT_ARGS="$MOUNT_ARGS -v $path:$path:ro"
    else
        echo "WARNING: Could not find $name on host! (Pattern: $search_pattern)"
    fi
}
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 270-277: Performs control-flow checks and command selection / 执行控制流判断与命令选择
```bash
IONIC_LINK="/usr/lib/x86_64-linux-gnu/libibverbs/libionic-rdmav34.so"
if [ -L "$IONIC_LINK" ]; then
    IONIC_REAL=$(readlink -f "$IONIC_LINK")
    if [ -f "$IONIC_REAL" ]; then
        echo "Ionic Driver: $IONIC_REAL"
        MOUNT_ARGS="$MOUNT_ARGS -v $IONIC_REAL:$IONIC_REAL:ro"
    fi
fi
```
**EN:** This block uses shell control flow to gate, branch, or repeat later commands. It prints status information.
**CN:** 该代码块使用 shell 控制流来判断、分支或重复后续命令。 它会打印状态信息。

### Lines 279-280: Comment or metadata block / 注释或元数据块
```bash
add_mount_if_exists "libnl-3" "libnl-3.so*"
add_mount_if_exists "libmnl" "libmnl.so*"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。

### Lines 282-282: Prints diagnostic status information / 打印诊断状态信息
```bash
echo "Mount args: $MOUNT_ARGS"
```
**EN:** This block prints resolved values so operators can confirm the runtime context before or after heavier actions run.
**CN:** 该代码块会打印解析后的关键值，便于操作者在执行重型步骤前后确认运行上下文。

### Lines 284-315: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
echo "Launching container: ci_sglang"
docker run -dt --user root \
  --device=/dev/kfd \
  --device=/dev/dri \
  ${DEVICE_FLAG} \
  -v "${GITHUB_WORKSPACE:-$PWD}:/sglang-checkout" \
  -v /sys/class/infiniband:/sys/class/infiniband:ro \
  -v /sys/class/infiniband_verbs:/sys/class/infiniband_verbs:ro \
  -v /sys/class/net:/sys/class/net:ro \
  -v /etc/libibverbs.d:/etc/libibverbs.d:ro \
  -v /usr/lib/x86_64-linux-gnu/libibverbs:/usr/lib/x86_64-linux-gnu/libibverbs:ro \
  $MOUNT_ARGS \
  $CACHE_VOLUME \
  --privileged \
  --network=host \
  --ipc=host \
  --ulimit memlock=-1 \
  --cap-add=IPC_LOCK \
  --cap-add=SYS_PTRACE \
  --security-opt seccomp=unconfined \
  --group-add video \
  --group-add rdma \
  --shm-size 32g \
  -e HF_TOKEN="${HF_TOKEN:-}" \
  -e HF_HOME=/sgl-data/hf-cache \
  -e HF_HUB_ETAG_TIMEOUT=300 \
  -e HF_HUB_DOWNLOAD_TIMEOUT=300 \
  -e MIOPEN_USER_DB_PATH=/sgl-data/miopen-cache \
  -e MIOPEN_CUSTOM_CACHE_DIR=/sgl-data/miopen-cache \
  -w /sglang-checkout \
  --name ci_sglang \
  "${IMAGE}"
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, prints status information.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、打印状态信息。

### Lines 317-320: Runs container build or execution steps / 执行容器构建或运行步骤
```bash
# The checkout is owned by the runner (non-root) but the container runs as
# root.  Git >= 2.35.2 rejects cross-user repos; mark the mount as safe so
# setuptools-scm / vcs_versioning can resolve the package version.
docker exec ci_sglang git config --global --add safe.directory /sglang-checkout
```
**EN:** This block performs one cohesive shell step inside the overall automation flow. It manages containerized steps, queries repository state.
**CN:** 该代码块在整体自动化流程中完成一个相对独立的 shell 步骤。 它会管理容器化步骤、查询仓库状态。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Git state inspection** / Git 状态检查
- **Container execution** / 容器执行
- **Runner provisioning** / 运行器准备

## Dependencies / 依赖关系
- **Commands / 外部命令**: `docker`, `git`, `python3`, `echo`, `grep`, `sed`, `find`, `head`, `sh`, `curl`
- **Environment variables / 环境变量**: `BASH_REMATCH`, `CACHE_HOST`, `CACHE_VOLUME`, `DEFAULT_MI30X_BASE_TAG`, `DEFAULT_MI35X_BASE_TAG`, `DEVICE_FLAG`, `DOCKERHUB_AMD_TOKEN`, `DOCKERHUB_AMD_USERNAME`, `GITHUB_WORKSPACE`, `GPU_ARCH`, `HF_TOKEN`, `HOSTNAME_VALUE`, `IMAGE`, `IONIC_LINK`, `IONIC_REAL`
- **Referenced files / 引用文件**: `amd_ci_start_container.sh`, `python/tools/get_version_tag.py`
