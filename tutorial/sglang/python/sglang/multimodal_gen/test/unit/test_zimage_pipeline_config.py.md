# test_zimage_pipeline_config.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/unit/test_zimage_pipeline_config.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates zimage pipeline config with focused assertions and fixtures. Key symbols include `TestZImagePipelineConfig`. / 该测试模块通过有针对性的断言与夹具，验证 zimage pipeline config 的实现。 关键符号包括 `TestZImagePipelineConfig`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports and module setup / 导入与模块初始化
```python
import unittest
from types import SimpleNamespace
from unittest.mock import patch

import torch

from sglang.multimodal_gen.configs.pipeline_configs.zimage import ZImagePipelineConfig
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 10-41: Class `TestZImagePipelineConfig` / 类 `TestZImagePipelineConfig`
```python
class TestZImagePipelineConfig(unittest.TestCase):
    @patch("sglang.multimodal_gen.configs.pipeline_configs.zimage.get_sp_world_size")
    def test_zimage_negative_prompt_rotary_embeddings_use_negative_prompt_len(
        self, mock_get_sp_world_size
    ) -> None:
        """Negative CFG branch should build RoPE positions from negative prompt embeds."""
        mock_get_sp_world_size.return_value = 1

        config = ZImagePipelineConfig()
        pos_seq_len = 19
        neg_seq_len = 45
        batch = SimpleNamespace(
            prompt_embeds=[torch.ones(pos_seq_len, 2560)],
            negative_prompt_embeds=[torch.ones(neg_seq_len, 2560)],
# ...
        cap_pos_ids, image_pos_ids = neg_kwargs["freqs_cis"]
        neg_cap_padded_len = 64
        self.assertEqual(cap_pos_ids.shape, (neg_cap_padded_len, 3))
        self.assertEqual(image_pos_ids[0].tolist(), [neg_cap_padded_len + 1, 0, 0])
```
**EN:** This class models `TestZImagePipelineConfig` as a specialization of `unittest.TestCase`. Important methods include `test_zimage_negative_prompt_rotary_embeddings_use_negative_prompt_len`.
**CN:** 该类实现 `TestZImagePipelineConfig`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `test_zimage_negative_prompt_rotary_embeddings_use_negative_prompt_len`。

### Lines 42-45: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Automated verification / 自动化验证
- Image generation flow / 图像生成流程
- Symbol `TestZImagePipelineConfig` anchors the module API / 符号 `TestZImagePipelineConfig` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.pipeline_configs.zimage`
- **External / 外部**: `unittest`, `unittest.mock`, `torch`
- **Stdlib / 标准库**: `types`
