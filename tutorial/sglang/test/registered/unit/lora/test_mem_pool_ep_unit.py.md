# test_mem_pool_ep_unit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/lora/test_mem_pool_ep_unit.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates mem pool ep unit behavior in SGLang's unit / lora area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / lora 领域中与 mem pool ep unit 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: supporting statements / 辅助语句
```python
"""Unit tests for LoRAMemoryPool's MoE expert-parallel (EP) handling.

Covers the global->local expert-id remapping and per-rank buffer sizing
introduced so that per-expert MoE LoRA buffers stay aligned with the
Triton MoE runner's local-id dispatch under `--ep > 1`.

The tests exercise the class behavior without standing up a full server
or distributed groups: `LoRAMemoryPool` is instantiated via `__new__`
and only the fields the helpers read are populated. This keeps the
tests hermetic (CPU-only, no CUDA, no MoE EP group).

Usage:
    python -m pytest test/registered/unit/lora/test_mem_pool_ep_unit.py -v
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 16-16: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 17-18: supporting source context / 辅助源码上下文
```python

# CPU-only unit test; no CUDA/distributed dependencies.
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 19-20: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=9, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=9, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 22-33: module imports and dependencies / 模块导入与依赖
```python
import types
import unittest
import unittest.mock as mock

import torch

from sglang.srt.lora.mem_pool import (
    LoRAMemoryPool,
    _get_moe_ep_context,
    _get_moe_tp_context,
    _moe_runner_keeps_global_expert_ids,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `types`, `unittest`, `unittest.mock`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `types`, `unittest`, `unittest.mock`, `torch`。

### Lines 36-61: function make pool / 函数 make pool
```python
def _make_pool(
    *,
    num_experts_global: int,
    moe_ep_size: int,
    moe_ep_rank: int,
    moe_use_local_expert_ids: bool,
) -> LoRAMemoryPool:
    """Construct a minimal LoRAMemoryPool for helper-level tests.

    Bypasses `__init__` (which requires a real base model, HF config, and
    device allocations) and sets only the fields consulted by the EP
    helpers under test.
    """
    pool = LoRAMemoryPool.__new__(LoRAMemoryPool)
    pool.moe_ep_size = moe_ep_size
    pool.moe_ep_rank = moe_ep_rank
    pool.moe_use_local_expert_ids = moe_use_local_expert_ids
    # Helpers under test in this module don't consult moe_tp_size, but set
    # defaults so accidental reads don't AttributeError.
    pool.moe_tp_size = 1
    pool.moe_tp_rank = 0
    if moe_use_local_expert_ids and num_experts_global % moe_ep_size == 0:
        pool._num_experts_local = num_experts_global // moe_ep_size
    else:
        pool._num_experts_local = num_experts_global
    return pool
```
**EN:** Construct a minimal LoRAMemoryPool for helper-level tests. This block implements `_make_pool` and captures one focused piece of the module's behavior.
**CN:** Construct a minimal LoRAMemoryPool for helper-level tests. 该代码块实现 `_make_pool`，承担模块行为中的一个聚焦逻辑片段。

### Lines 64-73: function make fake base model / 函数 make fake base model
```python
def _make_fake_base_model(num_experts: int) -> torch.nn.Module:
    """Return a `torch.nn.Module` whose `.config` exposes `num_experts`.

    Used by `_get_num_experts` / `_get_num_local_experts` which walk the
    HF config object. No real weights needed.
    """
    model = torch.nn.Linear(4, 4, bias=False)
    cfg = types.SimpleNamespace(num_experts=num_experts)
    model.config = cfg
    return model
```
**EN:** Return a `torch.nn.Module` whose `.config` exposes `num_experts`. This block implements `_make_fake_base_model` and captures one focused piece of the module's behavior.
**CN:** Return a `torch.nn.Module` whose `.config` exposes `num_experts`. 该代码块实现 `_make_fake_base_model`，承担模块行为中的一个聚焦逻辑片段。

### Lines 76-76: class TestNumExpertHelpers declaration / 类 TestNumExpertHelpers 声明
```python
class TestNumExpertHelpers(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 77-77: supporting statements / 辅助语句
```python
    """`_get_num_experts` / `_get_num_local_experts` / buffer-dim picker."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 79-81: test case num experts read from config / 测试用例 num experts read from config
```python
    def test_num_experts_read_from_config(self):
        model = _make_fake_base_model(num_experts=8)
        self.assertEqual(LoRAMemoryPool._get_num_experts(model), 8)
```
**EN:** This test exercises `test_num_experts_read_from_config` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_num_experts_read_from_config`。

### Lines 83-91: test case num local experts no ep / 测试用例 num local experts no ep
```python
    def test_num_local_experts_no_ep(self):
        pool = _make_pool(
            num_experts_global=8,
            moe_ep_size=1,
            moe_ep_rank=0,
            moe_use_local_expert_ids=False,
        )
        model = _make_fake_base_model(num_experts=8)
        self.assertEqual(pool._get_num_local_experts(model), 8)
```
**EN:** This test exercises `test_num_local_experts_no_ep` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_num_local_experts_no_ep`。

### Lines 93-101: test case num local experts with ep / 测试用例 num local experts with ep
```python
    def test_num_local_experts_with_ep(self):
        pool = _make_pool(
            num_experts_global=8,
            moe_ep_size=4,
            moe_ep_rank=2,
            moe_use_local_expert_ids=True,
        )
        model = _make_fake_base_model(num_experts=8)
        self.assertEqual(pool._get_num_local_experts(model), 2)
```
**EN:** This test exercises `test_num_local_experts_with_ep` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_num_local_experts_with_ep`。

### Lines 103-114: test case num local experts with ep but backend keeps global ids / 测试用例 num local experts with ep but backend keeps global ids
```python
    def test_num_local_experts_with_ep_but_backend_keeps_global_ids(self):
        """FlashInfer-style backends keep global topk_ids, so even under EP
        the LoRA buffers must remain globally-keyed.
        """
        pool = _make_pool(
            num_experts_global=8,
            moe_ep_size=4,
            moe_ep_rank=2,
            moe_use_local_expert_ids=False,
        )
        model = _make_fake_base_model(num_experts=8)
        self.assertEqual(pool._get_num_local_experts(model), 8)
```
**EN:** FlashInfer-style backends keep global topk_ids, so even under EP the LoRA buffers must remain globally-keyed. This test exercises `test_num_local_experts_with_ep_but_backend_keeps_global_ids` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** FlashInfer-style backends keep global topk_ids, so even under EP the LoRA buffers must remain globally-keyed. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_num_local_experts_with_ep_but_backend_keeps_global_ids`。

### Lines 116-131: test case uneven split disables local mapping / 测试用例 uneven split disables local mapping
```python
    def test_uneven_split_disables_local_mapping(self):
        """Shouldn't happen in practice (base MoE requires even split), but
        `__init__` must fold uneven splits into `moe_use_local_expert_ids ==
        False` so `_get_num_local_experts` returns the global count and no
        remapping happens anywhere downstream.
        """
        # Simulate what `LoRAMemoryPool.__init__` would set for an uneven
        # split: the divisibility guard there forces the flag to False.
        pool = _make_pool(
            num_experts_global=7,
            moe_ep_size=4,
            moe_ep_rank=0,
            moe_use_local_expert_ids=False,
        )
        model = _make_fake_base_model(num_experts=7)
        self.assertEqual(pool._get_num_local_experts(model), 7)
```
**EN:** Shouldn't happen in practice (base MoE requires even split), but `__init__` must fold uneven splits into `moe_use_local_expert_ids == False` so `_get_num_local_experts` returns the global count and no remapping happens anywhere downstream. This test exercises `test_uneven_split_disables_local_mapping` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Shouldn't happen in practice (base MoE requires even split), but `__init__` must fold uneven splits into `moe_use_local_expert_ids == False` so `_get_num_local_experts` returns the global count and no remapping happens anywhere downstream. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_uneven_split_disables_local_mapping`。

### Lines 134-134: class TestGlobalToLocalExpertId declaration / 类 TestGlobalToLocalExpertId 声明
```python
class TestGlobalToLocalExpertId(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 135-135: supporting statements / 辅助语句
```python
    """`_global_to_local_expert_id` — the per-rank filter + remap."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 137-145: test case passthrough without ep / 测试用例 passthrough without ep
```python
    def test_passthrough_without_ep(self):
        pool = _make_pool(
            num_experts_global=8,
            moe_ep_size=1,
            moe_ep_rank=0,
            moe_use_local_expert_ids=False,
        )
        for gid in range(8):
            self.assertEqual(pool._global_to_local_expert_id(gid), gid)
```
**EN:** This test exercises `test_passthrough_without_ep` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_passthrough_without_ep`。

### Lines 147-159: test case rank0 of ep4 owns first quarter / 测试用例 rank0 of ep4 owns first quarter
```python
    def test_rank0_of_ep4_owns_first_quarter(self):
        pool = _make_pool(
            num_experts_global=8,
            moe_ep_size=4,
            moe_ep_rank=0,
            moe_use_local_expert_ids=True,
        )
        # Owned: 0, 1 -> local 0, 1
        self.assertEqual(pool._global_to_local_expert_id(0), 0)
        self.assertEqual(pool._global_to_local_expert_id(1), 1)
        # Not owned by rank 0.
        for gid in (2, 3, 4, 5, 6, 7):
            self.assertIsNone(pool._global_to_local_expert_id(gid))
```
**EN:** This test exercises `test_rank0_of_ep4_owns_first_quarter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rank0_of_ep4_owns_first_quarter`。

### Lines 161-172: test case rank2 of ep4 owns third quarter / 测试用例 rank2 of ep4 owns third quarter
```python
    def test_rank2_of_ep4_owns_third_quarter(self):
        pool = _make_pool(
            num_experts_global=8,
            moe_ep_size=4,
            moe_ep_rank=2,
            moe_use_local_expert_ids=True,
        )
        # Owned globals 4, 5 -> local 0, 1
        self.assertEqual(pool._global_to_local_expert_id(4), 0)
        self.assertEqual(pool._global_to_local_expert_id(5), 1)
        for gid in (0, 1, 2, 3, 6, 7):
            self.assertIsNone(pool._global_to_local_expert_id(gid))
```
**EN:** This test exercises `test_rank2_of_ep4_owns_third_quarter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rank2_of_ep4_owns_third_quarter`。

### Lines 174-185: test case last rank owns last slice / 测试用例 last rank owns last slice
```python
    def test_last_rank_owns_last_slice(self):
        pool = _make_pool(
            num_experts_global=128,
            moe_ep_size=4,
            moe_ep_rank=3,
            moe_use_local_expert_ids=True,
        )
        # Local 0 <-> global 96, local 31 <-> global 127.
        self.assertEqual(pool._global_to_local_expert_id(96), 0)
        self.assertEqual(pool._global_to_local_expert_id(127), 31)
        self.assertIsNone(pool._global_to_local_expert_id(95))
        self.assertIsNone(pool._global_to_local_expert_id(128))
```
**EN:** This test exercises `test_last_rank_owns_last_slice` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_last_rank_owns_last_slice`。

### Lines 188-188: class TestIterLocalExpertWeightsDict declaration / 类 TestIterLocalExpertWeightsDict 声明
```python
class TestIterLocalExpertWeightsDict(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 189-189: supporting statements / 辅助语句
```python
    """`_iter_local_expert_weights` with dict input (the common case)."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 191-202: test case passthrough without ep / 测试用例 passthrough without ep
```python
    def test_passthrough_without_ep(self):
        pool = _make_pool(
            num_experts_global=4,
            moe_ep_size=1,
            moe_ep_rank=0,
            moe_use_local_expert_ids=False,
        )
        weights = {gid: torch.full((2,), float(gid)) for gid in range(4)}
        got = {lid: w.tolist() for lid, w in pool._iter_local_expert_weights(weights)}
        self.assertEqual(
            got, {0: [0.0, 0.0], 1: [1.0, 1.0], 2: [2.0, 2.0], 3: [3.0, 3.0]}
        )
```
**EN:** This test exercises `test_passthrough_without_ep` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_passthrough_without_ep`。

### Lines 204-214: test case rank0 of ep4 filters and remaps / 测试用例 rank0 of ep4 filters and remaps
```python
    def test_rank0_of_ep4_filters_and_remaps(self):
        pool = _make_pool(
            num_experts_global=8,
            moe_ep_size=4,
            moe_ep_rank=0,
            moe_use_local_expert_ids=True,
        )
        weights = {gid: torch.full((2,), float(gid)) for gid in range(8)}
        got = {lid: w.tolist() for lid, w in pool._iter_local_expert_weights(weights)}
        # Rank 0 sees globals 0,1 remapped to locals 0,1.
        self.assertEqual(got, {0: [0.0, 0.0], 1: [1.0, 1.0]})
```
**EN:** This test exercises `test_rank0_of_ep4_filters_and_remaps` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rank0_of_ep4_filters_and_remaps`。

### Lines 216-226: test case rank3 of ep4 filters and remaps / 测试用例 rank3 of ep4 filters and remaps
```python
    def test_rank3_of_ep4_filters_and_remaps(self):
        pool = _make_pool(
            num_experts_global=8,
            moe_ep_size=4,
            moe_ep_rank=3,
            moe_use_local_expert_ids=True,
        )
        weights = {gid: torch.full((2,), float(gid)) for gid in range(8)}
        got = {lid: w.tolist() for lid, w in pool._iter_local_expert_weights(weights)}
        # Rank 3 sees globals 6,7 remapped to locals 0,1.
        self.assertEqual(got, {0: [6.0, 6.0], 1: [7.0, 7.0]})
```
**EN:** This test exercises `test_rank3_of_ep4_filters_and_remaps` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rank3_of_ep4_filters_and_remaps`。

### Lines 228-247: test case sparse dict only yields owned experts / 测试用例 sparse dict only yields owned experts
```python
    def test_sparse_dict_only_yields_owned_experts(self):
        """Adapters may only target a subset of experts. The iterator must
        still correctly filter and remap whatever subset is provided.
        """
        pool = _make_pool(
            num_experts_global=8,
            moe_ep_size=4,
            moe_ep_rank=2,
            moe_use_local_expert_ids=True,
        )
        # Only globals 1, 4, 5, 7 present in adapter.
        weights = {
            1: torch.full((2,), 1.0),
            4: torch.full((2,), 4.0),
            5: torch.full((2,), 5.0),
            7: torch.full((2,), 7.0),
        }
        # Rank 2 owns globals 4, 5 -> locals 0, 1.
        got = {lid: w.tolist() for lid, w in pool._iter_local_expert_weights(weights)}
        self.assertEqual(got, {0: [4.0, 4.0], 1: [5.0, 5.0]})
```
**EN:** Adapters may only target a subset of experts. This test exercises `test_sparse_dict_only_yields_owned_experts` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Adapters may only target a subset of experts. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_sparse_dict_only_yields_owned_experts`。

### Lines 249-262: test case no experts owned yields nothing / 测试用例 no experts owned yields nothing
```python
    def test_no_experts_owned_yields_nothing(self):
        """Rank with no matching experts in a sparse dict yields nothing,
        leaves buffer zeroed.
        """
        pool = _make_pool(
            num_experts_global=8,
            moe_ep_size=4,
            moe_ep_rank=0,
            moe_use_local_expert_ids=True,
        )
        # Only globals 4, 5 present (owned by rank 2).
        weights = {4: torch.full((2,), 4.0), 5: torch.full((2,), 5.0)}
        got = list(pool._iter_local_expert_weights(weights))
        self.assertEqual(got, [])
```
**EN:** Rank with no matching experts in a sparse dict yields nothing, leaves buffer zeroed. This test exercises `test_no_experts_owned_yields_nothing` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Rank with no matching experts in a sparse dict yields nothing, leaves buffer zeroed. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_experts_owned_yields_nothing`。

### Lines 265-265: class TestIterLocalExpertWeightsTensor declaration / 类 TestIterLocalExpertWeightsTensor 声明
```python
class TestIterLocalExpertWeightsTensor(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 266-267: supporting statements / 辅助语句
```python
    """`_iter_local_expert_weights` with 3D tensor input (shared-outer and
    packed MoE-LoRA formats)."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 269-281: test case passthrough without ep / 测试用例 passthrough without ep
```python
    def test_passthrough_without_ep(self):
        pool = _make_pool(
            num_experts_global=4,
            moe_ep_size=1,
            moe_ep_rank=0,
            moe_use_local_expert_ids=False,
        )
        # [num_experts, rank, hidden] with values carrying the expert id.
        weights = torch.arange(4 * 2 * 3, dtype=torch.float32).reshape(4, 2, 3)
        got = [(lid, w.clone()) for lid, w in pool._iter_local_expert_weights(weights)]
        self.assertEqual([lid for lid, _ in got], [0, 1, 2, 3])
        for lid, w in got:
            self.assertTrue(torch.equal(w, weights[lid]))
```
**EN:** This test exercises `test_passthrough_without_ep` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_passthrough_without_ep`。

### Lines 283-295: test case rank1 of ep2 sees upper half / 测试用例 rank1 of ep2 sees upper half
```python
    def test_rank1_of_ep2_sees_upper_half(self):
        pool = _make_pool(
            num_experts_global=4,
            moe_ep_size=2,
            moe_ep_rank=1,
            moe_use_local_expert_ids=True,
        )
        weights = torch.arange(4 * 2 * 3, dtype=torch.float32).reshape(4, 2, 3)
        got = [(lid, w.clone()) for lid, w in pool._iter_local_expert_weights(weights)]
        # Rank 1 of EP=2 with 4 experts owns globals 2, 3 -> locals 0, 1.
        self.assertEqual([lid for lid, _ in got], [0, 1])
        self.assertTrue(torch.equal(got[0][1], weights[2]))
        self.assertTrue(torch.equal(got[1][1], weights[3]))
```
**EN:** This test exercises `test_rank1_of_ep2_sees_upper_half` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rank1_of_ep2_sees_upper_half`。

### Lines 297-313: test case rank with partial tensor coverage / 测试用例 rank with partial tensor coverage
```python
    def test_rank_with_partial_tensor_coverage(self):
        """Defensive: tensor has fewer experts than the expected local slice
        (e.g. sparse adapter).
        """
        pool = _make_pool(
            num_experts_global=8,
            moe_ep_size=4,
            moe_ep_rank=3,
            moe_use_local_expert_ids=True,
        )
        # Only 6 experts present in the tensor; rank 3 expects global 6,7.
        # So it should still yield local 0 mapped to global 6; global 7 is
        # beyond the tensor length and must be skipped safely.
        weights = torch.arange(6 * 2, dtype=torch.float32).reshape(6, 2)
        # Note: this is 2D, not 3D -> should raise (sanity check).
        with self.assertRaises(TypeError):
            list(pool._iter_local_expert_weights(weights))
```
**EN:** Defensive: tensor has fewer experts than the expected local slice (e.g. This test exercises `test_rank_with_partial_tensor_coverage` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Defensive: tensor has fewer experts than the expected local slice (e.g. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rank_with_partial_tensor_coverage`。

### Lines 316-316: class TestModuleLevelHelpers declaration / 类 TestModuleLevelHelpers 声明
```python
class TestModuleLevelHelpers(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 317-319: supporting statements / 辅助语句
```python
    """`_get_moe_ep_context` / `_moe_runner_keeps_global_expert_ids`
    must degrade gracefully when the MoE EP group or runner backend is
    not yet initialized (e.g. in pure-TP launches or hermetic tests)."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 321-326: test case ep context defaults when group uninitialized / 测试用例 ep context defaults when group uninitialized
```python
    def test_ep_context_defaults_when_group_uninitialized(self):
        # Real process here: the MoE EP group isn't set up in a unit test.
        # The helper must return (1, 0) rather than raising.
        ep_size, ep_rank = _get_moe_ep_context()
        self.assertEqual(ep_size, 1)
        self.assertEqual(ep_rank, 0)
```
**EN:** This test exercises `test_ep_context_defaults_when_group_uninitialized` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ep_context_defaults_when_group_uninitialized`。

### Lines 328-333: test case tp context defaults when group uninitialized / 测试用例 tp context defaults when group uninitialized
```python
    def test_tp_context_defaults_when_group_uninitialized(self):
        # Mirror of `_get_moe_ep_context` for the MoE TP group: if it isn't
        # initialized (hermetic tests, pure-TP launches), fall back to (1, 0).
        tp_size, tp_rank = _get_moe_tp_context()
        self.assertEqual(tp_size, 1)
        self.assertEqual(tp_rank, 0)
```
**EN:** This test exercises `test_tp_context_defaults_when_group_uninitialized` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tp_context_defaults_when_group_uninitialized`。

### Lines 335-337: test case keeps global expert ids defaults to false / 测试用例 keeps global expert ids defaults to false
```python
    def test_keeps_global_expert_ids_defaults_to_false(self):
        # Without a specific flashinfer backend selected, default is False.
        self.assertFalse(_moe_runner_keeps_global_expert_ids())
```
**EN:** This test exercises `test_keeps_global_expert_ids_defaults_to_false` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_keeps_global_expert_ids_defaults_to_false`。

### Lines 340-340: class TestPoolInitPicksUpEpContext declaration / 类 TestPoolInitPicksUpEpContext 声明
```python
class TestPoolInitPicksUpEpContext(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 341-342: supporting statements / 辅助语句
```python
    """`LoRAMemoryPool.__init__` should read EP context from the module-
    level helpers and set `moe_use_local_expert_ids` correctly."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 344-392: method new pool with ep / 方法 new pool with ep
```python
    def _new_pool_with_ep(
        self,
        ep_size: int,
        ep_rank: int,
        keeps_global: bool,
        num_experts: int = 8,
        moe_tp_size: int = 1,
        moe_tp_rank: int = 0,
        tp_size: int = 1,
        tp_rank: int = 0,
    ) -> LoRAMemoryPool:
        """Construct a pool with `__init__` called, but stop before
        `init_buffers` — we only care about the EP-context state.
        """
        with (
            mock.patch(
                "sglang.srt.lora.mem_pool._get_moe_ep_context",
                return_value=(ep_size, ep_rank),
            ),
            mock.patch(
                "sglang.srt.lora.mem_pool._get_moe_tp_context",
                return_value=(moe_tp_size, moe_tp_rank),
            ),
            mock.patch(
                "sglang.srt.lora.mem_pool._moe_runner_keeps_global_expert_ids",
                return_value=keeps_global,
            ),
            mock.patch.object(LoRAMemoryPool, "init_buffers", lambda self, _m: None),
        ):
            hf_cfg = types.SimpleNamespace(
                num_hidden_layers=1,
                hidden_size=8,
                vocab_size=32,
                num_experts=num_experts,
            )
            base_model = torch.nn.Linear(8, 8, bias=False)
            base_model.config = hf_cfg
            return LoRAMemoryPool(
                base_hf_config=hf_cfg,
                max_loras_per_batch=1,
                dtype=torch.bfloat16,
                tp_size=tp_size,
                tp_rank=tp_rank,
                max_lora_rank=8,
                target_modules={"qkv_proj"},
                base_model=base_model,
                eviction_policy="lru",
                lora_added_tokens_size=0,
            )
```
**EN:** Construct a pool with `__init__` called, but stop before `init_buffers` — we only care about the EP-context state. This block implements `_new_pool_with_ep` and captures one focused piece of the module's behavior.
**CN:** Construct a pool with `__init__` called, but stop before `init_buffers` — we only care about the EP-context state. 该代码块实现 `_new_pool_with_ep`，承担模块行为中的一个聚焦逻辑片段。

### Lines 394-398: test case no ep / 测试用例 no ep
```python
    def test_no_ep(self):
        pool = self._new_pool_with_ep(ep_size=1, ep_rank=0, keeps_global=False)
        self.assertEqual(pool.moe_ep_size, 1)
        self.assertEqual(pool.moe_ep_rank, 0)
        self.assertFalse(pool.moe_use_local_expert_ids)
```
**EN:** This test exercises `test_no_ep` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_ep`。

### Lines 400-404: test case ep4 triton backend / 测试用例 ep4 triton backend
```python
    def test_ep4_triton_backend(self):
        pool = self._new_pool_with_ep(ep_size=4, ep_rank=2, keeps_global=False)
        self.assertEqual(pool.moe_ep_size, 4)
        self.assertEqual(pool.moe_ep_rank, 2)
        self.assertTrue(pool.moe_use_local_expert_ids)
```
**EN:** This test exercises `test_ep4_triton_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ep4_triton_backend`。

### Lines 406-413: test case ep4 flashinfer cutlass keeps global / 测试用例 ep4 flashinfer cutlass keeps global
```python
    def test_ep4_flashinfer_cutlass_keeps_global(self):
        """FlashInfer CUTLASS keeps global topk_ids, so LoRA buffers stay
        globally-keyed even under EP.
        """
        pool = self._new_pool_with_ep(ep_size=4, ep_rank=2, keeps_global=True)
        self.assertEqual(pool.moe_ep_size, 4)
        self.assertEqual(pool.moe_ep_rank, 2)
        self.assertFalse(pool.moe_use_local_expert_ids)
```
**EN:** FlashInfer CUTLASS keeps global topk_ids, so LoRA buffers stay globally-keyed even under EP. This test exercises `test_ep4_flashinfer_cutlass_keeps_global` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** FlashInfer CUTLASS keeps global topk_ids, so LoRA buffers stay globally-keyed even under EP. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ep4_flashinfer_cutlass_keeps_global`。

### Lines 415-426: test case ep with uneven split falls back to global ids / 测试用例 ep with uneven split falls back to global ids
```python
    def test_ep_with_uneven_split_falls_back_to_global_ids(self):
        """If `num_experts % ep_size != 0` (shouldn't happen in practice,
        base MoE requires even split) `__init__` must fall back to
        globally-keyed buffers rather than silently truncating the local
        slice — otherwise non-zero ranks drop every LoRA weight.
        """
        pool = self._new_pool_with_ep(
            ep_size=4, ep_rank=1, keeps_global=False, num_experts=7
        )
        self.assertEqual(pool.moe_ep_size, 4)
        self.assertEqual(pool.moe_ep_rank, 1)
        self.assertFalse(pool.moe_use_local_expert_ids)
```
**EN:** If `num_experts % ep_size != 0` (shouldn't happen in practice, base MoE requires even split) `__init__` must fall back to globally-keyed buffers rather than silently truncating the local slice — otherwise non-zero ranks drop every LoRA weight. This test exercises `test_ep_with_uneven_split_falls_back_to_global_ids` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** If `num_experts % ep_size != 0` (shouldn't happen in practice, base MoE requires even split) `__init__` must fall back to globally-keyed buffers rather than silently truncating the local slice — otherwise non-zero ranks drop every LoRA weight. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ep_with_uneven_split_falls_back_to_global_ids`。

### Lines 428-444: test case init captures moe tp context / 测试用例 init captures moe tp context
```python
    def test_init_captures_moe_tp_context(self):
        """`__init__` must capture moe_tp_size/rank so per-expert MoE LoRA
        buffers can be sharded by the MoE-TP group (not the outer attn TP).
        Under `--tp N --ep N` the MoE TP group degenerates to size 1.
        """
        pool = self._new_pool_with_ep(
            ep_size=4,
            ep_rank=0,
            keeps_global=False,
            tp_size=4,
            tp_rank=0,
            moe_tp_size=1,
            moe_tp_rank=0,
        )
        self.assertEqual(pool.tp_size, 4)
        self.assertEqual(pool.moe_tp_size, 1)
        self.assertEqual(pool.moe_tp_rank, 0)
```
**EN:** `__init__` must capture moe_tp_size/rank so per-expert MoE LoRA buffers can be sharded by the MoE-TP group (not the outer attn TP). This test exercises `test_init_captures_moe_tp_context` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** `__init__` must capture moe_tp_size/rank so per-expert MoE LoRA buffers can be sharded by the MoE-TP group (not the outer attn TP). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_captures_moe_tp_context`。

### Lines 447-484: function fake base model with hidden dim / 函数 fake base model with hidden dim
```python
def _fake_base_model_with_hidden_dim(num_experts: int) -> torch.nn.Module:
    """Fake base model that implements `get_hidden_dim` for MoE + attention
    modules. Matches the signatures `LoRAMemoryPool.get_lora_{A,B}_shape`
    call through `sglang.srt.lora.utils.get_hidden_dim`.
    """

    class _Model(torch.nn.Module):
        def __init__(self):
            super().__init__()
            self.lin = torch.nn.Linear(4, 4, bias=False)
            self.config = types.SimpleNamespace(
                num_hidden_layers=1,
                hidden_size=64,
                num_attention_heads=8,
                num_key_value_heads=8,
                head_dim=8,
                intermediate_size=256,
                moe_intermediate_size=192,
                vocab_size=32,
                num_experts=num_experts,
            )

        def get_hidden_dim(self, module_name: str, layer_idx: int):
            cfg = self.config
            if module_name == "qkv_proj":
                head = cfg.head_dim
                return cfg.hidden_size, head * (
                    cfg.num_attention_heads + cfg.num_key_value_heads * 2
                )
            if module_name == "o_proj":
                return cfg.head_dim * cfg.num_attention_heads, cfg.hidden_size
            if module_name == "gate_up_proj_moe":
                return cfg.hidden_size, cfg.moe_intermediate_size * 2
            if module_name == "down_proj_moe":
                return cfg.moe_intermediate_size, cfg.hidden_size
            raise NotImplementedError(module_name)

    return _Model()
```
**EN:** Fake base model that implements `get_hidden_dim` for MoE + attention modules. This block implements `_fake_base_model_with_hidden_dim` and captures one focused piece of the module's behavior.
**CN:** Fake base model that implements `get_hidden_dim` for MoE + attention modules. 该代码块实现 `_fake_base_model_with_hidden_dim`，承担模块行为中的一个聚焦逻辑片段。

### Lines 487-487: class TestMoeBufferShardsByMoeTp declaration / 类 TestMoeBufferShardsByMoeTp 声明
```python
class TestMoeBufferShardsByMoeTp(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 488-497: supporting statements / 辅助语句
```python
    """Regression: per-expert MoE LoRA buffers must shard by `moe_tp_size`,
    not the outer attention `tp_size`.

    Under `--tp N --ep N` (e.g. tp=4, ep=4) `moe_tp_size == 1`, so per-
    expert weights span the full MoE intermediate dim on every rank; the
    corresponding LoRA buffer must match. Before the fix, the buffer was
    divided by `tp_size` (= 4) while `FusedMoEWithLoRA.slice_moe_lora_*`
    kept the weight full-width, producing a 4x shape-mismatch assert at
    load time. Non-MoE modules still shard by the outer `tp_size`.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 499-529: method pool / 方法 pool
```python
    def _pool(
        self,
        *,
        tp_size: int,
        moe_tp_size: int,
        num_experts: int = 128,
        ep_size: int = 1,
        ep_rank: int = 0,
    ) -> LoRAMemoryPool:
        pool = LoRAMemoryPool.__new__(LoRAMemoryPool)
        pool.max_loras_per_batch = 2
        pool.tp_size = tp_size
        pool.tp_rank = 0
        pool.moe_ep_size = ep_size
        pool.moe_ep_rank = ep_rank
        pool.moe_tp_size = moe_tp_size
        pool.moe_tp_rank = 0
        pool.moe_use_local_expert_ids = ep_size > 1
        pool._num_experts_local = (
            num_experts // ep_size if pool.moe_use_local_expert_ids else num_experts
        )
        pool.experts_shared_outer_loras = False
        pool.base_hf_config = types.SimpleNamespace(
            hidden_size=64,
            num_attention_heads=8,
            num_key_value_heads=8,
            head_dim=8,
            intermediate_size=256,
            moe_intermediate_size=192,
        )
        return pool
```
**EN:** This block implements `_pool` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_pool`，承担模块行为中的一个聚焦逻辑片段。

### Lines 531-548: test case moe down proj uses moe tp not attn tp / 测试用例 moe down proj uses moe tp not attn tp
```python
    def test_moe_down_proj_uses_moe_tp_not_attn_tp(self):
        """down_proj_moe is row-parallel: LoRA-A input_dim = moe_inter must
        be divided by `moe_tp_size`, NOT `tp_size`. This is the exact shape
        that failed at load time on `--tp 4 --ep 4` before the fix.
        """
        pool = self._pool(tp_size=4, moe_tp_size=1, num_experts=128, ep_size=4)
        model = _fake_base_model_with_hidden_dim(num_experts=128)
        num_local = 128 // 4  # 32
        # A: input_dim = moe_inter / moe_tp_size = 192 / 1 = 192 (pre-fix: 48).
        self.assertEqual(
            pool.get_lora_A_shape("down_proj_moe", model, 8, 0),
            (2, num_local, 8, 192),
        )
        # B: output_dim = hidden_size, not row-parallel -> unsharded.
        self.assertEqual(
            pool.get_lora_B_shape("down_proj_moe", model, 8, 0),
            (2, num_local, 64, 8),
        )
```
**EN:** down_proj_moe is row-parallel: LoRA-A input_dim = moe_inter must be divided by `moe_tp_size`, NOT `tp_size`. This test exercises `test_moe_down_proj_uses_moe_tp_not_attn_tp` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** down_proj_moe is row-parallel: LoRA-A input_dim = moe_inter must be divided by `moe_tp_size`, NOT `tp_size`. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_down_proj_uses_moe_tp_not_attn_tp`。

### Lines 550-567: test case moe gate up proj uses moe tp not attn tp / 测试用例 moe gate up proj uses moe tp not attn tp
```python
    def test_moe_gate_up_proj_uses_moe_tp_not_attn_tp(self):
        """gate_up_proj_moe is column-parallel: LoRA-B output_dim =
        moe_inter*2 must be divided by `moe_tp_size`, not `tp_size`.
        """
        pool = self._pool(tp_size=4, moe_tp_size=1, num_experts=128, ep_size=4)
        model = _fake_base_model_with_hidden_dim(num_experts=128)
        num_local = 128 // 4
        # A: input_dim = hidden_size, not row-parallel -> unsharded. Rank
        # dim is `max_lora_dim * stacked_multiply` (2 for gate_up).
        self.assertEqual(
            pool.get_lora_A_shape("gate_up_proj_moe", model, 8, 0),
            (2, num_local, 16, 64),
        )
        # B: output_dim = moe_inter*2 / moe_tp_size = 384 / 1 = 384 (pre-fix: 96).
        self.assertEqual(
            pool.get_lora_B_shape("gate_up_proj_moe", model, 8, 0),
            (2, num_local, 384, 8),
        )
```
**EN:** gate_up_proj_moe is column-parallel: LoRA-B output_dim = moe_inter*2 must be divided by `moe_tp_size`, not `tp_size`. This test exercises `test_moe_gate_up_proj_uses_moe_tp_not_attn_tp` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** gate_up_proj_moe is column-parallel: LoRA-B output_dim = moe_inter*2 must be divided by `moe_tp_size`, not `tp_size`. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_gate_up_proj_uses_moe_tp_not_attn_tp`。

### Lines 569-591: test case moe tp gt1 still shards moe dims / 测试用例 moe tp gt1 still shards moe dims
```python
    def test_moe_tp_gt1_still_shards_moe_dims(self):
        """Under `--tp 8 --ep 4` the MoE TP group has size 2, so per-expert
        weights ARE sharded along the MoE inner dim — the LoRA buffer must
        follow.
        """
        pool = self._pool(tp_size=8, moe_tp_size=2, num_experts=128, ep_size=4)
        model = _fake_base_model_with_hidden_dim(num_experts=128)
        num_local = 128 // 4
        # 192 / 2 = 96
        self.assertEqual(
            pool.get_lora_A_shape("down_proj_moe", model, 8, 0),
            (2, num_local, 8, 96),
        )
        # 384 / 2 = 192 (B: moe_inter*2 / moe_tp_size).
        self.assertEqual(
            pool.get_lora_B_shape("gate_up_proj_moe", model, 8, 0),
            (2, num_local, 192, 8),
        )
        # A: input_dim = hidden_size, unaffected by MoE TP.
        self.assertEqual(
            pool.get_lora_A_shape("gate_up_proj_moe", model, 8, 0),
            (2, num_local, 16, 64),
        )
```
**EN:** Under `--tp 8 --ep 4` the MoE TP group has size 2, so per-expert weights ARE sharded along the MoE inner dim — the LoRA buffer must follow. This test exercises `test_moe_tp_gt1_still_shards_moe_dims` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Under `--tp 8 --ep 4` the MoE TP group has size 2, so per-expert weights ARE sharded along the MoE inner dim — the LoRA buffer must follow. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_tp_gt1_still_shards_moe_dims`。

### Lines 593-608: test case non moe modules unaffected by moe tp / 测试用例 non moe modules unaffected by moe tp
```python
    def test_non_moe_modules_unaffected_by_moe_tp(self):
        """Non-MoE modules must continue to shard by the outer `tp_size`;
        the MoE-TP substitution applies only to `*_moe` modules.
        """
        pool = self._pool(tp_size=4, moe_tp_size=1, num_experts=128, ep_size=4)
        model = _fake_base_model_with_hidden_dim(num_experts=128)
        # o_proj is row-parallel: A input_dim sharded by tp_size, B unsharded.
        o_a = pool.get_lora_A_shape("o_proj", model, 8, 0)
        o_b = pool.get_lora_B_shape("o_proj", model, 8, 0)
        # head_dim*num_heads / tp_size = 64 / 4 = 16; B output = hidden_size = 64.
        self.assertEqual(o_a, (2, 8, 16))
        self.assertEqual(o_b, (2, 64, 8))
        # qkv_proj is column-parallel: A unsharded, B sharded by tp_size.
        q_b = pool.get_lora_B_shape("qkv_proj", model, 8, 0)
        # head_dim * (heads + 2*kv_heads) / tp_size = 8 * 24 / 4 = 48.
        self.assertEqual(q_b, (2, 48, 8))
```
**EN:** Non-MoE modules must continue to shard by the outer `tp_size`; the MoE-TP substitution applies only to `*_moe` modules. This test exercises `test_non_moe_modules_unaffected_by_moe_tp` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Non-MoE modules must continue to shard by the outer `tp_size`; the MoE-TP substitution applies only to `*_moe` modules. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_moe_modules_unaffected_by_moe_tp`。

### Lines 611-611: class TestLoadBufferPassesMoeTpRankToSlice declaration / 类 TestLoadBufferPassesMoeTpRankToSlice 声明
```python
class TestLoadBufferPassesMoeTpRankToSlice(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 612-629: supporting statements / 辅助语句
```python
    """Regression: `load_lora_weight_to_buffer` must hand `moe_tp_rank` (not
    the outer `tp_rank`) to `slice_moe_lora_{a,b}_weights`.

    Per-expert MoE weights are sharded along
    `moe_tp_size = tp_size // ep_size // dp_size`, NOT the outer `tp_size`.
    The bug only surfaces when those two values differ — i.e. when
    `1 < ep_size < tp_size`. Concrete reproducer (`tp=4 ep=2`):

      moe_tp_size = 2; outer rank 3 has moe_tp_rank=1.
      `intermediate_size_per_partition = moe_inter / 2 = 384`.
      Slicing with the OUTER rank (3) computes `start = 3 * 384 = 1152`,
      which is past the full `moe_inter = 768`, returning a `[r, 0]`-shaped
      tensor that fails the shape-match assert in `load_lora_weight_tensor`.

    This test exercises `load_lora_weight_to_buffer` end-to-end with a
    minimal mocked `FusedMoEWithLoRA` whose slicer captures-and-raises so
    we don't need to satisfy buffer-copy shape constraints.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 631-631: class _StopAfterCapture declaration / 类 _StopAfterCapture 声明
```python
    class _StopAfterCapture(Exception):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `Exception`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `Exception`。

### Lines 632-634: supporting statements / 辅助语句
```python
        """Sentinel raised from the mocked slicer to short-circuit
        execution before the buffer-copy phase (which would need real
        shapes the test does not provide)."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 636-715: test case moe tp rank used for slicing when ep lt tp (part 1/2) / 测试用例 moe tp rank used for slicing when ep lt tp（第 1/2 部分）
```python
    def test_moe_tp_rank_used_for_slicing_when_ep_lt_tp(self):
        from sglang.srt.lora.layers import FusedMoEWithLoRA

        # tp=4 ep=2 → moe_tp_size=2. Pick OUTER rank 3 so moe_tp_rank=1.
        # The two values differ; the bug would surface on this exact rank.
        pool = LoRAMemoryPool.__new__(LoRAMemoryPool)
        pool.tp_size = 4
        pool.tp_rank = 3
        pool.moe_tp_size = 2
        pool.moe_tp_rank = 1
        pool.moe_ep_size = 2
        pool.moe_ep_rank = 1
        pool.moe_use_local_expert_ids = True
        pool._num_experts_local = 1
        pool.num_layer = 1
        pool.target_modules = {"gate_up_proj", "down_proj"}
        pool.experts_shared_outer_loras = False
        pool.strict_loading = False
        pool.lora_added_tokens_size = 0
        # Tiny placeholder buffers — the mocked slicer raises before any of
        # this is read in the buffer-copy phase.
        pool.A_buffer = {
            "gate_up_proj_moe": [torch.zeros(1, 1, 1, 1)],
            "down_proj_moe": [torch.zeros(1, 1, 1, 1)],
        }
        pool.B_buffer = {
            "gate_up_proj_moe": [torch.zeros(1, 1, 1, 1)],
            "down_proj_moe": [torch.zeros(1, 1, 1, 1)],
        }
        pool.embedding_A_buffer = {}
        pool.embedding_B_buffer = {}
        pool.lm_head_A_buffer = {}
        pool.lm_head_B_buffer = {}
        pool.new_embeddings_buffer = {}

        captured_ranks = []

        moe_mod = mock.MagicMock(spec=FusedMoEWithLoRA)

        def capture_a(weights, tp_rank, target_module):
            captured_ranks.append(("A", target_module, tp_rank))
            raise TestLoadBufferPassesMoeTpRankToSlice._StopAfterCapture()

        def capture_b(weights, tp_rank, target_module):
            captured_ranks.append(("B", target_module, tp_rank))
            raise TestLoadBufferPassesMoeTpRankToSlice._StopAfterCapture()

        moe_mod.slice_moe_lora_a_weights.side_effect = capture_a
        moe_mod.slice_moe_lora_b_weights.side_effect = capture_b

        # Adapter with one per-expert MoE LoRA-A weight. The expert regex
        # `experts\.(\d+)\.` must match the key, which routes the weight
        # into `temp_A_buffer["gate_up_proj_moe"]` — the dict shape that
        # makes `temp_A_buffer.get("gate_up_proj_moe") is not None` true,
        # which in turn triggers `slice_moe_lora_a_weights` (and the
        # capture).
        adapter = mock.MagicMock()
        adapter.config.r = 4
        adapter.scaling = 1.0
        adapter.embedding_layers = {}
        adapter.added_tokens_embeddings = {}
        adapter.layers = [
            types.SimpleNamespace(
                weights={
                    "model.layers.0.mlp.experts.0.gate_up_proj.lora_A.weight": (
                        torch.zeros(8, 4)
                    ),
                },
            )
        ]

        with self.assertRaises(TestLoadBufferPassesMoeTpRankToSlice._StopAfterCapture):
            pool.load_lora_weight_to_buffer(
                uid="test",
                buffer_id=0,
                lora_adapter=adapter,
                lora_modules=[{"mlp.experts": moe_mod}],
                lora_embed_tokens_module=None,
                lora_lm_head_module=None,
            )
```
**EN:** This test exercises `test_moe_tp_rank_used_for_slicing_when_ep_lt_tp` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_tp_rank_used_for_slicing_when_ep_lt_tp`。 这一段对应同一逻辑块的第 1 部分。

### Lines 716-727: test case moe tp rank used for slicing when ep lt tp (part 2/2) / 测试用例 moe tp rank used for slicing when ep lt tp（第 2/2 部分）
```python

        self.assertGreater(len(captured_ranks), 0, "slicing was never invoked")
        for ab, target_module, rank in captured_ranks:
            self.assertEqual(
                rank,
                pool.moe_tp_rank,
                f"slice_moe_lora_{ab.lower()}_weights for {target_module} "
                f"received rank={rank}; expected moe_tp_rank="
                f"{pool.moe_tp_rank} (outer tp_rank is {pool.tp_rank}). "
                "Passing the outer tp_rank slices past "
                "intermediate_size_per_partition when ep_size < tp_size.",
            )
```
**EN:** This test exercises `test_moe_tp_rank_used_for_slicing_when_ep_lt_tp` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_tp_rank_used_for_slicing_when_ep_lt_tp`。 这一段对应同一逻辑块的第 2 部分。

### Lines 730-731: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_pool`: Construct a minimal LoRAMemoryPool for helper-level tests. / 该代码块实现 `_make_pool`，承担模块行为中的一个聚焦逻辑片段。
- `_make_fake_base_model`: Return a `torch.nn.Module` whose `.config` exposes `num_experts`. / 该代码块实现 `_make_fake_base_model`，承担模块行为中的一个聚焦逻辑片段。
- `TestNumExpertHelpers`: `_get_num_experts` / `_get_num_local_experts` / buffer-dim picker. / 用于组织相关测试、夹具或辅助方法。
- `TestGlobalToLocalExpertId`: `_global_to_local_expert_id` — the per-rank filter + remap. / 用于组织相关测试、夹具或辅助方法。
- `TestIterLocalExpertWeightsDict`: `_iter_local_expert_weights` with dict input (the common case). / 用于组织相关测试、夹具或辅助方法。
- `TestIterLocalExpertWeightsTensor`: `_iter_local_expert_weights` with 3D tensor input (shared-outer and packed MoE-LoRA formats). / 用于组织相关测试、夹具或辅助方法。
- `TestModuleLevelHelpers`: `_get_moe_ep_context` / `_moe_runner_keeps_global_expert_ids` must degrade gracefully when the MoE EP group or runner backend is not yet initialized (e.g. / 用于组织相关测试、夹具或辅助方法。
- `TestPoolInitPicksUpEpContext`: `LoRAMemoryPool.__init__` should read EP context from the module- level helpers and set `moe_use_local_expert_ids` correctly. / 用于组织相关测试、夹具或辅助方法。
- `TestNumExpertHelpers.test_num_experts_read_from_config`: This test exercises `test_num_experts_read_from_config` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_num_experts_read_from_config`。
- `TestNumExpertHelpers.test_num_local_experts_no_ep`: This test exercises `test_num_local_experts_no_ep` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_num_local_experts_no_ep`。
- `TestNumExpertHelpers.test_num_local_experts_with_ep`: This test exercises `test_num_local_experts_with_ep` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_num_local_experts_with_ep`。
- `TestNumExpertHelpers.test_num_local_experts_with_ep_but_backend_keeps_global_ids`: FlashInfer-style backends keep global topk_ids, so even under EP the LoRA buffers must remain globally-keyed. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_num_local_experts_with_ep_but_backend_keeps_global_ids`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `types`, `unittest`, `unittest.mock`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.lora.mem_pool`

- **Total lines / 总行数**: 731
