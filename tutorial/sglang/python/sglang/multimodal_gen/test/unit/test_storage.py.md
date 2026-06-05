# test_storage.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/unit/test_storage.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates storage with focused assertions and fixtures. Key symbols include `_create_temp_file`, `test_upload_file_success`, `test_upload_and_cleanup`. / 该测试模块通过有针对性的断言与夹具，验证 storage 的实现。 关键符号包括 `_create_temp_file`, `test_upload_file_success`, `test_upload_and_cleanup`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and module setup / 导入与模块初始化
```python
"""
Test suite for S3 CloudStorage integration.

Tests verify file upload, cleanup, URL generation, and error handling.
"""

import asyncio
import importlib
import os
from types import SimpleNamespace

import pytest

import sglang.multimodal_gen.runtime.entrypoints.openai.storage as storage_mod
from sglang.multimodal_gen.runtime.entrypoints.openai.storage import CloudStorage
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 18-22: Function `_create_temp_file` / 函数 `_create_temp_file`
```python
def _create_temp_file(tmp_path, name="test.png", content=b"\x89PNG\r\n\x1a\nfake"):
    """Create a temporary test file."""
    p = tmp_path / name
    p.write_bytes(content)
    return str(p)
```
**EN:** This function drives `_create_temp_file` with inputs such as `tmp_path`, `name`, `content`. Create a temporary test file.
**CN:** 这个函数负责 `_create_temp_file`，主要处理 `tmp_path`, `name`, `content` 等输入。 文档字符串说明：Create a temporary test file.

### Lines 28-53: Function `test_upload_file_success` / 函数 `test_upload_file_success`
```python
def test_upload_file_success(tmp_path):
    """Test successful upload with correct URL generation."""
    file_path = _create_temp_file(tmp_path, "image.png")

    storage_mod.cloud_storage.enabled = True
    storage_mod.cloud_storage.bucket_name = "my-bucket"
    storage_mod.cloud_storage.endpoint_url = "https://s3.example.com"
    storage_mod.cloud_storage.region_name = None

    called = {}

    def fake_upload(local_path, bucket, key, ExtraArgs=None):
        called["local_path"] = local_path
        called["bucket"] = bucket
# ...
    assert called["local_path"] == file_path
    assert called["bucket"] == "my-bucket"
    assert called["key"] == "image.png"
    assert called["extra"]["ContentType"] == "image/png"
```
**EN:** This function drives `test_upload_file_success` with inputs such as `tmp_path`. Test successful upload with correct URL generation.
**CN:** 这个函数负责 `test_upload_file_success`，主要处理 `tmp_path` 等输入。 文档字符串说明：Test successful upload with correct URL generation.

### Lines 56-72: Function `test_upload_and_cleanup` / 函数 `test_upload_and_cleanup`
```python
def test_upload_and_cleanup(tmp_path):
    """Test that local file is deleted after successful upload."""
    file_path = _create_temp_file(tmp_path, "cleanup.png")

    storage_mod.cloud_storage.enabled = True
    storage_mod.cloud_storage.bucket_name = "my-bucket"
    storage_mod.cloud_storage.endpoint_url = "https://s3.example.com"
    storage_mod.cloud_storage.client = SimpleNamespace(
        upload_file=lambda *args, **kwargs: None
    )

    assert os.path.exists(file_path)

    url = asyncio.run(storage_mod.cloud_storage.upload_and_cleanup(file_path))

    assert url == "https://s3.example.com/my-bucket/cleanup.png"
    assert not os.path.exists(file_path)
```
**EN:** This function drives `test_upload_and_cleanup` with inputs such as `tmp_path`. Test that local file is deleted after successful upload.
**CN:** 这个函数负责 `test_upload_and_cleanup`，主要处理 `tmp_path` 等输入。 文档字符串说明：Test that local file is deleted after successful upload.

### Lines 75-91: Function `test_upload_failure_preserves_file` / 函数 `test_upload_failure_preserves_file`
```python
def test_upload_failure_preserves_file(tmp_path):
    """Test that file is preserved when upload fails."""
    file_path = _create_temp_file(tmp_path, "preserve.png")

    storage_mod.cloud_storage.enabled = True
    storage_mod.cloud_storage.bucket_name = "my-bucket"
    storage_mod.cloud_storage.endpoint_url = "https://s3.example.com"

    def fake_upload_raises(*args, **kwargs):
        raise RuntimeError("simulated failure")

    storage_mod.cloud_storage.client = SimpleNamespace(upload_file=fake_upload_raises)

    result = asyncio.run(storage_mod.cloud_storage.upload_and_cleanup(file_path))

    assert result is None
    assert os.path.exists(file_path)
```
**EN:** This function drives `test_upload_failure_preserves_file` with inputs such as `tmp_path`. Test that file is preserved when upload fails.
**CN:** 这个函数负责 `test_upload_failure_preserves_file`，主要处理 `tmp_path` 等输入。 文档字符串说明：Test that file is preserved when upload fails.

### Lines 94-107: Function `test_disabled_storage_returns_none` / 函数 `test_disabled_storage_returns_none`
```python
def test_disabled_storage_returns_none(tmp_path):
    """Test that disabled storage returns None."""
    file_path = _create_temp_file(tmp_path, "test.png")

    prev_enabled = storage_mod.cloud_storage.enabled
    storage_mod.cloud_storage.enabled = False

    try:
        result = asyncio.run(
            storage_mod.cloud_storage.upload_file(file_path, "test.png")
        )
        assert result is None
    finally:
        storage_mod.cloud_storage.enabled = prev_enabled
```
**EN:** This function drives `test_disabled_storage_returns_none` with inputs such as `tmp_path`. Test that disabled storage returns None.
**CN:** 这个函数负责 `test_disabled_storage_returns_none`，主要处理 `tmp_path` 等输入。 文档字符串说明：Test that disabled storage returns None.

### Lines 110-124: Function `test_aws_url_with_region` / 函数 `test_aws_url_with_region`
```python
def test_aws_url_with_region(tmp_path):
    """Test AWS S3 URL generation with specific region."""
    file_path = _create_temp_file(tmp_path, "aws.png")

    storage_mod.cloud_storage.enabled = True
    storage_mod.cloud_storage.bucket_name = "aws-bucket"
    storage_mod.cloud_storage.endpoint_url = None
    storage_mod.cloud_storage.region_name = "us-west-2"
    storage_mod.cloud_storage.client = SimpleNamespace(
        upload_file=lambda *args, **kwargs: None
    )

    url = asyncio.run(storage_mod.cloud_storage.upload_file(file_path, "aws.png"))

    assert url == "https://aws-bucket.s3.us-west-2.amazonaws.com/aws.png"
```
**EN:** This function drives `test_aws_url_with_region` with inputs such as `tmp_path`. Test AWS S3 URL generation with specific region.
**CN:** 这个函数负责 `test_aws_url_with_region`，主要处理 `tmp_path` 等输入。 文档字符串说明：Test AWS S3 URL generation with specific region.

### Lines 127-141: Function `test_aws_url_default_region` / 函数 `test_aws_url_default_region`
```python
def test_aws_url_default_region(tmp_path):
    """Test AWS S3 URL defaults to us-east-1 when region not specified."""
    file_path = _create_temp_file(tmp_path, "default.png")

    storage_mod.cloud_storage.enabled = True
    storage_mod.cloud_storage.bucket_name = "default-bucket"
    storage_mod.cloud_storage.endpoint_url = None
    storage_mod.cloud_storage.region_name = None
    storage_mod.cloud_storage.client = SimpleNamespace(
        upload_file=lambda *args, **kwargs: None
    )

    url = asyncio.run(storage_mod.cloud_storage.upload_file(file_path, "default.png"))

    assert url == "https://default-bucket.s3.us-east-1.amazonaws.com/default.png"
```
**EN:** This function drives `test_aws_url_default_region` with inputs such as `tmp_path`. Test AWS S3 URL defaults to us-east-1 when region not specified.
**CN:** 这个函数负责 `test_aws_url_default_region`，主要处理 `tmp_path` 等输入。 文档字符串说明：Test AWS S3 URL defaults to us-east-1 when region not specified.

### Lines 144-159: Function `test_custom_endpoint_url` / 函数 `test_custom_endpoint_url`
```python
def test_custom_endpoint_url(tmp_path):
    """Test URL generation with custom endpoint (MinIO/OSS/COS)."""
    file_path = _create_temp_file(tmp_path, "custom.png")

    storage_mod.cloud_storage.enabled = True
    storage_mod.cloud_storage.bucket_name = "custom-bucket"
    storage_mod.cloud_storage.endpoint_url = "https://minio.example.com/"
    storage_mod.cloud_storage.region_name = None
    storage_mod.cloud_storage.client = SimpleNamespace(
        upload_file=lambda *args, **kwargs: None
    )

    url = asyncio.run(storage_mod.cloud_storage.upload_file(file_path, "custom.png"))

    # Verify trailing slash is stripped
    assert url == "https://minio.example.com/custom-bucket/custom.png"
```
**EN:** This function drives `test_custom_endpoint_url` with inputs such as `tmp_path`. Test URL generation with custom endpoint (MinIO/OSS/COS).
**CN:** 这个函数负责 `test_custom_endpoint_url`，主要处理 `tmp_path` 等输入。 文档字符串说明：Test URL generation with custom endpoint (MinIO/OSS/COS).

### Lines 162-188: Function `test_content_type_detection` / 函数 `test_content_type_detection`
```python
def test_content_type_detection(tmp_path):
    """Test Content-Type header for different file extensions."""
    storage_mod.cloud_storage.enabled = True
    storage_mod.cloud_storage.bucket_name = "test-bucket"
    storage_mod.cloud_storage.endpoint_url = "https://s3.test"

    test_cases = [
        ("image.png", "image/png"),
        ("image.jpg", "image/jpeg"),
        ("image.jpeg", "image/jpeg"),
        ("image.webp", "image/webp"),
        ("video.mp4", "video/mp4"),
        ("file.bin", "application/octet-stream"),
    ]
# ...
        file_path = _create_temp_file(tmp_path, filename)
        asyncio.run(storage_mod.cloud_storage.upload_file(file_path, filename))

        assert called["content_type"] == expected_type
```
**EN:** This function drives `test_content_type_detection` with inputs such as `tmp_path`. Test Content-Type header for different file extensions.
**CN:** 这个函数负责 `test_content_type_detection`，主要处理 `tmp_path` 等输入。 文档字符串说明：Test Content-Type header for different file extensions.

### Lines 189-195: Top-level configuration / 顶层配置
```python


# requires moto and boto3
has_moto = (
    importlib.util.find_spec("moto") is not None
    and importlib.util.find_spec("boto3") is not None
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 198-232: Function `test_integration_with_moto` / 函数 `test_integration_with_moto`
```python
@pytest.mark.skipif(not has_moto, reason="moto/boto3 not installed")
def test_integration_with_moto(tmp_path):
    """Integration test using moto to mock real S3 service."""
    import boto3
    from moto import mock_aws

    os.environ["SGLANG_CLOUD_STORAGE_TYPE"] = "s3"
    os.environ["SGLANG_S3_BUCKET_NAME"] = "integration-test"
    os.environ["SGLANG_S3_REGION_NAME"] = "us-east-1"

    with mock_aws():
        s3 = boto3.client("s3", region_name="us-east-1")
        s3.create_bucket(Bucket="integration-test")

# ...
        "SGLANG_S3_BUCKET_NAME",
        "SGLANG_S3_REGION_NAME",
    ]:
        os.environ.pop(key, None)
```
**EN:** This function drives `test_integration_with_moto` with inputs such as `tmp_path`. Integration test using moto to mock real S3 service.
**CN:** 这个函数负责 `test_integration_with_moto`，主要处理 `tmp_path` 等输入。 文档字符串说明：Integration test using moto to mock real S3 service.

## Key Concepts / 关键概念
- Artifact storage management / 产物存储管理
- Automated verification / 自动化验证
- Pytest-based assertions / 基于 Pytest 的断言
- Command-line interface / 命令行接口
- OpenAI-compatible protocol / 兼容 OpenAI 的协议

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.entrypoints.openai.storage`
- **External / 外部**: `pytest`, `boto3`, `moto`
- **Stdlib / 标准库**: `asyncio`, `importlib`, `os`, `types`
