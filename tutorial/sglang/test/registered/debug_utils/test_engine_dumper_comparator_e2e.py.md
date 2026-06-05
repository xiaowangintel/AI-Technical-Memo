# test_engine_dumper_comparator_e2e.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/test_engine_dumper_comparator_e2e.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on debug utils engine dumper comparator e2e in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 debug utils engine dumper comparator e2e 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Document the module
```python
"""E2E test: source patcher + dumper + comparator on SGLang server.

Patches Qwen3MoeDecoderLayer.forward (and related methods) to insert
dumper.dump() calls at 7 points, launches servers with Qwen3-30B-A3B
(MOE model), runs inference, verifies patched dump fields exist, then
runs comparator to verify numerical consistency.

Test cases:
- test_patch_dump_and_compare: TP=2 baseline vs TP=4 target
- test_dp_attention: TP=2 baseline vs TP=2+DP=2+dp-attention target

The dumper.apply_source_patches() auto-injects ``from ... import dumper``
so the YAML only needs ``dumper.dump(...)`` calls.
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 16-24: Import dependencies
```python
import os
import subprocess
import sys
import tempfile
from pathlib import Path
from typing import Optional

import pytest
import requests
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 26-28: Define module constants
```python
pytestmark = pytest.mark.filterwarnings(
    "ignore:Unknown config option. asyncio_mode:pytest.PytestConfigWarning",
)
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 30-36: Import dependencies
```python
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 38-44: Register CI metadata
```python
register_cuda_ci(est_time=300, suite="nightly-4-gpu", nightly=True)
register_amd_ci(
    est_time=300,
    suite="nightly-amd-4-gpu",
    nightly=True,
    disabled="TP=2 vs TP=4 numerical mismatch on AMD (comparator fails tolerance check)",
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 46-103: Define module constants (part 1)
```python
MODEL = "Qwen/Qwen3-30B-A3B"
BASELINE_TP = 2
TARGET_TP = 4
EXP_NAME = "e2e_source_patcher"
DUMPER_FILTER = "layer_id in [0, 1, 2]"

_FIELDS_TO_VERIFY: list[str] = [
    # decoder layer level (aligned with miles)
    "layer_input",
    "attn_output",
    "pre_mlp_residual",
    "mlp_output",
    # attention internals
    "attn_pre_o_proj",
    # moe internals
    "moe_router_logits",
    "moe_expert_output",
]

PATCH_CONFIG_YAML: str = """\
patches:
  # --- decoder layer level (aligned with miles test) ---
  - target: sglang.srt.models.qwen3_moe.Qwen3MoeDecoderLayer.forward
    edits:
      - match: |
          hidden_states, residual = (
              self.layer_communicator.prepare_attn_and_capture_last_layer_outputs(
                  hidden_states,
                  residual,
                  forward_batch,
                  captured_last_layer_outputs=captured_last_layer_outputs,
                  **kwargs,
              )
          )
        append: "dumper.dump('layer_input', hidden_states, dims='t h # tp:replicated')"
      - match: |
          hidden_states = self.self_attn(
              positions=positions,
              hidden_states=hidden_states,
              forward_batch=forward_batch,
          )
        append: "dumper.dump('attn_output', hidden_states, dims='t h[attn_tp:partial] # tp:replicated')"
      - match: |
          hidden_states, residual = self.layer_communicator.prepare_mlp(
              hidden_states, residual, forward_batch
          )
        append: "dumper.dump('pre_mlp_residual', hidden_states, dims='t h # tp:replicated')"
      - match: |
          hidden_states = self.mlp(
              hidden_states, forward_batch, should_allreduce_fusion, use_reduce_scatter
          )
        append: "dumper.dump('mlp_output', hidden_states, dims='t h[moe_tp:partial] # tp:replicated')"

  # --- attention internals ---
  - target: sglang.srt.models.qwen3_moe.Qwen3MoeAttention.forward_core
    edits:
      - match: "output, _ = self.o_proj(attn_output)"
        prepend: "dumper.dump('attn_pre_o_proj', attn_output, dims='t attn_h[attn_tp] # tp:replicated')"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 105-163: Define module constants (part 2)
```python
  # --- moe internals ---
  - target: sglang.srt.models.qwen3_moe.Qwen3MoeSparseMoeBlock.forward_normal
    edits:
      - match: "router_logits, _ = self.gate(hidden_states)"
        append: "dumper.dump('moe_router_logits', router_logits, dims='t num_experts # tp:replicated')"
      - match: "final_hidden_states = self.experts(hidden_states, topk_output)"
        append: "dumper.dump('moe_expert_output', final_hidden_states, dims='t h[moe_tp:partial] # tp:replicated')"
"""

PATCH_CONFIG_DP_ATTENTION_YAML: str = """\
patches:
  # --- decoder layer level (aligned with miles test) ---
  # dp-attention TP=2 DP=2 uses only 2 GPUs:
  #   GPU 0: tp=0, attn_tp=0 (attn_tp_size=1), attn_dp=0
  #   GPU 1: tp=1, attn_tp=0 (attn_tp_size=1), attn_dp=1
  # All sub-axes (attn_tp, moe_tp, attn_dp) are uniquely determined by tp_rank,
  # so only tp:replicated is needed — sub-axes are auto-resolved as implicitly replicated.
  #
  # Attn tensors are NOT TP-sharded (attn_tp_size=1).
  # mlp_output is still moe_tp:partial — the reduce-scatter happens in
  # postprocess_layer(), after the dump point.
  # layer_input is dumped after prepare_attn which DP-distributes tokens,
  # so it needs dp:=attn_dp to filter to the non-empty DP rank.
  - target: sglang.srt.models.qwen3_moe.Qwen3MoeDecoderLayer.forward
    edits:
      - match: |
          hidden_states, residual = (
              self.layer_communicator.prepare_attn_and_capture_last_layer_outputs(
                  hidden_states,
                  residual,
                  forward_batch,
                  captured_last_layer_outputs=captured_last_layer_outputs,
                  **kwargs,
              )
          )
        append: "dumper.dump('layer_input', hidden_states, dims='t h # tp:replicated dp:=attn_dp')"
      - match: |
          hidden_states = self.self_attn(
              positions=positions,
              hidden_states=hidden_states,
              forward_batch=forward_batch,
          )
        append: "dumper.dump('attn_output', hidden_states, dims='t h # tp:replicated')"
      - match: |
          hidden_states, residual = self.layer_communicator.prepare_mlp(
              hidden_states, residual, forward_batch
          )
        append: "dumper.dump('pre_mlp_residual', hidden_states, dims='t h # tp:replicated')"
      - match: |
          hidden_states = self.mlp(
              hidden_states, forward_batch, should_allreduce_fusion, use_reduce_scatter
          )
        append: "dumper.dump('mlp_output', hidden_states, dims='t h[moe_tp:partial] # tp:replicated')"

  # --- attention internals ---
  - target: sglang.srt.models.qwen3_moe.Qwen3MoeAttention.forward_core
    edits:
      - match: "output, _ = self.o_proj(attn_output)"
        prepend: "dumper.dump('attn_pre_o_proj', attn_output, dims='t attn_h # tp:replicated dp:=attn_dp')"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 165-172: Define module constants (part 3)
```python
  # --- moe internals ---
  - target: sglang.srt.models.qwen3_moe.Qwen3MoeSparseMoeBlock.forward_normal
    edits:
      - match: "router_logits, _ = self.gate(hidden_states)"
        append: "dumper.dump('moe_router_logits', router_logits, dims='t num_experts # tp:replicated')"
      - match: "final_hidden_states = self.experts(hidden_states, topk_output)"
        append: "dumper.dump('moe_expert_output', final_hidden_states, dims='t h[moe_tp:partial] # tp:replicated')"
"""
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 175-175: Define class TestSourcePatcherE2ESGLang
```python
class TestSourcePatcherE2ESGLang:
```
**EN:** This declaration introduces the `TestSourcePatcherE2ESGLang` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSourcePatcherE2ESGLang` 测试类，并说明它通过继承承担的职责。

### Lines 176-176: Document the class `TestSourcePatcherE2ESGLang`
```python
    """E2E: patch Qwen3Moe forward -> dump -> compare."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestSourcePatcherE2ESGLang`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestSourcePatcherE2ESGLang`的设计意图。

### Lines 178-183: Run test: patch dump and compare
```python
    def test_patch_dump_and_compare(self, tmp_path: Path) -> None:
        """TP=2 baseline vs TP=4 target."""
        _run_e2e_scenario(
            tmp_path=tmp_path,
            target_tp=TARGET_TP,
        )
```
**EN:** This test method exercises patch dump and compare and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 patch dump and compare 场景，并验证观测到的行为是否符合预期契约。

### Lines 185-216: Run test: dp attention
```python
    def test_dp_attention(self, tmp_path: Path) -> None:
        """TP=2 baseline vs TP=2+DP=2+dp-attention target.

        In dp-attention mode (attn_tp_size=1, attn_dp_size=2), attention
        tensors are NOT TP-sharded and mlp_output is still moe_tp:partial
        (the reduce-scatter happens in postprocess_layer, after the dump
        point).  A separate patch config with corrected dims is used for
        the target.

        Comparison is limited to step 0 (prefill) because the decode
        step has tokens on both DP ranks, which breaks the dp:=attn_dp
        single-rank assumption and causes comparator errors.

        mlp_output is allowed to fail because the FusedMoE dispatcher
        combine path may include an implicit all-reduce that makes the
        dumped value differ from the raw partial expert output.  All
        other tensors (layer_input, attn_output, attn_pre_o_proj,
        pre_mlp_residual, moe_router_logits, moe_expert_output) must
        pass at step 0.
        """
        _run_e2e_scenario(
            tmp_path=tmp_path,
            target_tp=BASELINE_TP,
            extra_target_server_args=["--dp", "2", "--enable-dp-attention"],
            target_patch_config_yaml=PATCH_CONFIG_DP_ATTENTION_YAML,
            extra_comparator_args=[
                "--end-step",
                "0",
                "--allow-failed-pattern",
                "mlp_output",
            ],
        )
```
**EN:** This test method exercises dp attention and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 dp attention 场景，并验证观测到的行为是否符合预期契约。

### Lines 222-275: Define helper: run e2e scenario (part 1)
```python
def _run_e2e_scenario(
    *,
    tmp_path: Path,
    target_tp: int,
    extra_target_server_args: Optional[list[str]] = None,
    target_patch_config_yaml: Optional[str] = None,
    extra_comparator_args: Optional[list[str]] = None,
) -> None:
    """Full e2e: write patch config -> baseline run -> target run -> compare."""
    base_url: str = DEFAULT_URL_FOR_TEST

    baseline_config_path: Path = tmp_path / "patch_config.yaml"
    baseline_config_path.write_text(PATCH_CONFIG_YAML)

    target_config_path: Path = tmp_path / "patch_config_target.yaml"
    target_config_path.write_text(target_patch_config_yaml or PATCH_CONFIG_YAML)

    baseline_dir: Path = tmp_path / "baseline"
    _run_server_and_generate(
        dump_dir=baseline_dir,
        config_path=baseline_config_path,
        tp=BASELINE_TP,
        base_url=base_url,
    )
    _verify_patched_fields(dump_dir=baseline_dir, field_names=_FIELDS_TO_VERIFY)

    target_dir: Path = tmp_path / "target"
    _run_server_and_generate(
        dump_dir=target_dir,
        config_path=target_config_path,
        tp=target_tp,
        base_url=base_url,
        extra_server_args=extra_target_server_args,
    )
    _verify_patched_fields(dump_dir=target_dir, field_names=_FIELDS_TO_VERIFY)

    baseline_exp: Path = baseline_dir / EXP_NAME
    target_exp: Path = target_dir / EXP_NAME

    cmd: list[str] = [
        "python",
        "-m",
        "sglang.srt.debug_utils.comparator",
        "--baseline-path",
        str(baseline_exp),
        "--target-path",
        str(target_exp),
        "--output-format",
        "json",
        "--allow-skipped-pattern",
        "input_ids|positions",
    ]
    if extra_comparator_args:
        cmd.extend(extra_comparator_args)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 277-286: Define helper: run e2e scenario (part 2)
```python
    result: subprocess.CompletedProcess[str] = subprocess.run(
        cmd,
        capture_output=True,
        text=True,
    )

    debug_file: Path = _save_comparator_output(
        stdout=result.stdout, stderr=result.stderr
    )
    print(f"Comparator debug output: {debug_file}")
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 288-290: Define helper: run e2e scenario (part 3)
```python
    assert result.returncode == 0, (
        f"Comparator failed (rc={result.returncode}). " f"Debug output: {debug_file}"
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 293-350: Define helper: run server and generate
```python
def _run_server_and_generate(
    *,
    dump_dir: Path,
    config_path: Path,
    tp: int,
    base_url: str,
    extra_server_args: Optional[list[str]] = None,
) -> None:
    """Launch SGLang server with source patcher + dumper, send a generate request."""
    env: dict[str, str] = {
        **os.environ,
        "DUMPER_SOURCE_PATCHER_CONFIG": str(config_path),
        "DUMPER_DIR": str(dump_dir),
        "DUMPER_EXP_NAME": EXP_NAME,
        "DUMPER_SERVER_PORT": "reuse",
    }

    server_args: list[str] = [
        "--tp",
        str(tp),
        "--max-total-tokens",
        "128",
        "--mem-fraction-static",
        "0.5",
        "--disable-cuda-graph",
        "--disable-piecewise-cuda-graph",
        "--disable-radix-cache",
    ]
    if extra_server_args:
        server_args.extend(extra_server_args)

    proc = popen_launch_server(
        MODEL,
        base_url,
        timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
        other_args=server_args,
        env=env,
    )
    try:
        requests.post(
            f"{base_url}/dumper/configure",
            json={
                "enable": True,
                "filter": DUMPER_FILTER,
                "cleanup_previous": True,
            },
        ).raise_for_status()

        resp = requests.post(
            f"{base_url}/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {"max_new_tokens": 1, "temperature": 0},
            },
        )
        assert resp.status_code == 200, f"Generate failed: {resp.text}"
    finally:
        kill_process_tree(proc.pid)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 353-360: Define helper: verify patched fields
```python
def _verify_patched_fields(*, dump_dir: Path, field_names: list[str]) -> None:
    """Verify that patched dump fields exist as .pt files."""
    for field in field_names:
        matches: list[Path] = list(dump_dir.rglob(f"*name={field}*.pt"))
        assert len(matches) > 0, (
            f"Expected patched field '{field}' not found under {dump_dir}. "
            f"Available files: {sorted(f.name for f in dump_dir.rglob('*.pt'))[:20]}"
        )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 363-371: Define helper: save comparator output
```python
def _save_comparator_output(*, stdout: str, stderr: str) -> Path:
    """Save comparator stdout+stderr to a temp file that persists for debugging."""
    fd, path_str = tempfile.mkstemp(prefix="comparator_e2e_", suffix=".log", dir="/tmp")
    with os.fdopen(fd, "w") as f:
        f.write("=== STDOUT ===\n")
        f.write(stdout)
        f.write("\n=== STDERR ===\n")
        f.write(stderr)
    return Path(path_str)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 374-375: Expose unittest entrypoint
```python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v"]))
```
**EN:** This conditional keeps the module executable as a standalone unittest target.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Server lifecycle helpers start the target runtime before assertions begin. / 服务生命周期辅助函数会在断言开始前启动目标运行时。
- Explicit teardown avoids leaked child processes between runs. / 显式清理可以避免多次运行之间遗留子进程。
- HTTP requests validate the externally visible API contract rather than only internal helpers. / HTTP 请求验证的是对外暴露的 API 契约，而不只是内部辅助函数。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `pathlib`, `pytest`, `requests`, `subprocess`, `sys`, `tempfile`, `typing`
- Notable symbols / 关键符号: `register_cuda_ci`, `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `requests.post`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
