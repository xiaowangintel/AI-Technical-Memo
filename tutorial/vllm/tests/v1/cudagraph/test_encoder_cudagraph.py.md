# test_encoder_cudagraph.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/cudagraph/test_encoder_cudagraph.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Unit tests for EncoderCudaGraphManager. / 该文件的文档字符串表明其用途：`unit tests for encodercudagraphmanager`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-13)
```python
"""Unit tests for EncoderCudaGraphManager.

Test organization:
  No GPU required:
    - TestFindBudgetGraph      — greedy budget selection logic
    - TestGetCumulativeStats   — hit/miss rate statistics
    - TestGetInputModality     — modality routing from mm_kwargs keys
  GPU required:
    - TestEncoderCudaGraphCaptureReplay — capture, replay, fallback, counters, chunking
    - TestEncoderCudaGraphVideoReplay   — video modality capture, replay
"""
```
**EN:** Module docstring that declares the scope of the file: Unit tests for EncoderCudaGraphManager.
**CN:** 模块文档字符串直接说明了文件范围：`unit tests for encodercudagraphmanager`。

### Imports and setup / 导入与设置 (lines 15-28)
```python
from typing import Any

import pytest
import torch

from vllm.platforms import current_platform
from vllm.v1.worker.encoder_cudagraph import (
    EncoderCudaGraphManager,
)
from vllm.v1.worker.encoder_cudagraph_defs import (
    EncoderCudaGraphCaptureInputs,
    EncoderCudaGraphConfig,
    EncoderCudaGraphReplayBuffers,
)
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.platforms, vllm.v1.worker.encoder_cudagraph, vllm.v1.worker.encoder_cudagraph_defs, vllm.config.compilation`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.platforms, vllm.v1.worker.encoder_cudagraph, vllm.v1.worker.encoder_cudagraph_defs, vllm.config.compilation`。

### _MockCompilationConfig (lines 35-45)
```python
class _MockCompilationConfig:
    """Minimal mock for VllmConfig.compilation_config."""

    def __init__(
        self,
        token_budgets: list[int] | None = None,
        max_mm_items: int = 0,
    ):
        self.encoder_cudagraph_token_budgets = token_budgets or []
        self.encoder_cudagraph_max_vision_items_per_batch = max_mm_items
        self.encoder_cudagraph_max_frames_per_batch = None
```
**EN:** Class `_MockCompilationConfig` groups 0 test method(s) and 1 helper/fixture method(s).
**CN:** 类 `_MockCompilationConfig` 组织了 0 个测试方法，以及 1 个辅助或 fixture 方法。

### _MockMultimodalConfig (lines 48-54)
```python
class _MockMultimodalConfig:
    mm_encoder_tp_mode = "replicate"

    def get_limit_per_prompt(self, modality: str) -> int:
        # Image-only mocks — return 0 for "video" to short-circuit the
        # max_frames_per_batch branch, so tests don't need a video-frame mock.
        return 0
```
**EN:** Class `_MockMultimodalConfig` groups 0 test method(s) and 1 helper/fixture method(s).
**CN:** 类 `_MockMultimodalConfig` 组织了 0 个测试方法，以及 1 个辅助或 fixture 方法。

### _MockModelConfig (lines 57-58)
```python
class _MockModelConfig:
    multimodal_config = _MockMultimodalConfig()
```
**EN:** Class `_MockModelConfig` groups 0 test method(s).
**CN:** 类 `_MockModelConfig` 组织了 0 个测试方法。

### _MockParallelConfig (lines 61-62)
```python
class _MockParallelConfig:
    tensor_parallel_size = 1
```
**EN:** Class `_MockParallelConfig` groups 0 test method(s).
**CN:** 类 `_MockParallelConfig` 组织了 0 个测试方法。

### _MockVllmConfig (lines 65-75)
```python
class _MockVllmConfig:
    """Minimal mock for VllmConfig used in __init__ tests."""

    def __init__(
        self,
        token_budgets: list[int] | None = None,
        max_mm_items: int = 0,
    ):
        self.compilation_config = _MockCompilationConfig(token_budgets, max_mm_items)
        self.model_config = _MockModelConfig()
        self.parallel_config = _MockParallelConfig()
```
**EN:** Class `_MockVllmConfig` groups 0 test method(s) and 1 helper/fixture method(s).
**CN:** 类 `_MockVllmConfig` 组织了 0 个测试方法，以及 1 个辅助或 fixture 方法。

### _MockModel (lines 78-94)
```python
class _MockModel:
    """Minimal mock implementing SupportsEncoderCudaGraph for __init__."""

    def __init__(self, min_budget: int = 4, max_budget: int = 128):
        self._min_budget = min_budget
        self._max_budget = max_budget

    def get_encoder_cudagraph_config(self) -> EncoderCudaGraphConfig:
        return EncoderCudaGraphConfig(
            modalities=["image"],
            input_key_by_modality={"image": "pixel_values"},
            buffer_keys=["dummy_buf"],
            out_hidden_size=32,
        )

    def get_encoder_cudagraph_budget_range(self, vllm_config):
        return (self._min_budget, self._max_budget)
```
**EN:** Class `_MockModel` groups 0 test method(s) and 3 helper/fixture method(s).
**CN:** 类 `_MockModel` 组织了 0 个测试方法，以及 3 个辅助或 fixture 方法。

### _make_manager_with_budgets (lines 97-111)
```python
def _make_manager_with_budgets(budgets: list[int]) -> EncoderCudaGraphManager:
    """Create a minimal EncoderCudaGraphManager with only token_budgets set.

    Skips the parts of __init__ that require a real VllmConfig / model
    by patching the attributes directly after construction.
    """
    mgr = object.__new__(EncoderCudaGraphManager)
    mgr.token_budgets = sorted(budgets)
    mgr.max_batch_size = 16
    mgr.use_dp = False
    mgr.budget_graphs = {}
    mgr.graph_hits = 0
    mgr.graph_misses = 0
    mgr.log_stats_interval = 100
    return mgr
```
**EN:** Helper function `_make_manager_with_budgets` encapsulates reusable logic for `manager with budgets`. Inputs: `budgets`. Key calls include `object.__new__, sorted`.
**CN:** 辅助函数 `_make_manager_with_budgets` 封装了与 `manager with budgets` 相关的可复用逻辑。 输入参数：`budgets`。 关键调用包括 `object.__new__, sorted`。

### TestGenerateBudgets (lines 119-136)
```python
class TestGenerateBudgets:
    """Auto-generate power-of-2 budgets from min to max."""

    def test_exact_powers_of_2(self):
        result = EncoderCudaGraphManager._generate_budgets(64, 1024)
        assert result == [64, 128, 256, 512, 1024]

    def test_max_not_power_of_2(self):
        result = EncoderCudaGraphManager._generate_budgets(64, 800)
        assert result == [64, 128, 256, 512, 800]

    def test_min_equals_max(self):
        result = EncoderCudaGraphManager._generate_budgets(64, 64)
        assert result == [64]

    def test_large_range(self):
        result = EncoderCudaGraphManager._generate_budgets(64, 8192)
        assert result == [64, 128, 256, 512, 1024, 2048, 4096, 8192]
```
**EN:** Class `TestGenerateBudgets` groups 4 test method(s). Representative scenarios: `test_exact_powers_of_2, test_max_not_power_of_2, test_min_equals_max, test_large_range`.
**CN:** 类 `TestGenerateBudgets` 组织了 4 个测试方法。 代表性场景：`test_exact_powers_of_2, test_max_not_power_of_2, test_min_equals_max, test_large_range`。

### TestFindBudgetGraph (lines 144-177)
```python
class TestFindBudgetGraph:
    """Budget greedy selection: smallest budget >= total_tokens."""

    @pytest.mark.parametrize(
        "total_tokens,budgets,expected",
        [
            # Exact match
            (2048, [2048, 4096, 8192], 2048),
            # Below smallest budget — picks smallest
            (100, [2048, 4096, 8192], 2048),
            # Zero tokens — picks smallest
            (0, [2048, 4096, 8192], 2048),
            # Between budgets — picks next one up
            (2049, [2048, 4096, 8192], 4096),
            (4097, [2048, 4096, 8192], 8192),
            # Exceeds all budgets — returns None (eager fallback)
            (9000, [2048, 4096, 8192], None),
            # Single budget, fits
    # ... excerpt omitted for brevity ...
    def test_find_budget(self, total_tokens, budgets, expected):
        assert result == expected
    def test_budgets_are_sorted(self):
        """Manager always sorts budgets ascending at init."""
        mgr = _make_manager_with_budgets([8192, 2048, 4096])
        assert mgr.token_budgets == [2048, 4096, 8192]
        # Budget selection still works correctly after sorting
        assert mgr._find_smallest_fitting_budget_given_tokens(3000) == 4096
```
**EN:** Class `TestFindBudgetGraph` groups 2 test method(s). Representative scenarios: `test_find_budget, test_budgets_are_sorted`.
**CN:** 类 `TestFindBudgetGraph` 组织了 2 个测试方法。 代表性场景：`test_find_budget, test_budgets_are_sorted`。

### TestGetCumulativeStats (lines 185-221)
```python
class TestGetCumulativeStats:
    """Statistics tracking and reporting."""

    def test_initial_stats_are_zero(self):
        mgr = _make_manager_with_budgets([2048])
        stats = mgr.get_cumulative_stats()
        assert stats["graph_hits"] == 0
        assert stats["graph_misses"] == 0
        assert stats["hit_rate"] == 0.0
    def test_hit_rate_calculation(self):
        mgr.graph_hits = 75
        mgr.graph_misses = 25
        assert stats["graph_hits"] == 75
        assert stats["graph_misses"] == 25
        assert stats["hit_rate"] == pytest.approx(0.75)
    # ... excerpt omitted for brevity ...
        assert mgr.get_cumulative_stats()["hit_rate"] == pytest.approx(1.0)
        assert mgr.get_cumulative_stats()["hit_rate"] == pytest.approx(0.0)
    def test_stats_report_budget_info(self):
        budgets = [2048, 4096, 8192]
        mgr = _make_manager_with_budgets(budgets)
        assert stats["num_budgets"] == 0  # no graphs captured yet
        assert stats["token_budgets"] == budgets
```
**EN:** Class `TestGetCumulativeStats` groups 5 test method(s). Representative scenarios: `test_initial_stats_are_zero, test_hit_rate_calculation, test_all_hits, test_all_misses, test_stats_report_budget_info`.
**CN:** 类 `TestGetCumulativeStats` 组织了 5 个测试方法。 代表性场景：`test_initial_stats_are_zero, test_hit_rate_calculation, test_all_hits, test_all_misses, test_stats_report_budget_info`。

### Module state / 模块级状态 (lines 229-239)
```python
_SPATIAL_MERGE = 2
_HIDDEN = 32
_PATCH_SIZE = 4  # H/W per patch in grid_thw units
_TEMPORAL_PATCH = 1
_IN_CHANNELS = 3
# flattened_patch_size = in_channels * temporal_patch * patch_size^2
_FLAT = _IN_CHANNELS * _TEMPORAL_PATCH * _PATCH_SIZE * _PATCH_SIZE  # 48

# Test budgets: small to keep capture fast
_BUDGETS = [16, 64]
_MAX_BATCH = 4
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_SPATIAL_MERGE, _HIDDEN, _PATCH_SIZE, _TEMPORAL_PATCH, _IN_CHANNELS, _FLAT, ...`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_SPATIAL_MERGE, _HIDDEN, _PATCH_SIZE, _TEMPORAL_PATCH, _IN_CHANNELS, _FLAT, ...`。

### _count_input_patches (lines 242-243)
```python
def _count_input_patches(grid_thw_list: list[list[int]]) -> int:
    return sum(t * h * w for t, h, w in grid_thw_list)
```
**EN:** Helper function `_count_input_patches` encapsulates reusable logic for `count input patches`. Inputs: `grid_thw_list`. Key calls include `sum`.
**CN:** 辅助函数 `_count_input_patches` 封装了与 `count input patches` 相关的可复用逻辑。 输入参数：`grid_thw_list`。 关键调用包括 `sum`。

### _count_output_tokens (lines 246-250)
```python
def _count_output_tokens(
    grid_thw_list: list[list[int]], spatial_merge_size: int
) -> int:
    m = spatial_merge_size
    return sum(t * (h // m) * (w // m) for t, h, w in grid_thw_list)
```
**EN:** Helper function `_count_output_tokens` encapsulates reusable logic for `count output tokens`. Inputs: `grid_thw_list, spatial_merge_size`. Key calls include `sum`.
**CN:** 辅助函数 `_count_output_tokens` 封装了与 `count output tokens` 相关的可复用逻辑。 输入参数：`grid_thw_list, spatial_merge_size`。 关键调用包括 `sum`。

### SimpleMockViTModel (lines 253-395)
```python
class SimpleMockViTModel(torch.nn.Module):
    """Minimal ViT model for CUDA graph tests.

    Implements the SupportsEncoderCudaGraph protocol by providing
    all required methods. The forward pass projects patches and
    simulates spatial merge by averaging groups of m^2 patches.
    """
    supports_encoder_cudagraph = True
    def __init__(self):
        super().__init__()
        self.proj = torch.nn.Linear(_FLAT, _HIDDEN)
        self.spatial_merge_size = _SPATIAL_MERGE
        self.out_hidden_size = _HIDDEN
    def get_encoder_cudagraph_config(self) -> EncoderCudaGraphConfig:
        return EncoderCudaGraphConfig(
    # ... excerpt omitted for brevity ...
        return "image"
        return (4, 128)
        return len(mm_kwargs["image_grid_thw"])
        return [t * (h // m) * (w // m) for t, h, w in mm_kwargs["image_grid_thw"]]
        return [t * h * w for t, h, w in mm_kwargs["image_grid_thw"]]
            return {
    def _forward(self, pixel_values: torch.Tensor) -> torch.Tensor:
        m2 = _SPATIAL_MERGE**2
        out = self.proj(pixel_values)
        n_out = out.shape[0] // m2
        return out[: n_out * m2].view(n_out, m2, _HIDDEN).mean(dim=1)
```
**EN:** Class `SimpleMockViTModel` groups 0 test method(s) and 13 helper/fixture method(s). Bases: `torch.nn.Module`.
**CN:** 类 `SimpleMockViTModel` 组织了 0 个测试方法，以及 13 个辅助或 fixture 方法。 基类：`torch.nn.Module`。

### _make_manager_for_gpu (lines 398-423)
```python
def _make_manager_for_gpu(
    model: SimpleMockViTModel,
    token_budgets: list[int],
    max_batch_size: int,
    device: torch.device,
    dtype: torch.dtype,
    *,
    max_frames_per_batch: int | None = None,
) -> EncoderCudaGraphManager:
    """Create EncoderCudaGraphManager bypassing VllmConfig for GPU tests."""
    mgr = object.__new__(EncoderCudaGraphManager)
    mgr.token_budgets = sorted(token_budgets)
    mgr.max_batch_size = max_batch_size
    mgr.max_frames_per_batch = (
        max_frames_per_batch if max_frames_per_batch is not None else max_batch_size * 2
    )
    mgr.use_dp = False
    mgr.budget_graphs = {}
    mgr.graph_hits = 0
    mgr.graph_misses = 0
    mgr.log_stats_interval = 100
    mgr.model = model
    mgr.config = model.get_encoder_cudagraph_config()
    mgr.device = device
    mgr.dtype = dtype
    return mgr
```
**EN:** Helper function `_make_manager_for_gpu` encapsulates reusable logic for `manager for GPU`. Inputs: `model, token_budgets, max_batch_size, device, dtype`. Key calls include `object.__new__, sorted, model.get_encoder_cudagraph_config`.
**CN:** 辅助函数 `_make_manager_for_gpu` 封装了与 `manager for gpu` 相关的可复用逻辑。 输入参数：`model, token_budgets, max_batch_size, device, dtype`。 关键调用包括 `object.__new__, sorted, model.get_encoder_cudagraph_config`。

### _make_pixel_values (lines 426-433)
```python
def _make_pixel_values(
    grid_thw_list: list[list[int]],
    device: torch.device,
    dtype: torch.dtype,
) -> torch.Tensor:
    """Random pixel_values matching the total input patch count."""
    n = _count_input_patches(grid_thw_list)
    return torch.randn(n, _FLAT, device=device, dtype=dtype)
```
**EN:** Helper function `_make_pixel_values` encapsulates reusable logic for `pixel values`. Inputs: `grid_thw_list, device, dtype`. Key calls include `_count_input_patches, torch.randn`.
**CN:** 辅助函数 `_make_pixel_values` 封装了与 `pixel values` 相关的可复用逻辑。 输入参数：`grid_thw_list, device, dtype`。 关键调用包括 `_count_input_patches, torch.randn`。

### _make_mm_kwargs (lines 436-445)
```python
def _make_mm_kwargs(
    grid_thw_list: list[list[int]],
    device: torch.device,
    dtype: torch.dtype,
) -> dict[str, Any]:
    """Create mm_kwargs for testing."""
    return {
        "pixel_values": _make_pixel_values(grid_thw_list, device, dtype),
        "image_grid_thw": grid_thw_list,
    }
```
**EN:** Helper function `_make_mm_kwargs` encapsulates reusable logic for `mm kwargs`. Inputs: `grid_thw_list, device, dtype`. Key calls include `_make_pixel_values`.
**CN:** 辅助函数 `_make_mm_kwargs` 封装了与 `mm kwargs` 相关的可复用逻辑。 输入参数：`grid_thw_list, device, dtype`。 关键调用包括 `_make_pixel_values`。

### _make_video_mm_kwargs (lines 448-457)
```python
def _make_video_mm_kwargs(
    grid_thw_list: list[list[int]],
    device: torch.device,
    dtype: torch.dtype,
) -> dict[str, Any]:
    """Create video mm_kwargs (pixel_values_videos / video_grid_thw) for testing."""
    return {
        "pixel_values_videos": _make_pixel_values(grid_thw_list, device, dtype),
        "video_grid_thw": grid_thw_list,
    }
```
**EN:** Helper function `_make_video_mm_kwargs` encapsulates reusable logic for `video mm kwargs`. Inputs: `grid_thw_list, device, dtype`. Key calls include `_make_pixel_values`.
**CN:** 辅助函数 `_make_video_mm_kwargs` 封装了与 `视频 mm kwargs` 相关的可复用逻辑。 输入参数：`grid_thw_list, device, dtype`。 关键调用包括 `_make_pixel_values`。

### TestEncoderCudaGraphCaptureReplay (lines 466-542)
```python
class TestEncoderCudaGraphCaptureReplay:
    def setup_method(self):
        self.device = torch.device("cuda:0")
        self.dtype = torch.float16
        self.model = SimpleMockViTModel().to(self.device).half()
        self.mgr = _make_manager_for_gpu(
            self.model, _BUDGETS, _MAX_BATCH, self.device, self.dtype
        )
        self.mgr.capture()

    # --- capture ---
    def test_capture_creates_one_graph_per_budget(self):
        assert len(self.mgr.budget_graphs) == len(_BUDGETS)
        assert set(self.mgr.budget_graphs.keys()) == set(_BUDGETS)
    # --- output shape ---
    # ... excerpt omitted for brevity ...
        assert result is not None
        assert len(result) == 2
        assert result[0].shape == (4, _HIDDEN)
        assert result[1].shape == (16, _HIDDEN)
        mm_kwargs = _make_mm_kwargs(grid_thw, self.device, self.dtype)
        result = self.mgr.execute(mm_kwargs)
        assert len(result) == n_images
        for out in result:
            assert out.shape == (4, _HIDDEN)
```
**EN:** Class `TestEncoderCudaGraphCaptureReplay` groups 7 test method(s) and 1 helper/fixture method(s). Representative scenarios: `test_capture_creates_one_graph_per_budget, test_execute_returns_one_tensor_per_image, test_execute_output_tokens_per_image, test_eager_fallback_when_tokens_exceed_all_budgets, test_hit_counter_increments_by_num_images, test_miss_counter_increments_by_num_images, ...`.
**CN:** 类 `TestEncoderCudaGraphCaptureReplay` 组织了 7 个测试方法，以及 1 个辅助或 fixture 方法。 代表性场景：`test_capture_creates_one_graph_per_budget, test_execute_returns_one_tensor_per_image, test_execute_output_tokens_per_image, test_eager_fallback_when_tokens_exceed_all_budgets, test_hit_counter_increments_by_num_images, test_miss_counter_increments_by_num_images, ...`。

### SimpleMockViTVideoModel (lines 550-688)
```python
class SimpleMockViTVideoModel(SimpleMockViTModel):
    """ViT mock that supports both image and video modalities.

    Reuses SimpleMockViTModel's NN weights and _forward() logic.
    Only the protocol methods that are key-dependent are overridden.
    """
    def get_encoder_cudagraph_config(self) -> EncoderCudaGraphConfig:
        return EncoderCudaGraphConfig(
            modalities=["image", "video"],
            input_key_by_modality={
                "image": "pixel_values",
                "video": "pixel_values_videos",
            },
            buffer_keys=["dummy_buf"],
            out_hidden_size=_HIDDEN,
        )
    # ... excerpt omitted for brevity ...
        return "video" if "video_grid_thw" in mm_kwargs else "image"
        return mm_kwargs[key]
        return len(self._get_grid_thw(mm_kwargs))
        return [t * (h // m) * (w // m) for t, h, w in self._get_grid_thw(mm_kwargs)]
        return [t * h * w for t, h, w in self._get_grid_thw(mm_kwargs)]
        self, mm_kwargs: dict[str, Any], buffers: dict[str, torch.Tensor]
    ) -> torch.Tensor:
        return self._forward(self._get_pixel_values(mm_kwargs))
    def encoder_eager_forward(self, mm_kwargs: dict[str, Any]) -> torch.Tensor:
```
**EN:** Class `SimpleMockViTVideoModel` groups 0 test method(s) and 12 helper/fixture method(s). Bases: `SimpleMockViTModel`.
**CN:** 类 `SimpleMockViTVideoModel` 组织了 0 个测试方法，以及 12 个辅助或 fixture 方法。 基类：`SimpleMockViTModel`。

### TestGetInputModality (lines 696-729)
```python
class TestGetInputModality:
    """get_input_modality returns correct modality based on mm_kwargs keys."""

    def test_image_only_model_always_returns_image(self):
        model = SimpleMockViTModel()
        mm_kwargs = {
            "pixel_values": torch.zeros(1, _FLAT),
            "image_grid_thw": [[1, 4, 4]],
        }
        assert model.get_input_modality(mm_kwargs) == "image"
    def test_video_model_returns_image_for_image_kwargs(self):
        model = SimpleMockViTVideoModel()
    # ... excerpt omitted for brevity ...
        assert model.get_input_modality(mm_kwargs) == "video"
        cfg = model.get_encoder_cudagraph_config()
        assert "image" in cfg.modalities
        assert "video" in cfg.modalities
        assert cfg.input_key_by_modality["image"] == "pixel_values"
        assert cfg.input_key_by_modality["video"] == "pixel_values_videos"
```
**EN:** Class `TestGetInputModality` groups 4 test method(s). Representative scenarios: `test_image_only_model_always_returns_image, test_video_model_returns_image_for_image_kwargs, test_video_model_returns_video_for_video_kwargs, test_video_model_config_has_both_modalities`.
**CN:** 类 `TestGetInputModality` 组织了 4 个测试方法。 代表性场景：`test_image_only_model_always_returns_image, test_video_model_returns_image_for_image_kwargs, test_video_model_returns_video_for_video_kwargs, test_video_model_config_has_both_modalities`。

### Module state / 模块级状态 (lines 736-737)
```python
_VIDEO_MAX_BATCH = 4
_VIDEO_MAX_FRAMES = 8  # 2 frames per item at max_batch_size=4
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_VIDEO_MAX_BATCH, _VIDEO_MAX_FRAMES`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_VIDEO_MAX_BATCH, _VIDEO_MAX_FRAMES`。

### TestEncoderCudaGraphVideoReplay (lines 741-824)
```python
class TestEncoderCudaGraphVideoReplay:
    def setup_method(self):
        self.device = torch.device("cuda:0")
        self.dtype = torch.float16
        self.model = SimpleMockViTVideoModel().to(self.device).half()
        self.mgr = _make_manager_for_gpu(
            self.model,
            _BUDGETS,
            _VIDEO_MAX_BATCH,
            self.device,
            self.dtype,
            max_frames_per_batch=_VIDEO_MAX_FRAMES,
        )
        self.mgr.capture()

    # --- capture ---
    def test_capture_creates_one_graph_per_budget(self):
    # ... excerpt omitted for brevity ...
        assert len(self.mgr.budget_graphs) == len(_BUDGETS)
        assert set(self.mgr.budget_graphs.keys()) == set(_BUDGETS)
        assert result is not None
        assert len(result) == 2
        assert result[0].shape == (8, _HIDDEN)
        assert len(img_result) == 2
        assert len(vid_result) == 1
        assert img_result[0].shape == (4, _HIDDEN)
        assert vid_result[0].shape == (8, _HIDDEN)
```
**EN:** Class `TestEncoderCudaGraphVideoReplay` groups 7 test method(s) and 1 helper/fixture method(s). Representative scenarios: `test_capture_creates_one_graph_per_budget, test_video_execute_returns_one_tensor_per_video, test_video_output_tokens_per_item, test_video_eager_fallback_when_tokens_exceed_all_budgets, test_video_hit_counter_increments_by_num_videos, test_video_miss_counter_increments_for_oversized_video, ...`.
**CN:** 类 `TestEncoderCudaGraphVideoReplay` 组织了 7 个测试方法，以及 1 个辅助或 fixture 方法。 代表性场景：`test_capture_creates_one_graph_per_budget, test_video_execute_returns_one_tensor_per_video, test_video_output_tokens_per_item, test_video_eager_fallback_when_tokens_exceed_all_budgets, test_video_hit_counter_increments_by_num_videos, test_video_miss_counter_increments_for_oversized_video, ...`。

### TestInitInvariantValidation (lines 832-934)
```python
class TestInitInvariantValidation:
    """Ensure max_batch_size <= min(token_budgets) for all config paths."""

    def _make_mgr(
        self,
        token_budgets=None,
        max_mm_items=0,
        min_budget=4,
        max_budget=128,
    ):
        vllm_config = _MockVllmConfig(token_budgets, max_mm_items)
        model = _MockModel(min_budget, max_budget)
        return EncoderCudaGraphManager(
            vllm_config=vllm_config,
            device=torch.device("cpu"),
            dtype=torch.float32,
            model=model,
        )
    # ... excerpt omitted for brevity ...
        assert mgr.max_batch_size <= min(mgr.token_budgets)
        with pytest.raises(ValueError, match="must be <= smallest token budget"):
        assert mgr.max_batch_size == 32
        assert mgr.token_budgets == [64, 128]
        assert mgr.max_batch_size == 64
    def test_user_budgets_negative_raises(self):
        from vllm.config.compilation import CompilationConfig
        with pytest.raises(ValueError, match="must be positive"):
            CompilationConfig(encoder_cudagraph_token_budgets=[-1, 64])
```
**EN:** Class `TestInitInvariantValidation` groups 13 test method(s) and 1 helper/fixture method(s). Representative scenarios: `test_auto_inferred_invariant_holds, test_auto_inferred_small_range, test_user_specified_bad_combo_raises, test_user_specified_valid_combo, test_user_specified_exact_boundary, test_user_max_mm_items_only_adjusts_budgets, ...`.
**CN:** 类 `TestInitInvariantValidation` 组织了 13 个测试方法，以及 1 个辅助或 fixture 方法。 代表性场景：`test_auto_inferred_invariant_holds, test_auto_inferred_small_range, test_user_specified_bad_combo_raises, test_user_specified_valid_combo, test_user_specified_exact_boundary, test_user_max_mm_items_only_adjusts_budgets, ...`。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化
- **EN:** Worker-side execution and device coordination
- **CN:** worker 侧执行与设备协同
- **EN:** GPU-specific execution assumptions
- **CN:** GPU 特定执行假设

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm.platforms, vllm.v1.worker.encoder_cudagraph, vllm.v1.worker.encoder_cudagraph_defs, vllm.config.compilation`.
- **CN:** 被测试的 vLLM 模块：`vllm.platforms, vllm.v1.worker.encoder_cudagraph, vllm.v1.worker.encoder_cudagraph_defs, vllm.config.compilation`。
- **EN:** Standard-library support: `typing`.
- **CN:** 标准库支持：`typing`。
