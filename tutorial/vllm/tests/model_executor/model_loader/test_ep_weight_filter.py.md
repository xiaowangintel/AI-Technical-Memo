# test_ep_weight_filter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/model_executor/model_loader/test_ep_weight_filter.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Tests for EP weight filtering during model loading. / 该文件主要围绕 Ep Weight Filter 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-19)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Tests for EP weight filtering during model loading."""

import glob
import tempfile

import huggingface_hub.constants
import pytest
import torch

from vllm.model_executor.model_loader.ep_weight_filter import (
    compute_local_expert_ids,
    parse_expert_id,
    should_skip_weight,
)
from vllm.model_executor.model_loader.weight_utils import (
    safetensors_weights_iterator,
)
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `glob`, `tempfile`, `huggingface_hub.constants`, `pytest`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Class: TestParseExpertId (lines 26-69)
```python
class TestParseExpertId:
    def test_routed_expert(self):
        name = "model.layers.0.mlp.experts.42.gate_proj.weight"
        assert parse_expert_id(name) == 42

    def test_large_expert_id(self):
        name = "model.layers.60.mlp.experts.383.down_proj.weight"
        assert parse_expert_id(name) == 383

    def test_shared_expert(self):
        # Shared experts use a different naming convention in most models
        name = "model.layers.0.mlp.shared_experts.gate_proj.weight"
        assert parse_expert_id(name) is None

    def test_attention_weight(self):
        name = "model.layers.0.self_attn.q_proj.weight"
        assert parse_expert_id(name) is None

    def test_embedding(self):
# ... omitted for brevity ...
        assert parse_expert_id(name) is None

    def test_fused_3d_expert_down_proj(self):
        name = "model.layers.10.mlp.experts.down_proj.weight"
        assert parse_expert_id(name) is None

    def test_expert_scale(self):
        # NVFP4 quantized models have scale tensors for experts
        name = "model.layers.5.mlp.experts.100.gate_proj.weight_scale"
        assert parse_expert_id(name) == 100

    def test_expert_zero_id(self):
        name = "model.layers.0.mlp.experts.0.up_proj.weight"
        assert parse_expert_id(name) == 0
```
**EN:** Groups related scenarios for Testparseexpertid. The class contains 10 test method(s).
**CN:** 该类把与 Testparseexpertid 相关的场景组织在一起。 其中包含 10 个测试方法。

### Class: TestComputeLocalExpertIds (lines 77-157)
```python
class TestComputeLocalExpertIds:
    def test_ep_disabled(self):
        assert compute_local_expert_ids(64, ep_size=1, ep_rank=0) is None

    def test_even_split(self):
        # 64 experts, EP=8 → 8 per rank
        ids = compute_local_expert_ids(64, ep_size=8, ep_rank=0)
        assert ids == set(range(0, 8))

        ids = compute_local_expert_ids(64, ep_size=8, ep_rank=7)
        assert ids == set(range(56, 64))

    def test_uneven_split(self):
        # 10 experts, EP=3 → ranks get 4, 3, 3
        ids_0 = compute_local_expert_ids(10, ep_size=3, ep_rank=0)
        ids_1 = compute_local_expert_ids(10, ep_size=3, ep_rank=1)
        ids_2 = compute_local_expert_ids(10, ep_size=3, ep_rank=2)

        assert len(ids_0) == 4
# ... omitted for brevity ...
            assert all_ids.isdisjoint(ids)
            all_ids |= ids
        assert all_ids == set(range(384))

    def test_round_robin_uneven(self):
        # 10 experts, EP=3: rank 0→{0,3,6,9}, rank 1→{1,4,7}, rank 2→{2,5,8}
        rr = "round_robin"
        ids_0 = compute_local_expert_ids(10, 3, 0, placement=rr)
        ids_1 = compute_local_expert_ids(10, 3, 1, placement=rr)
        ids_2 = compute_local_expert_ids(10, 3, 2, placement=rr)
        assert ids_0 == {0, 3, 6, 9}
        assert ids_1 == {1, 4, 7}
        assert ids_2 == {2, 5, 8}
        assert ids_0 | ids_1 | ids_2 == set(range(10))
```
**EN:** Groups related scenarios for Testcomputelocalexpertids. The class contains 9 test method(s).
**CN:** 该类把与 Testcomputelocalexpertids 相关的场景组织在一起。 其中包含 9 个测试方法。

### Class: TestShouldSkipWeight (lines 165-210)
```python
class TestShouldSkipWeight:
    def setup_method(self):
        # Simulate EP=8, rank=0 → experts 0-47
        self.local_ids = compute_local_expert_ids(384, ep_size=8, ep_rank=0)

    def test_no_filter(self):
        assert not should_skip_weight("anything", None)

    def test_dense_not_skipped(self):
        assert not should_skip_weight(
            "model.layers.0.self_attn.q_proj.weight", self.local_ids
        )

    def test_local_expert_not_skipped(self):
        assert not should_skip_weight(
            "model.layers.0.mlp.experts.10.gate_proj.weight", self.local_ids
        )

    def test_remote_expert_skipped(self):
# ... omitted for brevity ...
    def test_shared_expert_not_skipped(self):
        assert not should_skip_weight(
            "model.layers.0.mlp.shared_experts.gate_proj.weight", self.local_ids
        )

    def test_embedding_not_skipped(self):
        assert not should_skip_weight("model.embed_tokens.weight", self.local_ids)

    def test_fused_3d_expert_not_skipped(self):
        # 3D fused-expert tensors (gpt-oss style) have no numeric id.
        # Must not be skipped — weight_loader handles slicing later.
        assert not should_skip_weight(
            "model.layers.0.mlp.experts.gate_proj.weight", self.local_ids
        )
```
**EN:** Groups related scenarios for Testshouldskipweight. The class contains 8 test method(s) and 1 helper/setup method(s).
**CN:** 该类把与 Testshouldskipweight 相关的场景组织在一起。 其中包含 8 个测试方法，以及 1 个辅助或初始化方法。

### Class: TestSafetensorsWeightsIteratorWithEpFilter (lines 218-256)
```python
class TestSafetensorsWeightsIteratorWithEpFilter:
    """Verify that EP filtering produces a strict subset of unfiltered loading
    and that all expected dense + local expert weights are present."""

    @pytest.fixture(scope="class")
    def gpt2_files(self):
        """Download GPT-2 safetensors to a temp dir (shared across class)."""
        with tempfile.TemporaryDirectory() as tmpdir:
            huggingface_hub.constants.HF_HUB_OFFLINE = False
            from vllm.model_executor.model_loader.weight_utils import (
                download_weights_from_hf,
            )

            download_weights_from_hf(
                "openai-community/gpt2",
                allow_patterns=["*.safetensors"],
                cache_dir=tmpdir,
            )
            files = glob.glob(f"{tmpdir}/**/*.safetensors", recursive=True)
# ... omitted for brevity ...
        filtered_weights = dict(
            safetensors_weights_iterator(gpt2_files, False, local_expert_ids=None)
        )
        assert set(all_weights.keys()) == set(filtered_weights.keys())

    def test_empty_filter_skips_experts_only(self, gpt2_files):
        """GPT-2 has no expert weights, so even an empty local_expert_ids
        set should return all weights (all are dense)."""
        all_weights = dict(safetensors_weights_iterator(gpt2_files, False))
        filtered_weights = dict(
            safetensors_weights_iterator(gpt2_files, False, local_expert_ids=set())
        )
        # GPT-2 has no experts, so nothing should be filtered
        assert set(all_weights.keys()) == set(filtered_weights.keys())
```
**EN:** Groups related scenarios for Testsafetensorsweightsiteratorwithepfilter. The class contains 2 test method(s) and 1 helper/setup method(s).
**CN:** 该类把与 Testsafetensorsweightsiteratorwithepfilter 相关的场景组织在一起。 其中包含 2 个测试方法，以及 1 个辅助或初始化方法。

### Class: TestEpFilterOnSyntheticMoeWeights (lines 259-361)
```python
class TestEpFilterOnSyntheticMoeWeights:
    """Create synthetic safetensors files with expert-like naming and verify
    that the filter correctly skips non-local experts."""

    @pytest.fixture
    def synthetic_moe_files(self, tmp_path):
        """Create synthetic safetensors with expert-patterned tensor names."""
        from safetensors.torch import save_file

        tensors = {}
        # Dense weights
        tensors["model.embed_tokens.weight"] = torch.randn(100, 64)
        tensors["model.layers.0.self_attn.q_proj.weight"] = torch.randn(64, 64)
        tensors["model.layers.0.input_layernorm.weight"] = torch.randn(64)
        # Expert weights: 8 experts
        for expert_id in range(8):
            tensors[f"model.layers.0.mlp.experts.{expert_id}.gate_proj.weight"] = (
                torch.randn(128, 64)
            )
# ... omitted for brevity ...
        assert len(all_expert_names) == 24

    def test_tensor_values_match(self, synthetic_moe_files):
        """Filtered tensors have identical values to unfiltered ones."""
        files, _ = synthetic_moe_files
        all_weights = dict(safetensors_weights_iterator(files, False))

        local_ids = compute_local_expert_ids(8, ep_size=2, ep_rank=0)
        filtered = dict(
            safetensors_weights_iterator(files, False, local_expert_ids=local_ids)
        )

        for name, tensor in filtered.items():
            assert torch.equal(tensor, all_weights[name]), f"Tensor mismatch for {name}"
```
**EN:** Groups related scenarios for Testepfilteronsyntheticmoeweights. The class contains 5 test method(s) and 1 helper/setup method(s).
**CN:** 该类把与 Testepfilteronsyntheticmoeweights 相关的场景组织在一起。 其中包含 5 个测试方法，以及 1 个辅助或初始化方法。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `glob`, `tempfile`
- **Third-party / 第三方依赖**: `huggingface_hub.constants`, `pytest`, `torch`, `safetensors.torch`
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.model_loader.ep_weight_filter`, `vllm.model_executor.model_loader.weight_utils`
