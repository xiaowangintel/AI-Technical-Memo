# test_gguf_download.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/test_gguf_download.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers model-facing behavior. The file defines 8 test(s), 0 fixture(s), and 4 helper/class block(s) to validate this area. / [CN] 该文件覆盖面向模型的行为。它定义了 8 个测试、0 个 fixture，以及 4 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L11)
```python
from unittest.mock import MagicMock, patch

import pytest

from vllm.config import ModelConfig
from vllm.config.load import LoadConfig
from vllm.model_executor.model_loader.gguf_loader import GGUFModelLoader
from vllm.model_executor.model_loader.weight_utils import download_gguf
```
**EN:** Imports standard-library modules such as `unittest.mock.MagicMock`, `unittest.mock.patch`, third-party packages like `pytest`, project helpers such as `vllm.config.ModelConfig`, `vllm.config.load.LoadConfig`, `vllm.model_executor.model_loader.gguf_loader.GGUFModelLoader`.
**CN:** 导入标准库模块（如 `unittest.mock.MagicMock`、`unittest.mock.patch`）、第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.config.ModelConfig`、`vllm.config.load.LoadConfig`、`vllm.model_executor.model_loader.gguf_loader.GGUFModelLoader`）。

### Class / 类: TestGGUFDownload (L14-L96)
```python
class TestGGUFDownload:
    """Test GGUF model downloading functionality."""

    @patch("vllm.model_executor.model_loader.weight_utils.download_weights_from_hf")
    def test_download_gguf_single_file(self, mock_download):
        """Test downloading a single GGUF file."""
        # Setup mock
        mock_folder = "/tmp/mock_cache"
        mock_download.return_value = mock_folder

        # Mock glob to return a single file
        with patch("glob.glob") as mock_glob:
            mock_glob.side_effect = lambda pattern, **kwargs: (
                [f"{mock_folder}/model-IQ1_S.gguf"] if "IQ1_S" in pattern else []
# ... 61 lines omitted for brevity ...
    @patch("glob.glob", return_value=[])
    def test_download_gguf_no_files_found(self, mock_glob, mock_download):
        """Test error when no GGUF files are found."""
        mock_folder = "/tmp/mock_cache"
        mock_download.return_value = mock_folder

        with pytest.raises(ValueError, match="Downloaded GGUF files not found"):
            download_gguf("unsloth/Qwen3-0.6B-GGUF", "IQ1_S")
```
**EN:** This class groups related scenarios in `TestGGUFDownload`. It contains 4 test method(s) and 0 supporting method(s). Representative methods include `test_download_gguf_single_file`, `test_download_gguf_sharded_files`, `test_download_gguf_subdir`.
**CN:** 该类将与 `TestGGUFDownload` 相关的场景组织在一起。 它包含 4 个测试方法和 0 个辅助方法。 代表性方法包括 `test_download_gguf_single_file`、`test_download_gguf_sharded_files`、`test_download_gguf_subdir`。

### Test method / 测试方法: TestGGUFDownload.test_download_gguf_single_file (L17-L47)
```python
    @patch("vllm.model_executor.model_loader.weight_utils.download_weights_from_hf")
    def test_download_gguf_single_file(self, mock_download):
        """Test downloading a single GGUF file."""
        # Setup mock
        mock_folder = "/tmp/mock_cache"
        mock_download.return_value = mock_folder

        # Mock glob to return a single file
        with patch("glob.glob") as mock_glob:
            mock_glob.side_effect = lambda pattern, **kwargs: (
                [f"{mock_folder}/model-IQ1_S.gguf"] if "IQ1_S" in pattern else []
            )

            result = download_gguf("unsloth/Qwen3-0.6B-GGUF", "IQ1_S")

            # Verify download_weights_from_hf was called with correct patterns
            mock_download.assert_called_once_with(
                model_name_or_path="unsloth/Qwen3-0.6B-GGUF",
# ... 5 lines omitted for brevity ...
                    "*/*-IQ1_S-*.gguf",
                ],
                revision=None,
                ignore_patterns=None,
            )

            # Verify result is the file path, not folder
            assert result == f"{mock_folder}/model-IQ1_S.gguf"
```
**EN:** This test validates `TestGGUFDownload.test_download_gguf_single_file`. Key inputs are `mock_download`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `result == f'{mock_folder}/model-IQ1_S.gguf'`.
**CN:** 这个测试验证 `TestGGUFDownload.test_download_gguf_single_file`。 关键输入包括 `mock_download`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `result == f'{mock_folder}/model-IQ1_S.gguf'`。

### Test method / 测试方法: TestGGUFDownload.test_download_gguf_sharded_files (L49-L69)
```python
    @patch("vllm.model_executor.model_loader.weight_utils.download_weights_from_hf")
    def test_download_gguf_sharded_files(self, mock_download):
        """Test downloading sharded GGUF files."""
        mock_folder = "/tmp/mock_cache"
        mock_download.return_value = mock_folder

        # Mock glob to return sharded files
        with patch("glob.glob") as mock_glob:
            mock_glob.side_effect = lambda pattern, **kwargs: (
                [
                    f"{mock_folder}/model-Q2_K-00001-of-00002.gguf",
                    f"{mock_folder}/model-Q2_K-00002-of-00002.gguf",
                ]
                if "Q2_K" in pattern
                else []
            )

            result = download_gguf("unsloth/gpt-oss-120b-GGUF", "Q2_K")

            # Should return the first file after sorting
            assert result == f"{mock_folder}/model-Q2_K-00001-of-00002.gguf"
```
**EN:** This test validates `TestGGUFDownload.test_download_gguf_sharded_files`. Key inputs are `mock_download`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `result == f'{mock_folder}/model-Q2_K-00001-of-00002.gguf'`.
**CN:** 这个测试验证 `TestGGUFDownload.test_download_gguf_sharded_files`。 关键输入包括 `mock_download`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `result == f'{mock_folder}/model-Q2_K-00001-of-00002.gguf'`。

### Test method / 测试方法: TestGGUFDownload.test_download_gguf_subdir (L71-L86)
```python
    @patch("vllm.model_executor.model_loader.weight_utils.download_weights_from_hf")
    def test_download_gguf_subdir(self, mock_download):
        """Test downloading GGUF files from subdirectory."""
        mock_folder = "/tmp/mock_cache"
        mock_download.return_value = mock_folder

        with patch("glob.glob") as mock_glob:
            mock_glob.side_effect = lambda pattern, **kwargs: (
                [f"{mock_folder}/Q2_K/model-Q2_K.gguf"]
                if "Q2_K" in pattern or "**/*.gguf" in pattern
                else []
            )

            result = download_gguf("unsloth/gpt-oss-120b-GGUF", "Q2_K")

            assert result == f"{mock_folder}/Q2_K/model-Q2_K.gguf"
```
**EN:** This test validates `TestGGUFDownload.test_download_gguf_subdir`. Key inputs are `mock_download`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `result == f'{mock_folder}/Q2_K/model-Q2_K.gguf'`.
**CN:** 这个测试验证 `TestGGUFDownload.test_download_gguf_subdir`。 关键输入包括 `mock_download`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `result == f'{mock_folder}/Q2_K/model-Q2_K.gguf'`。

### Test method / 测试方法: TestGGUFDownload.test_download_gguf_no_files_found (L88-L96)
```python
    @patch("vllm.model_executor.model_loader.weight_utils.download_weights_from_hf")
    @patch("glob.glob", return_value=[])
    def test_download_gguf_no_files_found(self, mock_glob, mock_download):
        """Test error when no GGUF files are found."""
        mock_folder = "/tmp/mock_cache"
        mock_download.return_value = mock_folder

        with pytest.raises(ValueError, match="Downloaded GGUF files not found"):
            download_gguf("unsloth/Qwen3-0.6B-GGUF", "IQ1_S")
```
**EN:** This test validates `TestGGUFDownload.test_download_gguf_no_files_found`. Key inputs are `mock_glob`, `mock_download`. It checks an expected failure path with `pytest.raises`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `TestGGUFDownload.test_download_gguf_no_files_found`。 关键输入包括 `mock_glob`、`mock_download`。 它使用 `pytest.raises` 检查预期失败路径。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Class / 类: TestGGUFModelLoader (L99-L224)
```python
class TestGGUFModelLoader:
    """Test GGUFModelLoader class methods."""

    @patch("os.path.isfile", return_value=True)
    def test_prepare_weights_local_file(self, mock_isfile):
        """Test _prepare_weights with local file."""
        load_config = LoadConfig(load_format="gguf")
        loader = GGUFModelLoader(load_config)

        # Create a simple mock ModelConfig with only the model attribute
        model_config = MagicMock()
        model_config.model = "/path/to/model.gguf"

        result = loader._prepare_weights(model_config)
# ... 104 lines omitted for brevity ...

        # Create ModelConfig with a valid repo_id to avoid validation errors
        # Then test _prepare_weights with invalid format
        model_config = ModelConfig(model="unsloth/Qwen3-0.6B")
        # Manually set model to invalid format after creation
        model_config.model = "invalid-format"
        with pytest.raises(ValueError, match="Unrecognised GGUF reference"):
            loader._prepare_weights(model_config)
```
**EN:** This class groups related scenarios in `TestGGUFModelLoader`. It contains 4 test method(s) and 0 supporting method(s). Representative methods include `test_prepare_weights_local_file`, `test_prepare_weights_repo_filename`, `test_prepare_weights_repo_quant_type`.
**CN:** 该类将与 `TestGGUFModelLoader` 相关的场景组织在一起。 它包含 4 个测试方法和 0 个辅助方法。 代表性方法包括 `test_prepare_weights_local_file`、`test_prepare_weights_repo_filename`、`test_prepare_weights_repo_quant_type`。

### Test method / 测试方法: TestGGUFModelLoader.test_prepare_weights_local_file (L102-L114)
```python
    @patch("os.path.isfile", return_value=True)
    def test_prepare_weights_local_file(self, mock_isfile):
        """Test _prepare_weights with local file."""
        load_config = LoadConfig(load_format="gguf")
        loader = GGUFModelLoader(load_config)

        # Create a simple mock ModelConfig with only the model attribute
        model_config = MagicMock()
        model_config.model = "/path/to/model.gguf"

        result = loader._prepare_weights(model_config)
        assert result == "/path/to/model.gguf"
        mock_isfile.assert_called_once_with("/path/to/model.gguf")
```
**EN:** This test validates `TestGGUFModelLoader.test_prepare_weights_local_file`. Key inputs are `mock_isfile`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `result == '/path/to/model.gguf'`.
**CN:** 这个测试验证 `TestGGUFModelLoader.test_prepare_weights_local_file`。 关键输入包括 `mock_isfile`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `result == '/path/to/model.gguf'`。

### Test method / 测试方法: TestGGUFModelLoader.test_prepare_weights_repo_filename (L116-L136)
```python
    @patch("vllm.model_executor.model_loader.gguf_loader.hf_hub_download")
    @patch("os.path.isfile", return_value=False)
    def test_prepare_weights_repo_filename(self, mock_isfile, mock_hf_download):
        """Test _prepare_weights with repo_id/filename.gguf format."""
        load_config = LoadConfig(load_format="gguf")
        loader = GGUFModelLoader(load_config)

        mock_hf_download.return_value = "/downloaded/model.gguf"

        model_config = MagicMock()
        model_config.model = "unsloth/Qwen3-0.6B-GGUF/model.gguf"
        model_config.revision = "abc123"

        result = loader._prepare_weights(model_config)
        assert result == "/downloaded/model.gguf"
        mock_hf_download.assert_called_once_with(
            repo_id="unsloth/Qwen3-0.6B-GGUF",
            filename="model.gguf",
            revision="abc123",
            cache_dir=None,
        )
```
**EN:** This test validates `TestGGUFModelLoader.test_prepare_weights_repo_filename`. Key inputs are `mock_isfile`, `mock_hf_download`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `result == '/downloaded/model.gguf'`.
**CN:** 这个测试验证 `TestGGUFModelLoader.test_prepare_weights_repo_filename`。 关键输入包括 `mock_isfile`、`mock_hf_download`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `result == '/downloaded/model.gguf'`。

### Test method / 测试方法: TestGGUFModelLoader.test_prepare_weights_repo_quant_type (L138-L185)
```python
    @patch("vllm.config.model.get_hf_image_processor_config", return_value=None)
    @patch("vllm.transformers_utils.config.file_or_path_exists", return_value=True)
    @patch("vllm.config.model.get_config")
    @patch("vllm.config.model.is_gguf", return_value=True)
    @patch("vllm.model_executor.model_loader.gguf_loader.download_gguf")
    @patch("os.path.isfile", return_value=False)
    def test_prepare_weights_repo_quant_type(
        self,
        mock_isfile,
        mock_download_gguf,
        mock_is_gguf,
        mock_get_config,
        mock_file_exists,
        mock_get_image_config,
    ):
        """Test _prepare_weights with repo_id:quant_type format."""
        mock_hf_config = MagicMock()
        mock_hf_config.architectures = ["Qwen3ForCausalLM"]
# ... 22 lines omitted for brevity ...
        assert result == "/downloaded/model-IQ1_S.gguf"
        mock_download_gguf.assert_called_once_with(
            "unsloth/Qwen3-0.6B-GGUF",
            "IQ1_S",
            cache_dir=None,
            revision=None,
            ignore_patterns=["original/**/*"],
        )
```
**EN:** This test validates `TestGGUFModelLoader.test_prepare_weights_repo_quant_type`. Key inputs are `mock_isfile`, `mock_download_gguf`, `mock_is_gguf`, `mock_get_config`, `mock_file_exists`, `mock_get_image_config`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `result == '/downloaded/model-IQ1_S.gguf'`.
**CN:** 这个测试验证 `TestGGUFModelLoader.test_prepare_weights_repo_quant_type`。 关键输入包括 `mock_isfile`、`mock_download_gguf`、`mock_is_gguf`、`mock_get_config`、`mock_file_exists`、`mock_get_image_config`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `result == '/downloaded/model-IQ1_S.gguf'`。

### Test method / 测试方法: TestGGUFModelLoader.test_prepare_weights_invalid_format (L187-L224)
```python
    @patch("vllm.config.model.get_hf_image_processor_config", return_value=None)
    @patch("vllm.config.model.get_config")
    @patch("vllm.config.model.is_gguf", return_value=False)
    @patch("vllm.transformers_utils.gguf_utils.check_gguf_file", return_value=False)
    @patch("os.path.isfile", return_value=False)
    def test_prepare_weights_invalid_format(
        self,
        mock_isfile,
        mock_check_gguf,
        mock_is_gguf,
        mock_get_config,
        mock_get_image_config,
    ):
        """Test _prepare_weights with invalid format."""
        mock_hf_config = MagicMock()
        mock_hf_config.architectures = ["Qwen3ForCausalLM"]

        class MockTextConfig:
# ... 12 lines omitted for brevity ...

        # Create ModelConfig with a valid repo_id to avoid validation errors
        # Then test _prepare_weights with invalid format
        model_config = ModelConfig(model="unsloth/Qwen3-0.6B")
        # Manually set model to invalid format after creation
        model_config.model = "invalid-format"
        with pytest.raises(ValueError, match="Unrecognised GGUF reference"):
            loader._prepare_weights(model_config)
```
**EN:** This test validates `TestGGUFModelLoader.test_prepare_weights_invalid_format`. Key inputs are `mock_isfile`, `mock_check_gguf`, `mock_is_gguf`, `mock_get_config`, `mock_get_image_config`. It checks an expected failure path with `pytest.raises`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `TestGGUFModelLoader.test_prepare_weights_invalid_format`。 关键输入包括 `mock_isfile`、`mock_check_gguf`、`mock_is_gguf`、`mock_get_config`、`mock_get_image_config`。 它使用 `pytest.raises` 检查预期失败路径。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

## Key Concepts / 关键概念
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `unittest.mock.MagicMock`, `unittest.mock.patch`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `vllm.config.ModelConfig`, `vllm.config.load.LoadConfig`, `vllm.model_executor.model_loader.gguf_loader.GGUFModelLoader`, `vllm.model_executor.model_loader.weight_utils.download_gguf`
