# storage.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/entrypoints/openai/storage.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the service entrypoint layer. It centers on `CloudStorage`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于服务入口层。它围绕 `CloudStorage` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: module setup and imports / 模块初始化与导入
```python
import asyncio
import os
from typing import Optional

from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `asyncio`, `os`, `typing`, and `sglang.multimodal_gen.runtime.utils.logging_utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `asyncio`、`os`、`typing` 和 `sglang.multimodal_gen.runtime.utils.logging_utils`。这些依赖为后续实现提供所需符号。

### Lines 7-7: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 10-10: `CloudStorage` class overview / `CloudStorage` 类概览
```python
class CloudStorage:
```
**EN:** This block defines class `CloudStorage`. It encapsulates cloud storage behavior.
**CN:** 该代码块定义了类 `CloudStorage`。 它用于封装 cloud storage 相关行为。

### Lines 11-41: `__init__` implementation / `__init__` 实现
```python
    def __init__(self):
        self.enabled = os.getenv("SGLANG_CLOUD_STORAGE_TYPE", "").lower() == "s3"
        if not self.enabled:
            return

        try:
            import boto3
        except ImportError:
            logger.error(
                "boto3 is not installed. Please install it with `pip install boto3` to use cloud storage."
            )
            self.enabled = False
            return

        self.bucket_name = os.getenv("SGLANG_S3_BUCKET_NAME")
        if not self.bucket_name:
            self.enabled = False
            return

        endpoint_url = os.getenv("SGLANG_S3_ENDPOINT_URL") or None
        region_name = os.getenv("SGLANG_S3_REGION_NAME") or None

        self.client = boto3.client(
            "s3",
            aws_access_key_id=os.getenv("SGLANG_S3_ACCESS_KEY_ID"),
            aws_secret_access_key=os.getenv("SGLANG_S3_SECRET_ACCESS_KEY"),
            endpoint_url=endpoint_url,
            region_name=region_name,
        )
        self.endpoint_url = endpoint_url
        self.region_name = region_name
```
**EN:** This block defines method `__init__` on `CloudStorage`. It initializes the instance state. Key calls include `os.getenv`, `boto3.client`, `os.getenv.lower`, and `logger.error`. The implementation branches on conditions, handles exceptional paths.
**CN:** 该代码块定义了 `CloudStorage` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `os.getenv`、`boto3.client`、`os.getenv.lower` 和 `logger.error`。 实现中包含条件分支，处理异常路径。

### Lines 43-44: `is_enabled` implementation / `is_enabled` 实现
```python
    def is_enabled(self) -> bool:
        return self.enabled
```
**EN:** This block defines method `is_enabled` on `CloudStorage`. It handles is enabled logic.
**CN:** 该代码块定义了 `CloudStorage` 的方法 `is_enabled`。 它用于处理 is enabled 相关逻辑。

### Lines 46-89: `upload_file` implementation / `upload_file` 实现
```python
    async def upload_file(self, local_path: str, destination_key: str) -> Optional[str]:
        if not self.is_enabled():
            return None

        def _sync_upload():
            """Synchronous part of the upload to run in a thread."""
            ext = os.path.splitext(local_path)[1].lower()
            content_type = {
                ".png": "image/png",
                ".jpg": "image/jpeg",
                ".jpeg": "image/jpeg",
                ".webp": "image/webp",
                ".mp4": "video/mp4",
                ".glb": "model/gltf-binary",
                ".obj": "text/plain",
            }.get(ext, "application/octet-stream")

            # Use the client created once in __init__
            self.client.upload_file(
                local_path,
                self.bucket_name,
                destination_key,
                ExtraArgs={"ContentType": content_type},
            )

        try:
            # Offload the blocking I/O call to a thread executor
            await asyncio.get_running_loop().run_in_executor(None, _sync_upload)
        except Exception as e:
            # If upload fails, log the error and return None for fallback
            logger.error(f"Upload failed for {destination_key}: {e}")
            return None

        # Simplified URL generation with a default region
        if self.endpoint_url:
            url = (
                f"{self.endpoint_url.rstrip('/')}/{self.bucket_name}/{destination_key}"
            )
        else:
            region = self.region_name or "us-east-1"
            url = f"https://{self.bucket_name}.s3.{region}.amazonaws.com/{destination_key}"

        logger.info(f"Uploaded {local_path} to {url}")
        return url
```
**EN:** This block defines method `upload_file` on `CloudStorage`. It handles upload file logic. Key calls include `logger.info`, `self.is_enabled`, `os.path.splitext.lower`, `get`, and `self.client.upload_file`. The implementation branches on conditions, handles exceptional paths. Parameters such as `local_path`, and `destination_key` drive the behavior in this section.
**CN:** 该代码块定义了 `CloudStorage` 的方法 `upload_file`。 它用于处理 upload file 相关逻辑。 关键调用包括 `logger.info`、`self.is_enabled`、`os.path.splitext.lower`、`get` 和 `self.client.upload_file`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `local_path` 和 `destination_key` 等参数驱动。

### Lines 91-105: `upload_and_cleanup` implementation / `upload_and_cleanup` 实现
```python
    async def upload_and_cleanup(self, file_path: str) -> Optional[str]:
        """Helper to upload a file and delete the local copy if successful."""
        if not self.is_enabled():
            return None

        key = os.path.basename(file_path)
        url = await self.upload_file(file_path, key)

        if url:
            try:
                # pass if removal fails
                os.remove(file_path)
            except OSError as e:
                logger.warning(f"Failed to remove temporary file {file_path}: {e}")
        return url
```
**EN:** This block defines method `upload_and_cleanup` on `CloudStorage`. Helper to upload a file and delete the local copy if successful. Key calls include `os.path.basename`, `self.is_enabled`, `self.upload_file`, `os.remove`, and `logger.warning`. The implementation branches on conditions, handles exceptional paths. Parameters such as `file_path` drive the behavior in this section.
**CN:** 该代码块定义了 `CloudStorage` 的方法 `upload_and_cleanup`。 它用于处理 upload and cleanup 相关逻辑。 关键调用包括 `os.path.basename`、`self.is_enabled`、`self.upload_file`、`os.remove` 和 `logger.warning`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `file_path` 等参数驱动。

### Lines 109-109: supporting statements / 辅助语句
```python
cloud_storage = CloudStorage()
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `cloud_storage`. The code collaborates with `CloudStorage`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `cloud_storage` 等名称。 代码会与 `CloudStorage` 协同工作。

## Key Concepts / 关键概念
- `CloudStorage`: Primary class that encapsulates cloud storage behavior. / 核心类，用于封装 cloud storage 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `os`, `typing`
- **Third-party / 第三方依赖**: `boto3`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 109
