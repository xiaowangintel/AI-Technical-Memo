# test_mm_process_config.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/core/test_mm_process_config.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on core mm process config in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 core mm process config 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Import dependencies
```python
import unittest
from unittest.mock import MagicMock, patch

from sglang.srt.server_args import ServerArgs
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 7-8: Register CI metadata
```python
register_cuda_ci(est_time=9, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=1, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 11-11: Define class TestMmProcessConfigValidation
```python
class TestMmProcessConfigValidation(unittest.TestCase):
```
**EN:** This declaration introduces the `TestMmProcessConfigValidation` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMmProcessConfigValidation` 测试类，并说明它通过继承承担的职责。

### Lines 12-12: Document the class `TestMmProcessConfigValidation`
```python
    """Server-args validation for mm_process_config."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestMmProcessConfigValidation`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestMmProcessConfigValidation`的设计意图。

### Lines 14-19: Run test: valid config accepted
```python
    def test_valid_config_accepted(self):
        args = ServerArgs(
            model_path="dummy",
            mm_process_config={"image": {"max_pixels": 5000000}},
        )
        self.assertEqual(args.mm_process_config, {"image": {"max_pixels": 5000000}})
```
**EN:** This test method exercises valid config accepted and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 valid config accepted 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 21-23: Run test: empty config accepted
```python
    def test_empty_config_accepted(self):
        args = ServerArgs(model_path="dummy", mm_process_config={})
        self.assertEqual(args.mm_process_config, {})
```
**EN:** This test method exercises empty config accepted and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty config accepted 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 25-29: Run test: none config defaults to empty dict
```python
    def test_none_config_defaults_to_empty_dict(self):
        args = ServerArgs(model_path="dummy", mm_process_config=None)
        # None is kept as-is for dummy models (default happens after early return)
        # but for real models it would be set to {}
        self.assertIsNone(args.mm_process_config)
```
**EN:** This test method exercises none config defaults to empty dict and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 none config defaults to empty dict 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 31-34: Run test: top level non dict rejected
```python
    def test_top_level_non_dict_rejected(self):
        with self.assertRaises(TypeError) as ctx:
            ServerArgs(model_path="dummy", mm_process_config="bad")
        self.assertIn("mm_process_config must be a dict", str(ctx.exception))
```
**EN:** This test method exercises top level non dict rejected and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 top level non dict rejected 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 36-39: Run test: modality non dict rejected image
```python
    def test_modality_non_dict_rejected_image(self):
        with self.assertRaises(TypeError) as ctx:
            ServerArgs(model_path="dummy", mm_process_config={"image": "bad"})
        self.assertIn("mm_process_config['image'] must be a dict", str(ctx.exception))
```
**EN:** This test method exercises modality non dict rejected image and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 modality non dict rejected image 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 41-44: Run test: modality non dict rejected video
```python
    def test_modality_non_dict_rejected_video(self):
        with self.assertRaises(TypeError) as ctx:
            ServerArgs(model_path="dummy", mm_process_config={"video": 123})
        self.assertIn("mm_process_config['video'] must be a dict", str(ctx.exception))
```
**EN:** This test method exercises modality non dict rejected video and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 modality non dict rejected video 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 46-49: Run test: modality non dict rejected audio
```python
    def test_modality_non_dict_rejected_audio(self):
        with self.assertRaises(TypeError) as ctx:
            ServerArgs(model_path="dummy", mm_process_config={"audio": [1, 2]})
        self.assertIn("mm_process_config['audio'] must be a dict", str(ctx.exception))
```
**EN:** This test method exercises modality non dict rejected audio and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 modality non dict rejected audio 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 51-58: Run test: multi modality config accepted
```python
    def test_multi_modality_config_accepted(self):
        config = {
            "image": {"max_pixels": 1048576},
            "video": {"max_pixels": 602112},
            "audio": {"sample_rate": 16000},
        }
        args = ServerArgs(model_path="dummy", mm_process_config=config)
        self.assertEqual(args.mm_process_config, config)
```
**EN:** This test method exercises multi modality config accepted and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi modality config accepted 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 61-61: Define class TestBaseProcessorConfigExtraction
```python
class TestBaseProcessorConfigExtraction(unittest.TestCase):
```
**EN:** This declaration introduces the `TestBaseProcessorConfigExtraction` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestBaseProcessorConfigExtraction` 测试类，并说明它通过继承承担的职责。

### Lines 62-62: Document the class `TestBaseProcessorConfigExtraction`
```python
    """Verify BaseMultimodalProcessor.__init__ extracts configs from server_args."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestBaseProcessorConfigExtraction`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestBaseProcessorConfigExtraction`的设计意图。

### Lines 64-84: Define helper: make processor
```python
    def _make_processor(self, mm_process_config):
        """Create a BaseMultimodalProcessor via the real __init__ with mocked deps."""
        from sglang.srt.multimodal.processors.base_processor import (
            BaseMultimodalProcessor,
        )

        server_args = MagicMock()
        server_args.mm_process_config = mm_process_config

        hf_config = MagicMock()
        mock_hf_processor = MagicMock()

        # Call real __init__ so we test actual config extraction
        with patch.object(BaseMultimodalProcessor, "__abstractmethods__", set()):
            proc = BaseMultimodalProcessor(
                hf_config=hf_config,
                server_args=server_args,
                _processor=mock_hf_processor,
                transport_mode=None,
            )
        return proc
```
**EN:** This helper function encapsulates reusable logic inside `TestBaseProcessorConfigExtraction` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestBaseProcessorConfigExtraction` 内部调用，从而让场景结构更清晰。

### Lines 86-95: Run test: configs extracted
```python
    def test_configs_extracted(self):
        config = {
            "image": {"max_pixels": 5000000},
            "video": {"fps": 3},
            "audio": {"sample_rate": 16000},
        }
        proc = self._make_processor(config)
        self.assertEqual(proc.image_config, {"max_pixels": 5000000})
        self.assertEqual(proc.video_config, {"fps": 3})
        self.assertEqual(proc.audio_config, {"sample_rate": 16000})
```
**EN:** This test method exercises configs extracted and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 configs extracted 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 97-101: Run test: empty config yields empty dicts
```python
    def test_empty_config_yields_empty_dicts(self):
        proc = self._make_processor({})
        self.assertEqual(proc.image_config, {})
        self.assertEqual(proc.video_config, {})
        self.assertEqual(proc.audio_config, {})
```
**EN:** This test method exercises empty config yields empty dicts and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty config yields empty dicts 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 104-104: Define class TestProcessMmDataKwargs
```python
class TestProcessMmDataKwargs(unittest.TestCase):
```
**EN:** This declaration introduces the `TestProcessMmDataKwargs` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestProcessMmDataKwargs` 测试类，并说明它通过继承承担的职责。

### Lines 105-105: Document the class `TestProcessMmDataKwargs`
```python
    """Verify process_mm_data injects per-modality kwargs correctly."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestProcessMmDataKwargs`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestProcessMmDataKwargs`的设计意图。

### Lines 107-140: Define helper: make base processor
```python
    def _make_base_processor(self, mm_process_config):
        """Create a BaseMultimodalProcessor with process_mm_data testable."""
        from sglang.srt.multimodal.processors.base_processor import (
            BaseMultimodalProcessor,
        )

        server_args = MagicMock()
        server_args.mm_process_config = mm_process_config
        server_args.disable_fast_image_processor = True
        server_args.keep_mm_feature_on_device = True

        mock_processor = MagicMock()
        mock_processor.__class__.__name__ = "TestProcessor"
        # Capture kwargs passed to __call__
        captured_kwargs = {}

        def capture_call(**kwargs):
            captured_kwargs.update(kwargs)
            return {}

        mock_processor.__call__ = MagicMock(side_effect=capture_call)

        with patch.object(BaseMultimodalProcessor, "__abstractmethods__", set()):
            with patch.object(BaseMultimodalProcessor, "__init__", lambda self: None):
                proc = BaseMultimodalProcessor()

        proc.server_args = server_args
        proc._processor = mock_processor
        proc.image_config = mm_process_config.get("image", {})
        proc.video_config = mm_process_config.get("video", {})
        proc.audio_config = mm_process_config.get("audio", {})
        proc.FEATURE_NAMES = []

        return proc, mock_processor, captured_kwargs
```
**EN:** This helper function encapsulates reusable logic inside `TestProcessMmDataKwargs` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestProcessMmDataKwargs` 内部调用，从而让场景结构更清晰。

### Lines 142-151: Run test: images kwargs injected
```python
    def test_images_kwargs_injected(self):
        config = {"image": {"max_pixels": 5000000}}
        proc, mock_proc, _ = self._make_base_processor(config)

        proc.process_mm_data("test", images=["img1"])

        call_kwargs = mock_proc.__call__.call_args
        self.assertEqual(
            call_kwargs.kwargs.get("images_kwargs"), {"max_pixels": 5000000}
        )
```
**EN:** This test method exercises images kwargs injected and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 images kwargs injected 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 153-162: Run test: videos kwargs injected
```python
    def test_videos_kwargs_injected(self):
        config = {"video": {"fps": 3, "max_frames": 60}}
        proc, mock_proc, _ = self._make_base_processor(config)

        proc.process_mm_data("test", videos=["vid1"])

        call_kwargs = mock_proc.__call__.call_args
        self.assertEqual(
            call_kwargs.kwargs.get("videos_kwargs"), {"fps": 3, "max_frames": 60}
        )
```
**EN:** This test method exercises videos kwargs injected and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 videos kwargs injected 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 164-180: Run test: no collision with overlapping keys
```python
    def test_no_collision_with_overlapping_keys(self):
        """Core test: image and video both have max_pixels but stay separate."""
        config = {
            "image": {"max_pixels": 1048576},
            "video": {"max_pixels": 602112},
        }
        proc, mock_proc, _ = self._make_base_processor(config)

        proc.process_mm_data("test", images=["img1"], videos=["vid1"])

        call_kwargs = mock_proc.__call__.call_args
        self.assertEqual(
            call_kwargs.kwargs.get("images_kwargs"), {"max_pixels": 1048576}
        )
        self.assertEqual(
            call_kwargs.kwargs.get("videos_kwargs"), {"max_pixels": 602112}
        )
```
**EN:** This test method exercises no collision with overlapping keys and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no collision with overlapping keys 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 182-188: Run test: empty config no kwargs injected
```python
    def test_empty_config_no_kwargs_injected(self):
        proc, mock_proc, _ = self._make_base_processor({})

        proc.process_mm_data("test", images=["img1"])

        call_kwargs = mock_proc.__call__.call_args
        self.assertNotIn("images_kwargs", call_kwargs.kwargs)
```
**EN:** This test method exercises empty config no kwargs injected and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty config no kwargs injected 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 190-202: Run test: audio kwargs preserved with config
```python
    def test_audio_kwargs_preserved_with_config(self):
        """audio_config merges with existing truncation=False."""
        config = {"audio": {"sample_rate": 16000}}
        proc, mock_proc, _ = self._make_base_processor(config)
        # Simulate a processor that uses singular "audio" key
        mock_proc.__class__.__name__ = "Gemma3nProcessor"

        proc.process_mm_data("test", audios=["aud1"])

        call_kwargs = mock_proc.__call__.call_args
        audio_kw = call_kwargs.kwargs.get("audio_kwargs", {})
        self.assertFalse(audio_kw.get("truncation", True))
        self.assertEqual(audio_kw.get("sample_rate"), 16000)
```
**EN:** This test method exercises audio kwargs preserved with config and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 audio kwargs preserved with config 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 205-205: Define class TestOverrideProcessorsConfigInjection
```python
class TestOverrideProcessorsConfigInjection(unittest.TestCase):
```
**EN:** This declaration introduces the `TestOverrideProcessorsConfigInjection` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestOverrideProcessorsConfigInjection` 测试类，并说明它通过继承承担的职责。

### Lines 206-206: Document the class `TestOverrideProcessorsConfigInjection`
```python
    """Regression tests for processors that override process_mm_data."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestOverrideProcessorsConfigInjection`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestOverrideProcessorsConfigInjection`的设计意图。

### Lines 208-231: Define helper: make override processor
```python
    def _make_override_processor(self, processor_cls, mm_process_config):
        """Create an override processor with mocked dependencies."""
        server_args = MagicMock()
        server_args.mm_process_config = mm_process_config
        server_args.disable_fast_image_processor = True
        server_args.keep_mm_feature_on_device = False

        mock_hf_processor = MagicMock()
        mock_hf_processor.__class__.__name__ = "TestProcessor"
        # Ernie processor accesses result["images"] after __call__,
        # so return {"images": None} to pass the None-guard safely.
        mock_hf_processor.__call__ = MagicMock(return_value={"images": None})

        with patch.object(processor_cls, "__init__", lambda self: None):
            proc = processor_cls()

        proc.server_args = server_args
        proc._processor = mock_hf_processor
        proc.image_config = mm_process_config.get("image", {})
        proc.video_config = mm_process_config.get("video", {})
        proc.audio_config = mm_process_config.get("audio", {})
        proc.FEATURE_NAMES = []

        return proc, mock_hf_processor
```
**EN:** This helper function encapsulates reusable logic inside `TestOverrideProcessorsConfigInjection` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestOverrideProcessorsConfigInjection` 内部调用，从而让场景结构更清晰。

### Lines 233-251: Run test: ernie45 vl injects images kwargs
```python
    def test_ernie45_vl_injects_images_kwargs(self):
        from sglang.srt.multimodal.processors.ernie45_vl import (
            Ernie4_5_VLImageProcessor,
        )

        config = {"image": {"max_pixels": 2000000}, "video": {"max_pixels": 500000}}
        proc, mock_proc = self._make_override_processor(
            Ernie4_5_VLImageProcessor, config
        )

        proc.process_mm_data("test", images=["img1"], videos=["vid1"])

        call_kwargs = mock_proc.__call__.call_args
        self.assertEqual(
            call_kwargs.kwargs.get("images_kwargs"), {"max_pixels": 2000000}
        )
        self.assertEqual(
            call_kwargs.kwargs.get("videos_kwargs"), {"max_pixels": 500000}
        )
```
**EN:** This test method exercises ernie45 vl injects images kwargs and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 ernie45 vl injects images kwargs 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 253-268: Run test: midashenglm injects audio kwargs
```python
    def test_midashenglm_injects_audio_kwargs(self):
        from sglang.srt.multimodal.processors.midashenglm import (
            MiDashengLMMultimodalProcessor,
        )

        config = {"audio": {"sample_rate": 16000}}
        proc, mock_proc = self._make_override_processor(
            MiDashengLMMultimodalProcessor, config
        )

        proc.process_mm_data("test", audios=["aud1"])

        call_kwargs = mock_proc.__call__.call_args
        audio_kw = call_kwargs.kwargs.get("audio_kwargs", {})
        self.assertFalse(audio_kw.get("truncation", True))
        self.assertEqual(audio_kw.get("sample_rate"), 16000)
```
**EN:** This test method exercises midashenglm injects audio kwargs and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 midashenglm injects audio kwargs 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 270-286: Run test: midashenglm user config overrides truncation
```python
    def test_midashenglm_user_config_overrides_truncation(self):
        """User config can override the default truncation=False."""
        from sglang.srt.multimodal.processors.midashenglm import (
            MiDashengLMMultimodalProcessor,
        )

        config = {"audio": {"truncation": True}}
        proc, mock_proc = self._make_override_processor(
            MiDashengLMMultimodalProcessor, config
        )

        proc.process_mm_data("test", audios=["aud1"])

        call_kwargs = mock_proc.__call__.call_args
        audio_kw = call_kwargs.kwargs.get("audio_kwargs", {})
        # User config can override truncation if they explicitly set it
        self.assertTrue(audio_kw.get("truncation"))
```
**EN:** This test method exercises midashenglm user config overrides truncation and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 midashenglm user config overrides truncation 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 289-290: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.multimodal.processors.base_processor`, `sglang.srt.multimodal.processors.ernie45_vl`, `sglang.srt.multimodal.processors.midashenglm`, `sglang.srt.server_args`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `unittest`, `unittest.mock`
- Notable symbols / 关键符号: `register_cuda_ci`, `register_amd_ci`, `unittest.main`
