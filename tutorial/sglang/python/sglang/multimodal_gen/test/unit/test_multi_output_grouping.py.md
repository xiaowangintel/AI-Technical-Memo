# test_multi_output_grouping.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/unit/test_multi_output_grouping.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates multi output grouping with focused assertions and fixtures. Key symbols include `CountingDedupStage`, `CountingLatentStage`, `TestMultiOutputGrouping`. / 该测试模块通过有针对性的断言与夹具，验证 multi output grouping 的实现。 关键符号包括 `CountingDedupStage`, `CountingLatentStage`, `TestMultiOutputGrouping`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and module setup / 导入与模块初始化
```python
import unittest
from types import SimpleNamespace

import torch

from sglang.multimodal_gen.configs.sample.sampling_params import SamplingParams
from sglang.multimodal_gen.runtime.entrypoints.utils import (
    expand_request_outputs,
    normalize_output_seeds,
)
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
from sglang.multimodal_gen.runtime.pipelines_core.stages.base import PipelineStage
from sglang.multimodal_gen.runtime.pipelines_core.stages.latent_preparation import (
    LatentPreparationStage,
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 18-38: Class `CountingDedupStage` / 类 `CountingDedupStage`
```python
class CountingDedupStage(PipelineStage):
    deduplicated_output_fields = ("prompt_embeds",)
    deduplicated_tensor_tree_output_fields = ("timesteps",)
    deduplicated_deepcopy_output_fields = ("scheduler",)
    deduplicated_extra_tensor_tree_output_keys = ("mu",)

    def __init__(self):
        self.server_args = SimpleNamespace(comfyui_mode=True)
        self.forward_calls = 0

    def build_dedup_fingerprint(self, batch: Req, server_args):
        return batch.prompt

    def forward(self, batch: Req, server_args) -> Req:
# ...
        batch.timesteps = torch.tensor([value])
        batch.scheduler = {"state": [value]}
        batch.extra["mu"] = torch.tensor([value])
        return batch
```
**EN:** This class models `CountingDedupStage` as a specialization of `PipelineStage`. Important methods include `__init__`, `build_dedup_fingerprint`, `forward`.
**CN:** 该类实现 `CountingDedupStage`，并继承/扩展 `PipelineStage`。 其中较重要的方法包括 `__init__`, `build_dedup_fingerprint`, `forward`。

### Lines 41-67: Class `CountingLatentStage` / 类 `CountingLatentStage`
```python
class CountingLatentStage(LatentPreparationStage):
    def __init__(self):
        self.server_args = SimpleNamespace(comfyui_mode=True)
        self.prepare_group_calls = 0
        self.forward_calls = 0

    def build_dedup_fingerprint(self, batch: Req, server_args):
        return batch.prompt

    def _prepare_grouped_latents(
        self,
        batches: list[Req],
        server_args,
    ) -> Req:
# ...
    def forward(self, batch: Req, server_args) -> Req:
        self.forward_calls += 1
        batch.latents = torch.tensor([[[100.0 + self.forward_calls]]])
        return batch
```
**EN:** This class models `CountingLatentStage` as a specialization of `LatentPreparationStage`. Important methods include `__init__`, `build_dedup_fingerprint`, `_prepare_grouped_latents`, `forward`.
**CN:** 该类实现 `CountingLatentStage`，并继承/扩展 `LatentPreparationStage`。 其中较重要的方法包括 `__init__`, `build_dedup_fingerprint`, `_prepare_grouped_latents`, `forward`。

### Lines 70-194: Class `TestMultiOutputGrouping` / 类 `TestMultiOutputGrouping`
```python
class TestMultiOutputGrouping(unittest.TestCase):
    def test_normalize_output_seeds_from_int(self):
        self.assertEqual(
            normalize_output_seeds(10, num_outputs_per_prompt=3),
            [10, 11, 12],
        )

    def test_normalize_output_seeds_from_per_prompt_list(self):
        self.assertEqual(
            normalize_output_seeds([3, 5], num_outputs_per_prompt=2),
            [3, 5],
        )

    def test_normalize_output_seeds_from_total_list(self):
# ...
        self.assertTrue(torch.equal(reqs[0].latents, torch.tensor([[[0.0]]])))
        self.assertTrue(torch.equal(reqs[1].latents, torch.tensor([[[1.0]]])))
        self.assertTrue(torch.equal(reqs[0].latent_ids, torch.tensor([[[10.0]]])))
        self.assertTrue(torch.equal(reqs[1].latent_ids, torch.tensor([[[11.0]]])))
```
**EN:** This class models `TestMultiOutputGrouping` as a specialization of `unittest.TestCase`. Important methods include `test_normalize_output_seeds_from_int`, `test_normalize_output_seeds_from_per_prompt_list`, `test_normalize_output_seeds_from_total_list`, `test_normalize_output_seeds_rejects_mismatched_list`.
**CN:** 该类实现 `TestMultiOutputGrouping`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `test_normalize_output_seeds_from_int`, `test_normalize_output_seeds_from_per_prompt_list`, `test_normalize_output_seeds_from_total_list`, `test_normalize_output_seeds_rejects_mismatched_list`。

### Lines 195-198: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Sampling parameter control / 采样参数控制
- Automated verification / 自动化验证

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.entrypoints.utils`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.base`, `sglang.multimodal_gen.runtime.pipelines_core.stages.latent_preparation`
- **External / 外部**: `unittest`, `torch`
- **Stdlib / 标准库**: `types`
