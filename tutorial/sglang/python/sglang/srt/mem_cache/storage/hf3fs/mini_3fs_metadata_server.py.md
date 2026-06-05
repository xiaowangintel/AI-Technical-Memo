# mini_3fs_metadata_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/storage/hf3fs/mini_3fs_metadata_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the mini 3 FS metadata server logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的mini 3 fs元数据服务器相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and setup / 导入与初始化
```python
import argparse
import atexit
import json
import logging
import threading
from collections import OrderedDict
from pathlib import Path
from typing import Dict, List, Optional, Tuple

import orjson
import requests
from fastapi import FastAPI, HTTPException, Request, Response
from fastapi.responses import ORJSONResponse
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

from sglang.srt.mem_cache.hicache_storage import PoolName
from sglang.srt.mem_cache.storage.hf3fs.storage_hf3fs import Hf3fsMetadataInterface
```
**EN:** Imports `argparse`, `atexit`, `json`, `logging`, `threading`, `collections` and other helpers used by the surrounding scope.
**CN:** 导入 `argparse`, `atexit`, `json`, `logging`, `threading`, `collections` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 19-20: Comment block / 注释块
```python

# --- Configuration ---
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 21-23: Control flow block / 控制流代码块
```python
logging.basicConfig(
    level=logging.INFO, format="%(asctime)s - %(levelname)s - %(message)s"
)
```
**EN:** Implements top-level control flow needed for setup or execution.
**CN:** 实现顶层初始化或执行所需的控制流。

### Lines 24-26: Comment block / 注释块
```python


# --- Data Models ---
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 27-29: RankMetadata declaration / RankMetadata 声明
```python
class RankMetadata:
    """Holds all metadata for a single rank."""

```
**EN:** Holds all metadata for a single rank. Declares the `RankMetadata` class.
**CN:** 声明 `RankMetadata` 类。

### Lines 30-34: __init__ implementation / __init__ 实现
```python
    def __init__(self, num_pages: int):
        self.lock = threading.Lock()
        self.num_pages = num_pages
        self.free_pages: List[int] = list(range(num_pages))
        self.key_to_index: OrderedDict[str, int] = OrderedDict()
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `RankMetadata`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `RankMetadata`。

### Lines 35-36: Comment block / 注释块
```python
        # Todo: Support multi files for HF3FS

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 37-40: exists_keys implementation / exists_keys 实现
```python
    def exists_keys(self, keys: List[str]) -> List[bool]:
        """Check if keys exist in metadata."""
        with self.lock:
            return [key in self.key_to_index for key in keys]
```
**EN:** Check if keys exist in metadata. Implements the exists keys routine for this scope. It belongs to `RankMetadata`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的exists keys例程。 该方法属于 `RankMetadata`。它会向调用方返回计算结果。

### Lines 42-66: reserve_and_allocate_page_indices implementation / reserve_and_allocate_page_indices 实现
```python
    def reserve_and_allocate_page_indices(
        self, keys: List[Tuple[str, str]]
    ) -> List[Tuple[bool, int]]:
        """Reserve and allocate page indices for keys."""
        with self.lock:
            results = [None] * len(keys)
            new_keys_to_process = []

            for i, (key, prefix_key) in enumerate(keys):
                if key in self.key_to_index:
                    results[i] = (True, self.key_to_index[key])
                    self.key_to_index.move_to_end(key)
# ... omitted for brevity ...

                results[i] = (False, page_index)

            return results
```
**EN:** Reserve and allocate page indices for keys. Implements the reserve AND allocate page indices routine for this scope. It belongs to `RankMetadata`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的reserve and allocate页indices例程。 该方法属于 `RankMetadata`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 68-81: confirm_write implementation / confirm_write 实现
```python
    def confirm_write(
        self,
        written_keys_to_confirm: List[Tuple[str, int]],
        pages_to_release: List[int],
    ) -> None:
        """Confirm write operations and release pages."""
        with self.lock:
            for key, page_index in written_keys_to_confirm:
                self.key_to_index[key] = page_index
                self.key_to_index.move_to_end(key)

            for page_index in pages_to_release:
                if page_index not in self.free_pages:
                    self.free_pages.append(page_index)
```
**EN:** Confirm write operations and release pages. Implements the confirm write routine for this scope. It belongs to `RankMetadata`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的confirm write例程。 该方法属于 `RankMetadata`。实现过程中会遍历输入或受管条目。

### Lines 83-93: delete_keys implementation / delete_keys 实现
```python
    def delete_keys(self, keys: List[str]) -> int:
        """Delete keys and return count of deleted keys."""
        with self.lock:
            count = 0
            for key in keys:
                if key in self.key_to_index:
                    page_index = self.key_to_index.pop(key)
                    if page_index not in self.free_pages:
                        self.free_pages.append(page_index)
                    count += 1
            return count
```
**EN:** Delete keys and return count of deleted keys. Implements the delete keys routine for this scope. It belongs to `RankMetadata`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的delete keys例程。 该方法属于 `RankMetadata`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 95-99: clear_all implementation / clear_all 实现
```python
    def clear_all(self) -> None:
        """Clear all metadata."""
        with self.lock:
            self.free_pages = list(range(self.num_pages))
            self.key_to_index.clear()
```
**EN:** Clear all metadata. Resets internal state and returns the object to a clean baseline. It belongs to `RankMetadata`.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `RankMetadata`。

### Lines 101-111: get_page_indices implementation / get_page_indices 实现
```python
    def get_page_indices(self, keys: List[str]) -> List[Optional[int]]:
        """Get page indices for keys."""
        with self.lock:
            results = []
            for key in keys:
                if key in self.key_to_index:
                    results.append(self.key_to_index[key])
                    self.key_to_index.move_to_end(key)
                else:
                    results.append(None)
            return results
```
**EN:** Get page indices for keys. Retrieves the requested data or state from the current object. It belongs to `RankMetadata`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `RankMetadata`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 114-116: GlobalMetadataState declaration / GlobalMetadataState 声明
```python
class GlobalMetadataState:
    """Manages the state for all ranks and persistence."""

```
**EN:** Manages the state for all ranks and persistence. Declares the `GlobalMetadataState` class.
**CN:** 声明 `GlobalMetadataState` 类。

### Lines 117-123: __init__ implementation / __init__ 实现
```python
    def __init__(self, persistence_path: Optional[str], save_interval: int):
        self.global_lock = threading.RLock()
        self.ranks: Dict[str, RankMetadata] = {}
        self.persistence_path = Path(persistence_path) if persistence_path else None
        self.save_interval = save_interval
        self.save_timer: Optional[threading.Timer] = None
        self.is_shutting_down = False
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `GlobalMetadataState`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `GlobalMetadataState`。

### Lines 125-152: load_from_disk implementation / load_from_disk 实现
```python
    def load_from_disk(self):
        if not self.persistence_path or not self.persistence_path.exists():
            logging.info("Persistence file not found. Starting with a clean state.")
            return

        logging.info(f"Loading state from {self.persistence_path}")
        try:
            with open(self.persistence_path, "r") as f:
                persisted_data = json.load(f)

            with self.global_lock:
                for key_str, data in persisted_data.items():
# ... omitted for brevity ...
                f"Failed to load or parse persistence file: {e}. Starting fresh.",
                exc_info=True,
            )
            self.ranks.clear()
```
**EN:** Loads state from an external or serialized representation. It belongs to `GlobalMetadataState`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `GlobalMetadataState`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 154-176: save_to_disk implementation / save_to_disk 实现
```python
    def save_to_disk(self):
        if not self.persistence_path:
            return

        logging.info("Persisting metadata to disk...")
        with self.global_lock:
            serializable_state = {}
            for key_str, rank_meta in self.ranks.items():
                with rank_meta.lock:
                    serializable_state[key_str] = {
                        "num_pages": rank_meta.num_pages,
                        "free_pages": rank_meta.free_pages,
# ... omitted for brevity ...
            temp_path.rename(self.persistence_path)
            logging.info(f"Metadata successfully persisted to {self.persistence_path}")
        except Exception as e:
            logging.error(f"Failed to save metadata to disk: {e}", exc_info=True)
```
**EN:** Persists the current state to an external destination. It belongs to `GlobalMetadataState`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 将当前状态持久化到外部位置。 该方法属于 `GlobalMetadataState`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 178-183: schedule_save implementation / schedule_save 实现
```python
    def schedule_save(self):
        if self.is_shutting_down or not self.persistence_path:
            return
        self.save_to_disk()
        self.save_timer = threading.Timer(self.save_interval, self.schedule_save)
        self.save_timer.start()
```
**EN:** Implements the schedule save routine for this scope. It belongs to `GlobalMetadataState`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的schedule save例程。 该方法属于 `GlobalMetadataState`。它会向调用方返回计算结果。

### Lines 185-191: shutdown implementation / shutdown 实现
```python
    def shutdown(self):
        logging.info("Shutting down metadata server...")
        self.is_shutting_down = True
        if self.save_timer:
            self.save_timer.cancel()
        self.save_to_disk()
        logging.info("Shutdown complete.")
```
**EN:** Implements the shutdown routine for this scope. It belongs to `GlobalMetadataState`.
**CN:** 实现当前作用域中的shutdown例程。 该方法属于 `GlobalMetadataState`。

### Lines 192-194: Comment block / 注释块
```python


# --- Global MetadataServer implementation ---
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 195-197: Hf3fsMetadataServer declaration / Hf3fsMetadataServer 声明
```python
class Hf3fsMetadataServer:
    """HF3FS Metadata Server that manages metadata for multiple ranks."""

```
**EN:** HF3FS Metadata Server that manages metadata for multiple ranks. Declares the `Hf3fsMetadataServer` class.
**CN:** 声明 `Hf3fsMetadataServer` 类。

### Lines 198-202: __init__ implementation / __init__ 实现
```python
    def __init__(self, persistence_path: Optional[str] = None, save_interval: int = 60):
        self.state = GlobalMetadataState(persistence_path, save_interval)
        self.app = FastAPI(default_response_class=ORJSONResponse)

        self._setup_routes()
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `Hf3fsMetadataServer`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `Hf3fsMetadataServer`。

### Lines 204-214: _setup_routes implementation / _setup_routes 实现
```python
    def _setup_routes(self):
        """Setup FastAPI routes."""
        self.app.post("/{rank}/initialize")(self.initialize)
        self.app.post("/{rank}/exists")(self.exists)
        self.app.post("/{rank}/reserve_and_allocate_page_indices")(
            self.reserve_and_allocate_page_indices
        )
        self.app.post("/{rank}/confirm_write")(self.confirm_write)
        self.app.post("/{rank}/delete_keys")(self.delete_keys)
        self.app.post("/{rank}/clear")(self.clear)
        self.app.post("/{rank}/get_page_indices")(self.get_page_indices)
```
**EN:** Setup FastAPI routes. Implements the setup routes routine for this scope. It belongs to `Hf3fsMetadataServer`.
**CN:** 实现当前作用域中的setup routes例程。 该方法属于 `Hf3fsMetadataServer`。

### Lines 216-218: _rank_key implementation / _rank_key 实现
```python
    def _rank_key(self, rank: int, namespace: str) -> str:
        """Generate the composite key for rank+namespace."""
        return f"{rank}:{namespace}"
```
**EN:** Generate the composite key for rank+namespace. Implements the rank KEY routine for this scope. It belongs to `Hf3fsMetadataServer`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的rank key例程。 该方法属于 `Hf3fsMetadataServer`。它会向调用方返回计算结果。

### Lines 220-228: get_rank_metadata implementation / get_rank_metadata 实现
```python
    def get_rank_metadata(self, rank: int, namespace: str = "kv") -> RankMetadata:
        """Get rank metadata with proper error handling."""
        key = self._rank_key(rank, namespace)
        if key not in self.state.ranks:
            raise HTTPException(
                status_code=404,
                detail=f"Rank {rank} namespace '{namespace}' not initialized. Please call /{rank}/initialize first.",
            )
        return self.state.ranks[key]
```
**EN:** Get rank metadata with proper error handling. Retrieves the requested data or state from the current object. It belongs to `Hf3fsMetadataServer`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `Hf3fsMetadataServer`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 230-233: _read_json implementation / _read_json 实现
```python
    async def _read_json(self, request: Request) -> dict:
        """Parse request JSON using orjson if available."""
        body = await request.body()
        return orjson.loads(body)
```
**EN:** Parse request JSON using orjson if available. Implements the read json routine for this scope. It belongs to `Hf3fsMetadataServer`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的read json例程。 该方法属于 `Hf3fsMetadataServer`。它会向调用方返回计算结果。

### Lines 235-237: _json_response implementation / _json_response 实现
```python
    def _json_response(self, content: dict):
        """Return ORJSONResponse when available to bypass jsonable_encoder."""
        return ORJSONResponse(content)
```
**EN:** Return ORJSONResponse when available to bypass jsonable_encoder. Implements the json response routine for this scope. It belongs to `Hf3fsMetadataServer`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的json response例程。 该方法属于 `Hf3fsMetadataServer`。它会向调用方返回计算结果。

### Lines 239-259: initialize implementation / initialize 实现
```python
    async def initialize(self, rank: int, request: Request):
        """Initialize a rank with specified number of pages."""
        data = await self._read_json(request)
        num_pages = data["num_pages"]
        namespace = data.get("namespace", "kv")
        key = self._rank_key(rank, namespace)
        with self.state.global_lock:
            if key in self.state.ranks:
                logging.info(
                    f"Rank {rank} namespace '{namespace}' already exists. Initialization request ignored."
                )
                if self.state.ranks[key].num_pages != num_pages:
# ... omitted for brevity ...
                    f"Initializing new Rank {rank} namespace '{namespace}' with {num_pages} pages."
                )
                self.state.ranks[key] = RankMetadata(num_pages)
        return Response(status_code=204)
```
**EN:** Initialize a rank with specified number of pages. Implements the initialize routine for this scope. It belongs to `Hf3fsMetadataServer`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的initialize例程。 该方法属于 `Hf3fsMetadataServer`。它会向调用方返回计算结果。

### Lines 261-268: exists implementation / exists 实现
```python
    async def exists(self, rank: int, request: Request):
        """Check if keys exist in metadata."""
        data = await self._read_json(request)
        keys = data["keys"]
        namespace = data.get("namespace", "kv")
        metadata = self.get_rank_metadata(rank, namespace)
        results = metadata.exists_keys(keys)
        return self._json_response({"exists": results})
```
**EN:** Check if keys exist in metadata. Implements the exists routine for this scope. It belongs to `Hf3fsMetadataServer`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的exists例程。 该方法属于 `Hf3fsMetadataServer`。它会向调用方返回计算结果。

### Lines 270-277: reserve_and_allocate_page_indices implementation / reserve_and_allocate_page_indices 实现
```python
    async def reserve_and_allocate_page_indices(self, rank: int, request: Request):
        """Reserve and allocate page indices for keys."""
        data = await self._read_json(request)
        namespace = data.get("namespace", "kv")
        metadata = self.get_rank_metadata(rank, namespace)
        keys = data["keys"]
        results = metadata.reserve_and_allocate_page_indices(keys)
        return self._json_response({"indices": results})
```
**EN:** Reserve and allocate page indices for keys. Implements the reserve AND allocate page indices routine for this scope. It belongs to `Hf3fsMetadataServer`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的reserve and allocate页indices例程。 该方法属于 `Hf3fsMetadataServer`。它会向调用方返回计算结果。

### Lines 279-289: confirm_write implementation / confirm_write 实现
```python
    async def confirm_write(self, rank: int, request: Request):
        """Confirm write operations and release pages."""
        data = await self._read_json(request)
        namespace = data.get("namespace", "kv")
        metadata = self.get_rank_metadata(rank, namespace)
        success_written_keys = data.get("written_keys_to_confirm", [])
        released_pages = data.get("pages_to_release", [])

        metadata.confirm_write(success_written_keys, released_pages)

        return Response(status_code=204)
```
**EN:** Confirm write operations and release pages. Implements the confirm write routine for this scope. It belongs to `Hf3fsMetadataServer`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的confirm write例程。 该方法属于 `Hf3fsMetadataServer`。它会向调用方返回计算结果。

### Lines 291-297: delete_keys implementation / delete_keys 实现
```python
    async def delete_keys(self, rank: int, request: Request):
        """Delete keys from metadata."""
        data = await self._read_json(request)
        namespace = data.get("namespace", "kv")
        metadata = self.get_rank_metadata(rank, namespace)
        count = metadata.delete_keys(data["keys"])
        return Response(status_code=204)
```
**EN:** Delete keys from metadata. Implements the delete keys routine for this scope. It belongs to `Hf3fsMetadataServer`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的delete keys例程。 该方法属于 `Hf3fsMetadataServer`。它会向调用方返回计算结果。

### Lines 299-305: clear implementation / clear 实现
```python
    async def clear(self, rank: int, request: Request):
        """Clear all metadata for a rank."""
        data = await self._read_json(request)
        namespace = data.get("namespace", "kv")
        metadata = self.get_rank_metadata(rank, namespace)
        metadata.clear_all()
        return Response(status_code=204)
```
**EN:** Clear all metadata for a rank. Resets internal state and returns the object to a clean baseline. It belongs to `Hf3fsMetadataServer`. It returns a computed result to its caller.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `Hf3fsMetadataServer`。它会向调用方返回计算结果。

### Lines 307-314: get_page_indices implementation / get_page_indices 实现
```python
    async def get_page_indices(self, rank: int, request: Request):
        """Get page indices for keys."""
        data = await self._read_json(request)
        namespace = data.get("namespace", "kv")
        metadata = self.get_rank_metadata(rank, namespace)
        keys = data["keys"]
        results = metadata.get_page_indices(keys)
        return self._json_response({"indices": results})
```
**EN:** Get page indices for keys. Retrieves the requested data or state from the current object. It belongs to `Hf3fsMetadataServer`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `Hf3fsMetadataServer`。它会向调用方返回计算结果。

### Lines 316-333: run implementation / run 实现
```python
    def run(self, host: str = "0.0.0.0", port: int = 18000):
        """Run the metadata server."""
        self.state.load_from_disk()
        if self.state.persistence_path:
            self.state.schedule_save()
            atexit.register(self.state.shutdown)

        import uvicorn

        logging.info(f"Starting metadata server on http://{host}:{port}")
        if self.state.persistence_path:
            logging.info(
                f"Persistence is ENABLED. Saving to '{self.state.persistence_path}' every {self.state.save_interval} seconds."
            )
        else:
            logging.info("Persistence is DISABLED.")

        uvicorn.run(self.app, host=host, port=port)
```
**EN:** Run the metadata server. Implements the RUN routine for this scope. It belongs to `Hf3fsMetadataServer`.
**CN:** 实现当前作用域中的run例程。 该方法属于 `Hf3fsMetadataServer`。

### Lines 334-336: Comment block / 注释块
```python


# --- Client implementation ---
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 337-339: Hf3fsGlobalMetadataClient declaration / Hf3fsGlobalMetadataClient 声明
```python
class Hf3fsGlobalMetadataClient(Hf3fsMetadataInterface):
    """Global http metadata client for HF3FS."""

```
**EN:** Global http metadata client for HF3FS. Declares the `Hf3fsGlobalMetadataClient` class and connects it to `Hf3fsMetadataInterface`.
**CN:** 声明 `Hf3fsGlobalMetadataClient` 类，并将其关联到 `Hf3fsMetadataInterface`。

### Lines 340-353: __init__ implementation / __init__ 实现
```python
    def __init__(self, base_url: str, max_retries: int = 3):
        self.base_url = base_url.rstrip("/")
        self._session = requests.Session()

        retry_strategy = Retry(
            total=max_retries,
            backoff_factor=0.3,
            status_forcelist=[500, 502, 503, 504],
            allowed_methods=["GET", "POST"],
        )
        adapter = HTTPAdapter(
            max_retries=retry_strategy, pool_connections=256, pool_maxsize=256
        )
        self._session.mount("http://", adapter)
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `Hf3fsGlobalMetadataClient`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `Hf3fsGlobalMetadataClient`。

### Lines 355-368: _post implementation / _post 实现
```python
    def _post(self, endpoint: str, json_data: dict) -> dict:
        try:
            url = f"{self.base_url}/{endpoint}"
            headers = {"Content-Type": "application/json"}
            payload = orjson.dumps(json_data)  # type: ignore[union-attr]
            response = self._session.post(url, data=payload, headers=headers)
            response.raise_for_status()

            if response.status_code == 204 or not response.content:
                return {}
            return orjson.loads(response.content)  # type: ignore[union-attr]
        except requests.exceptions.RequestException as e:
            logging.error(f"Failed to POST to {endpoint} after retries: {e}")
            raise RuntimeError(f"Failed to connect to metadata server: {e}") from e
```
**EN:** Implements the post routine for this scope. It belongs to `Hf3fsGlobalMetadataClient`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 实现当前作用域中的post例程。 该方法属于 `Hf3fsGlobalMetadataClient`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 370-375: initialize implementation / initialize 实现
```python
    def initialize(
        self, rank: int, num_pages: int, namespace: PoolName = PoolName.KV
    ) -> None:
        self._post(
            f"{rank}/initialize", {"num_pages": num_pages, "namespace": str(namespace)}
        )
```
**EN:** Implements the initialize routine for this scope. It belongs to `Hf3fsGlobalMetadataClient`.
**CN:** 实现当前作用域中的initialize例程。 该方法属于 `Hf3fsGlobalMetadataClient`。

### Lines 377-384: reserve_and_allocate_page_indices implementation / reserve_and_allocate_page_indices 实现
```python
    def reserve_and_allocate_page_indices(
        self, rank: int, keys: List[Tuple[str, str]], namespace: PoolName = PoolName.KV
    ) -> List[Tuple[bool, int]]:
        response = self._post(
            f"{rank}/reserve_and_allocate_page_indices",
            {"keys": keys, "namespace": str(namespace)},
        )
        return [tuple(item) for item in response.get("indices")]
```
**EN:** Implements the reserve AND allocate page indices routine for this scope. It belongs to `Hf3fsGlobalMetadataClient`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的reserve and allocate页indices例程。 该方法属于 `Hf3fsGlobalMetadataClient`。它会向调用方返回计算结果。

### Lines 386-400: confirm_write implementation / confirm_write 实现
```python
    def confirm_write(
        self,
        rank: int,
        written_keys_to_confirm: List[Tuple[str, int]],
        pages_to_release: List[int],
        namespace: PoolName = PoolName.KV,
    ) -> None:
        self._post(
            f"{rank}/confirm_write",
            {
                "written_keys_to_confirm": written_keys_to_confirm,
                "pages_to_release": pages_to_release,
                "namespace": str(namespace),
            },
        )
```
**EN:** Implements the confirm write routine for this scope. It belongs to `Hf3fsGlobalMetadataClient`.
**CN:** 实现当前作用域中的confirm write例程。 该方法属于 `Hf3fsGlobalMetadataClient`。

### Lines 402-405: delete_keys implementation / delete_keys 实现
```python
    def delete_keys(
        self, rank: int, keys: List[str], namespace: PoolName = PoolName.KV
    ) -> None:
        self._post(f"{rank}/delete_keys", {"keys": keys, "namespace": str(namespace)})
```
**EN:** Implements the delete keys routine for this scope. It belongs to `Hf3fsGlobalMetadataClient`.
**CN:** 实现当前作用域中的delete keys例程。 该方法属于 `Hf3fsGlobalMetadataClient`。

### Lines 407-413: exists implementation / exists 实现
```python
    def exists(
        self, rank: int, keys: List[str], namespace: PoolName = PoolName.KV
    ) -> List[bool]:
        response = self._post(
            f"{rank}/exists", {"keys": keys, "namespace": str(namespace)}
        )
        return response.get("exists", [])
```
**EN:** Implements the exists routine for this scope. It belongs to `Hf3fsGlobalMetadataClient`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的exists例程。 该方法属于 `Hf3fsGlobalMetadataClient`。它会向调用方返回计算结果。

### Lines 415-416: clear implementation / clear 实现
```python
    def clear(self, rank: int, namespace: PoolName = PoolName.KV) -> None:
        self._post(f"{rank}/clear", {"namespace": str(namespace)})
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `Hf3fsGlobalMetadataClient`.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `Hf3fsGlobalMetadataClient`。

### Lines 418-424: get_page_indices implementation / get_page_indices 实现
```python
    def get_page_indices(
        self, rank: int, keys: List[str], namespace: PoolName = PoolName.KV
    ) -> List[Optional[int]]:
        response = self._post(
            f"{rank}/get_page_indices", {"keys": keys, "namespace": str(namespace)}
        )
        return response.get("indices")
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `Hf3fsGlobalMetadataClient`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `Hf3fsGlobalMetadataClient`。它会向调用方返回计算结果。

### Lines 427-429: Hf3fsLocalMetadataClient declaration / Hf3fsLocalMetadataClient 声明
```python
class Hf3fsLocalMetadataClient(Hf3fsMetadataInterface):
    """Local metadata client that directly operates on RankMetadata in memory without metadata server."""

```
**EN:** Local metadata client that directly operates on RankMetadata in memory without metadata server. Declares the `Hf3fsLocalMetadataClient` class and connects it to `Hf3fsMetadataInterface`.
**CN:** 声明 `Hf3fsLocalMetadataClient` 类，并将其关联到 `Hf3fsMetadataInterface`。

### Lines 430-431: __init__ implementation / __init__ 实现
```python
    def __init__(self):
        self._metadata: Dict[str, RankMetadata] = {}  # key: "rank:namespace"
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `Hf3fsLocalMetadataClient`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `Hf3fsLocalMetadataClient`。

### Lines 433-434: _ns_key implementation / _ns_key 实现
```python
    def _ns_key(self, rank: int, namespace: PoolName) -> str:
        return f"{rank}:{namespace}"
```
**EN:** Implements the NS KEY routine for this scope. It belongs to `Hf3fsLocalMetadataClient`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的ns key例程。 该方法属于 `Hf3fsLocalMetadataClient`。它会向调用方返回计算结果。

### Lines 436-442: _get_metadata implementation / _get_metadata 实现
```python
    def _get_metadata(self, rank: int, namespace) -> RankMetadata:
        key = self._ns_key(rank, namespace)
        if key not in self._metadata:
            raise RuntimeError(
                f"Namespace '{namespace}' for rank {rank} not initialized"
            )
        return self._metadata[key]
```
**EN:** Implements the get metadata routine for this scope. It belongs to `Hf3fsLocalMetadataClient`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get元数据例程。 该方法属于 `Hf3fsLocalMetadataClient`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 444-449: initialize implementation / initialize 实现
```python
    def initialize(
        self, rank: int, num_pages: int, namespace: PoolName = PoolName.KV
    ) -> None:
        key = self._ns_key(rank, namespace)
        if key not in self._metadata:
            self._metadata[key] = RankMetadata(num_pages)
```
**EN:** Implements the initialize routine for this scope. It belongs to `Hf3fsLocalMetadataClient`.
**CN:** 实现当前作用域中的initialize例程。 该方法属于 `Hf3fsLocalMetadataClient`。

### Lines 451-457: reserve_and_allocate_page_indices implementation / reserve_and_allocate_page_indices 实现
```python
    def reserve_and_allocate_page_indices(
        self, rank: int, keys: List[Tuple[str, str]], namespace: PoolName = PoolName.KV
    ) -> List[Tuple[bool, int]]:
        """Reserve and allocate page indices for keys."""
        return self._get_metadata(rank, namespace).reserve_and_allocate_page_indices(
            keys
        )
```
**EN:** Reserve and allocate page indices for keys. Implements the reserve AND allocate page indices routine for this scope. It belongs to `Hf3fsLocalMetadataClient`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的reserve and allocate页indices例程。 该方法属于 `Hf3fsLocalMetadataClient`。它会向调用方返回计算结果。

### Lines 459-469: confirm_write implementation / confirm_write 实现
```python
    def confirm_write(
        self,
        rank: int,
        written_keys_to_confirm: List[Tuple[str, int]],
        pages_to_release: List[int],
        namespace: PoolName = PoolName.KV,
    ) -> None:
        """Confirm write operations."""
        self._get_metadata(rank, namespace).confirm_write(
            written_keys_to_confirm, pages_to_release
        )
```
**EN:** Confirm write operations. Implements the confirm write routine for this scope. It belongs to `Hf3fsLocalMetadataClient`.
**CN:** 实现当前作用域中的confirm write例程。 该方法属于 `Hf3fsLocalMetadataClient`。

### Lines 471-475: delete_keys implementation / delete_keys 实现
```python
    def delete_keys(
        self, rank: int, keys: List[str], namespace: PoolName = PoolName.KV
    ) -> None:
        """Delete keys."""
        self._get_metadata(rank, namespace).delete_keys(keys)
```
**EN:** Delete keys. Implements the delete keys routine for this scope. It belongs to `Hf3fsLocalMetadataClient`.
**CN:** 实现当前作用域中的delete keys例程。 该方法属于 `Hf3fsLocalMetadataClient`。

### Lines 477-481: exists implementation / exists 实现
```python
    def exists(
        self, rank: int, keys: List[str], namespace: PoolName = PoolName.KV
    ) -> List[bool]:
        """Check if keys exist."""
        return self._get_metadata(rank, namespace).exists_keys(keys)
```
**EN:** Check if keys exist. Implements the exists routine for this scope. It belongs to `Hf3fsLocalMetadataClient`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的exists例程。 该方法属于 `Hf3fsLocalMetadataClient`。它会向调用方返回计算结果。

### Lines 483-485: clear implementation / clear 实现
```python
    def clear(self, rank: int, namespace: PoolName = PoolName.KV) -> None:
        """Clear all metadata for rank."""
        self._get_metadata(rank, namespace).clear_all()
```
**EN:** Clear all metadata for rank. Resets internal state and returns the object to a clean baseline. It belongs to `Hf3fsLocalMetadataClient`.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `Hf3fsLocalMetadataClient`。

### Lines 487-491: get_page_indices implementation / get_page_indices 实现
```python
    def get_page_indices(
        self, rank: int, keys: List[str], namespace: PoolName = PoolName.KV
    ) -> List[Optional[int]]:
        """Get page indices for keys."""
        return self._get_metadata(rank, namespace).get_page_indices(keys)
```
**EN:** Get page indices for keys. Retrieves the requested data or state from the current object. It belongs to `Hf3fsLocalMetadataClient`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `Hf3fsLocalMetadataClient`。它会向调用方返回计算结果。

### Lines 494-506: run_metadata_server implementation / run_metadata_server 实现
```python
def run_metadata_server(
    host: str = "0.0.0.0",
    port: int = 18000,
    persistence_path: Optional[str] = None,
    save_interval: int = 60,
):
    """Run the HF3FS metadata server."""
    global server
    server = Hf3fsMetadataServer(
        persistence_path=persistence_path, save_interval=save_interval
    )

    server.run(host=host, port=port)
```
**EN:** Run the HF3FS metadata server. Implements the RUN metadata server routine for this scope.
**CN:** 实现当前作用域中的run元数据服务器例程。

### Lines 507-509: Comment block / 注释块
```python


# --- Main Execution ---
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 510-532: Script entry point / 脚本入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="HF3FS Metadata Server")
    parser.add_argument(
        "--host", type=str, default="0.0.0.0", help="Host to bind the server to."
    )
    parser.add_argument(
        "--port", type=int, default=18000, help="Port to run the server on."
    )
    parser.add_argument(
        "--persistence-path",
        type=str,
        default=None,
# ... omitted for brevity ...
    )
    args = parser.parse_args()

    run_metadata_server(args.host, args.port, args.persistence_path, args.save_interval)
```
**EN:** Provides the executable entry point for running this module directly.
**CN:** 为直接运行该模块提供可执行入口。

## Key Concepts / 关键概念
- **`RankMetadata`**: Defines the `RankMetadata` type and its core responsibilities. / 定义 `RankMetadata` 类型及其核心职责。
- **`GlobalMetadataState`**: Defines the `GlobalMetadataState` type and its core responsibilities. / 定义 `GlobalMetadataState` 类型及其核心职责。
- **`Hf3fsMetadataServer`**: Defines the `Hf3fsMetadataServer` type and its core responsibilities. / 定义 `Hf3fsMetadataServer` 类型及其核心职责。
- **`Hf3fsGlobalMetadataClient`**: Defines the `Hf3fsGlobalMetadataClient` type and its core responsibilities. / 定义 `Hf3fsGlobalMetadataClient` 类型及其核心职责。
- **`Hf3fsLocalMetadataClient`**: Defines the `Hf3fsLocalMetadataClient` type and its core responsibilities. / 定义 `Hf3fsLocalMetadataClient` 类型及其核心职责。
- **`run_metadata_server`**: Provides the `run_metadata_server` entry point for module-level behavior. / 提供模块级行为的 `run_metadata_server` 入口。

## Dependencies / 依赖关系
- **External / 外部**: `argparse`, `atexit`, `json`, `logging`, `threading`, `collections`, `pathlib`, `typing`, `orjson`, `requests`, `fastapi`, `fastapi.responses`
- **Internal / 内部**: `sglang.srt.mem_cache.hicache_storage`, `sglang.srt.mem_cache.storage.hf3fs.storage_hf3fs`
