# test_modelopt_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/model_loader/test_modelopt_loader.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates modelopt loader behavior in SGLang's unit / model loader area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / model loader 领域中与 modelopt loader 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: supporting statements / 辅助语句
```python
"""
Unit tests for ModelOptModelLoader class.

This test module verifies the functionality of ModelOptModelLoader, which
applies NVIDIA Model Optimizer quantization to models during loading.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 8-24: module imports and dependencies / 模块导入与依赖
```python
import unittest
from unittest.mock import MagicMock, patch

import torch.nn as nn

from sglang.srt.configs.device_config import DeviceConfig
from sglang.srt.configs.load_config import LoadConfig
from sglang.srt.configs.model_config import ModelConfig
from sglang.srt.layers.modelopt_utils import QUANT_CFG_CHOICES
from sglang.srt.layers.quantization.modelopt_quant import (
    ModelOptMixedPrecisionConfig,
)
from sglang.srt.model_loader.loader import ModelOptModelLoader
from sglang.srt.models.utils import WeightsMapper
from sglang.srt.utils import get_device
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `unittest.mock`, `torch.nn`, `sglang.srt.configs.device_config`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `unittest.mock`, `torch.nn`, `sglang.srt.configs.device_config`。

### Lines 25-28: supporting source context / 辅助源码上下文
```python

# Note: PYTHONPATH=python should be set when running tests

# Constants for calibration parameters to avoid hard-coded values
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 29-33: CI registration and metadata / CI 注册与元数据
```python
CALIBRATION_BATCH_SIZE = 36
CALIBRATION_NUM_SAMPLES = 512
DEFAULT_DEVICE = "cuda:0"

register_cuda_ci(est_time=11, stage="base-b", runner_config="1-gpu-small")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 36-36: class TestModelOptModelLoader declaration / 类 TestModelOptModelLoader 声明
```python
class TestModelOptModelLoader(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 37-37: supporting statements / 辅助语句
```python
    """Test cases for ModelOptModelLoader functionality."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 39-88: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Set up test fixtures."""
        # Mock distributed functionality to avoid initialization errors
        self.mock_tp_rank = patch(
            "sglang.srt.distributed.parallel_state.get_tensor_model_parallel_rank",
            return_value=0,
        )
        self.mock_tp_rank.start()

        self.mock_rank0_log = patch("sglang.srt.model_loader.loader.rank0_log")
        self.mock_rank0_log.start()

        # Mock logger to avoid issues
        self.mock_logger = patch("sglang.srt.model_loader.loader.logger")
        self.mock_logger.start()

        # Mock all distributed functions that might be called
        self.mock_get_tp_group = patch(
            "sglang.srt.distributed.parallel_state.get_tp_group"
        )
        self.mock_get_tp_group.start()

        # Mock model parallel initialization check
        self.mock_mp_is_initialized = patch(
            "sglang.srt.distributed.parallel_state.model_parallel_is_initialized",
            return_value=True,
        )
        self.mock_mp_is_initialized.start()

        self.model_path = "TinyLlama/TinyLlama-1.1B-Chat-v1.0"
        self.load_config = LoadConfig()
        self.device_config = DeviceConfig(device=get_device())

        # Create a basic model config with unified quantization flag
        self.model_config = ModelConfig(
            model_path=self.model_path,
            quantization="modelopt_fp8",  # Use unified quantization approach
        )

        # Also create a unified quantization config for new tests
        self.unified_model_config = ModelConfig(
            model_path=self.model_path, quantization="modelopt_fp8"
        )

        # Mock base model
        self.mock_base_model = MagicMock(spec=nn.Module)
        self.mock_base_model.eval.return_value = self.mock_base_model
        self.mock_base_model.device = (
            DEFAULT_DEVICE  # Add device attribute for calibration tests
        )
```
**EN:** Set up test fixtures. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up test fixtures. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 90-97: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        """Clean up test fixtures."""
        # Stop mocks
        self.mock_tp_rank.stop()
        self.mock_rank0_log.stop()
        self.mock_logger.stop()
        self.mock_get_tp_group.stop()
        self.mock_mp_is_initialized.stop()
```
**EN:** Clean up test fixtures. This routine releases resources and restores state after the related tests finish.
**CN:** Clean up test fixtures. 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 99-178: test case successful fp8 quantization (part 1/2) / 测试用例 successful fp8 quantization（第 1/2 部分）
```python
    @patch("sglang.srt.model_loader.loader.QUANT_CFG_CHOICES", QUANT_CFG_CHOICES)
    @patch("sglang.srt.model_loader.loader.logger")
    def test_successful_fp8_quantization(self, mock_logger):
        """Test successful FP8 quantization workflow."""

        # Create loader instance
        loader = ModelOptModelLoader(self.load_config)

        # Mock modelopt modules
        mock_mtq = MagicMock()

        # Configure mtq mock with FP8_DEFAULT_CFG
        mock_fp8_cfg = MagicMock()
        mock_mtq.FP8_DEFAULT_CFG = mock_fp8_cfg
        mock_mtq.quantize.return_value = self.mock_base_model
        mock_mtq.print_quant_summary = MagicMock()

        # Create a custom load_model method for testing that simulates the real logic
        def mock_load_model(*, model_config, device_config):
            mock_logger.info("ModelOptModelLoader: Loading base model...")

            # Simulate loading base model (this is already mocked)
            model = self.mock_base_model

            # Simulate the quantization config lookup
            quant_choice_str = model_config._get_modelopt_quant_type()
            quant_cfg_name = QUANT_CFG_CHOICES.get(quant_choice_str)

            if not quant_cfg_name:
                raise ValueError(f"Invalid modelopt_quant choice: '{quant_choice_str}'")

            # Simulate getattr call and quantization
            if quant_cfg_name == "FP8_DEFAULT_CFG":
                quant_cfg = mock_fp8_cfg

                mock_logger.info(
                    f"Quantizing model with ModelOpt using config attribute: mtq.{quant_cfg_name}"
                )

                # Simulate mtq.quantize call
                quantized_model = mock_mtq.quantize(model, quant_cfg, forward_loop=None)
                mock_logger.info("Model successfully quantized with ModelOpt.")

                # Simulate print_quant_summary call
                mock_mtq.print_quant_summary(quantized_model)

                return quantized_model.eval()

            return model.eval()

        # Patch the load_model method with our custom implementation
        with patch.object(loader, "load_model", side_effect=mock_load_model):
            # Execute the load_model method
            result_model = loader.load_model(
                model_config=self.model_config, device_config=self.device_config
            )

            # Verify the quantization process
            mock_mtq.quantize.assert_called_once_with(
                self.mock_base_model, mock_fp8_cfg, forward_loop=None
            )

            # Verify logging
            mock_logger.info.assert_any_call(
                "ModelOptModelLoader: Loading base model..."
            )
            mock_logger.info.assert_any_call(
                "Quantizing model with ModelOpt using config attribute: mtq.FP8_DEFAULT_CFG"
            )
            mock_logger.info.assert_any_call(
                "Model successfully quantized with ModelOpt."
            )

            # Verify print_quant_summary was called
            mock_mtq.print_quant_summary.assert_called_once_with(self.mock_base_model)

            # Verify eval() was called on the returned model
            self.mock_base_model.eval.assert_called()

            # Verify we get back the expected model
```
**EN:** Test successful FP8 quantization workflow. This test exercises `test_successful_fp8_quantization` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** Test successful FP8 quantization workflow. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_successful_fp8_quantization`。 这一段对应同一逻辑块的第 1 部分。

### Lines 179-179: test case successful fp8 quantization (part 2/2) / 测试用例 successful fp8 quantization（第 2/2 部分）
```python
            self.assertEqual(result_model, self.mock_base_model)
```
**EN:** Test successful FP8 quantization workflow. This test exercises `test_successful_fp8_quantization` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** Test successful FP8 quantization workflow. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_successful_fp8_quantization`。 这一段对应同一逻辑块的第 2 部分。

### Lines 181-211: test case missing modelopt import / 测试用例 missing modelopt import
```python
    @patch("sglang.srt.model_loader.loader.logger")
    def test_missing_modelopt_import(self, mock_logger):
        """Test error handling when modelopt library is not available."""

        loader = ModelOptModelLoader(self.load_config)

        # Mock the base model loader method
        with patch.object(
            loader, "_load_modelopt_base_model", return_value=self.mock_base_model
        ):
            # Simulate missing modelopt by making import fail
            original_import = __import__

            def mock_import(name, *args, **kwargs):
                if name.startswith("modelopt"):
                    raise ImportError("No module named 'modelopt'")
                # Return default import behavior for other modules
                return original_import(name, *args, **kwargs)

            with patch("builtins.__import__", side_effect=mock_import):
                # Expect ImportError to be raised and logged
                with self.assertRaises(ImportError):
                    loader.load_model(
                        model_config=self.model_config, device_config=self.device_config
                    )

                # Verify error logging
                mock_logger.error.assert_called_with(
                    "NVIDIA Model Optimizer (modelopt) library not found. "
                    "Please install it to use ModelOpt quantization."
                )
```
**EN:** Test error handling when modelopt library is not available. This test exercises `test_missing_modelopt_import` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test error handling when modelopt library is not available. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_missing_modelopt_import`。

### Lines 213-267: test case calibration workflow integration / 测试用例 calibration workflow integration
```python
    @patch("sglang.srt.model_loader.loader.QUANT_CFG_CHOICES", QUANT_CFG_CHOICES)
    @patch("sglang.srt.model_loader.loader.AutoTokenizer")
    @patch("sglang.srt.model_loader.loader.logger")
    def test_calibration_workflow_integration(self, mock_logger, mock_auto_tokenizer):
        """Test end-to-end calibration workflow integration."""

        loader = ModelOptModelLoader(self.load_config)

        # Mock tokenizer
        mock_tokenizer = MagicMock()
        mock_tokenizer.padding_side = "right"
        mock_auto_tokenizer.from_pretrained.return_value = mock_tokenizer

        # Mock modelopt modules
        mock_mtq = MagicMock()
        mock_mto = MagicMock()
        mock_dataset_utils = MagicMock()

        # Configure quantization config
        mock_fp8_cfg = MagicMock()
        mock_mtq.FP8_DEFAULT_CFG = mock_fp8_cfg

        # Configure dataset utilities
        mock_calib_dataloader = MagicMock()
        mock_calibrate_loop = MagicMock()
        mock_dataset_utils.get_dataset_dataloader.return_value = mock_calib_dataloader
        mock_dataset_utils.create_forward_loop.return_value = mock_calibrate_loop

        # Configure model as not quantized initially
        mock_is_quantized = MagicMock(return_value=False)

        with patch.object(
            loader, "_load_modelopt_base_model", return_value=self.mock_base_model
        ):
            with patch.dict(
                "sys.modules",
                {
                    "modelopt": MagicMock(),
                    "modelopt.torch": MagicMock(),
                    "modelopt.torch.opt": mock_mto,
                    "modelopt.torch.quantization": mock_mtq,
                    "modelopt.torch.quantization.utils": MagicMock(
                        is_quantized=mock_is_quantized
                    ),
                    "modelopt.torch.utils": MagicMock(),
                    "modelopt.torch.utils.dataset_utils": mock_dataset_utils,
                },
            ):
                # Execute the load_model method to test the full workflow
                result_model = loader.load_model(
                    model_config=self.model_config, device_config=self.device_config
                )

                # Verify the model loading was successful
                self.assertEqual(result_model, self.mock_base_model)
```
**EN:** Test end-to-end calibration workflow integration. This test exercises `test_calibration_workflow_integration` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test end-to-end calibration workflow integration. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_calibration_workflow_integration`。

### Lines 268-272: supporting source context / 辅助源码上下文
```python

                # Verify key calibration components were used
                # Note: We can't easily verify the exact calls due to dynamic imports,
                # but we can verify the workflow completed successfully

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 273-352: test case quantized checkpoint restore (part 1/2) / 测试用例 quantized checkpoint restore（第 1/2 部分）
```python
    @patch("sglang.srt.model_loader.loader.QUANT_CFG_CHOICES", QUANT_CFG_CHOICES)
    @patch("sglang.srt.model_loader.loader.AutoTokenizer")
    @patch("sglang.srt.model_loader.loader.logger")
    def test_quantized_checkpoint_restore(self, mock_logger, mock_auto_tokenizer):
        """Test restoring from a quantized checkpoint."""

        # Create model config with checkpoint restore path
        config_with_restore = ModelConfig(
            model_path=self.model_path,
            quantization="modelopt_fp8",
        )

        # Create load config with checkpoint restore path
        load_config_with_restore = LoadConfig(
            modelopt_checkpoint_restore_path="/path/to/quantized/checkpoint"
        )

        loader = ModelOptModelLoader(load_config_with_restore)

        # Mock tokenizer
        mock_tokenizer = MagicMock()
        mock_auto_tokenizer.from_pretrained.return_value = mock_tokenizer

        # Mock modelopt modules
        mock_mtq = MagicMock()
        mock_mto = MagicMock()

        # Configure quantization config
        mock_fp8_cfg = MagicMock()
        mock_mtq.FP8_DEFAULT_CFG = mock_fp8_cfg

        # Configure model as not quantized initially
        mock_is_quantized = MagicMock(return_value=False)

        with patch.object(
            loader, "_load_modelopt_base_model", return_value=self.mock_base_model
        ):
            with patch.dict(
                "sys.modules",
                {
                    "modelopt": MagicMock(),
                    "modelopt.torch": MagicMock(),
                    "modelopt.torch.opt": mock_mto,
                    "modelopt.torch.quantization": mock_mtq,
                    "modelopt.torch.quantization.utils": MagicMock(
                        is_quantized=mock_is_quantized
                    ),
                },
            ):
                with patch.object(loader, "_setup_modelopt_quantization") as mock_setup:
                    # Mock the _setup_modelopt_quantization to simulate checkpoint restore
                    def mock_setup_quantization(
                        model,
                        tokenizer,
                        quant_cfg,
                        quantized_ckpt_restore_path=None,
                        **kwargs,
                    ):
                        if quantized_ckpt_restore_path:
                            mock_mto.restore(model, quantized_ckpt_restore_path)
                            print(
                                f"Restored quantized model from {quantized_ckpt_restore_path}"
                            )
                            return

                    mock_setup.side_effect = mock_setup_quantization

                    # Execute the load_model method
                    result_model = loader.load_model(
                        model_config=config_with_restore,
                        device_config=self.device_config,
                    )

                    # Verify the setup was called with restore path
                    mock_setup.assert_called_once()
                    call_args = mock_setup.call_args
                    # Check that the restore path was passed correctly
                    self.assertIn("quantized_ckpt_restore_path", call_args[1])
                    self.assertEqual(
                        call_args[1]["quantized_ckpt_restore_path"],
```
**EN:** Test restoring from a quantized checkpoint. This test exercises `test_quantized_checkpoint_restore` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** Test restoring from a quantized checkpoint. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_quantized_checkpoint_restore`。 这一段对应同一逻辑块的第 1 部分。

### Lines 353-362: test case quantized checkpoint restore (part 2/2) / 测试用例 quantized checkpoint restore（第 2/2 部分）
```python
                        "/path/to/quantized/checkpoint",
                    )

                    # Verify restore was called
                    mock_mto.restore.assert_called_once_with(
                        self.mock_base_model, "/path/to/quantized/checkpoint"
                    )

                    # Verify we get the expected model back
                    self.assertEqual(result_model, self.mock_base_model)
```
**EN:** Test restoring from a quantized checkpoint. This test exercises `test_quantized_checkpoint_restore` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** Test restoring from a quantized checkpoint. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_quantized_checkpoint_restore`。 这一段对应同一逻辑块的第 2 部分。

### Lines 364-443: test case quantized checkpoint save (part 1/2) / 测试用例 quantized checkpoint save（第 1/2 部分）
```python
    @patch("sglang.srt.model_loader.loader.QUANT_CFG_CHOICES", QUANT_CFG_CHOICES)
    @patch("sglang.srt.model_loader.loader.AutoTokenizer")
    @patch("sglang.srt.model_loader.loader.logger")
    def test_quantized_checkpoint_save(self, mock_logger, mock_auto_tokenizer):
        """Test saving quantized checkpoint after calibration."""

        # Create model config with checkpoint save path
        config_with_save = ModelConfig(
            model_path=self.model_path,
            quantization="modelopt_fp8",
        )

        # Create load config with checkpoint save path
        load_config_with_save = LoadConfig(
            modelopt_checkpoint_save_path="/path/to/save/checkpoint"
        )

        loader = ModelOptModelLoader(load_config_with_save)

        # Mock tokenizer
        mock_tokenizer = MagicMock()
        mock_auto_tokenizer.from_pretrained.return_value = mock_tokenizer

        # Mock modelopt modules
        mock_mtq = MagicMock()
        mock_mto = MagicMock()
        mock_dataset_utils = MagicMock()

        # Configure quantization config
        mock_fp8_cfg = MagicMock()
        mock_mtq.FP8_DEFAULT_CFG = mock_fp8_cfg

        # Configure model as not quantized initially
        mock_is_quantized = MagicMock(return_value=False)

        with patch.object(
            loader, "_load_modelopt_base_model", return_value=self.mock_base_model
        ):
            with patch.dict(
                "sys.modules",
                {
                    "modelopt": MagicMock(),
                    "modelopt.torch": MagicMock(),
                    "modelopt.torch.opt": mock_mto,
                    "modelopt.torch.quantization": mock_mtq,
                    "modelopt.torch.quantization.utils": MagicMock(
                        is_quantized=mock_is_quantized
                    ),
                    "modelopt.torch.utils": MagicMock(),
                    "modelopt.torch.utils.dataset_utils": mock_dataset_utils,
                },
            ):
                with patch.object(loader, "_setup_modelopt_quantization") as mock_setup:
                    # Mock the _setup_modelopt_quantization to simulate checkpoint save
                    def mock_setup_quantization(
                        model,
                        tokenizer,
                        quant_cfg,
                        quantized_ckpt_save_path=None,
                        **kwargs,
                    ):
                        # Simulate calibration and quantization
                        mock_mtq.quantize(model, quant_cfg, forward_loop=MagicMock())
                        mock_mtq.print_quant_summary(model)

                        # Save checkpoint if path provided
                        if quantized_ckpt_save_path:
                            mock_mto.save(model, quantized_ckpt_save_path)
                            print(
                                f"Quantized model saved to {quantized_ckpt_save_path}"
                            )

                    mock_setup.side_effect = mock_setup_quantization

                    # Execute the load_model method
                    result_model = loader.load_model(
                        model_config=config_with_save, device_config=self.device_config
                    )

                    # Verify the setup was called with save path
```
**EN:** Test saving quantized checkpoint after calibration. This test exercises `test_quantized_checkpoint_save` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** Test saving quantized checkpoint after calibration. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_quantized_checkpoint_save`。 这一段对应同一逻辑块的第 1 部分。

### Lines 444-459: test case quantized checkpoint save (part 2/2) / 测试用例 quantized checkpoint save（第 2/2 部分）
```python
                    mock_setup.assert_called_once()
                    call_args = mock_setup.call_args
                    # Check that the save path was passed correctly
                    self.assertIn("quantized_ckpt_save_path", call_args[1])
                    self.assertEqual(
                        call_args[1]["quantized_ckpt_save_path"],
                        "/path/to/save/checkpoint",
                    )

                    # Verify save was called
                    mock_mto.save.assert_called_once_with(
                        self.mock_base_model, "/path/to/save/checkpoint"
                    )

                    # Verify we get the expected model back
                    self.assertEqual(result_model, self.mock_base_model)
```
**EN:** Test saving quantized checkpoint after calibration. This test exercises `test_quantized_checkpoint_save` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** Test saving quantized checkpoint after calibration. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_quantized_checkpoint_save`。 这一段对应同一逻辑块的第 2 部分。

### Lines 461-478: test case unified quantization flag support / 测试用例 unified quantization flag support
```python
    def test_unified_quantization_flag_support(self):
        """Test that ModelOptModelLoader supports unified quantization flags."""
        # Test modelopt_fp8
        config_fp8 = ModelConfig(
            model_path=self.model_path, quantization="modelopt_fp8"
        )
        self.assertEqual(config_fp8._get_modelopt_quant_type(), "fp8")

        # Test modelopt_fp4
        config_fp4 = ModelConfig(
            model_path=self.model_path, quantization="modelopt_fp4"
        )
        self.assertEqual(config_fp4._get_modelopt_quant_type(), "nvfp4")

        # Test auto-detection
        config_auto = ModelConfig(model_path=self.model_path, quantization="modelopt")
        # Should default to fp8 when no config is detected
        self.assertEqual(config_auto._get_modelopt_quant_type(), "fp8")
```
**EN:** Test that ModelOptModelLoader supports unified quantization flags. This test exercises `test_unified_quantization_flag_support` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that ModelOptModelLoader supports unified quantization flags. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unified_quantization_flag_support`。

### Lines 481-481: class TestModelOptLoaderIntegration declaration / 类 TestModelOptLoaderIntegration 声明
```python
class TestModelOptLoaderIntegration(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 482-482: supporting statements / 辅助语句
```python
    """Integration tests for ModelOptModelLoader with Engine API."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 484-525: test case engine with modelopt quant parameter / 测试用例 engine with modelopt quant parameter
```python
    @patch("sglang.srt.model_loader.loader.get_model_loader")
    @patch("sglang.srt.entrypoints.engine.Engine.__init__")
    def test_engine_with_modelopt_quant_parameter(
        self, mock_engine_init, mock_get_model_loader
    ):
        """Test that Engine properly handles modelopt_quant parameter."""

        # Mock the Engine.__init__ to avoid actual initialization
        mock_engine_init.return_value = None

        # Mock get_model_loader to return our ModelOptModelLoader
        mock_loader = MagicMock(spec=ModelOptModelLoader)
        mock_get_model_loader.return_value = mock_loader

        # Import here to avoid circular imports during test discovery
        # import sglang as sgl  # Commented out since not directly used

        # Test that we can create an engine with modelopt_quant parameter
        # This would normally trigger the ModelOptModelLoader selection
        try:
            engine_args = {
                "model_path": "TinyLlama/TinyLlama-1.1B-Chat-v1.0",
                "modelopt_quant": "fp8",
                "log_level": "error",  # Suppress logs during testing
            }

            # This tests the parameter parsing and server args creation
            from sglang.srt.server_args import ServerArgs

            server_args = ServerArgs(**engine_args)

            # Verify that modelopt_quant is properly set
            self.assertEqual(server_args.modelopt_quant, "fp8")

        except Exception as e:
            # If there are missing dependencies or initialization issues,
            # we can still verify the parameter is accepted
            if "modelopt_quant" not in str(e):
                # The parameter was accepted, which is what we want to test
                pass
            else:
                self.fail(f"modelopt_quant parameter not properly handled: {e}")
```
**EN:** Test that Engine properly handles modelopt_quant parameter. This test exercises `test_engine_with_modelopt_quant_parameter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that Engine properly handles modelopt_quant parameter. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_engine_with_modelopt_quant_parameter`。

### Lines 527-565: test case engine with modelopt quant cli argument / 测试用例 engine with modelopt quant cli argument
```python
    @patch("sglang.srt.model_loader.loader.get_model_loader")
    @patch("sglang.srt.entrypoints.engine.Engine.__init__")
    def test_engine_with_modelopt_quant_cli_argument(
        self, mock_engine_init, mock_get_model_loader
    ):
        """Test that CLI argument --modelopt-quant is properly parsed."""

        # Mock the Engine.__init__ to avoid actual initialization
        mock_engine_init.return_value = None

        # Mock get_model_loader to return our ModelOptModelLoader
        mock_loader = MagicMock(spec=ModelOptModelLoader)
        mock_get_model_loader.return_value = mock_loader

        # Test CLI argument parsing
        import argparse

        from sglang.srt.server_args import ServerArgs

        # Create parser and add arguments
        parser = argparse.ArgumentParser()
        ServerArgs.add_cli_args(parser)

        # Test parsing with modelopt_quant argument
        args = parser.parse_args(
            [
                "--model-path",
                "TinyLlama/TinyLlama-1.1B-Chat-v1.0",
                "--modelopt-quant",
                "fp8",
            ]
        )

        # Convert to ServerArgs using the proper from_cli_args method
        server_args = ServerArgs.from_cli_args(args)

        # Verify that modelopt_quant was properly parsed
        self.assertEqual(server_args.modelopt_quant, "fp8")
        self.assertEqual(server_args.model_path, "TinyLlama/TinyLlama-1.1B-Chat-v1.0")
```
**EN:** Test that CLI argument --modelopt-quant is properly parsed. This test exercises `test_engine_with_modelopt_quant_cli_argument` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that CLI argument --modelopt-quant is properly parsed. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_engine_with_modelopt_quant_cli_argument`。

### Lines 568-568: class TestParseQuantHfConfig declaration / 类 TestParseQuantHfConfig 声明
```python
class TestParseQuantHfConfig(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 569-584: class-level constants and configuration for `TestParseQuantHfConfig` / 类级常量与配置
```python
    """Tests for _parse_quant_hf_config and _parse_modelopt_quant_config.

    Regression tests for the fix where quant_method='modelopt' ignoring quant_algo.
    """

    # (quant_config_input, expected_quant_method)
    _MODELOPT_CASES = [
        ({"quant_method": "modelopt", "quant_algo": "FP8"}, "modelopt_fp8"),
        ({"quant_method": "modelopt", "quant_algo": "FP4"}, "modelopt_fp4"),
        ({"quant_method": "modelopt", "quant_algo": "NVFP4"}, "modelopt_fp4"),
        ({"quant_method": "modelopt", "quant_algo": "MIXED_PRECISION"}, "w4afp8"),
        ({"quant_algo": "FP8"}, "modelopt_fp8"),
        ({"quant_algo": "FP4"}, "modelopt_fp4"),
        ({"quant_algo": "MIXED_PRECISION"}, "w4afp8"),
        ({"quant_method": "modelopt"}, "modelopt"),
    ]
```
**EN:** This block defines shared names such as `_MODELOPT_CASES`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `_MODELOPT_CASES` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 586-602: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Set up a real ModelConfig using TinyLlama (already used elsewhere)."""
        self.mock_tp_rank = patch(
            "sglang.srt.distributed.parallel_state.get_tensor_model_parallel_rank",
            return_value=0,
        )
        self.mock_tp_rank.start()

        self.mock_mp_is_initialized = patch(
            "sglang.srt.distributed.parallel_state.model_parallel_is_initialized",
            return_value=True,
        )
        self.mock_mp_is_initialized.start()

        self.model_config = ModelConfig(
            model_path="TinyLlama/TinyLlama-1.1B-Chat-v1.0",
        )
```
**EN:** Set up a real ModelConfig using TinyLlama (already used elsewhere). This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up a real ModelConfig using TinyLlama (already used elsewhere). 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 604-606: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        self.mock_tp_rank.stop()
        self.mock_mp_is_initialized.stop()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 608-614: test case modelopt quant parsing / 测试用例 modelopt quant parsing
```python
    def test_modelopt_quant_parsing(self):
        """Modelopt quant configs must resolve to the correct quant_method."""
        for quant_cfg_input, expected in self._MODELOPT_CASES:
            with self.subTest(quant_cfg=quant_cfg_input):
                self.model_config.hf_config.quantization_config = dict(quant_cfg_input)
                result = self.model_config._parse_quant_hf_config()
                self.assertEqual(result["quant_method"], expected)
```
**EN:** Modelopt quant configs must resolve to the correct quant_method. This test exercises `test_modelopt_quant_parsing` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Modelopt quant configs must resolve to the correct quant_method. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_modelopt_quant_parsing`。

### Lines 616-624: test case non modelopt quant method unchanged / 测试用例 non modelopt quant method unchanged
```python
    def test_non_modelopt_quant_method_unchanged(self):
        """Non-modelopt quant_method (e.g. 'gptq') must NOT enter the modelopt path."""
        self.model_config.hf_config.quantization_config = {
            "quant_method": "gptq",
            "bits": 4,
        }
        result = self.model_config._parse_quant_hf_config()
        self.assertEqual(result["quant_method"], "gptq")
        self.assertNotIn("quant_algo", result)
```
**EN:** Non-modelopt quant_method (e.g. This test exercises `test_non_modelopt_quant_method_unchanged` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Non-modelopt quant_method (e.g. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_modelopt_quant_method_unchanged`。

### Lines 627-627: class TestModelOptMixedPrecisionConfig declaration / 类 TestModelOptMixedPrecisionConfig 声明
```python
class TestModelOptMixedPrecisionConfig(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 628-638: test case nemotron mixed precision uses modelopt mixed / 测试用例 nemotron mixed precision uses modelopt mixed
```python
    def test_nemotron_mixed_precision_uses_modelopt_mixed(self):
        model_config = ModelConfig.__new__(ModelConfig)
        model_config.hf_config = MagicMock()
        model_config.hf_config.model_type = "nemotron_h"
        model_config.hf_config.architectures = ["NemotronHForCausalLM"]

        result = model_config._parse_modelopt_quant_config(
            {"quantization": {"quant_algo": "MIXED_PRECISION"}}
        )

        self.assertEqual(result["quant_method"], "modelopt_mixed")
```
**EN:** This test exercises `test_nemotron_mixed_precision_uses_modelopt_mixed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nemotron_mixed_precision_uses_modelopt_mixed`。

### Lines 640-646: test case mixed precision override does not hijack w4afp8 / 测试用例 mixed precision override does not hijack w4afp8
```python
    def test_mixed_precision_override_does_not_hijack_w4afp8(self):
        self.assertIsNone(
            ModelOptMixedPrecisionConfig.override_quantization_method(
                {"quant_method": "w4afp8", "quant_algo": "MIXED_PRECISION"},
                "w4afp8",
            )
        )
```
**EN:** This test exercises `test_mixed_precision_override_does_not_hijack_w4afp8` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_precision_override_does_not_hijack_w4afp8`。

### Lines 648-649: test case mixed precision uses nvfp4 min capability / 测试用例 mixed precision uses nvfp4 min capability
```python
    def test_mixed_precision_uses_nvfp4_min_capability(self):
        self.assertEqual(ModelOptMixedPrecisionConfig.get_min_capability(), 100)
```
**EN:** This test exercises `test_mixed_precision_uses_nvfp4_min_capability` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_precision_uses_nvfp4_min_capability`。

### Lines 651-685: test case mixed precision quant layer resolution after mapping / 测试用例 mixed precision quant layer resolution after mapping
```python
    def test_mixed_precision_quant_layer_resolution_after_mapping(self):
        quant_config = ModelOptMixedPrecisionConfig.from_config(
            {
                "quant_algo": "MIXED_PRECISION",
                "quantized_layers": {
                    "backbone.layers.0.mixer.in_proj": {"quant_algo": "FP8"},
                    "backbone.layers.1.mixer.experts.0.up_proj": {
                        "quant_algo": "NVFP4",
                        "group_size": 16,
                    },
                    "backbone.layers.2.mixer.q_proj": {"quant_algo": "FP8"},
                    "backbone.layers.2.mixer.k_proj": {"quant_algo": "FP8"},
                    "backbone.layers.2.mixer.v_proj": {"quant_algo": "FP8"},
                },
                "packed_modules_mapping": {
                    "qkv_proj": ["q_proj", "k_proj", "v_proj"],
                },
            }
        )
        quant_config.apply_weight_name_mapper(
            WeightsMapper(orig_to_new_prefix={"backbone.": "model."})
        )

        self.assertEqual(
            quant_config._resolve_quant_algo("model.layers.0.mixer.in_proj"),
            "FP8",
        )
        self.assertEqual(
            quant_config._resolve_quant_algo("model.layers.1.mixer.experts"),
            "NVFP4",
        )
        self.assertEqual(
            quant_config._resolve_quant_algo("model.layers.2.mixer.qkv_proj"),
            "FP8",
        )
```
**EN:** This test exercises `test_mixed_precision_quant_layer_resolution_after_mapping` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_precision_quant_layer_resolution_after_mapping`。

### Lines 688-689: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestModelOptModelLoader`: Test cases for ModelOptModelLoader functionality. / 用于组织相关测试、夹具或辅助方法。
- `TestModelOptLoaderIntegration`: Integration tests for ModelOptModelLoader with Engine API. / 用于组织相关测试、夹具或辅助方法。
- `TestParseQuantHfConfig`: Tests for _parse_quant_hf_config and _parse_modelopt_quant_config. / 用于组织相关测试、夹具或辅助方法。
- `TestModelOptMixedPrecisionConfig`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestModelOptModelLoader.setUp`: Set up test fixtures. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestModelOptModelLoader.tearDown`: Clean up test fixtures. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestModelOptModelLoader.test_successful_fp8_quantization`: Test successful FP8 quantization workflow. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_successful_fp8_quantization`。
- `TestModelOptModelLoader.test_missing_modelopt_import`: Test error handling when modelopt library is not available. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_missing_modelopt_import`。
- `TestModelOptModelLoader.test_calibration_workflow_integration`: Test end-to-end calibration workflow integration. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_calibration_workflow_integration`。
- `TestModelOptModelLoader.test_quantized_checkpoint_restore`: Test restoring from a quantized checkpoint. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_quantized_checkpoint_restore`。
- `TestModelOptModelLoader.test_quantized_checkpoint_save`: Test saving quantized checkpoint after calibration. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_quantized_checkpoint_save`。
- `TestModelOptModelLoader.test_unified_quantization_flag_support`: Test that ModelOptModelLoader supports unified quantization flags. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unified_quantization_flag_support`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `unittest.mock`
- **Third-party modules / 第三方模块**: `torch.nn`
- **Internal modules / 内部模块**: `sglang.srt.configs.device_config`, `sglang.srt.configs.load_config`, `sglang.srt.configs.model_config`, `sglang.srt.layers.modelopt_utils`, `sglang.srt.layers.quantization.modelopt_quant`, `sglang.srt.model_loader.loader`, `sglang.srt.models.utils`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 689
