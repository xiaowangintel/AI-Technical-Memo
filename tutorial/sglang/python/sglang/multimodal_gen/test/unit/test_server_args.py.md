# test_server_args.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/unit/test_server_args.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates server args with focused assertions and fixtures. Key symbols include `TestServerArgsPathExpansion`, `TestOffloadDefaults`, `TestFSDPShardConditions`. / 该测试模块通过有针对性的断言与夹具，验证 server args 的实现。 关键符号包括 `TestServerArgsPathExpansion`, `TestOffloadDefaults`, `TestFSDPShardConditions`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: Imports and module setup / 导入与模块初始化
```python
import os
import sys
import unittest
from unittest.mock import patch

from sglang.multimodal_gen.configs.models.fsdp import (
    is_module_list_entry,
    is_module_list_entry_in,
    is_zimage_layer,
)
from sglang.multimodal_gen.configs.pipeline_configs.base import (
    ModelTaskType,
    PipelineConfig,
)
# ...
    QwenImageTransformer2DModel,
)
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.utils import FlexibleArgumentParser
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 30-263: Class `TestServerArgsPathExpansion` / 类 `TestServerArgsPathExpansion`
```python
class TestServerArgsPathExpansion(unittest.TestCase):
    def _from_dict_without_model_resolution(self, kwargs):
        with patch.object(
            PipelineConfig, "from_kwargs", return_value=QwenImagePipelineConfig()
        ):
            return ServerArgs.from_dict(kwargs)

    def test_tilde_model_path_is_expanded(self):
        args = self._from_dict_without_model_resolution(
            {"model_path": "~/fake/local/model"}
        )
        expected = os.path.expanduser("~/fake/local/model")
        self.assertEqual(args.model_path, expected)
        self.assertFalse(args.model_path.startswith("~"))
# ...

        self.assertEqual(
            server_args.layerwise_offload_components, ["transformer", "text_encoder"]
        )
```
**EN:** This class models `TestServerArgsPathExpansion` as a specialization of `unittest.TestCase`. Important methods include `_from_dict_without_model_resolution`, `test_tilde_model_path_is_expanded`, `test_absolute_path_is_unchanged`, `test_component_paths_are_expanded_before_pipeline_resolution`.
**CN:** 该类实现 `TestServerArgsPathExpansion`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `_from_dict_without_model_resolution`, `test_tilde_model_path_is_expanded`, `test_absolute_path_is_unchanged`, `test_component_paths_are_expanded_before_pipeline_resolution`。

### Lines 266-851: Class `TestOffloadDefaults` / 类 `TestOffloadDefaults`
```python
class TestOffloadDefaults(unittest.TestCase):
    def _from_dict_with_pipeline_config(
        self,
        pipeline_config,
        *,
        memory_gb=80,
        available_memory_gb=None,
        kwargs=None,
    ):
        def get_available_gpu_memory(device_id=0, **_kwargs):
            if isinstance(available_memory_gb, dict):
                return available_memory_gb[device_id]
            if available_memory_gb is not None:
                return available_memory_gb
# ...
            server_args = ServerArgs.from_cli_args(args, unknown_args)

        self.assertFalse(server_args.use_fsdp_inference)
        self.assertFalse(server_args.enable_cfg_parallel)
```
**EN:** This class models `TestOffloadDefaults` as a specialization of `unittest.TestCase`. Important methods include `_from_dict_with_pipeline_config`, `_from_dict_with_task_type`, `test_vae_cpu_offload_defaults_false_for_video_generation`, `test_vae_cpu_offload_defaults_false_on_low_memory_gpu`.
**CN:** 该类实现 `TestOffloadDefaults`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `_from_dict_with_pipeline_config`, `_from_dict_with_task_type`, `test_vae_cpu_offload_defaults_false_for_video_generation`, `test_vae_cpu_offload_defaults_false_on_low_memory_gpu`。

### Lines 854-886: Class `TestFSDPShardConditions` / 类 `TestFSDPShardConditions`
```python
class TestFSDPShardConditions(unittest.TestCase):
    def test_helpers_match_only_direct_block_entries(self):
        self.assertTrue(
            is_module_list_entry("transformer_blocks.0", "transformer_blocks")
        )
        self.assertFalse(
            is_module_list_entry("transformer_blocks.0.ff.net.0", "transformer_blocks")
        )
        self.assertTrue(
            is_module_list_entry_in(
                "single_transformer_blocks.12",
                ("transformer_blocks", "single_transformer_blocks"),
            )
        )
# ...
    def test_zimage_condition_keeps_inner_numbered_modules(self):
        self.assertTrue(is_zimage_layer("layers.0.mlp.0", None))
        self.assertTrue(is_zimage_layer("noise_refiner.0.attention.to_out.0", None))
        self.assertFalse(is_zimage_layer("transformer_blocks.0", None))
```
**EN:** This class models `TestFSDPShardConditions` as a specialization of `unittest.TestCase`. Important methods include `test_helpers_match_only_direct_block_entries`, `test_qwen_dit_has_fsdp_shard_condition`, `test_zimage_condition_keeps_inner_numbered_modules`.
**CN:** 该类实现 `TestFSDPShardConditions`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `test_helpers_match_only_direct_block_entries`, `test_qwen_dit_has_fsdp_shard_condition`, `test_zimage_condition_keeps_inner_numbered_modules`。

### Lines 889-921: Class `TestModelIdResolution` / 类 `TestModelIdResolution`
```python
class TestModelIdResolution(unittest.TestCase):
    def setUp(self):
        _get_config_info.cache_clear()

    def test_model_id_overrides_arbitrary_local_path(self):
        # a local path whose directory name does not match any HF repo name;
        # --model-id tells the engine which config to use
        info = _get_config_info("/data/my-custom-qwen", model_id="Qwen-Image")
        self.assertIsNotNone(info)

        self.assertIs(info.pipeline_config_cls, QwenImagePipelineConfig)

    def test_model_id_works_after_tilde_expansion(self):
        # simulate the full flow: user passes ~/..., engine expands and resolves
# ...
        # unrecognized model_id: should warn and fall back to path-based detection
        # with an unresolvable path, expect RuntimeError from the detector step
        with self.assertRaises((RuntimeError, Exception)):
            _get_config_info("/data/no-such-model", model_id="NonExistentModelXYZ")
```
**EN:** This class models `TestModelIdResolution` as a specialization of `unittest.TestCase`. Important methods include `setUp`, `test_model_id_overrides_arbitrary_local_path`, `test_model_id_works_after_tilde_expansion`, `test_hf_cache_snapshot_path_resolves_registered_nvfp4_model`.
**CN:** 该类实现 `TestModelIdResolution`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `setUp`, `test_model_id_overrides_arbitrary_local_path`, `test_model_id_works_after_tilde_expansion`, `test_hf_cache_snapshot_path_resolves_registered_nvfp4_model`。

### Lines 924-1030: Class `TestPerRoleParallelism` / 类 `TestPerRoleParallelism`
```python
class TestPerRoleParallelism(unittest.TestCase):
    """Test per-role parallelism args and get_role_parallelism helper."""

    def _from_dict(self, kwargs):
        with patch.object(
            PipelineConfig, "from_kwargs", return_value=QwenImagePipelineConfig()
        ):
            return ServerArgs.from_dict(kwargs)

    def test_defaults_are_none(self):
        args = self._from_dict({"model_path": "/fake"})
        from sglang.multimodal_gen.runtime.disaggregation.roles import RoleType

        for role in [RoleType.ENCODER, RoleType.DENOISER, RoleType.DECODER]:
# ...
        self.assertEqual(args.denoiser_ulysses, 2)
        self.assertEqual(args.denoiser_ring, 2)
        self.assertEqual(args.encoder_tp, 1)
        self.assertIsNone(args.decoder_tp)
```
**EN:** This class models `TestPerRoleParallelism` as a specialization of `unittest.TestCase`. Test per-role parallelism args and get_role_parallelism helper. Important methods include `_from_dict`, `test_defaults_are_none`, `test_encoder_overrides`, `test_denoiser_overrides`.
**CN:** 该类实现 `TestPerRoleParallelism`，并继承/扩展 `unittest.TestCase`。 文档字符串指出：Test per-role parallelism args and get_role_parallelism helper. 其中较重要的方法包括 `_from_dict`, `test_defaults_are_none`, `test_encoder_overrides`, `test_denoiser_overrides`。

### Lines 1033-1068: Class `TestPipelineResolutionCliOverride` / 类 `TestPipelineResolutionCliOverride`
```python
class TestPipelineResolutionCliOverride(unittest.TestCase):
    def setUp(self):
        _get_config_info.cache_clear()

    def test_resolution_flag_overrides_qwen_image_layered_pipeline_config(self):
        parser = FlexibleArgumentParser()
        ServerArgs.add_cli_args(parser)
        argv = [
            "--model-path",
            "Qwen/Qwen-Image-Layered",
            "--resolution",
            "768",
        ]

# ...
            server_args = ServerArgs.from_cli_args(args, unknown_args)

        self.assertTrue(server_args.pipeline_config.disable_autocast)
        self.assertTrue(server_args.disable_autocast)
```
**EN:** This class models `TestPipelineResolutionCliOverride` as a specialization of `unittest.TestCase`. Important methods include `setUp`, `test_resolution_flag_overrides_qwen_image_layered_pipeline_config`, `test_disable_autocast_is_preserved_after_pipeline_config_resolution`.
**CN:** 该类实现 `TestPipelineResolutionCliOverride`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `setUp`, `test_resolution_flag_overrides_qwen_image_layered_pipeline_config`, `test_disable_autocast_is_preserved_after_pipeline_config_resolution`。

### Lines 1069-1072: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Pipeline orchestration / 流水线编排
- Caching strategy / 缓存策略
- Request disaggregation / 请求解耦

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.fsdp`, `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.configs.pipeline_configs.ltx_2`, `sglang.multimodal_gen.configs.pipeline_configs.mova`, `sglang.multimodal_gen.configs.pipeline_configs.qwen_image`, `sglang.multimodal_gen.configs.pipeline_configs.wan`, `sglang.multimodal_gen.configs.pipeline_configs.zimage`, `sglang.multimodal_gen.registry`
- **External / 外部**: `unittest`, `unittest.mock`
- **Stdlib / 标准库**: `os`, `sys`
