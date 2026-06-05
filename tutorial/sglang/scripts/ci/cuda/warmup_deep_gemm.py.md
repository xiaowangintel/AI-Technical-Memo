# warmup_deep_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/cuda/warmup_deep_gemm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `warmup_deep_gemm` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `warmup_deep_gemm` 流程，主要负责CI 编排、测试执行。它属于 `cuda` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Lightweight DeepGEMM JIT compilation warmup without loading model weights.

Reads model config.json from HF cache to derive kernel shapes, then compiles
DeepGEMM kernels directly. This avoids the expensive model weight loading step
that the full `sglang.compile_deep_gemm` requires.

Supports DeepSeek V2/V3 family models. Falls back to `sglang.compile_deep_gemm`
for unsupported architectures.

Usage:
    python3 scripts/ci/cuda/warmup_deep_gemm.py \
        deepseek-ai/DeepSeek-V3-0324:8 \
        deepseek-ai/DeepSeek-V3.2:8
"""
```
**EN:** Lightweight DeepGEMM JIT compilation warmup without loading model weights.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 17-27: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import hashlib
import json
import os
import signal
import subprocess
import sys
import threading
import time
from math import ceil
from pathlib import Path
from typing import Dict, List
```
**EN:** This block loads hashlib, json, os, signal, subprocess, sys, threading, time. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 hashlib, json, os, signal, subprocess, sys, threading, time。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 31-77: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
MARKER_DIR = os.path.join(os.path.expanduser("~"), ".cache", "sglang", "warmup_markers")

# Outer cap for stuck fallback subprocesses; CRASH_MARKERS abort sooner.
FALLBACK_TIMEOUT_SEC = 600

# Per-model launch flags forwarded to `sglang.compile_deep_gemm`. DeepGEMM
# cache key includes per-rank N/K (depends on tp/dp/ep) — must match each
# model's `other_args` in test/registered/ or warmed shapes won't be hit.
FALLBACK_ARGS: Dict[str, List[str]] = {
    "deepseek-ai/DeepSeek-V3.2": ["--dp", "8", "--enable-dp-attention"],
    "zai-org/GLM-5-FP8": ["--dp", "8", "--enable-dp-attention"],
    "XiaomiMiMo/MiMo-V2-Flash": [
        "--dp",
        "2",
        "--enable-dp-attention",
        "--attention-backend",
        "fa3",
    ],
    # --mm-enable-dp-encoder is required: without it DP0 runs the vision
    # encoder alone and DP1 deadlocks at the next collective.
    "XiaomiMiMo/MiMo-V2.5": [
        "--dp",
        "2",
        "--enable-dp-attention",
        "--mm-enable-dp-encoder",
        "--attention-backend",
        "fa3",
        "--mm-attention-backend",
        "fa3",
    ],
}

# compile_deep_gemm polls /v1/models for the full timeout even after a TP rank
# dies; the watcher uses these to kill the group within seconds instead.
CRASH_MARKERS = (
    "Scheduler hit an exception",
    "Received sigquit from a child",
)

# Configure DeepGEMM cache before importing deep_gemm
os.environ["DG_JIT_CACHE_DIR"] = os.getenv(
    "SGLANG_DG_CACHE_DIR",
    os.path.join(os.path.expanduser("~"), ".cache", "deep_gemm"),
)
os.environ["DG_JIT_USE_NVRTC"] = os.getenv("SGL_DG_USE_NVRTC", "0")

BLOCK_SIZE = 128
```
**EN:** This section defines MARKER_DIR, FALLBACK_TIMEOUT_SEC, CRASH_MARKERS, BLOCK_SIZE, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 MARKER_DIR, FALLBACK_TIMEOUT_SEC, CRASH_MARKERS, BLOCK_SIZE，用于把可复用的默认值集中在模块顶部。

### Lines 80-100: Defines the `get_config_json` routine / 定义 `get_config_json` 例程
```python
def get_config_json(model_name):
    """Load config.json for a cached model from HF cache."""
    cache_dir = os.environ.get(
        "HF_HOME", os.path.join(os.path.expanduser("~"), ".cache", "huggingface")
    )
    hub_dir = os.path.join(cache_dir, "hub")
    safe_name = "models--" + model_name.replace("/", "--")
    snapshots_dir = os.path.join(hub_dir, safe_name, "snapshots")

    if not os.path.isdir(snapshots_dir):
        return None

    snapshots = sorted(
        Path(snapshots_dir).iterdir(), key=lambda p: p.stat().st_mtime, reverse=True
    )
    for snapshot in snapshots:
        config_path = snapshot / "config.json"
        if config_path.exists():
            with open(config_path) as f:
                return json.load(f)
    return None
```
**EN:** This block defines `get_config_json`. Load config.json for a cached model from HF cache. It accepts 1 parameter(s): model_name. Internally it touches filesystem paths, checks file existence, loads structured data, reads or writes files.
**CN:** 该代码块定义了 `get_config_json`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：model_name。内部会处理文件系统路径、检查文件是否存在、加载结构化数据、读写文件。

### Lines 103-109: Defines the `is_deepseek_v2v3` routine / 定义 `is_deepseek_v2v3` 例程
```python
def is_deepseek_v2v3(config):
    """Check if a model is from the DeepSeek V2/V3 family."""
    architectures = config.get("architectures", [])
    model_type = config.get("model_type", "")
    return any(
        "DeepseekV2" in a or "DeepseekV3" in a for a in architectures
    ) or model_type in ("deepseek_v2", "deepseek_v3")
```
**EN:** This block defines `is_deepseek_v2v3`. Check if a model is from the DeepSeek V2/V3 family. It accepts 1 parameter(s): config. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `is_deepseek_v2v3`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：config。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 112-166: Defines the `compute_deepseek_v2v3_shapes` routine / 定义 `compute_deepseek_v2v3_shapes` 例程
```python
def compute_deepseek_v2v3_shapes(config, tp):
    """Compute all DeepGEMM (kernel_type, N, K, num_groups) for DeepSeek V2/V3.

    Shape derivation based on:
    - MoE: python/sglang/srt/layers/moe/fused_moe_triton/layer.py
    - MLA: python/sglang/srt/models/deepseek_v2.py
    - FP8: python/sglang/srt/layers/quantization/fp8_kernel.py
    """
    shapes = []

    hidden_size = config["hidden_size"]
    num_attention_heads = config.get("num_attention_heads", 128)
    kv_lora_rank = config.get("kv_lora_rank", 512)
    qk_nope_head_dim = config.get("qk_nope_head_dim", 128)
    v_head_dim = config.get("v_head_dim", 128)
    n_routed_experts = config.get("n_routed_experts", 0)
    n_shared_experts = config.get("n_shared_experts", 0)
    moe_intermediate_size = config.get("moe_intermediate_size", 0)

    num_local_heads = num_attention_heads // tp
    # Shared expert fusion is enabled by default (disable_shared_experts_fusion=False)
    # so the FusedMoE weight tensor includes shared experts
    num_local_experts = n_routed_experts + n_shared_experts

    # --- MoE expert GEMM shapes ---
    # FusedMoE shards intermediate_size across TP ranks (column parallel for gate/up,
    # row parallel for down). All experts are replicated on each TP rank.
    if n_routed_experts > 0 and moe_intermediate_size > 0:
        moe_inter_per_tp = moe_intermediate_size // tp

        # Gate-Up projection: (tokens, hidden_size) @ (experts, 2*inter_per_tp, hidden_size)^T
        # Both masked and contiguous paths are used at runtime
        shapes.append(("MASKED", moe_inter_per_tp * 2, hidden_size, num_local_experts))
        shapes.append(("CONTIG", moe_inter_per_tp * 2, hidden_size, num_local_experts))

        # Down projection: (tokens, inter_per_tp) @ (experts, hidden_size, inter_per_tp)^T
        shapes.append(("MASKED", hidden_size, moe_inter_per_tp, num_local_experts))
        shapes.append(("CONTIG", hidden_size, moe_inter_per_tp, num_local_experts))

    # --- MLA attention GEMM shapes (masked grouped GEMM) ---
    if kv_lora_rank > 0 and num_local_heads > 0:
        # Q_nope -> compressed K: (heads, m, qk_nope_head_dim) @ (heads, kv_lora_rank, qk_nope_head_dim)^T
        shapes.append(("MASKED", kv_lora_rank, qk_nope_head_dim, num_local_heads))

        # Attention output -> V: (heads, m, kv_lora_rank) @ (heads, v_head_dim, kv_lora_rank)^T
        shapes.append(("MASKED", v_head_dim, kv_lora_rank, num_local_heads))

    # --- kv_b_proj (non-grouped GEMM via FP8 kernel) ---
    # ColumnParallelLinear(kv_lora_rank, num_heads * (qk_nope + v_head_dim))
    # Per TP rank: N = num_local_heads * (qk_nope_head_dim + v_head_dim)
    if kv_lora_rank > 0 and num_local_heads > 0:
        kv_b_proj_n = num_local_heads * (qk_nope_head_dim + v_head_dim)
        shapes.append(("NORMAL", kv_b_proj_n, kv_lora_rank, 1))

    return shapes
```
**EN:** This block defines `compute_deepseek_v2v3_shapes`. Compute all DeepGEMM (kernel_type, N, K, num_groups) for DeepSeek V2/V3. It accepts 2 parameter(s): config, tp. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `compute_deepseek_v2v3_shapes`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：config, tp。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 169-184: Defines the `get_architecture_key` routine / 定义 `get_architecture_key` 例程
```python
def get_architecture_key(config, tp):
    """Key for dedup: models with same key share DeepGEMM kernels."""
    if config is None:
        return None
    fields = [
        config.get("hidden_size", 0),
        config.get("moe_intermediate_size", 0),
        config.get("n_routed_experts", 0),
        config.get("n_shared_experts", 0),
        config.get("num_attention_heads", 0),
        config.get("kv_lora_rank", 0),
        config.get("qk_nope_head_dim", 0),
        config.get("v_head_dim", 0),
        tp,
    ]
    return tuple(fields)
```
**EN:** This block defines `get_architecture_key`. Key for dedup: models with same key share DeepGEMM kernels. It accepts 2 parameter(s): config, tp. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `get_architecture_key`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：config, tp。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 187-206: Defines the `compute_m_list` routine / 定义 `compute_m_list` 例程
```python
def compute_m_list(fast_warmup=False, chunked_prefill_size=8192):
    """Compute the list of M values to compile (matches compile_utils.py logic)."""
    m_list = []
    if fast_warmup:
        m_list += list(range(1, 1025))
        next_m, sample_step = 1024, 2
        max_prefill_bs = min(chunked_prefill_size, 32 * 1024)
        while next_m < max_prefill_bs:
            m_list += list(range(next_m, 2 * next_m, sample_step))
            next_m *= 2
            sample_step *= 2
        m_list.append(max_prefill_bs)
        m_list = sorted(set(m_list))
    else:
        m_max = 16 * 1024
        if chunked_prefill_size > 8192:
            m_max = chunked_prefill_size * 2
        m_max = min(128 * 1024, m_max)
        m_list = list(range(1, m_max + 1))
    return m_list
```
**EN:** This block defines `compute_m_list`. Compute the list of M values to compile (matches compile_utils.py logic). It accepts 2 parameter(s): fast_warmup, chunked_prefill_size. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `compute_m_list`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：fast_warmup, chunked_prefill_size。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 209-217: Defines the `_empty_token_fp8` routine / 定义 `_empty_token_fp8` 例程
```python
def _empty_token_fp8(size):
    """Create FP8 token tensor + per-block scale tensor."""
    import torch

    *dims, k = size
    return (
        torch.empty(size, device="cuda", dtype=torch.float8_e4m3fn),
        torch.empty((*dims, ceil(k / BLOCK_SIZE)), device="cuda", dtype=torch.float32),
    )
```
**EN:** This block defines `_empty_token_fp8`. Create FP8 token tensor + per-block scale tensor. It accepts 1 parameter(s): size. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `_empty_token_fp8`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：size。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 220-232: Defines the `_empty_block_fp8` routine / 定义 `_empty_block_fp8` 例程
```python
def _empty_block_fp8(size):
    """Create FP8 block tensor + per-block scale tensor."""
    import torch

    *dims, n, k = size
    return (
        torch.empty(size, device="cuda", dtype=torch.float8_e4m3fn),
        torch.empty(
            (*dims, ceil(n / BLOCK_SIZE), ceil(k / BLOCK_SIZE)),
            device="cuda",
            dtype=torch.float32,
        ),
    )
```
**EN:** This block defines `_empty_block_fp8`. Create FP8 block tensor + per-block scale tensor. It accepts 1 parameter(s): size. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `_empty_block_fp8`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：size。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 235-249: Defines the `get_memory_requirement` routine / 定义 `get_memory_requirement` 例程
```python
def get_memory_requirement(kernel_type, max_m, n, k, num_groups):
    """Estimate GPU memory needed in GB for compilation buffers."""
    _GB = 1 << 30
    if kernel_type == "NORMAL":
        return (max_m * k + n * k + max_m * n * 2) / _GB
    elif kernel_type == "CONTIG":
        return (max_m * k + num_groups * n * k + max_m * 4 + max_m * n * 2) / _GB
    elif kernel_type == "MASKED":
        return (
            num_groups * max_m * k
            + num_groups * n * k
            + num_groups * 4
            + num_groups * max_m * n * 2
        ) / _GB
    return 0
```
**EN:** This block defines `get_memory_requirement`. Estimate GPU memory needed in GB for compilation buffers. It accepts 5 parameter(s): kernel_type, max_m, n, k, num_groups. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `get_memory_requirement`。其文档字符串说明了该逻辑的预期职责。它接收 5 个参数：kernel_type, max_m, n, k, num_groups。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 252-329: Defines the `compile_one_shape` routine / 定义 `compile_one_shape` 例程
```python
def compile_one_shape(kernel_type, n, k, num_groups, m_list):
    """Compile DeepGEMM kernels for one (kernel_type, N, K, num_groups) shape."""
    import deep_gemm
    import torch
    from tqdm import tqdm

    # Filter M list for contiguous layout alignment
    if kernel_type == "CONTIG":
        m_alignment = deep_gemm.get_mk_alignment_for_contiguous_layout()
        m_list = sorted(set(m for m in m_list if m % m_alignment == 0))

    if not m_list:
        return

    max_m = max(m_list)

    # Reduce max_m if not enough GPU memory
    mem_free = torch.cuda.mem_get_info()[0] / (1 << 30)
    mem_required = get_memory_requirement(kernel_type, max_m, n, k, num_groups)
    if mem_required > mem_free:
        while (
            get_memory_requirement(kernel_type, max_m, n, k, num_groups) > mem_free
            and max_m > 4096
        ):
            max_m //= 2
        print(
            f"  Memory {mem_free:.1f}GB < required {mem_required:.1f}GB, "
            f"reducing max_m to {max_m}"
        )
        m_list = [m for m in m_list if m <= max_m]

    get_compile_mode = getattr(deep_gemm, "get_compile_mode", None)
    set_compile_mode = getattr(deep_gemm, "set_compile_mode", None)
    old_mode = get_compile_mode() if get_compile_mode is not None else None
    if set_compile_mode is not None:
        set_compile_mode(1)
    try:
        if kernel_type == "NORMAL":
            lhs_q, lhs_s = _empty_token_fp8((max_m, k))
            rhs_q, rhs_s = _empty_block_fp8((n, k))
            out = torch.empty((max_m, n), device="cuda", dtype=torch.bfloat16)
            for m in tqdm(m_list, desc=f"  NORMAL N={n} K={k}"):
                deep_gemm.fp8_gemm_nt((lhs_q[:m], lhs_s[:m]), (rhs_q, rhs_s), out[:m])

        elif kernel_type == "CONTIG":
            lhs_q, lhs_s = _empty_token_fp8((max_m, k))
            rhs_q, rhs_s = _empty_block_fp8((num_groups, n, k))
            m_indices = torch.zeros((max_m,), device="cuda", dtype=torch.int32)
            out = torch.empty((max_m, n), device="cuda", dtype=torch.bfloat16)
            for m in tqdm(m_list, desc=f"  CONTIG N={n} K={k} G={num_groups}"):
                deep_gemm.m_grouped_fp8_gemm_nt_contiguous(
                    (lhs_q[:m], lhs_s[:m]),
                    (rhs_q, rhs_s),
                    out[:m],
                    m_indices[:m],
                )

        elif kernel_type == "MASKED":
            lhs_q, lhs_s = _empty_token_fp8((num_groups, max_m, k))
            rhs_q, rhs_s = _empty_block_fp8((num_groups, n, k))
            masked_m = torch.zeros((num_groups,), device="cuda", dtype=torch.int32)
            out = torch.empty(
                (num_groups, max_m, n), device="cuda", dtype=torch.bfloat16
            )
            for m in tqdm(m_list, desc=f"  MASKED N={n} K={k} G={num_groups}"):
                deep_gemm.fp8_m_grouped_gemm_nt_masked(
                    (lhs_q, lhs_s),
                    (rhs_q, rhs_s),
                    out,
                    masked_m=masked_m,
                    expected_m=m,
                )
    finally:
        if set_compile_mode is not None and old_mode is not None:
            set_compile_mode(old_mode)

    torch.cuda.current_stream().synchronize()
    torch.cuda.empty_cache()
```
**EN:** This block defines `compile_one_shape`. Compile DeepGEMM kernels for one (kernel_type, N, K, num_groups) shape. It accepts 5 parameter(s): kernel_type, n, k, num_groups, m_list. Internally it emits status messages.
**CN:** 该代码块定义了 `compile_one_shape`。其文档字符串说明了该逻辑的预期职责。它接收 5 个参数：kernel_type, n, k, num_groups, m_list。内部会输出状态信息。

### Lines 332-339: Defines the `compile_shapes_lightweight` routine / 定义 `compile_shapes_lightweight` 例程
```python
def compile_shapes_lightweight(shapes, m_list):
    """Compile all DeepGEMM shapes directly (no model loading)."""
    for i, (kernel_type, n, k, num_groups) in enumerate(shapes, 1):
        print(f"\n[{i}/{len(shapes)}] {kernel_type} N={n} K={k} G={num_groups}")
        t0 = time.time()
        compile_one_shape(kernel_type, n, k, num_groups, m_list)
        elapsed = time.time() - t0
        print(f"  Done in {elapsed:.1f}s")
```
**EN:** This block defines `compile_shapes_lightweight`. Compile all DeepGEMM shapes directly (no model loading). It accepts 2 parameter(s): shapes, m_list. Internally it emits status messages.
**CN:** 该代码块定义了 `compile_shapes_lightweight`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：shapes, m_list。内部会输出状态信息。

### Lines 342-358: Defines the `_kill_pg_and_wait` routine / 定义 `_kill_pg_and_wait` 例程
```python
def _kill_pg_and_wait(proc):
    """SIGTERM the subprocess's process group, escalate to SIGKILL if needed."""
    try:
        os.killpg(os.getpgid(proc.pid), signal.SIGTERM)
    except (ProcessLookupError, OSError):
        pass
    try:
        return proc.wait(timeout=10)
    except subprocess.TimeoutExpired:
        try:
            os.killpg(os.getpgid(proc.pid), signal.SIGKILL)
        except (ProcessLookupError, OSError):
            pass
        try:
            return proc.wait(timeout=5)
        except subprocess.TimeoutExpired:
            return -1
```
**EN:** This block defines `_kill_pg_and_wait`. SIGTERM the subprocess's process group, escalate to SIGKILL if needed. It accepts 1 parameter(s): proc. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `_kill_pg_and_wait`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：proc。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 361-376: Defines the `get_version_key` routine / 定义 `get_version_key` 例程
```python
def get_version_key():
    """Hash of Python + Triton + PyTorch versions; invalidates markers on upgrade."""
    parts = [sys.version]
    try:
        import triton  # noqa: WPS433

        parts.append(f"triton={triton.__version__}")
    except ImportError:
        parts.append("triton=none")
    try:
        import torch  # noqa: WPS433

        parts.append(f"torch={torch.__version__}")
    except ImportError:
        parts.append("torch=none")
    return hashlib.sha256("|".join(parts).encode()).hexdigest()[:12]
```
**EN:** This block defines `get_version_key`. Hash of Python + Triton + PyTorch versions; invalidates markers on upgrade. It takes no explicit parameters at the top level. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `get_version_key`。其文档字符串说明了该逻辑的预期职责。它在顶层定义中不接收显式参数。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 379-387: Defines the `get_fallback_marker_path` routine / 定义 `get_fallback_marker_path` 例程
```python
def get_fallback_marker_path(model, tp, extra_args):
    """Marker path for one (model, tp, extra_args) fallback invocation."""
    args_blob = json.dumps(list(extra_args))
    args_hash = hashlib.md5(args_blob.encode()).hexdigest()[:8]
    safe_model = model.replace("/", "--")
    return os.path.join(
        MARKER_DIR,
        f"deepgemm_fallback_{safe_model}_tp{tp}_{args_hash}_{get_version_key()}.done",
    )
```
**EN:** This block defines `get_fallback_marker_path`. Marker path for one (model, tp, extra_args) fallback invocation. It accepts 3 parameter(s): model, tp, extra_args. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `get_fallback_marker_path`。其文档字符串说明了该逻辑的预期职责。它接收 3 个参数：model, tp, extra_args。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 390-391: Defines the `check_fallback_marker` routine / 定义 `check_fallback_marker` 例程
```python
def check_fallback_marker(model, tp, extra_args):
    return os.path.exists(get_fallback_marker_path(model, tp, extra_args))
```
**EN:** This block defines `check_fallback_marker`. It accepts 3 parameter(s): model, tp, extra_args. Internally it checks file existence.
**CN:** 该代码块定义了 `check_fallback_marker`。它接收 3 个参数：model, tp, extra_args。内部会检查文件是否存在。

### Lines 394-408: Defines the `write_fallback_marker` routine / 定义 `write_fallback_marker` 例程
```python
def write_fallback_marker(model, tp, extra_args):
    marker = get_fallback_marker_path(model, tp, extra_args)
    os.makedirs(os.path.dirname(marker), exist_ok=True)
    Path(marker).write_text(
        json.dumps(
            {
                "model": model,
                "tp": tp,
                "extra_args": list(extra_args),
                "version_key": get_version_key(),
                "timestamp": time.time(),
            }
        )
    )
    print(f"  Wrote marker: {marker}")
```
**EN:** This block defines `write_fallback_marker`. It accepts 3 parameter(s): model, tp, extra_args. Internally it touches filesystem paths, emits status messages.
**CN:** 该代码块定义了 `write_fallback_marker`。它接收 3 个参数：model, tp, extra_args。内部会处理文件系统路径、输出状态信息。

### Lines 411-486: Defines the `fallback_compile_deep_gemm` routine / 定义 `fallback_compile_deep_gemm` 例程
```python
def fallback_compile_deep_gemm(model, tp):
    """Fall back to full sglang.compile_deep_gemm (loads model weights).

    Runs in its own process group so a hung subprocess (e.g. one TP rank
    crashes and the rest deadlock on NCCL collectives) can be killed
    cleanly without leaking children. Watches subprocess output for crash
    markers so a deterministic failure aborts in seconds rather than burning
    the full FALLBACK_TIMEOUT_SEC.
    """
    extra_args = FALLBACK_ARGS.get(model, [])
    print(
        f"Falling back to full compile_deep_gemm for {model} "
        f"(tp={tp}, extra_args={extra_args})..."
    )
    cmd = [
        sys.executable,
        "-m",
        "sglang.compile_deep_gemm",
        "--model",
        model,
        "--tp",
        str(tp),
        "--trust-remote-code",
        "--model-loader-extra-config",
        '{"enable_multithread_load": true, "num_threads": 64}',
        # Cap compile_deep_gemm's own /v1/models polling loop so it gives up
        # before our outer timeout has to SIGTERM it.
        "--timeout",
        str(FALLBACK_TIMEOUT_SEC),
        *extra_args,
    ]

    crashed = threading.Event()
    proc = subprocess.Popen(
        cmd,
        preexec_fn=os.setsid,
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,
        bufsize=1,
        text=True,
    )

    def _watch():
        # Stream child output to our stdout while scanning for crash markers.
        for line in proc.stdout:
            sys.stdout.write(line)
            sys.stdout.flush()
            if not crashed.is_set() and any(m in line for m in CRASH_MARKERS):
                crashed.set()

    watcher = threading.Thread(target=_watch, daemon=True)
    watcher.start()

    deadline = time.monotonic() + FALLBACK_TIMEOUT_SEC
    while True:
        rc = proc.poll()
        if rc is not None:
            watcher.join(timeout=2)
            if rc != 0:
                print(f"Warning: fallback failed for {model} (exit code {rc})")
            return rc == 0
        if crashed.is_set():
            print(
                f"Warning: detected crash marker in {model} (tp={tp}) subprocess; "
                "killing process group and continuing."
            )
            _kill_pg_and_wait(proc)
            return False
        if time.monotonic() >= deadline:
            print(
                f"Warning: fallback timed out after {FALLBACK_TIMEOUT_SEC}s for "
                f"{model} (tp={tp}); killing process group and continuing."
            )
            _kill_pg_and_wait(proc)
            return False
        time.sleep(2)
```
**EN:** This block defines `fallback_compile_deep_gemm`. Fall back to full sglang.compile_deep_gemm (loads model weights). It accepts 2 parameter(s): model, tp. Internally it emits status messages.
**CN:** 该代码块定义了 `fallback_compile_deep_gemm`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：model, tp。内部会输出状态信息。

### Lines 489-599: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    if len(sys.argv) < 2 or sys.argv[1] in ("-h", "--help"):
        print("Usage: warmup_deep_gemm.py model1:tp1 [model2:tp2 ...]")
        print("\nDerives DeepGEMM kernel shapes from config.json without loading model")
        print(
            "weights. Falls back to full compile_deep_gemm for unknown architectures."
        )
        sys.exit(0)

    # Parse model:tp pairs
    model_tp_pairs = []
    for arg in sys.argv[1:]:
        if ":" not in arg:
            print(f"Error: expected model:tp format, got '{arg}'")
            sys.exit(1)
        model, tp_str = arg.rsplit(":", 1)
        model_tp_pairs.append((model, int(tp_str)))

    fast_warmup = os.environ.get("SGLANG_JIT_DEEPGEMM_FAST_WARMUP", "0").lower() in (
        "1",
        "true",
    )
    print(f"=== DeepGEMM Lightweight Warmup ({len(model_tp_pairs)} model(s)) ===")
    print(f"    Fast warmup: {fast_warmup}")
    print(
        f"    Cache dir: {os.environ.get('DG_JIT_CACHE_DIR', '~/.cache/deep_gemm')}\n"
    )

    # Load configs and deduplicate by architecture
    seen_keys = {}
    to_process = []  # (model, tp, config_or_None, shapes_or_None)

    for model, tp in model_tp_pairs:
        config = get_config_json(model)
        if config is None:
            print(f"  SKIP   {model} (tp={tp}): config.json not in HF cache")
            continue

        # Models with FALLBACK_ARGS launch with extra dp/ep/dp-attention flags
        # that change per-rank N/K. The lightweight path doesn't model those —
        # it computes attention shapes assuming TP-only sharding — so dedup'ing
        # such a model to a no-override DeepSeek V2/V3 lookalike (e.g. V3.2 →
        # V3-0324) silently picks the wrong attention shapes for the test's
        # actual launch config. Force these through fallback so the populated
        # cache matches the real test.
        has_fallback_override = model in FALLBACK_ARGS

        key = get_architecture_key(config, tp)
        if key in seen_keys and not has_fallback_override:
            print(f"  DEDUP  {model} (tp={tp}): same shapes as {seen_keys[key]}")
            continue

        if is_deepseek_v2v3(config) and not has_fallback_override:
            shapes = compute_deepseek_v2v3_shapes(config, tp)
            seen_keys[key] = model
            to_process.append((model, tp, config, shapes))
            print(f"  FOUND  {model} (tp={tp}): {len(shapes)} DeepGEMM shape(s)")
        else:
            seen_keys[key] = model
            to_process.append((model, tp, config, None))
            if has_fallback_override:
                print(
                    f"  FOUND  {model} (tp={tp}): forced fallback (extra args "
                    f"{FALLBACK_ARGS[model]})"
                )
            else:
                arch = config.get("architectures", ["unknown"])
                print(
                    f"  FOUND  {model} (tp={tp}): unknown arch {arch}, "
                    "will use fallback"
                )

    if not to_process:
        print("\nNo models to process. Done.")
        return

    m_list = compute_m_list(fast_warmup=fast_warmup)
    print(f"\nM list: {len(m_list)} values (range {min(m_list)}-{max(m_list)})")

    for model, tp, config, shapes in to_process:
        print(f"\n{'=' * 60}")
        print(f"Model: {model} (tp={tp})")
        print(f"{'=' * 60}")

        if shapes is None:
            # Fallback path: full sglang.compile_deep_gemm with the test's launch
            # flags. Loading model weights inside that subprocess is the dominant
            # cost (45-170s/model) and dwarfs the actual DeepGEMM compile, so we
            # skip the whole fallback when the marker says we already populated
            # this cache. Cache is keyed on (model, tp, extra_args, version_key).
            extra_args = FALLBACK_ARGS.get(model, [])
            if check_fallback_marker(model, tp, extra_args):
                print(
                    f"  SKIP fallback (warm marker found): {model} (tp={tp}, "
                    f"extra_args={extra_args})"
                )
                continue
            if fallback_compile_deep_gemm(model, tp):
                write_fallback_marker(model, tp, extra_args)
            continue

        # Print shape summary
        for kernel_type, n, k, num_groups in shapes:
            print(f"  {kernel_type:8s} N={n:<6d} K={k:<6d} G={num_groups}")

        t0 = time.time()
        compile_shapes_lightweight(shapes, m_list)
        elapsed = time.time() - t0
        print(f"\nCompleted {model} in {elapsed:.1f}s")

    print("\nDeepGEMM lightweight warmup complete.")
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it emits status messages, controls process exit status.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会输出状态信息、控制进程退出状态。

### Lines 602-603: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
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
- **Standard library / 标准库**: `hashlib`, `json`, `math`, `os`, `pathlib`, `signal`, `subprocess`, `sys`, `threading`, `time`, `typing`
- **Third-party modules / 第三方模块**: `deep_gemm`, `torch`, `tqdm`, `triton`
