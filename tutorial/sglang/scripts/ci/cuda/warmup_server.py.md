# warmup_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/cuda/warmup_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `warmup_server` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `warmup_server` 流程，主要负责CI 编排、测试执行。它属于 `cuda` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Full server warmup to pre-warm Triton autotuning and CUDA graph capture.

On cold H200 nodes (new nodes or after container recreation), CUDA graph capture
triggers Triton autotuning which takes ~330s per server launch. This script
launches actual servers with CUDA graphs enabled to cache the autotuned kernels,
so subsequent test launches are fast (~30-60s).

Uses marker files to skip warmup on already-warm nodes. Marker files are
invalidated when Python, Triton, or PyTorch versions change.

Usage:
    python3 scripts/ci/cuda/warmup_server.py \
        deepseek-ai/DeepSeek-V3-0324:8
"""
```
**EN:** Full server warmup to pre-warm Triton autotuning and CUDA graph capture.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 17-25: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import hashlib
import json
import os
import signal
import subprocess
import sys
import tempfile
import time
from pathlib import Path
```
**EN:** This block loads hashlib, json, os, signal, subprocess, sys, tempfile, time. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 hashlib, json, os, signal, subprocess, sys, tempfile, time。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 28-28: Top-level expr logic / 顶层 expr 逻辑
```python
sys.path.insert(0, os.path.dirname(__file__))
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 29-29: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
from warmup_deep_gemm import get_architecture_key, get_config_json
```
**EN:** This block loads warmup_deep_gemm. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 warmup_deep_gemm。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 31-34: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
MARKER_DIR = os.path.join(os.path.expanduser("~"), ".cache", "sglang", "warmup_markers")
HEALTH_POLL_INTERVAL = 10  # seconds between health checks
SERVER_STARTUP_TIMEOUT = 900  # 15 min max to wait for server ready
DEFAULT_PORT = 39876
```
**EN:** This section defines MARKER_DIR, HEALTH_POLL_INTERVAL, SERVER_STARTUP_TIMEOUT, DEFAULT_PORT, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 MARKER_DIR, HEALTH_POLL_INTERVAL, SERVER_STARTUP_TIMEOUT, DEFAULT_PORT，用于把可复用的默认值集中在模块顶部。

### Lines 37-52: Defines the `get_version_key` routine / 定义 `get_version_key` 例程
```python
def get_version_key():
    """Hash of Python + Triton + PyTorch versions to invalidate markers on upgrades."""
    parts = [sys.version]
    try:
        import triton

        parts.append(f"triton={triton.__version__}")
    except ImportError:
        parts.append("triton=none")
    try:
        import torch

        parts.append(f"torch={torch.__version__}")
    except ImportError:
        parts.append("torch=none")
    return hashlib.sha256("|".join(parts).encode()).hexdigest()[:12]
```
**EN:** This block defines `get_version_key`. Hash of Python + Triton + PyTorch versions to invalidate markers on upgrades. It takes no explicit parameters at the top level. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `get_version_key`。其文档字符串说明了该逻辑的预期职责。它在顶层定义中不接收显式参数。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 55-61: Defines the `get_marker_path` routine / 定义 `get_marker_path` 例程
```python
def get_marker_path(model, tp):
    """Get the marker file path for a model:tp pair."""
    version_key = get_version_key()
    safe_model = model.replace("/", "--")
    return os.path.join(
        MARKER_DIR, f"server_warmup_{safe_model}_tp{tp}_{version_key}.done"
    )
```
**EN:** This block defines `get_marker_path`. Get the marker file path for a model:tp pair. It accepts 2 parameter(s): model, tp. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `get_marker_path`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：model, tp。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 64-67: Defines the `check_marker` routine / 定义 `check_marker` 例程
```python
def check_marker(model, tp):
    """Check if warmup marker exists (node already warm)."""
    marker = get_marker_path(model, tp)
    return os.path.exists(marker)
```
**EN:** This block defines `check_marker`. Check if warmup marker exists (node already warm). It accepts 2 parameter(s): model, tp. Internally it checks file existence.
**CN:** 该代码块定义了 `check_marker`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：model, tp。内部会检查文件是否存在。

### Lines 70-84: Defines the `write_marker` routine / 定义 `write_marker` 例程
```python
def write_marker(model, tp):
    """Write warmup marker after successful warmup."""
    marker = get_marker_path(model, tp)
    os.makedirs(os.path.dirname(marker), exist_ok=True)
    Path(marker).write_text(
        json.dumps(
            {
                "model": model,
                "tp": tp,
                "version_key": get_version_key(),
                "timestamp": time.time(),
            }
        )
    )
    print(f"  Wrote marker: {marker}")
```
**EN:** This block defines `write_marker`. Write warmup marker after successful warmup. It accepts 2 parameter(s): model, tp. Internally it touches filesystem paths, emits status messages.
**CN:** 该代码块定义了 `write_marker`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：model, tp。内部会处理文件系统路径、输出状态信息。

### Lines 87-123: Defines the `kill_server` routine / 定义 `kill_server` 例程
```python
def kill_server(proc):
    """Kill server process tree."""
    if proc.poll() is None:
        try:
            os.killpg(os.getpgid(proc.pid), signal.SIGTERM)
        except (ProcessLookupError, OSError):
            pass
        try:
            proc.wait(timeout=15)
        except subprocess.TimeoutExpired:
            try:
                os.killpg(os.getpgid(proc.pid), signal.SIGKILL)
            except (ProcessLookupError, OSError):
                pass
            try:
                proc.wait(timeout=5)
            except subprocess.TimeoutExpired:
                pass

    # sglang's scheduler_TP* and detokenizer workers spawn through
    # multiprocessing with their own session/process group, so they escape
    # killpg on launch_server and stay alive holding GPU memory after a
    # readiness-timeout or unclean exit. Kill any survivors by name so the
    # next model (or the next CI step) starts with empty GPUs.
    for pattern in ("sglang::scheduler", "sglang::detokenizer"):
        try:
            subprocess.run(
                ["pkill", "-9", "-f", pattern],
                timeout=5,
                check=False,
                stdout=subprocess.DEVNULL,
                stderr=subprocess.DEVNULL,
            )
        except (FileNotFoundError, subprocess.TimeoutExpired):
            pass
    # Let the driver release device memory before the caller measures it.
    time.sleep(2)
```
**EN:** This block defines `kill_server`. Kill server process tree. It accepts 1 parameter(s): proc. Internally it executes external commands.
**CN:** 该代码块定义了 `kill_server`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：proc。内部会执行外部命令。

### Lines 126-142: Defines the `wait_for_server` routine / 定义 `wait_for_server` 例程
```python
def wait_for_server(base_url, proc, timeout):
    """Poll /health_generate until server is ready or timeout."""
    import requests

    start = time.time()
    while time.time() - start < timeout:
        ret = proc.poll()
        if ret is not None:
            return False, f"Server exited with code {ret}"
        try:
            resp = requests.get(f"{base_url}/health_generate", timeout=5)
            if resp.status_code == 200:
                return True, None
        except requests.RequestException:
            pass
        time.sleep(HEALTH_POLL_INTERVAL)
    return False, "Timed out waiting for server"
```
**EN:** This block defines `wait_for_server`. Poll /health_generate until server is ready or timeout. It accepts 3 parameter(s): base_url, proc, timeout. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `wait_for_server`。其文档字符串说明了该逻辑的预期职责。它接收 3 个参数：base_url, proc, timeout。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 145-163: Defines the `send_generate_request` routine / 定义 `send_generate_request` 例程
```python
def send_generate_request(base_url):
    """Send one /generate request to exercise the full inference path."""
    import requests

    payload = {
        "input_ids": [0, 1, 2, 3],
        "sampling_params": {
            "max_new_tokens": 8,
            "temperature": 0,
        },
    }
    try:
        resp = requests.post(f"{base_url}/generate", json=payload, timeout=120)
        if resp.status_code == 200:
            print("  Generate request succeeded")
        else:
            print(f"  Warning: generate request returned {resp.status_code}")
    except requests.RequestException as e:
        print(f"  Warning: generate request failed: {e}")
```
**EN:** This block defines `send_generate_request`. Send one /generate request to exercise the full inference path. It accepts 1 parameter(s): base_url. Internally it emits status messages.
**CN:** 该代码块定义了 `send_generate_request`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：base_url。内部会输出状态信息。

### Lines 166-239: Defines the `warmup_one_model` routine / 定义 `warmup_one_model` 例程
```python
def warmup_one_model(model, tp, port):
    """Launch server, wait for ready, send one request, then kill."""
    base_url = f"http://127.0.0.1:{port}"

    cmd = [
        sys.executable,
        "-m",
        "sglang.launch_server",
        "--model-path",
        model,
        "--tp",
        str(tp),
        "--host",
        "127.0.0.1",
        "--port",
        str(port),
        "--trust-remote-code",
        "--model-loader-extra-config",
        '{"enable_multithread_load": true, "num_threads": 64}',
    ]

    # Use a temp file for server output to avoid pipe buffer deadlock
    # (server logs can exceed the 64KB pipe buffer during CUDA graph capture)
    log_file = tempfile.NamedTemporaryFile(
        mode="w", prefix="warmup_server_", suffix=".log", delete=False
    )
    log_path = log_file.name

    print(f"  Launching server: {' '.join(cmd)}")
    print(f"  Server log: {log_path}")
    proc = subprocess.Popen(
        cmd,
        stdout=log_file,
        stderr=subprocess.STDOUT,
        preexec_fn=os.setsid,
    )

    try:
        # Wait for server to be ready (includes CUDA graph capture)
        print(
            f"  Waiting for server (timeout={SERVER_STARTUP_TIMEOUT}s, "
            f"polling every {HEALTH_POLL_INTERVAL}s)..."
        )
        ok, err = wait_for_server(base_url, proc, SERVER_STARTUP_TIMEOUT)
        if not ok:
            print(f"  Warning: server not ready: {err}")
            return False

        print("  Server ready, sending generate request...")
        send_generate_request(base_url)
        return True

    finally:
        # Surface the tail of the server log so CI captures validation
        # messages, exceptions, and warmup progress (the launch_server
        # subprocess writes stdout/stderr to the tempfile, not our stdout).
        try:
            log_file.flush()
            with open(log_path) as f:
                lines = f.readlines()
            print(f"  --- server log tail ({len(lines)} lines, last 30) ---")
            for line in lines[-30:]:
                print(f"    | {line.rstrip()}")
            print("  --- end server log ---")
        except Exception:
            pass

        print("  Killing server...")
        kill_server(proc)
        log_file.close()
        try:
            os.unlink(log_path)
        except OSError:
            pass
```
**EN:** This block defines `warmup_one_model`. Launch server, wait for ready, send one request, then kill. It accepts 3 parameter(s): model, tp, port. Internally it reads or writes files, emits status messages.
**CN:** 该代码块定义了 `warmup_one_model`。其文档字符串说明了该逻辑的预期职责。它接收 3 个参数：model, tp, port。内部会读写文件、输出状态信息。

### Lines 242-331: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    if len(sys.argv) < 2 or sys.argv[1] in ("-h", "--help"):
        print("Usage: warmup_server.py model1:tp1 [model2:tp2 ...]")
        print(
            "\nLaunches full servers with CUDA graphs enabled to pre-warm"
            " Triton autotuning."
        )
        print("Skips instantly on warm nodes (marker file exists).")
        sys.exit(0)

    # Parse model:tp pairs
    model_tp_pairs = []
    for arg in sys.argv[1:]:
        if ":" not in arg:
            print(f"Error: expected model:tp format, got '{arg}'")
            sys.exit(1)
        model, tp_str = arg.rsplit(":", 1)
        model_tp_pairs.append((model, int(tp_str)))

    print(f"=== Server CUDA Graph Warmup ({len(model_tp_pairs)} model(s)) ===")
    print(f"    Marker dir: {MARKER_DIR}")
    print(f"    Version key: {get_version_key()}\n")

    # Deduplicate by architecture and check markers
    seen_keys = {}
    to_warmup = []

    for model, tp in model_tp_pairs:
        # Check marker first (fast path)
        if check_marker(model, tp):
            print(f"  SKIP   {model} (tp={tp}): already warm (marker exists)")
            continue

        # Architecture dedup
        config = get_config_json(model)
        if config is not None:
            key = get_architecture_key(config, tp)
            if key in seen_keys:
                print(
                    f"  DEDUP  {model} (tp={tp}): same architecture as {seen_keys[key]}"
                )
                continue
            seen_keys[key] = model

        to_warmup.append((model, tp))
        print(f"  QUEUE  {model} (tp={tp}): needs warmup")

    if not to_warmup:
        print("\nAll models already warm. Done.")
        return

    print(f"\n{len(to_warmup)} model(s) to warm up.\n")

    port = DEFAULT_PORT
    for i, (model, tp) in enumerate(to_warmup, 1):
        print(f"\n{'=' * 60}")
        print(f"[{i}/{len(to_warmup)}] {model} (tp={tp})")
        print(f"{'=' * 60}")

        t0 = time.time()
        success = warmup_one_model(model, tp, port)
        elapsed = time.time() - t0

        if success:
            print(f"  Completed in {elapsed:.0f}s")
            write_marker(model, tp)
            # Also write markers for dedup'd models that share this architecture
            config = get_config_json(model)
            if config is not None:
                key = get_architecture_key(config, tp)
                for other_model, other_tp in model_tp_pairs:
                    if (other_model, other_tp) == (model, tp):
                        continue
                    other_config = get_config_json(other_model)
                    if other_config is not None:
                        other_key = get_architecture_key(other_config, other_tp)
                        if other_key == key and not check_marker(other_model, other_tp):
                            write_marker(other_model, other_tp)
                            print(
                                f"  Also marked {other_model} (tp={other_tp}) as warm (same arch)"
                            )
        else:
            print(
                f"  Warning: warmup failed after {elapsed:.0f}s (non-fatal, tests will still work)"
            )

        # Use a different port for the next model to avoid bind conflicts
        port += 100

    print("\nServer CUDA graph warmup complete.")
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it emits status messages, controls process exit status.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会输出状态信息、控制进程退出状态。

### Lines 334-335: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **Subprocess orchestration** / 子进程编排
- **Filesystem coordination** / 文件系统协同
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Structured data handling** / 结构化数据处理

## Dependencies / 依赖关系
- **Standard library / 标准库**: `hashlib`, `json`, `os`, `pathlib`, `signal`, `subprocess`, `sys`, `tempfile`, `time`
- **Third-party modules / 第三方模块**: `requests`, `torch`, `triton`, `warmup_deep_gemm`
