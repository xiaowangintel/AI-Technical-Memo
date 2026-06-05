# nextqa.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/hicache/nextqa.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on hicache nextqa. It primarily loads, filters, or serializes benchmark datasets and reads or writes local files and intermediate artifacts. / 该 Python 模块聚焦于 hicache nextqa 相关流程。它主要用于加载、筛选或序列化基准测试数据集、读写本地文件及中间产物。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Imports and setup / 导入与初始化
```python
import os
import sys
from typing import List

import av
from datasets import load_dataset
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会加载、筛选或序列化基准测试数据集。

### Lines 9-21: Function `find_video_files` / 函数 `find_video_files`
```python
def find_video_files(video_dir) -> List[str]:
    if os.path.isfile(video_dir):
        return [video_dir]

    video_files = []
    for root, dirs, files in os.walk(video_dir):
        for file in files:
            if file.endswith((".mp4", ".avi", ".mov")):
                video_files.append(os.path.join(root, file))
            # if file is dir
            elif os.path.isdir(file):
                video_files.extend(find_video_files(file))
    return video_files
```
**EN:** `find_video_files` is a function that implements the core logic for this scope. It returns `video_files` to the caller. Notable calls include `os.path.isfile`, `os.walk`, `file.endswith`.
**CN:** `find_video_files` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `video_files`。其中较关键的调用包括 `os.path.isfile`, `os.walk`, `file.endswith`。

### Lines 24-27: Function `video_frames` / 函数 `video_frames`
```python
def video_frames(video_path, max_frames) -> int:
    container = av.open(video_path)
    total_frames = container.streams.video[0].frames
    return min(total_frames, max_frames)
```
**EN:** `video_frames` is a function that reads or writes local files and intermediate artifacts. It returns `min(total_frames, max_frames)` to the caller. Notable calls include `av.open`, `min`.
**CN:** `video_frames` 是一个函数，用于读写本地文件及中间产物。它会向调用方返回 `min(total_frames, max_frames)`。其中较关键的调用包括 `av.open`, `min`。

### Lines 30-30: Class `Video` declaration / 类 `Video` 声明
```python
class Video:
```
**EN:** This block introduces class `Video`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `Video`，用于通过统一接口组织相关行为。

### Lines 31-33: Method `Video.__init__` / 方法 `Video.__init__`
```python
    def __init__(self, video_path, num_frames):
        self.path = video_path
        self.num_frames = num_frames
```
**EN:** `Video.__init__` is a method that implements the core logic for this scope.
**CN:** `Video.__init__` 是一个方法，用于实现当前作用域的核心逻辑。

### Lines 35-36: Method `Video.__str__` / 方法 `Video.__str__`
```python
    def __str__(self):
        return f"Video({self.path}, {self.num_frames})"
```
**EN:** `Video.__str__` is a method that implements the core logic for this scope. It returns `f'Video({self.path}, {self.num_frames})'` to the caller.
**CN:** `Video.__str__` 是一个方法，用于实现当前作用域的核心逻辑。它会向调用方返回 `f'Video({self.path}, {self.num_frames})'`。

### Lines 38-39: Method `Video.__iter__` / 方法 `Video.__iter__`
```python
    def __iter__(self):
        return iter((self.path, self.num_frames))
```
**EN:** `Video.__iter__` is a method that implements the core logic for this scope. It returns `iter((self.path, self.num_frames))` to the caller. Notable calls include `iter`.
**CN:** `Video.__iter__` 是一个方法，用于实现当前作用域的核心逻辑。它会向调用方返回 `iter((self.path, self.num_frames))`。其中较关键的调用包括 `iter`。

### Lines 42-42: Class `VideoPrompt` declaration / 类 `VideoPrompt` 声明
```python
class VideoPrompt(Video):
```
**EN:** This block introduces class `VideoPrompt`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `VideoPrompt`，用于通过统一接口组织相关行为。

### Lines 43-45: Method `VideoPrompt.__init__` / 方法 `VideoPrompt.__init__`
```python
    def __init__(self, video_path, num_frames, prompt):
        super().__init__(video_path, num_frames)
        self.prompt = prompt
```
**EN:** `VideoPrompt.__init__` is a method that implements the core logic for this scope. Notable calls include `super().__init__`, `super`.
**CN:** `VideoPrompt.__init__` 是一个方法，用于实现当前作用域的核心逻辑。其中较关键的调用包括 `super().__init__`, `super`。

### Lines 47-48: Method `VideoPrompt.__str__` / 方法 `VideoPrompt.__str__`
```python
    def __str__(self):
        return f"VideoPrompt({self.path}, {self.num_frames}, {self.prompt})"
```
**EN:** `VideoPrompt.__str__` is a method that implements the core logic for this scope. It returns `f'VideoPrompt({self.path}, {self.num_frames}, {self.prompt})'` to the caller.
**CN:** `VideoPrompt.__str__` 是一个方法，用于实现当前作用域的核心逻辑。它会向调用方返回 `f'VideoPrompt({self.path}, {self.num_frames}, {self.prompt})'`。

### Lines 50-51: Method `VideoPrompt.__iter__` / 方法 `VideoPrompt.__iter__`
```python
    def __iter__(self):
        return iter((self.path, self.num_frames, self.prompt))
```
**EN:** `VideoPrompt.__iter__` is a method that implements the core logic for this scope. It returns `iter((self.path, self.num_frames, self.prompt))` to the caller. Notable calls include `iter`.
**CN:** `VideoPrompt.__iter__` 是一个方法，用于实现当前作用域的核心逻辑。它会向调用方返回 `iter((self.path, self.num_frames, self.prompt))`。其中较关键的调用包括 `iter`。

### Lines 54-54: Class `VideoLoader` declaration / 类 `VideoLoader` 声明
```python
class VideoLoader:
```
**EN:** This block introduces class `VideoLoader`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `VideoLoader`，用于通过统一接口组织相关行为。

### Lines 55-55: Class-level state / 类级状态
```python
    pass
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。

### Lines 58-58: Class `VideoFileLoader` declaration / 类 `VideoFileLoader` 声明
```python
class VideoFileLoader(VideoLoader):
```
**EN:** This block introduces class `VideoFileLoader`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `VideoFileLoader`，用于通过统一接口组织相关行为。

### Lines 59-61: Class-level state / 类级状态
```python
    """
    Load all the videos in a directory
    """
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。

### Lines 63-69: Method `VideoFileLoader.__init__` / 方法 `VideoFileLoader.__init__`
```python
    def __init__(self, video_dir, batch_size=1, max_frames=sys.maxsize):
        super().__init__()
        self.video_dir = video_dir
        self.video_files = find_video_files(video_dir)
        self.batch_size = batch_size
        self.max_frames = max_frames
        print(f"batch_size: {batch_size}, max_frames: {max_frames}")
```
**EN:** `VideoFileLoader.__init__` is a method that implements the core logic for this scope. Notable calls include `super().__init__`, `find_video_files`, `print`.
**CN:** `VideoFileLoader.__init__` 是一个方法，用于实现当前作用域的核心逻辑。其中较关键的调用包括 `super().__init__`, `find_video_files`, `print`。

### Lines 71-82: Method `VideoFileLoader.__iter__` / 方法 `VideoFileLoader.__iter__`
```python
    def __iter__(self):  # (file, number of frames)
        if self.batch_size == 1:
            for video_file in self.video_files:
                yield Video(video_file, video_frames(video_file, self.max_frames))
        else:
            batch = []
            for video_file in self.video_files:
                video = Video(video_file, video_frames(video_file, self.max_frames))
                batch.append(video)
                if len(batch) == self.batch_size:
                    yield batch
                    batch = []
```
**EN:** `VideoFileLoader.__iter__` is a method that implements the core logic for this scope. Notable calls include `Video`, `batch.append`, `video_frames`.
**CN:** `VideoFileLoader.__iter__` 是一个方法，用于实现当前作用域的核心逻辑。其中较关键的调用包括 `Video`, `batch.append`, `video_frames`。

### Lines 85-85: Class `NExTQALoader` declaration / 类 `NExTQALoader` 声明
```python
class NExTQALoader(VideoLoader):
```
**EN:** This block introduces class `NExTQALoader`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `NExTQALoader`，用于通过统一接口组织相关行为。

### Lines 86-89: Class-level state / 类级状态
```python
    """
    Load vdideos and prompts from NExT dataset
    set: train, test or validation
    """
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized. In practice it loads, filters, or serializes benchmark datasets.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。 从实现上看，它会加载、筛选或序列化基准测试数据集。

### Lines 91-112: Method `NExTQALoader.__init__` / 方法 `NExTQALoader.__init__`
```python
    def __init__(
        self, video_dir, batch_size=1, max_frames=sys.maxsize, dset="test", task="OE"
    ):
        """
        task: 'MV' or 'OE'
        """
        super().__init__()
        self.task = task
        print(f"Loading the {dset} data of {task} from lmms-lab/NExTQA")
        self.ds = load_dataset("lmms-lab/NExTQA", task)
        self.ds = self.ds[dset]

        # self.n = ds.num_rows
        self.video_dir = video_dir
        self.video_files = find_video_files(video_dir)
        self.video_to_path = dict()
        for video_file in self.video_files:
            video_id = video_file.split("/")[-1].split(".")[0]
            self.video_to_path[video_id] = video_file

        self.batch_size = batch_size
        self.max_frames = max_frames
```
**EN:** `NExTQALoader.__init__` is a method that loads, filters, or serializes benchmark datasets. The docstring frames it as: task: 'MV' or 'OE' Notable calls include `super().__init__`, `print`, `load_dataset`.
**CN:** `NExTQALoader.__init__` 是一个方法，用于加载、筛选或序列化基准测试数据集。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `super().__init__`, `print`, `load_dataset`。

### Lines 114-124: Method `NExTQALoader.get_video_prompt` / 方法 `NExTQALoader.get_video_prompt`
```python
    def get_video_prompt(self, entry, max_frames) -> VideoPrompt:
        # Get video
        video_id = entry["video"]
        video_path = self.video_to_path[video_id]
        assert os.path.exists(video_path), f"Video not found: {video_path}"
        num_frames = min(entry["frame_count"], max_frames)
        video = Video(video_path, num_frames)
        prompt = entry["question"] + "?"
        if self.task == "MC":  # add choices
            prompt += f' a0: {entry["a0"]}, a1: {entry["a1"]}, a2: {entry["a2"]}, a3: {entry["a3"]}'
        return VideoPrompt(video_path, num_frames, prompt)
```
**EN:** `NExTQALoader.get_video_prompt` is a method that implements the core logic for this scope. It returns `VideoPrompt(video_path, num_frames, prompt)` to the caller. Notable calls include `os.path.exists`, `min`, `Video`.
**CN:** `NExTQALoader.get_video_prompt` 是一个方法，用于实现当前作用域的核心逻辑。它会向调用方返回 `VideoPrompt(video_path, num_frames, prompt)`。其中较关键的调用包括 `os.path.exists`, `min`, `Video`。

### Lines 126-137: Method `NExTQALoader.__iter__` / 方法 `NExTQALoader.__iter__`
```python
    def __iter__(self):
        if self.batch_size == 1:
            for entry in self.ds:
                yield self.get_video_prompt(entry, self.max_frames)
        else:
            batch = []
            for entry in self.ds:
                video = self.get_video_prompt(entry, self.max_frames)
                batch.append(video)
                if len(batch) == self.batch_size:
                    yield batch
                    batch = []
```
**EN:** `NExTQALoader.__iter__` is a method that implements the core logic for this scope. Notable calls include `self.get_video_prompt`, `batch.append`, `len`.
**CN:** `NExTQALoader.__iter__` 是一个方法，用于实现当前作用域的核心逻辑。其中较关键的调用包括 `self.get_video_prompt`, `batch.append`, `len`。

### Lines 141-155: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    video_dir = "./videos"
    # video_loader = VideoFileLoader(video_dir, batch_size=16)
    # for batch in video_loader:
    #     print(f"Number of videos in batch: {len(batch)}")
    #     for video_file, num_frames in batch:
    #         print(f"Video: {video_file} number of frames: {num_frames}")

    video_loader = NExTQALoader(video_dir, batch_size=16, dset="test", task="OE")
    for batch in video_loader:
        print(f"Number of videos in batch: {len(batch)}")
        for video_file, num_frames, prompt in batch:
            print(
                f"Video: {video_file} number of frames: {num_frames}, prompt: {prompt}"
            )
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `sys`, `typing`
- **Third-party / 第三方依赖**: `av`, `datasets`
