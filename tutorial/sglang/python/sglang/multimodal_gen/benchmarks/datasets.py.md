# datasets.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/benchmarks/datasets.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the multimodal generation module. It centers on `RequestFuncInput`, `RequestFuncOutput`, and `BaseDataset`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于多模态生成模块。它围绕 `RequestFuncInput`、`RequestFuncOutput` 和 `BaseDataset` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: module setup and imports / 模块初始化与导入
```python
import glob
import json
import os
import random
import re
import subprocess
import uuid
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from typing import Any, Dict, List, Optional

import requests
from PIL import Image

from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `glob`, `json`, `os`, `random`, `re`, and `subprocess`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `glob`、`json`、`os`、`random`、`re` 和 `subprocess`。这些依赖为后续实现提供所需符号。

### Lines 17-17: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 21-21: `RequestFuncInput` class overview / `RequestFuncInput` 类概览
```python
class RequestFuncInput:
```
**EN:** This block defines class `RequestFuncInput`. It encapsulates request func input behavior.
**CN:** 该代码块定义了类 `RequestFuncInput`。 它用于封装 request func input 相关行为。

### Lines 22-34: supporting statements / 辅助语句
```python
    prompt: str
    api_url: str = ""
    model: str = ""
    num_outputs_per_prompt: int = 1
    width: Optional[int] = None
    height: Optional[int] = None
    num_frames: Optional[int] = None
    fps: Optional[int] = None
    extra_body: Dict[str, Any] = field(default_factory=dict)
    image_paths: Optional[List[str]] = None
    request_id: str = field(default_factory=lambda: str(uuid.uuid4()))
    slo_ms: Optional[float] = None
    num_inference_steps: Optional[int] = None
```
**EN:** This block gathers supporting statements inside `RequestFuncInput`. It updates names such as `prompt`, `api_url`, `model`, `num_outputs_per_prompt`, `width`, and `height`. The code collaborates with `field`, `str`, and `uuid.uuid4`.
**CN:** 该代码块汇集了位于 `RequestFuncInput` 内部的辅助语句。 它会更新 `prompt`、`api_url`、`model`、`num_outputs_per_prompt`、`width` 和 `height` 等名称。 代码会与 `field`、`str` 和 `uuid.uuid4` 协同工作。

### Lines 38-38: `RequestFuncOutput` class overview / `RequestFuncOutput` 类概览
```python
class RequestFuncOutput:
```
**EN:** This block defines class `RequestFuncOutput`. It encapsulates request func output behavior.
**CN:** 该代码块定义了类 `RequestFuncOutput`。 它用于封装 request func output 相关行为。

### Lines 39-46: supporting statements / 辅助语句
```python
    success: bool = False
    latency: float = 0.0
    error: str = ""
    start_time: float = 0.0
    response_body: Dict[str, Any] = field(default_factory=dict)
    peak_memory_mb: float = 0.0
    slo_achieved: Optional[bool] = None
    output_count: int = 0
```
**EN:** This block gathers supporting statements inside `RequestFuncOutput`. It updates names such as `success`, `latency`, `error`, `start_time`, `response_body`, and `peak_memory_mb`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `RequestFuncOutput` 内部的辅助语句。 它会更新 `success`、`latency`、`error`、`start_time`、`response_body` 和 `peak_memory_mb` 等名称。 代码会与 `field` 协同工作。

### Lines 49-50: `is_dir_not_empty` implementation / `is_dir_not_empty` 实现
```python
def is_dir_not_empty(path: str) -> bool:
    return os.path.isdir(path) and bool(os.listdir(path))
```
**EN:** This block defines function `is_dir_not_empty`. It handles is dir not empty logic. Key calls include `os.path.isdir`, `bool`, and `os.listdir`. Parameters such as `path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `is_dir_not_empty`。 它用于处理 is dir not empty 相关逻辑。 关键调用包括 `os.path.isdir`、`bool` 和 `os.listdir`。 本段逻辑主要由 `path` 等参数驱动。

### Lines 53-53: `BaseDataset` class overview / `BaseDataset` 类概览
```python
class BaseDataset(ABC):
```
**EN:** This block defines class `BaseDataset`. It encapsulates base dataset behavior. It inherits from `ABC`.
**CN:** 该代码块定义了类 `BaseDataset`。 它用于封装 base dataset 相关行为。 它继承自 `ABC`。

### Lines 54-58: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, args, api_url: str = "", model: str = ""):
        self.args = args
        self.api_url = api_url
        self.model = model
        self.items: List[Dict[str, Any]] = []
```
**EN:** This block defines method `__init__` on `BaseDataset`. It initializes the instance state. Parameters such as `args`, `api_url`, and `model` drive the behavior in this section.
**CN:** 该代码块定义了 `BaseDataset` 的方法 `__init__`。 它用于初始化实例状态。 本段逻辑主要由 `args`、`api_url` 和 `model` 等参数驱动。

### Lines 60-62: `__len__` implementation / `__len__` 实现
```python
    @abstractmethod
    def __len__(self) -> int:
        pass
```
**EN:** This block defines method `__len__` on `BaseDataset`. It handles len logic.
**CN:** 该代码块定义了 `BaseDataset` 的方法 `__len__`。 它用于处理 len 相关逻辑。

### Lines 64-66: `__getitem__` implementation / `__getitem__` 实现
```python
    @abstractmethod
    def __getitem__(self, idx: int) -> RequestFuncInput:
        pass
```
**EN:** This block defines method `__getitem__` on `BaseDataset`. It handles getitem logic. Parameters such as `idx` drive the behavior in this section.
**CN:** 该代码块定义了 `BaseDataset` 的方法 `__getitem__`。 它用于处理 getitem 相关逻辑。 本段逻辑主要由 `idx` 等参数驱动。

### Lines 68-69: `get_requests` implementation / `get_requests` 实现
```python
    def get_requests(self) -> List[RequestFuncInput]:
        return [self[i] for i in range(len(self))]
```
**EN:** This block defines method `get_requests` on `BaseDataset`. It retrieves requests. Key calls include `range`, and `len`.
**CN:** 该代码块定义了 `BaseDataset` 的方法 `get_requests`。 它用于获取requests。 关键调用包括 `range` 和 `len`。

### Lines 72-77: `VBenchDataset` class overview / `VBenchDataset` 类概览
```python
class VBenchDataset(BaseDataset):
    """
    Dataset loader for VBench prompts.
    Supports t2v, i2v.
    """
```
**EN:** This block defines class `VBenchDataset`. Dataset loader for VBench prompts. Supports t2v, i2v. It inherits from `BaseDataset`.
**CN:** 该代码块定义了类 `VBenchDataset`。 它用于封装 vbench dataset 相关行为。 它继承自 `BaseDataset`。

### Lines 78-79: supporting statements / 辅助语句
```python
    T2V_PROMPT_URL = "https://raw.githubusercontent.com/Vchitect/VBench/master/prompts/prompts_per_dimension/subject_consistency.txt"
    I2V_DOWNLOAD_SCRIPT_URL = "https://raw.githubusercontent.com/Vchitect/VBench/master/vbench2_beta_i2v/download_data.sh"
```
**EN:** This block gathers supporting statements inside `VBenchDataset`. It updates names such as `T2V_PROMPT_URL`, and `I2V_DOWNLOAD_SCRIPT_URL`.
**CN:** 该代码块汇集了位于 `VBenchDataset` 内部的辅助语句。 它会更新 `T2V_PROMPT_URL` 和 `I2V_DOWNLOAD_SCRIPT_URL` 等名称。

### Lines 81-84: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, args, api_url: str = "", model: str = ""):
        super().__init__(args, api_url, model)
        self.cache_dir = os.path.join(os.path.expanduser("~"), ".cache", "sglang")
        self.items = self._load_data()
```
**EN:** This block defines method `__init__` on `VBenchDataset`. It initializes the instance state. Key calls include `super.__init__`, `os.path.join`, `self._load_data`, `os.path.expanduser`, and `super`. Parameters such as `args`, `api_url`, and `model` drive the behavior in this section.
**CN:** 该代码块定义了 `VBenchDataset` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`os.path.join`、`self._load_data`、`os.path.expanduser` 和 `super`。 本段逻辑主要由 `args`、`api_url` 和 `model` 等参数驱动。

### Lines 86-94: `_load_data` implementation / `_load_data` 实现
```python
    def _load_data(self) -> List[Dict[str, Any]]:
        if self.args.task_name in ("text-to-video", "text-to-image", "video-to-video"):
            return self._load_t2v_prompts()
        elif self.args.task_name in ("image-to-video", "image-to-image"):
            return self._load_i2v_data()
        else:
            raise ValueError(
                f"Illegal task name is found in VBenchDataset {self.args.task_name}"
            )
```
**EN:** This block defines method `_load_data` on `VBenchDataset`. It loads data. Key calls include `self._load_t2v_prompts`, `self._load_i2v_data`, and `ValueError`. The implementation branches on conditions.
**CN:** 该代码块定义了 `VBenchDataset` 的方法 `_load_data`。 它用于加载data。 关键调用包括 `self._load_t2v_prompts`、`self._load_i2v_data` 和 `ValueError`。 实现中包含条件分支。

### Lines 96-102: `_download_file` implementation / `_download_file` 实现
```python
    def _download_file(self, url: str, dest_path: str) -> None:
        """Download a file from URL to destination path."""
        os.makedirs(os.path.dirname(dest_path), exist_ok=True)
        resp = requests.get(url)
        resp.raise_for_status()
        with open(dest_path, "w") as f:
            f.write(resp.text)
```
**EN:** This block defines method `_download_file` on `VBenchDataset`. Download a file from URL to destination path. Key calls include `os.makedirs`, `requests.get`, `resp.raise_for_status`, `os.path.dirname`, and `open`. The implementation uses context-managed resources. Parameters such as `url`, and `dest_path` drive the behavior in this section.
**CN:** 该代码块定义了 `VBenchDataset` 的方法 `_download_file`。 它用于处理 download file 相关逻辑。 关键调用包括 `os.makedirs`、`requests.get`、`resp.raise_for_status`、`os.path.dirname` 和 `open`。 实现中使用上下文管理资源。 本段逻辑主要由 `url` 和 `dest_path` 等参数驱动。

### Lines 104-124: `_load_t2v_prompts` implementation / `_load_t2v_prompts` 实现
```python
    def _load_t2v_prompts(self) -> List[Dict[str, Any]]:
        path = self.args.dataset_path

        if not path:
            path = os.path.join(self.cache_dir, "vbench_subject_consistency.txt")
            if not os.path.exists(path):
                logger.info(f"Downloading VBench T2V prompts to {path}...")
                try:
                    self._download_file(self.T2V_PROMPT_URL, path)
                except Exception as e:
                    logger.info(f"Failed to download VBench prompts: {e}")
                    return [{"prompt": "A cat sitting on a bench"}] * 50

        prompts = []
        with open(path, "r") as f:
            for line in f:
                line = line.strip()
                if line:
                    prompts.append({"prompt": line})

        return self._resize_data(prompts)
```
**EN:** This block defines method `_load_t2v_prompts` on `VBenchDataset`. It loads t2v prompts. Key calls include `self._resize_data`, `os.path.join`, `open`, `os.path.exists`, and `logger.info`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths, uses context-managed resources.
**CN:** 该代码块定义了 `VBenchDataset` 的方法 `_load_t2v_prompts`。 它用于加载t2v prompts。 关键调用包括 `self._resize_data`、`os.path.join`、`open`、`os.path.exists` 和 `logger.info`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径，使用上下文管理资源。

### Lines 126-177: `_auto_download_i2v_dataset` implementation / `_auto_download_i2v_dataset` 实现
```python
    def _auto_download_i2v_dataset(self) -> Optional[str]:
        """Auto-download VBench I2V dataset and return the dataset directory."""
        vbench_i2v_dir = os.path.join(self.cache_dir, "vbench_i2v", "vbench2_beta_i2v")
        info_json_path = os.path.join(vbench_i2v_dir, "data", "i2v-bench-info.json")
        crop_dir = os.path.join(vbench_i2v_dir, "data", "crop")
        origin_dir = os.path.join(vbench_i2v_dir, "data", "origin")

        if (
            os.path.exists(info_json_path)
            and is_dir_not_empty(crop_dir)
            and is_dir_not_empty(origin_dir)
        ):
            return vbench_i2v_dir

        logger.info(f"Downloading VBench I2V dataset to {vbench_i2v_dir}...")
        try:
            cache_root = os.path.join(self.cache_dir, "vbench_i2v")
            script_path = os.path.join(cache_root, "download_data.sh")

            self._download_file(self.I2V_DOWNLOAD_SCRIPT_URL, script_path)
            os.chmod(script_path, 0o755)

            logger.info("Executing download_data.sh (this may take a while)...")

            result = subprocess.run(
                ["bash", script_path],
                cwd=cache_root,
                capture_output=True,
                text=True,
            )
            if result.returncode != 0:
                raise RuntimeError(f"Download script failed: {result.stderr}")
            missing_packages = re.findall(r"(\S+): command not found", result.stderr)
            if missing_packages:
                missing_packages = list(set(missing_packages))
                package_list = ", ".join(f"'{cmd}'" for cmd in missing_packages)
                raise RuntimeError(
                    f"Download script failed because the following commands are not installed: {package_list}.\n"
                    "Please install them (e.g., on Ubuntu: `sudo apt install ...`) and try again."
                )
            logger.info(
                f"Successfully downloaded VBench I2V dataset to {vbench_i2v_dir}"
            )
        except Exception as e:
            logger.info(f"Failed to download VBench I2V dataset: {e}")
            logger.info("Please manually download following instructions at:")
            logger.info(
                "https://github.com/Vchitect/VBench/tree/master/vbench2_beta_i2v#22-download"
            )
            return None

        return vbench_i2v_dir if os.path.exists(info_json_path) else None
```
**EN:** This block defines method `_auto_download_i2v_dataset` on `VBenchDataset`. Auto-download VBench I2V dataset and return the dataset directory. Key calls include `os.path.join`, `logger.info`, `os.path.exists`, `is_dir_not_empty`, and `self._download_file`. The implementation branches on conditions, handles exceptional paths.
**CN:** 该代码块定义了 `VBenchDataset` 的方法 `_auto_download_i2v_dataset`。 它用于处理 auto download i2v dataset 相关逻辑。 关键调用包括 `os.path.join`、`logger.info`、`os.path.exists`、`is_dir_not_empty` 和 `self._download_file`。 实现中包含条件分支，处理异常路径。

### Lines 179-198: `_load_from_i2v_json` implementation / `_load_from_i2v_json` 实现
```python
    def _load_from_i2v_json(self, json_path: str) -> List[Dict[str, Any]]:
        """Load I2V data from i2v-bench-info.json format."""
        with open(json_path, "r") as f:
            items = json.load(f)

        base_dir = os.path.dirname(
            os.path.dirname(json_path)
        )  # Go up to vbench2_beta_i2v
        origin_dir = os.path.join(base_dir, "data", "origin")

        data = []
        for item in items:
            img_path = os.path.join(origin_dir, item.get("file_name", ""))
            if os.path.exists(img_path):
                data.append({"prompt": item.get("caption", ""), "image_path": img_path})
            else:
                logger.warning(f"Image not found: {img_path}")

        logger.info(f"Loaded {len(data)} I2V samples from VBench I2V dataset")
        return data
```
**EN:** This block defines method `_load_from_i2v_json` on `VBenchDataset`. Load I2V data from i2v-bench-info.json format. Key calls include `os.path.dirname`, `os.path.join`, `logger.info`, `open`, and `json.load`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `json_path` drive the behavior in this section.
**CN:** 该代码块定义了 `VBenchDataset` 的方法 `_load_from_i2v_json`。 它用于加载from i2v json。 关键调用包括 `os.path.dirname`、`os.path.join`、`logger.info`、`open` 和 `json.load`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `json_path` 等参数驱动。

### Lines 200-217: `_scan_directory_for_images` implementation / `_scan_directory_for_images` 实现
```python
    def _scan_directory_for_images(self, path: str) -> List[Dict[str, Any]]:
        """Scan directory for image files."""
        exts = ["*.jpg", "*.jpeg", "*.png", "*.webp"]
        files = []

        for ext in exts:
            files.extend(glob.glob(os.path.join(path, ext)))
            files.extend(glob.glob(os.path.join(path, ext.upper())))

            origin_dir = os.path.join(path, "data", "origin")
            if os.path.exists(origin_dir):
                files.extend(glob.glob(os.path.join(origin_dir, ext)))
                files.extend(glob.glob(os.path.join(origin_dir, ext.upper())))

        return [
            {"prompt": os.path.splitext(os.path.basename(f))[0], "image_path": f}
            for f in files
        ]
```
**EN:** This block defines method `_scan_directory_for_images` on `VBenchDataset`. Scan directory for image files. Key calls include `files.extend`, `os.path.join`, `os.path.exists`, `glob.glob`, and `os.path.splitext`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `path` drive the behavior in this section.
**CN:** 该代码块定义了 `VBenchDataset` 的方法 `_scan_directory_for_images`。 它用于处理 scan directory for images 相关逻辑。 关键调用包括 `files.extend`、`os.path.join`、`os.path.exists`、`glob.glob` 和 `os.path.splitext`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `path` 等参数驱动。

### Lines 219-230: `_create_dummy_data` implementation / `_create_dummy_data` 实现
```python
    def _create_dummy_data(self) -> List[Dict[str, Any]]:
        """Create dummy data with a placeholder image in cache directory."""
        logger.info("No I2V data found. Using dummy placeholders.")

        dummy_image = os.path.join(self.cache_dir, "dummy_image.jpg")
        if not os.path.exists(dummy_image):
            os.makedirs(self.cache_dir, exist_ok=True)
            img = Image.new("RGB", (100, 100), color="red")
            img.save(dummy_image)
            logger.info(f"Created dummy image at {dummy_image}")

        return [{"prompt": "A moving cat", "image_path": dummy_image}] * 10
```
**EN:** This block defines method `_create_dummy_data` on `VBenchDataset`. Create dummy data with a placeholder image in cache directory. Key calls include `logger.info`, `os.path.join`, `os.path.exists`, `os.makedirs`, and `Image.new`. The implementation branches on conditions.
**CN:** 该代码块定义了 `VBenchDataset` 的方法 `_create_dummy_data`。 它用于创建dummy data。 关键调用包括 `logger.info`、`os.path.join`、`os.path.exists`、`os.makedirs` 和 `Image.new`。 实现中包含条件分支。

### Lines 232-257: `_load_i2v_data` implementation / `_load_i2v_data` 实现
```python
    def _load_i2v_data(self) -> List[Dict[str, Any]]:
        """Load I2V data from VBench I2V dataset or user-provided path."""
        path = self.args.dataset_path
        if not path:
            path = self._auto_download_i2v_dataset()
            if not path:
                return self._resize_data(self._create_dummy_data())

        info_json_candidates = [
            os.path.join(path, "data", "i2v-bench-info.json"),
            path if path.endswith(".json") else None,
        ]

        for json_path in info_json_candidates:
            if json_path and os.path.exists(json_path):
                try:
                    return self._resize_data(self._load_from_i2v_json(json_path))
                except Exception as e:
                    logger.info(f"Failed to load {json_path}: {e}")

        if os.path.isdir(path):
            data = self._scan_directory_for_images(path)
            if data:
                return self._resize_data(data)

        return self._resize_data(self._create_dummy_data())
```
**EN:** This block defines method `_load_i2v_data` on `VBenchDataset`. Load I2V data from VBench I2V dataset or user-provided path. Key calls include `os.path.isdir`, `self._resize_data`, `self._auto_download_i2v_dataset`, `os.path.join`, and `self._scan_directory_for_images`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths.
**CN:** 该代码块定义了 `VBenchDataset` 的方法 `_load_i2v_data`。 它用于加载i2v data。 关键调用包括 `os.path.isdir`、`self._resize_data`、`self._auto_download_i2v_dataset`、`os.path.join` 和 `self._scan_directory_for_images`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。

### Lines 259-268: `_resize_data` implementation / `_resize_data` 实现
```python
    def _resize_data(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        """Resize data to match num_prompts."""
        if not self.args.num_prompts:
            return data

        if len(data) < self.args.num_prompts:
            factor = (self.args.num_prompts // len(data)) + 1
            data = data * factor

        return data[: self.args.num_prompts]
```
**EN:** This block defines method `_resize_data` on `VBenchDataset`. Resize data to match num_prompts. Key calls include `len`. The implementation branches on conditions. Parameters such as `data` drive the behavior in this section.
**CN:** 该代码块定义了 `VBenchDataset` 的方法 `_resize_data`。 它用于处理 resize data 相关逻辑。 关键调用包括 `len`。 实现中包含条件分支。 本段逻辑主要由 `data` 等参数驱动。

### Lines 270-271: `__len__` implementation / `__len__` 实现
```python
    def __len__(self) -> int:
        return len(self.items)
```
**EN:** This block defines method `__len__` on `VBenchDataset`. It handles len logic. Key calls include `len`.
**CN:** 该代码块定义了 `VBenchDataset` 的方法 `__len__`。 它用于处理 len 相关逻辑。 关键调用包括 `len`。

### Lines 273-285: `__getitem__` implementation / `__getitem__` 实现
```python
    def __getitem__(self, idx: int) -> RequestFuncInput:
        item = self.items[idx]
        return RequestFuncInput(
            prompt=item.get("prompt", ""),
            api_url=self.api_url,
            model=self.model,
            num_outputs_per_prompt=self.args.num_outputs_per_prompt,
            width=self.args.width,
            height=self.args.height,
            num_frames=self.args.num_frames,
            fps=self.args.fps,
            image_paths=[item["image_path"]] if "image_path" in item else None,
        )
```
**EN:** This block defines method `__getitem__` on `VBenchDataset`. It handles getitem logic. Key calls include `RequestFuncInput`, and `item.get`. Parameters such as `idx` drive the behavior in this section.
**CN:** 该代码块定义了 `VBenchDataset` 的方法 `__getitem__`。 它用于处理 getitem 相关逻辑。 关键调用包括 `RequestFuncInput` 和 `item.get`。 本段逻辑主要由 `idx` 等参数驱动。

### Lines 288-288: `RandomDataset` class overview / `RandomDataset` 类概览
```python
class RandomDataset(BaseDataset):
```
**EN:** This block defines class `RandomDataset`. It encapsulates random dataset behavior. It inherits from `BaseDataset`.
**CN:** 该代码块定义了类 `RandomDataset`。 它用于封装 random dataset 相关行为。 它继承自 `BaseDataset`。

### Lines 289-303: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, args, api_url: str = "", model: str = ""):
        super().__init__(args, api_url, model)
        self.num_prompts = args.num_prompts or 100

        self.random_request_config = args.random_request_config
        if self.random_request_config:
            self.random_request_config = json.loads(self.random_request_config)
            weights = [p.pop("weight") for p in self.random_request_config]
            seed = args.random_request_seed
            rng = random.Random(seed)
            self._sampled_requests = rng.choices(
                self.random_request_config, weights=weights, k=self.num_prompts
            )
        else:
            self._sampled_requests = None
```
**EN:** This block defines method `__init__` on `RandomDataset`. It initializes the instance state. Key calls include `super.__init__`, `json.loads`, `random.Random`, `rng.choices`, and `super`. The implementation branches on conditions. Parameters such as `args`, `api_url`, and `model` drive the behavior in this section.
**CN:** 该代码块定义了 `RandomDataset` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`json.loads`、`random.Random`、`rng.choices` 和 `super`。 实现中包含条件分支。 本段逻辑主要由 `args`、`api_url` 和 `model` 等参数驱动。

### Lines 305-309: `get_sampling_params` implementation / `get_sampling_params` 实现
```python
    def get_sampling_params(self, idx: int) -> dict:
        """Return the per-request sampling profile dict, or empty dict if not mix-diffusion."""
        if self._sampled_requests:
            return self._sampled_requests[idx]
        return {}
```
**EN:** This block defines method `get_sampling_params` on `RandomDataset`. Return the per-request sampling profile dict, or empty dict if not mix-diffusion. The implementation branches on conditions. Parameters such as `idx` drive the behavior in this section.
**CN:** 该代码块定义了 `RandomDataset` 的方法 `get_sampling_params`。 它用于获取sampling params。 实现中包含条件分支。 本段逻辑主要由 `idx` 等参数驱动。

### Lines 311-312: `__len__` implementation / `__len__` 实现
```python
    def __len__(self) -> int:
        return self.num_prompts
```
**EN:** This block defines method `__len__` on `RandomDataset`. It handles len logic.
**CN:** 该代码块定义了 `RandomDataset` 的方法 `__len__`。 它用于处理 len 相关逻辑。

### Lines 314-331: `__getitem__` implementation / `__getitem__` 实现
```python
    def __getitem__(self, idx: int) -> RequestFuncInput:
        profile = self._sampled_requests[idx] if self._sampled_requests else {}

        return RequestFuncInput(
            prompt=f"Random prompt {idx} for benchmarking diffusion models",
            api_url=self.api_url,
            model=self.model,
            num_outputs_per_prompt=profile.get(
                "num_outputs_per_prompt", self.args.num_outputs_per_prompt
            ),
            width=profile.get("width", self.args.width),
            height=profile.get("height", self.args.height),
            num_frames=profile.get("num_frames", self.args.num_frames),
            num_inference_steps=profile.get(
                "num_inference_steps", self.args.num_inference_steps
            ),
            fps=profile.get("fps", self.args.fps),
        )
```
**EN:** This block defines method `__getitem__` on `RandomDataset`. It handles getitem logic. Key calls include `RequestFuncInput`, and `profile.get`. Parameters such as `idx` drive the behavior in this section.
**CN:** 该代码块定义了 `RandomDataset` 的方法 `__getitem__`。 它用于处理 getitem 相关逻辑。 关键调用包括 `RequestFuncInput` 和 `profile.get`。 本段逻辑主要由 `idx` 等参数驱动。

## Key Concepts / 关键概念
- `RequestFuncInput`: Primary class that encapsulates request func input behavior. / 核心类，用于封装 request func input 相关行为。
- `RequestFuncOutput`: Primary class that encapsulates request func output behavior. / 核心类，用于封装 request func output 相关行为。
- `is_dir_not_empty`: Top-level function that handles is dir not empty logic. / 顶层函数，用于处理 is dir not empty 相关逻辑。
- `BaseDataset`: Primary class that encapsulates base dataset behavior. / 核心类，用于封装 base dataset 相关行为。
- `VBenchDataset`: Dataset loader for VBench prompts. / 核心类，用于封装 vbench dataset 相关行为。
- `RandomDataset`: Primary class that encapsulates random dataset behavior. / 核心类，用于封装 random dataset 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `glob`, `json`, `os`, `random`, `re`, `subprocess`, `uuid`, `abc`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `requests`, `PIL`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 331
