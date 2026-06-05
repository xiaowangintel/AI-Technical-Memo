# test_consistency_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/test_consistency_metrics.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates consistency metrics with focused assertions and fixtures. Key symbols include `_solid_image`, `test_consistency_gt_urls_are_pinned_to_ci_data_revision`, `test_pixel_metrics_identical_image`. / 该测试模块通过有针对性的断言与夹具，验证 consistency metrics 的实现。 关键符号包括 `_solid_image`, `test_consistency_gt_urls_are_pinned_to_ci_data_revision`, `test_pixel_metrics_identical_image`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and module setup / 导入与模块初始化
```python
import math

import numpy as np

from sglang.multimodal_gen.test import test_utils
from sglang.multimodal_gen.test.test_utils import (
    ConsistencyThresholds,
    LoadedConsistencyGT,
    compare_with_gt,
    compute_mean_abs_diff,
    compute_psnr,
    compute_ssim,
    save_consistency_failure_artifact,
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 17-18: Function `_solid_image` / 函数 `_solid_image`
```python
def _solid_image(value: int, size: int = 32) -> np.ndarray:
    return np.full((size, size, 3), value, dtype=np.uint8)
```
**EN:** This function drives `_solid_image` with inputs such as `value`, `size`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_solid_image`，主要处理 `value`, `size` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 21-26: Function `test_consistency_gt_urls_are_pinned_to_ci_data_revision` / 函数 `test_consistency_gt_urls_are_pinned_to_ci_data_revision`
```python
def test_consistency_gt_urls_are_pinned_to_ci_data_revision():
    revision_path = f"/ci-data/{test_utils.SGL_TEST_FILES_CI_DATA_REVISION}/"

    assert "/ci-data/main/" not in test_utils.SGL_TEST_FILES_CONSISTENCY_GT_ROOT
    assert revision_path in test_utils.SGL_TEST_FILES_OFFICIAL_CONSISTENCY_GT_BASE
    assert revision_path in test_utils.SGL_TEST_FILES_SGLANG_CONSISTENCY_GT_BASE
```
**EN:** This function drives `test_consistency_gt_urls_are_pinned_to_ci_data_revision`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `test_consistency_gt_urls_are_pinned_to_ci_data_revision`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 29-38: Function `test_pixel_metrics_identical_image` / 函数 `test_pixel_metrics_identical_image`
```python
def test_pixel_metrics_identical_image():
    image = _solid_image(128)

    ssim = compute_ssim(image, image)
    psnr = compute_psnr(image, image)
    mean_abs_diff = compute_mean_abs_diff(image, image)

    assert ssim == 1.0
    assert math.isinf(psnr)
    assert mean_abs_diff == 0.0
```
**EN:** This function drives `test_pixel_metrics_identical_image`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `test_pixel_metrics_identical_image`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 41-51: Function `test_pixel_metrics_detect_different_image` / 函数 `test_pixel_metrics_detect_different_image`
```python
def test_pixel_metrics_detect_different_image():
    image = _solid_image(128)
    other = _solid_image(0)

    ssim = compute_ssim(image, other)
    psnr = compute_psnr(image, other)
    mean_abs_diff = compute_mean_abs_diff(image, other)

    assert ssim < 0.95
    assert psnr < 28.0
    assert mean_abs_diff > 8.0
```
**EN:** This function drives `test_pixel_metrics_detect_different_image`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `test_pixel_metrics_detect_different_image`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 54-82: Function `test_compare_with_gt_passes_for_identical_image` / 函数 `test_compare_with_gt_passes_for_identical_image`
```python
def test_compare_with_gt_passes_for_identical_image(monkeypatch):
    gt_image = _solid_image(128)

    monkeypatch.setattr(
        test_utils,
        "compute_clip_embedding",
        lambda image: np.array([1.0, 0.0], dtype=np.float32),
    )

    result = compare_with_gt(
        output_frames=[gt_image.copy()],
        gt_data=LoadedConsistencyGT(
            images=[gt_image.copy()],
            embeddings=[np.array([1.0, 0.0], dtype=np.float32)],
# ...
    assert result.min_similarity == 1.0
    assert result.min_ssim == 1.0
    assert math.isinf(result.min_psnr)
    assert result.max_mean_abs_diff == 0.0
```
**EN:** This function drives `test_compare_with_gt_passes_for_identical_image` with inputs such as `monkeypatch`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `test_compare_with_gt_passes_for_identical_image`，主要处理 `monkeypatch` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 85-124: Function `test_compare_with_gt_uses_worst_frame_for_video` / 函数 `test_compare_with_gt_uses_worst_frame_for_video`
```python
def test_compare_with_gt_uses_worst_frame_for_video(monkeypatch):
    gt_frame_0 = _solid_image(128)
    gt_frame_1 = _solid_image(128)
    bad_frame = _solid_image(0)

    monkeypatch.setattr(
        test_utils,
        "compute_clip_embedding",
        lambda image: np.array([1.0, 0.0], dtype=np.float32),
    )

    result = compare_with_gt(
        output_frames=[gt_frame_0.copy(), bad_frame],
        gt_data=LoadedConsistencyGT(
# ...
        or not metric.psnr_passed
        or not metric.mean_abs_diff_passed
        for metric in result.frame_metrics
    )
```
**EN:** This function drives `test_compare_with_gt_uses_worst_frame_for_video` with inputs such as `monkeypatch`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `test_compare_with_gt_uses_worst_frame_for_video`，主要处理 `monkeypatch` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 127-174: Function `test_save_consistency_failure_artifact` / 函数 `test_save_consistency_failure_artifact`
```python
def test_save_consistency_failure_artifact(tmp_path, monkeypatch):
    gt_image = _solid_image(128)
    bad_image = _solid_image(0)

    monkeypatch.setattr(
        test_utils,
        "compute_clip_embedding",
        lambda image: np.array([1.0, 0.0], dtype=np.float32),
    )

    result = compare_with_gt(
        output_frames=[bad_image],
        gt_data=LoadedConsistencyGT(
            images=[gt_image],
# ...
    assert (tmp_path / "consistency_failures" / "index.html").exists()
    assert (
        tmp_path / "consistency_failures" / "generated" / "unit_image_fail_1gpu.png"
    ).exists()
```
**EN:** This function drives `test_save_consistency_failure_artifact` with inputs such as `tmp_path`, `monkeypatch`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `test_save_consistency_failure_artifact`，主要处理 `tmp_path`, `monkeypatch` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- Automated verification / 自动化验证
- Command-line interface / 命令行接口
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程
- Symbol `_solid_image` anchors the module API / 符号 `_solid_image` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.test`, `sglang.multimodal_gen.test.test_utils`
- **External / 外部**: `numpy`
- **Stdlib / 标准库**: `math`
