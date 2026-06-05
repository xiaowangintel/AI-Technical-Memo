# test_connector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/multimodal/media/test_connector.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Connector behavior in the Multimodal test area through focused pytest scenarios. It focuses on scenarios such as Url Images, Get Supported Suffixes, Image Equals. / 该文件在 Multimodal 测试域中，通过有针对性的 pytest 场景验证 Connector 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-34)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import asyncio
import mimetypes
import os
import shutil
import time
from tempfile import NamedTemporaryFile, TemporaryDirectory

import aiohttp
import numpy as np
import pybase64 as base64
import pytest
import requests
import torch
from PIL import Image, ImageChops

from vllm.multimodal.image import convert_image_mode
from vllm.multimodal.inputs import PlaceholderRange
from vllm.multimodal.media import MediaConnector

# Test different image extensions (JPG/PNG) and formats (gray/RGB/RGBA)
TEST_IMAGE_ASSETS = [
    "2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",  # "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg"
    "Grayscale_8bits_palette_sample_image.png",  # "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/Grayscale_8bits_palette_sample_image.png",
    "1280px-Venn_diagram_rgb.svg.png",  # "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/1280px-Venn_diagram_rgb.svg.png",
    "RGBA_comp.png",  # "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/RGBA_comp.png",
]

TEST_VIDEO_URLS = [
    "https://www.bogotobogo.com/python/OpenCV_Python/images/mean_shift_tracking/slow_traffic_small.mp4",
    "https://github.com/opencv/opencv/raw/refs/tags/4.12.0/samples/data/vtest.avi",
]
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `asyncio`, `mimetypes`, `aiohttp`, `numpy`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: url_images (lines 37-42)
```python
@pytest.fixture(scope="module")
def url_images(local_asset_server) -> dict[str, Image.Image]:
    return {
        image_url: local_asset_server.get_image_asset(image_url)
        for image_url in TEST_IMAGE_ASSETS
    }
```
**EN:** Provides a pytest fixture for Url Images. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`, `local_asset_server.get_image_asset`.
**CN:** 该代码块定义 pytest 夹具 `url_images`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `local_asset_server.get_image_asset` 构造或返回测试所需的值。

### Helper: get_supported_suffixes (lines 45-52)
```python
def get_supported_suffixes() -> tuple[str, ...]:
    # We should at least test the file types mentioned in GPT-4 with Vision
    OPENAI_SUPPORTED_SUFFIXES = (".png", ".jpeg", ".jpg", ".webp", ".gif")

    # Additional file types that are supported by us
    EXTRA_SUPPORTED_SUFFIXES = (".bmp", ".tiff")

    return OPENAI_SUPPORTED_SUFFIXES + EXTRA_SUPPORTED_SUFFIXES
```
**EN:** Implements a reusable helper for Get Supported Suffixes, reducing duplication across related tests.
**CN:** 该辅助函数为 Get Supported Suffixes 提供可复用逻辑，用于减少相关测试之间的重复代码。

### Helper: _image_equals (lines 55-56)
```python
def _image_equals(a: Image.Image, b: Image.Image) -> bool:
    return (np.asarray(a) == np.asarray(convert_image_mode(b, a.mode))).all()
```
**EN:** Implements a reusable helper for Image Equals, reducing duplication across related tests. It coordinates operations such as `(np.asarray(a) == np.asarray(convert_image_mode(b, a.mode))).all`, `np.asarray`, `convert_image_mode`.
**CN:** 该辅助函数为 Image Equals 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `(np.asarray(a) == np.asarray(convert_image_mode(b, a.mode))).all`, `np.asarray`, `convert_image_mode` 等操作。

### Test: test_fetch_image_http (lines 59-66)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("image_url", TEST_IMAGE_ASSETS, indirect=True)
async def test_fetch_image_http(image_url: str):
    connector = MediaConnector()

    image_sync = connector.fetch_image(image_url)
    image_async = await connector.fetch_image_async(image_url)
    assert _image_equals(image_sync, image_async)
```
**EN:** Async Checks Fetch Image Http under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `MediaConnector`, `connector.fetch_image` before asserting the expected outcome.
**CN:** 该测试用例验证 Fetch Image Http 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `MediaConnector`, `connector.fetch_image` 驱动目标逻辑，再断言预期结果。

### Test: test_fetch_image_base64 (lines 69-111)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("raw_image_url", TEST_IMAGE_ASSETS)
@pytest.mark.parametrize("suffix", get_supported_suffixes())
async def test_fetch_image_base64(
    url_images: dict[str, Image.Image], raw_image_url: str, suffix: str
):
    connector = MediaConnector(
        # Domain restriction should not apply to data URLs.
        allowed_media_domains=[
            "www.bogotobogo.com",
            "github.com",
        ]
    )
    url_image = url_images[raw_image_url]

    try:
        mime_type = Image.MIME[Image.registered_extensions()[suffix]]
    except KeyError:
        try:
# ... omitted for brevity ...

            raise

        base64_image = base64.b64encode(f.read()).decode("utf-8")
        data_url = f"data:{mime_type};base64,{base64_image}"

        data_image_sync = connector.fetch_image(data_url)
        if _image_equals(url_image, Image.open(f)):
            assert _image_equals(url_image, data_image_sync)
        else:
            pass  # Lossy format; only check that image can be opened

        data_image_async = await connector.fetch_image_async(data_url)
        assert _image_equals(data_image_sync, data_image_async)
```
**EN:** Async Checks Fetch Image Base64 under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `MediaConnector`, `get_supported_suffixes` before asserting the expected outcome.
**CN:** 该测试用例验证 Fetch Image Base64 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `MediaConnector`, `get_supported_suffixes` 驱动目标逻辑，再断言预期结果。

### Test: test_fetch_image_local_files (lines 114-152)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("image_url", TEST_IMAGE_ASSETS, indirect=True)
async def test_fetch_image_local_files(image_url: str):
    connector = MediaConnector()

    with TemporaryDirectory() as temp_dir:
        local_connector = MediaConnector(allowed_local_media_path=temp_dir)

        origin_image = connector.fetch_image(image_url)
        origin_image.save(
            os.path.join(temp_dir, os.path.basename(image_url)),
            quality=100,
            icc_profile=origin_image.info.get("icc_profile"),
        )

        image_async = await local_connector.fetch_image_async(
            f"file://{temp_dir}/{os.path.basename(image_url)}"
        )
        image_sync = local_connector.fetch_image(
# ... omitted for brevity ...
            await local_connector.fetch_image_async(
                f"file://{temp_dir}/../{os.path.basename(image_url)}"
            )
        with pytest.raises(RuntimeError, match="Cannot load local files"):
            await connector.fetch_image_async(
                f"file://{temp_dir}/../{os.path.basename(image_url)}"
            )

        with pytest.raises(ValueError, match="must be a subpath"):
            local_connector.fetch_image(
                f"file://{temp_dir}/../{os.path.basename(image_url)}"
            )
        with pytest.raises(RuntimeError, match="Cannot load local files"):
            connector.fetch_image(f"file://{temp_dir}/../{os.path.basename(image_url)}")
```
**EN:** Async Checks Fetch Image Local Files under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `MediaConnector`, `TemporaryDirectory` before asserting the expected outcome.
**CN:** 该测试用例验证 Fetch Image Local Files 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `MediaConnector`, `TemporaryDirectory` 驱动目标逻辑，再断言预期结果。

### Test: test_fetch_image_local_files_with_space_in_name (lines 155-179)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("image_url", [TEST_IMAGE_ASSETS[0]], indirect=True)
async def test_fetch_image_local_files_with_space_in_name(image_url: str):
    connector = MediaConnector()

    with TemporaryDirectory() as temp_dir:
        local_connector = MediaConnector(allowed_local_media_path=temp_dir)

        origin_image = connector.fetch_image(image_url)
        filename = "file name with space.jpg"
        origin_image.save(
            os.path.join(temp_dir, filename),
            quality=100,
            icc_profile=origin_image.info.get("icc_profile"),
        )

        try:
            image_async = await local_connector.fetch_image_async(
                f"file://{temp_dir}/{filename}"
            )
            image_sync = local_connector.fetch_image(f"file://{temp_dir}/{filename}")
        except FileNotFoundError as e:
            pytest.fail("Failed to fetch image with space in name: {}".format(e))
        # Check that the images are equal
        assert not ImageChops.difference(image_sync, image_async).getbbox()
```
**EN:** Async Checks Fetch Image Local Files With Space In Name under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `MediaConnector`, `TemporaryDirectory` before asserting the expected outcome.
**CN:** 该测试用例验证 Fetch Image Local Files With Space In Name 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `MediaConnector`, `TemporaryDirectory` 驱动目标逻辑，再断言预期结果。

### Test: test_fetch_image_error_conversion (lines 182-192)
```python
@pytest.mark.asyncio
async def test_fetch_image_error_conversion():
    connector = MediaConnector()
    broken_img = "data:image/png;base64,aGVsbG9fdmxsbV9jb21tdW5pdHkK"

    # PIL.UnidentifiedImageError should be converted to ValueError
    with pytest.raises(ValueError):
        await connector.fetch_image_async(broken_img)

    with pytest.raises(ValueError):
        connector.fetch_image(broken_img)
```
**EN:** Async Checks Fetch Image Error Conversion under a focused test scenario. The body exercises logic via `MediaConnector`, `pytest.raises`, `connector.fetch_image` before asserting the expected outcome.
**CN:** 该测试用例验证 Fetch Image Error Conversion 在特定场景下的行为。 函数体会先通过 `MediaConnector`, `pytest.raises`, `connector.fetch_image` 驱动目标逻辑，再断言预期结果。

### Test: test_fetch_video_http (lines 195-215)
```python
@pytest.mark.flaky(reruns=3, reruns_delay=5)
@pytest.mark.asyncio
@pytest.mark.parametrize("video_url", TEST_VIDEO_URLS)
@pytest.mark.parametrize("num_frames", [-1, 32, 1800])
async def test_fetch_video_http(video_url: str, num_frames: int):
    connector = MediaConnector(
        media_io_kwargs={
            "video": {
                "num_frames": num_frames,
            }
        }
    )

    try:
        video_sync, metadata_sync = connector.fetch_video(video_url)
        video_async, metadata_async = await connector.fetch_video_async(video_url)
    except (TimeoutError, asyncio.TimeoutError) as e:
        pytest.skip(f"Timeout fetching video (CI network flakiness): {e}")

    assert np.array_equal(video_sync, video_async)
    assert metadata_sync == metadata_async
```
**EN:** Async Checks Fetch Video Http under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.flaky`, `pytest.mark.parametrize`, `MediaConnector` before asserting the expected outcome.
**CN:** 该测试用例验证 Fetch Video Http 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.flaky`, `pytest.mark.parametrize`, `MediaConnector` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
test_fetch_video_http_with_dynamic_loader
test_placeholder_range_get_embeds_indices_in_range
test_placeholder_range_extract_embeds_range
test_allowed_media_domains
test_ssrf_bypass_backslash_in_url
test_ssrf_bypass_backslash_disallowed_domain
_make_cached_connector
test_cache_put_and_get
test_cache_ttl_expiry
test_cache_lru_eviction
test_cache_ttl_eviction_during_write
test_put_cached_bytes_missing_dir
test_get_cached_bytes_file_deleted_before_read
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Mocking and patching / 模拟与补丁**
  - **EN:** Several scenarios replace dependencies or environment state so the tests can isolate one behavior at a time.
  - **CN:** 多个场景会替换依赖或环境状态，从而把验证范围限定在单一行为上。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `mimetypes`, `os`, `shutil`, `time`, `tempfile`
- **Third-party / 第三方依赖**: `aiohttp`, `numpy`, `pybase64`, `pytest`, `requests`, `torch`, `PIL`
- **vLLM internal / vLLM 内部依赖**: `vllm.multimodal.image`, `vllm.multimodal.inputs`, `vllm.multimodal.media`
