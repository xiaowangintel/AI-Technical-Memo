# auto_benchmark_lib.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/auto_benchmark_lib.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file provides top-level SGLang runtime utilities, launch helpers, benchmarking entrypoints, or environment configuration used by the broader package. / 该文件提供 SGLang 顶层运行时工具、启动辅助逻辑、基准入口或环境配置，供更大的包范围使用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module-level supporting statements
```python
import argparse
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 2-2: Module-level supporting statements
```python
import csv
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 3-3: Module-level supporting statements
```python
import itertools
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 4-4: Module-level supporting statements
```python
import json
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 5-5: Module-level supporting statements
```python
import os
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 6-6: Module-level supporting statements
```python
import shlex
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 7-7: Module-level supporting statements
```python
import signal
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 8-8: Module-level supporting statements
```python
import subprocess
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 9-9: Module-level supporting statements
```python
import sys
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 10-10: Module-level supporting statements
```python
import time
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 11-11: Module-level supporting statements
```python
from copy import deepcopy
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 12-12: Module-level supporting statements
```python
from types import SimpleNamespace
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 13-13: Module-level supporting statements
```python
from typing import Any, Callable, Dict, Iterable, List, Optional, Sequence, Tuple
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 14-14: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 15-15: Module-level supporting statements
```python
import yaml
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 16-16: Module-level supporting statements
```python
from tqdm.auto import tqdm
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 17-17: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 18-18: Module-level supporting statements
```python
from sglang.benchmark.datasets import get_dataset
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 19-22: Module-level supporting statements
```python
from sglang.benchmark.datasets.autobench import (
    sample_autobench_requests,
    serialize_dataset_row_to_autobench,
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 23-23: Module-level supporting statements
```python
from sglang.benchmark.utils import get_tokenizer
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 24-24: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 25-30: Module-level supporting statements
```python
SUPPORTED_DATASETS = {
    "sharegpt",
    "custom",
    "random",
    "generated-shared-prefix",
}
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 31-31: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 32-37: Module-level supporting statements
```python
FLAG_ALIASES = {
    "tp": "tp_size",
    "pp": "pp_size",
    "dp": "dp_size",
    "ep": "ep_size",
}
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 38-38: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 39-39: Module-level supporting statements
```python
OOM_HINT = "Candidate likely OOMed. Increase GPU count or use GPUs with larger memory."
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 40-65: Module-level supporting statements
```python
PROGRESS_FLAG_KEYS = (
    "tp_size",
    "dp_size",
    "ep_size",
    "pp_size",
    "prefill_attention_backend",
    "decode_attention_backend",
    "attention_backend",
    "sampling_backend",
    "grammar_backend",
    "mem_fraction_static",
    "chunked_prefill_size",
    "prefill_max_requests",
    "max_prefill_tokens",
    "max_running_requests",
    "max_queued_requests",
    "schedule_policy",
    "schedule_conservativeness",
    "num_continuous_decode_steps",
    "stream_interval",
    "page_size",
    "cuda_graph_max_bs",
    "speculative_num_steps",
    "speculative_eagle_topk",
    "speculative_num_draft_tokens",
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 66-91: Module-level supporting statements
```python
PROGRESS_FLAG_ALIASES = {
    "tp_size": "tp",
    "dp_size": "dp",
    "ep_size": "ep",
    "pp_size": "pp",
    "prefill_attention_backend": "prefill",
    "decode_attention_backend": "decode",
    "attention_backend": "attn",
    "sampling_backend": "sampling",
    "grammar_backend": "grammar",
    "mem_fraction_static": "mfs",
    "chunked_prefill_size": "chunk",
    "prefill_max_requests": "prefill_req",
    "max_prefill_tokens": "prefill_tok",
    "max_running_requests": "mrr",
    "max_queued_requests": "mqr",
    "schedule_policy": "sched",
    "schedule_conservativeness": "sched_cons",
    "num_continuous_decode_steps": "decode_steps",
    "stream_interval": "stream",
    "page_size": "page",
    "cuda_graph_max_bs": "cg_bs",
    "speculative_num_steps": "spec_steps",
    "speculative_eagle_topk": "eagle_topk",
    "speculative_num_draft_tokens": "draft_tok",
}
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 92-92: Module-level supporting statements
```python
SENSITIVE_ENV_MARKERS = ("TOKEN", "KEY", "SECRET", "PASSWORD")
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 93-93: Module-level supporting statements
```python
DEFAULT_MAX_CANDIDATES = 8
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 94-94: Module-level supporting statements
```python
MAX_BINARY_SEARCH_ROUNDS = 5
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 95-95: Module-level supporting statements
```python
DEFAULT_BINARY_SEARCH_ROUNDS = 5
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 96-96: Module-level supporting statements
```python
MAX_SEARCH_DURATION_HOURS = 12.0
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 97-97: Module-level supporting statements
```python
DEFAULT_SEARCH_DURATION_HOURS = 12.0
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 98-99: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 100-100: SearchDeadlineExceeded class declaration
```python
class SearchDeadlineExceeded(RuntimeError):
```
**EN:** This block declares the `SearchDeadlineExceeded` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `SearchDeadlineExceeded` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 101-101: Class-level supporting statements
```python
    """Raised when the auto benchmark exhausts its global search budget."""
```
**EN:** This block contains supporting statements for the `SearchDeadlineExceeded` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`SearchDeadlineExceeded` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 102-103: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 104-106: load yaml function
```python
def load_yaml(path: str) -> Dict[str, Any]:
    with open(path, "r", encoding="utf-8") as f:
        return yaml.safe_load(f)
```
**EN:** This block uses `load_yaml` to load resources or configuration. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `load_yaml` 来加载资源或配置。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 107-108: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 109-110: as list function
```python
def as_list(value: Any) -> List[Any]:
    return value if isinstance(value, list) else [value]
```
**EN:** This block uses `as_list` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `as_list` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 111-112: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 113-114: slugify function
```python
def slugify(text: str) -> str:
    return "".join(ch.lower() if ch.isalnum() else "-" for ch in text).strip("-")
```
**EN:** This block uses `slugify` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `slugify` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 115-116: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 117-118: canonical flag name function
```python
def canonical_flag_name(name: str) -> str:
    return FLAG_ALIASES.get(name, name)
```
**EN:** This block uses `canonical_flag_name` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `canonical_flag_name` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 119-120: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 121-122: canonicalize flags function
```python
def canonicalize_flags(flags: Dict[str, Any]) -> Dict[str, Any]:
    return {canonical_flag_name(key): value for key, value in flags.items()}
```
**EN:** This block uses `canonicalize_flags` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `canonicalize_flags` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 123-124: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 125-133: flatten function
```python
def flatten(data: Dict[str, Any], prefix: str = "") -> Dict[str, Any]:
    flat: Dict[str, Any] = {}
    for key, value in data.items():
        name = f"{prefix}.{key}" if prefix else key
        if isinstance(value, dict):
            flat.update(flatten(value, name))
        else:
            flat[name] = value
    return flat
```
**EN:** This block uses `flatten` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `flatten` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 134-135: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 136-137: log line function
```python
def log_line(message: str) -> None:
    tqdm.write(message)
```
**EN:** This block uses `log_line` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `log_line` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 138-139: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 140-149: detect current cuda capability function
```python
def detect_current_cuda_capability() -> Optional[Tuple[int, int]]:
    try:
        import torch
    except ModuleNotFoundError:
        return None

    if not torch.cuda.is_available():
        return None
    major, minor = torch.cuda.get_device_capability()
    return int(major), int(minor)
```
**EN:** This block uses `detect_current_cuda_capability` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `detect_current_cuda_capability` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 150-151: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 152-161: is attention backend supported function
```python
def is_attention_backend_supported(
    backend: Any, capability: Optional[Tuple[int, int]]
) -> bool:
    if capability is None or backend in (None, ""):
        return True

    major, _minor = capability
    if backend == "fa3":
        return major in (8, 9)
    return True
```
**EN:** This block uses `is_attention_backend_supported` to bridge this module with an external system. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `is_attention_backend_supported` 来将本模块与外部系统桥接起来。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 162-163: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 164-175: is candidate supported on current device function
```python
def is_candidate_supported_on_current_device(
    candidate: Dict[str, Any], capability: Optional[Tuple[int, int]]
) -> bool:
    backend_keys = (
        "attention_backend",
        "prefill_attention_backend",
        "decode_attention_backend",
    )
    return all(
        is_attention_backend_supported(candidate.get(key), capability)
        for key in backend_keys
    )
```
**EN:** This block uses `is_candidate_supported_on_current_device` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `is_candidate_supported_on_current_device` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 176-177: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 178-181: append jsonl function
```python
def append_jsonl(path: str, records: Iterable[Dict[str, Any]]) -> None:
    with open(path, "a", encoding="utf-8") as f:
        for record in records:
            f.write(json.dumps(record, ensure_ascii=False) + "\n")
```
**EN:** This block uses `append_jsonl` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `append_jsonl` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 182-183: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 184-194: read jsonl function
```python
def read_jsonl(path: str) -> List[Dict[str, Any]]:
    if not path or not os.path.isfile(path):
        return []
    records: List[Dict[str, Any]] = []
    with open(path, "r", encoding="utf-8") as f:
        for line in f:
            line = line.strip()
            if not line:
                continue
            records.append(json.loads(line))
    return records
```
**EN:** This block uses `read_jsonl` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `read_jsonl` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 195-196: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 197-203: describe search tier function
```python
def describe_search_tier(tier: int) -> str:
    descriptions = {
        1: "tier 1: smallest and fastest sanity sweep",
        2: "tier 2: balanced default sweep",
        3: "tier 3: largest and slowest full search",
    }
    return descriptions.get(tier, f"tier {tier}")
```
**EN:** This block uses `describe_search_tier` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `describe_search_tier` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 204-205: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 206-218: install interrupt handlers function
```python
def install_interrupt_handlers() -> Dict[signal.Signals, Any]:
    previous = {}

    def handler(signum, _frame):  # type: ignore[no-untyped-def]
        raise KeyboardInterrupt(f"Interrupted by signal {signum}")

    for sig in (signal.SIGINT, signal.SIGTERM):
        try:
            previous[sig] = signal.getsignal(sig)
            signal.signal(sig, handler)
        except Exception:
            continue
    return previous
```
**EN:** This block uses `install_interrupt_handlers` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `install_interrupt_handlers` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 219-220: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 221-226: restore interrupt handlers function
```python
def restore_interrupt_handlers(previous: Dict[signal.Signals, Any]) -> None:
    for sig, handler in previous.items():
        try:
            signal.signal(sig, handler)
        except Exception:
            continue
```
**EN:** This block uses `restore_interrupt_handlers` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `restore_interrupt_handlers` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 227-228: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 229-251: collect stale server pids function
```python
def collect_stale_server_pids(port: int) -> List[int]:
    patterns = [
        ["lsof", "-ti", f"tcp:{port}", "-sTCP:LISTEN"],
        ["pgrep", "-f", f"sglang.launch_server.*--port {port}"],
        ["pgrep", "-f", f"sglang.launch_server.*--port={port}"],
        ["pgrep", "-f", f"sglang serve .*--port {port}"],
        ["pgrep", "-f", f"sglang serve .*--port={port}"],
    ]
    pids = set()
    for command in patterns:
        try:
            result = subprocess.run(
                command, capture_output=True, text=True, check=False
            )
        except FileNotFoundError:
            continue
        if result.returncode not in (0, 1):
            continue
        for line in result.stdout.splitlines():
            line = line.strip()
            if line.isdigit():
                pids.add(int(line))
    return sorted(pids)
```
**EN:** This block uses `collect_stale_server_pids` to bridge this module with an external system. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `collect_stale_server_pids` 来将本模块与外部系统桥接起来。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 252-253: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 254-271: kill pid or group function
```python
def kill_pid_or_group(pid: int) -> None:
    try:
        pgid = os.getpgid(pid)
    except ProcessLookupError:
        return

    for sig, delay in ((signal.SIGTERM, 1.0), (signal.SIGKILL, 0.0)):
        try:
            os.killpg(pgid, sig)
        except ProcessLookupError:
            return
        except PermissionError:
            try:
                os.kill(pid, sig)
            except ProcessLookupError:
                return
        if delay:
            time.sleep(delay)
```
**EN:** This block uses `kill_pid_or_group` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `kill_pid_or_group` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 272-273: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 274-280: preclean stale server function
```python
def preclean_stale_server(port: int) -> None:
    stale_pids = collect_stale_server_pids(port)
    if not stale_pids:
        return
    log_line(f"preclean_port={port} stale_pids={stale_pids}")
    for pid in stale_pids:
        kill_pid_or_group(pid)
```
**EN:** This block uses `preclean_stale_server` to bridge this module with an external system. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `preclean_stale_server` 来将本模块与外部系统桥接起来。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 281-282: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 283-286: normalize binary search rounds function
```python
def normalize_binary_search_rounds(value: Any) -> int:
    if value is None:
        return DEFAULT_BINARY_SEARCH_ROUNDS
    return max(1, min(int(value), MAX_BINARY_SEARCH_ROUNDS))
```
**EN:** This block uses `normalize_binary_search_rounds` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `normalize_binary_search_rounds` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 287-288: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 289-298: resolve max candidates function
```python
def resolve_max_candidates(search_cfg: Dict[str, Any]) -> Optional[int]:
    if "max_candidates" not in search_cfg:
        return DEFAULT_MAX_CANDIDATES
    configured = search_cfg.get("max_candidates")
    if configured is None:
        return None
    value = int(configured)
    if value < 1:
        raise ValueError("search.max_candidates must be >= 1 or null.")
    return value
```
**EN:** This block uses `resolve_max_candidates` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `resolve_max_candidates` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 299-300: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 301-315: estimate binary search trials function
```python
def estimate_binary_search_trials(
    lower: float, upper: float, tolerance: float, max_rounds: int
) -> int:
    if upper <= lower or tolerance <= 0:
        return 1

    trials = 0
    lo, hi = float(lower), float(upper)
    while hi - lo > tolerance and trials < max_rounds:
        qps = pick_qps_midpoint(lo, hi)
        if qps <= lo or qps >= hi:
            break
        hi = qps
        trials += 1
    return max(trials, 1)
```
**EN:** This block uses `estimate_binary_search_trials` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `estimate_binary_search_trials` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 316-317: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 318-322: pick qps midpoint function
```python
def pick_qps_midpoint(lower: float, upper: float) -> float:
    midpoint = round((lower + upper) / 2, 4)
    if lower < midpoint < upper:
        return midpoint
    return (lower + upper) / 2
```
**EN:** This block uses `pick_qps_midpoint` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `pick_qps_midpoint` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 323-324: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 325-334: estimate trials per candidate function
```python
def estimate_trials_per_candidate(benchmark_cfg: Dict[str, Any]) -> int:
    mode, values, tolerance, max_rounds = build_qps_plan(benchmark_cfg)
    max_concurrency_values = as_list(benchmark_cfg.get("max_concurrency", [None]))
    if mode == "fixed":
        per_concurrency = len(values)
    else:
        per_concurrency = estimate_binary_search_trials(
            values[0], values[1], tolerance, max_rounds
        )
    return max(1, per_concurrency) * len(max_concurrency_values)
```
**EN:** This block uses `estimate_trials_per_candidate` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `estimate_trials_per_candidate` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 335-336: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 337-346: describe qps plan function
```python
def describe_qps_plan(benchmark_cfg: Dict[str, Any]) -> str:
    mode, values, tolerance, max_rounds = build_qps_plan(benchmark_cfg)
    if mode == "fixed":
        return f"fixed qps values={values}"
    return (
        f"binary search qps lower={values[0]} upper={values[1]} "
        f"tolerance={tolerance} max_rounds={max_rounds} "
        "estimated_trials_per_max_concurrency="
        f"{estimate_binary_search_trials(values[0], values[1], tolerance, max_rounds)}"
    )
```
**EN:** This block uses `describe_qps_plan` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `describe_qps_plan` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 347-348: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 349-357: scenario plan text function
```python
def scenario_plan_text(scenario: Dict[str, Any]) -> str:
    cfg = scenario["cfg"]
    parts = [f"kind={cfg['kind']}", f"num_prompts={cfg.get('num_prompts', '')}"]
    if cfg["kind"] == "random":
        parts.append(f"input_len={cfg['random_input_len']}")
        parts.append(f"output_len={cfg['random_output_len']}")
    elif cfg.get("path"):
        parts.append(f"path={cfg['path']}")
    return ", ".join(str(part) for part in parts if part != "")
```
**EN:** This block uses `scenario_plan_text` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `scenario_plan_text` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 358-359: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 360-412: print run plan function
```python
def print_run_plan(
    config_path: str,
    output_dir: str,
    tier: int,
    max_candidates: Optional[int],
    benchmark_cfg: Dict[str, Any],
    scenarios: Sequence[Dict[str, Any]],
    server_cfg: Dict[str, Any],
    base_candidates: Sequence[Dict[str, Any]],
    speculative_enabled: bool,
    search_budget_hours: float,
    search_deadline: float,
) -> None:
    estimated_base_trials = (
        len(scenarios)
        * len(base_candidates)
        * estimate_trials_per_candidate(benchmark_cfg)
    )
    log_line("=== Auto Benchmark Plan ===")
    log_line(f"config={config_path}")
    log_line(f"output_dir={output_dir}")
    log_line(f"search.tier={tier} ({describe_search_tier(tier)})")
    log_line(
        "search.max_candidates="
        f"{max_candidates if max_candidates is not None else 'unbounded'}"
    )
    log_line(
        f"search.max_duration_hours={search_budget_hours:.1f} "
        f"(deadline {time.strftime('%Y-%m-%d %H:%M:%S', time.localtime(search_deadline))})"
    )
    log_line(f"qps_plan={describe_qps_plan(benchmark_cfg)}")
    log_line(
        "max_concurrency="
        f"{json.dumps(as_list(benchmark_cfg.get('max_concurrency', [None])), ensure_ascii=False)}"
    )
    log_line(f"estimated_base_trials={estimated_base_trials}")
    log_line("Planned scenarios:")
    for index, scenario in enumerate(scenarios, start=1):
        log_line(
            f"  [{index}/{len(scenarios)}] {scenario['display_name']}: "
            f"{scenario_plan_text(scenario)}"
        )
    log_line("Planned base candidates:")
    for index, candidate in enumerate(base_candidates, start=1):
        rendered = merge_host_port(server_cfg, candidate)
        log_line(
            f"  [{index}/{len(base_candidates)}] {json.dumps(rendered, ensure_ascii=False)}"
        )
    if speculative_enabled:
        log_line(
            "Speculative stage is enabled. Its candidate list will be printed after "
            "the best base configuration is known."
        )
```
**EN:** This block uses `print_run_plan` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `print_run_plan` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 413-414: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 415-425: estimated finish time function
```python
def estimated_finish_time(
    start_time: float, completed: int, total: Optional[int]
) -> str:
    if not total or completed <= 0:
        return "?"
    remaining_seconds = max(
        0.0, (time.time() - start_time) * (total - completed) / completed
    )
    return time.strftime(
        "%Y-%m-%d %H:%M:%S", time.localtime(time.time() + remaining_seconds)
    )
```
**EN:** This block uses `estimated_finish_time` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `estimated_finish_time` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 426-427: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 428-429: current time text function
```python
def current_time_text() -> str:
    return time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())
```
**EN:** This block uses `current_time_text` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `current_time_text` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 430-431: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 432-434: resolve search budget hours function
```python
def resolve_search_budget_hours(search_cfg: Dict[str, Any]) -> float:
    configured = search_cfg.get("max_duration_hours", DEFAULT_SEARCH_DURATION_HOURS)
    return max(0.0, min(float(configured), MAX_SEARCH_DURATION_HOURS))
```
**EN:** This block uses `resolve_search_budget_hours` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `resolve_search_budget_hours` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 435-436: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 437-438: format timestamp function
```python
def format_timestamp(timestamp: float) -> str:
    return time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(timestamp))
```
**EN:** This block uses `format_timestamp` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `format_timestamp` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 439-440: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 441-444: remaining search seconds function
```python
def remaining_search_seconds(search_deadline: Optional[float]) -> Optional[float]:
    if search_deadline is None:
        return None
    return max(0.0, search_deadline - time.time())
```
**EN:** This block uses `remaining_search_seconds` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `remaining_search_seconds` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 445-446: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 447-457: raise if search deadline reached function
```python
def raise_if_search_deadline_reached(
    search_deadline: Optional[float], budget_hours: float
) -> None:
    remaining = remaining_search_seconds(search_deadline)
    if remaining is None or remaining > 0:
        return
    raise SearchDeadlineExceeded(
        "search budget of "
        f"{budget_hours:.1f}h reached before the full search completed "
        f"(deadline {format_timestamp(search_deadline)})"
    )
```
**EN:** This block uses `raise_if_search_deadline_reached` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `raise_if_search_deadline_reached` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 458-459: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 460-474: summarize progress flags function
```python
def summarize_progress_flags(server_flags: Dict[str, Any], limit: int = 6) -> str:
    parts = []
    for key in PROGRESS_FLAG_KEYS:
        if key not in server_flags:
            continue
        value = server_flags[key]
        if value in (None, "", False):
            continue
        alias = PROGRESS_FLAG_ALIASES.get(key, key)
        parts.append(f"{alias}={value}")
        if len(parts) >= limit:
            break
    if not parts and server_flags.get("candidate_id") is not None:
        return f"candidate={server_flags['candidate_id']}"
    return ",".join(parts)
```
**EN:** This block uses `summarize_progress_flags` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `summarize_progress_flags` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 475-476: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 477-491: format best progress function
```python
def format_best_progress(record: Optional[Dict[str, Any]]) -> str:
    if not record or not record.get("metrics"):
        return "best pending"

    metrics = record["metrics"]
    flags = dict(record.get("server_flags", {}))
    flags["candidate_id"] = record.get("candidate_id")
    return (
        "best "
        f"qps={record.get('requested_qps', 0.0):.4f} "
        f"tok/s={metrics.get('output_throughput', 0.0):.1f} "
        f"ttft={metrics.get('mean_ttft_ms', 0.0):.1f}ms "
        f"tpot={metrics.get('mean_tpot_ms', 0.0):.1f}ms "
        f"cfg[{summarize_progress_flags(flags)}]"
    )
```
**EN:** This block uses `format_best_progress` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `format_best_progress` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 492-493: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 494-502: refresh progress eta function
```python
def refresh_progress_eta(
    pbar: tqdm, start_time: float, best_record: Optional[Dict[str, Any]] = None
) -> None:
    pbar.set_postfix_str(
        f"now {current_time_text()} | "
        f"finish {estimated_finish_time(start_time, int(pbar.n), pbar.total)} | "
        f"{format_best_progress(best_record)}",
        refresh=False,
    )
```
**EN:** This block uses `refresh_progress_eta` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `refresh_progress_eta` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 503-504: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 505-519: make progress bar function
```python
def make_progress_bar(
    desc: str, total: int, position: int, leave: bool
) -> Tuple[tqdm, float]:
    start_time = time.time()
    pbar = tqdm(
        total=total,
        desc=desc,
        dynamic_ncols=True,
        mininterval=1.0,
        position=position,
        leave=leave,
        bar_format="{l_bar}{bar}| {n_fmt}/{total_fmt} [{elapsed}<{remaining}, {rate_fmt}] {postfix}",
    )
    refresh_progress_eta(pbar, start_time)
    return pbar, start_time
```
**EN:** This block uses `make_progress_bar` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `make_progress_bar` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 520-521: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 522-531: advance progress function
```python
def advance_progress(
    pbar: tqdm,
    start_time: float,
    count: int = 1,
    best_record: Optional[Dict[str, Any]] = None,
) -> None:
    if pbar.total is not None and pbar.n + count > pbar.total:
        pbar.total = pbar.n + count
    pbar.update(count)
    refresh_progress_eta(pbar, start_time, best_record)
```
**EN:** This block uses `advance_progress` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `advance_progress` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 532-533: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 534-539: tail text function
```python
def tail_text(path: str, limit: int = 4000) -> str:
    if not path or not os.path.isfile(path):
        return ""
    with open(path, "r", encoding="utf-8", errors="ignore") as f:
        text = f.read()
    return text[-limit:]
```
**EN:** This block uses `tail_text` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `tail_text` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 540-541: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 542-555: cli args function
```python
def cli_args(flags: Dict[str, Any]) -> List[str]:
    args: List[str] = []
    for key, value in flags.items():
        if value is None or value is False:
            continue
        flag = f"--{key.replace('_', '-')}"
        if value is True:
            args.append(flag)
        elif isinstance(value, list):
            args.append(flag)
            args.extend(str(item) for item in value)
        else:
            args.extend([flag, str(value)])
    return args
```
**EN:** This block uses `cli_args` to store configuration or metadata. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `cli_args` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 556-557: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 558-572: classify failure function
```python
def classify_failure(message: str) -> Tuple[Optional[str], Optional[str]]:
    lower = message.lower()
    oom_markers = (
        "out of memory",
        "cuda out of memory",
        "hip out of memory",
        "cudnn_status_alloc_failed",
        "std::bad_alloc",
        "memoryerror",
        "memory allocation",
        "no available memory",
    )
    if any(marker in lower for marker in oom_markers):
        return "oom", OOM_HINT
    return None, None
```
**EN:** This block uses `classify_failure` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `classify_failure` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 573-574: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 575-585: prompt kind function
```python
def prompt_kind(prompt: Any) -> str:
    if isinstance(prompt, str):
        return "prompt"
    if isinstance(prompt, list) and prompt:
        if isinstance(prompt[0], dict):
            return "messages"
        if isinstance(prompt[0], str):
            return "multi_turn"
        if isinstance(prompt[0], int):
            return "token_ids"
    return "unknown"
```
**EN:** This block uses `prompt_kind` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `prompt_kind` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 586-587: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 588-602: summarize rows function
```python
def summarize_rows(rows: Sequence[Any]) -> Dict[str, Any]:
    kinds: Dict[str, int] = {}
    output_lens = [row.output_len for row in rows]
    for row in rows:
        kind = prompt_kind(row.prompt)
        kinds[kind] = kinds.get(kind, 0) + 1
    return {
        "num_requests": len(rows),
        "prompt_kinds": kinds,
        "output_len_min": min(output_lens) if output_lens else 0,
        "output_len_max": max(output_lens) if output_lens else 0,
        "output_len_avg": (
            round(sum(output_lens) / len(output_lens), 2) if output_lens else 0.0
        ),
    }
```
**EN:** This block uses `summarize_rows` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `summarize_rows` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 603-604: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 605-619: infer backend function
```python
def infer_backend(backend: str, rows: Sequence[Any]) -> str:
    if backend != "auto":
        return backend

    kinds = {prompt_kind(row.prompt) for row in rows}
    if kinds <= {"messages", "multi_turn"}:
        return "sglang-oai-chat"
    if kinds <= {"prompt"}:
        return "sglang-oai"
    if kinds <= {"token_ids"}:
        return "sglang"
    raise ValueError(
        f"Cannot infer backend for mixed prompt kinds: {sorted(kinds)}. "
        "Set benchmark.backend explicitly."
    )
```
**EN:** This block uses `infer_backend` to bridge this module with an external system. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `infer_backend` 来将本模块与外部系统桥接起来。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 620-621: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 622-637: looks like autobench function
```python
def looks_like_autobench(path: str) -> bool:
    if not path or not os.path.isfile(path):
        return False
    with open(path, "r", encoding="utf-8") as f:
        for line in f:
            line = line.strip()
            if not line:
                continue
            try:
                row = json.loads(line)
            except json.JSONDecodeError:
                return False
            return isinstance(row, dict) and any(
                key in row for key in ("prompt", "messages", "prompt_origin", "system")
            )
    return False
```
**EN:** This block uses `looks_like_autobench` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `looks_like_autobench` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 638-639: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 640-649: write autobench jsonl function
```python
def write_autobench_jsonl(
    path: str, rows: Sequence[Any], metadata: Optional[Dict[str, Any]] = None
) -> None:
    directory = os.path.dirname(path)
    if directory:
        os.makedirs(directory, exist_ok=True)
    with open(path, "w", encoding="utf-8") as f:
        for row in rows:
            record = serialize_dataset_row_to_autobench(row, metadata=metadata)
            f.write(json.dumps(record, ensure_ascii=False) + "\n")
```
**EN:** This block uses `write_autobench_jsonl` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `write_autobench_jsonl` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 650-651: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 652-681: normalize dataset cfg function
```python
def normalize_dataset_cfg(
    dataset_cfg: Optional[Dict[str, Any]], benchmark_cfg: Dict[str, Any]
) -> Dict[str, Any]:
    raw = {} if dataset_cfg is None else dataset_cfg
    if isinstance(raw, str):
        raw = {"kind": raw}
    cfg = dict(raw)

    if "kind" not in cfg and cfg.get("path") in SUPPORTED_DATASETS:
        cfg["kind"] = cfg["path"]
        cfg["path"] = ""

    if "kind" not in cfg and benchmark_cfg.get("dataset_path"):
        cfg["kind"] = "custom"
        cfg["path"] = benchmark_cfg["dataset_path"]

    if "num_prompts" not in cfg and benchmark_cfg.get("num_prompts") is not None:
        cfg["num_prompts"] = benchmark_cfg["num_prompts"]

    cfg["kind"] = cfg.get("kind", "custom")
    if cfg["kind"] == "autobench":
        cfg["kind"] = "custom"
    if cfg["kind"] not in SUPPORTED_DATASETS:
        raise ValueError(
            f"Unsupported dataset kind: {cfg['kind']}. "
            f"Supported: {sorted(SUPPORTED_DATASETS)}"
        )
    if cfg["kind"] == "custom" and not cfg.get("path"):
        raise ValueError("dataset.path is required when dataset.kind=custom.")
    return cfg
```
**EN:** This block uses `normalize_dataset_cfg` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `normalize_dataset_cfg` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 682-683: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 684-732: expand dataset scenarios function
```python
def expand_dataset_scenarios(dataset_cfg: Dict[str, Any]) -> List[Dict[str, Any]]:
    if dataset_cfg["kind"] != "random":
        name = dataset_cfg.get("scenario_name", "default")
        return [
            {
                "name": slugify(str(name)) or "default",
                "display_name": str(name),
                "cfg": dataset_cfg,
            }
        ]

    input_lens = as_list(
        dataset_cfg.get("input_len", dataset_cfg.get("random_input_len", 1024))
    )
    output_lens = as_list(
        dataset_cfg.get("output_len", dataset_cfg.get("random_output_len", 256))
    )
    if len(input_lens) != len(output_lens):
        raise ValueError(
            "random dataset input_len and output_len must have the same number of elements."
        )

    scenario_names = dataset_cfg.get("scenario_names")
    if scenario_names is not None and len(as_list(scenario_names)) != len(input_lens):
        raise ValueError(
            "dataset.scenario_names must match the length of input_len/output_len."
        )

    names = as_list(scenario_names) if scenario_names is not None else None
    scenarios = []
    for index, (input_len, output_len) in enumerate(zip(input_lens, output_lens)):
        cfg = dict(dataset_cfg)
        cfg["random_input_len"] = int(input_len)
        cfg["random_output_len"] = int(output_len)
        cfg["input_len"] = int(input_len)
        cfg["output_len"] = int(output_len)
        display_name = (
            str(names[index])
            if names is not None
            else f"input{int(input_len)}-output{int(output_len)}"
        )
        scenarios.append(
            {
                "name": slugify(display_name) or f"scenario-{index + 1}",
                "display_name": display_name,
                "cfg": cfg,
            }
        )
    return scenarios
```
**EN:** This block uses `expand_dataset_scenarios` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `expand_dataset_scenarios` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 733-734: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 735-771: build dataset args function
```python
def build_dataset_args(
    dataset_cfg: Dict[str, Any], tokenizer_path: str, model: Optional[str]
) -> SimpleNamespace:
    dataset_path = dataset_cfg.get("path", "")
    if dataset_cfg["kind"] == "sharegpt" and dataset_path in ("", None, "sharegpt"):
        dataset_path = ""
    is_random = dataset_cfg["kind"] == "random"

    return SimpleNamespace(
        dataset_name=dataset_cfg["kind"],
        dataset_path=dataset_path,
        tokenizer=tokenizer_path,
        model=model,
        num_prompts=int(dataset_cfg.get("num_prompts", 1000)),
        sharegpt_output_len=(dataset_cfg.get("output_len") if not is_random else None),
        sharegpt_context_len=dataset_cfg.get("context_len"),
        random_input_len=int(
            dataset_cfg.get("input_len", dataset_cfg.get("random_input_len", 1024))
        ),
        random_output_len=int(
            dataset_cfg.get("output_len", dataset_cfg.get("random_output_len", 256))
        ),
        random_range_ratio=float(dataset_cfg.get("random_range_ratio", 0.0)),
        prompt_suffix=dataset_cfg.get("prompt_suffix", ""),
        apply_chat_template=bool(dataset_cfg.get("apply_chat_template", False)),
        gsp_num_groups=int(dataset_cfg.get("gsp_num_groups", 64)),
        gsp_prompts_per_group=int(dataset_cfg.get("gsp_prompts_per_group", 16)),
        gsp_system_prompt_len=int(dataset_cfg.get("gsp_system_prompt_len", 2048)),
        gsp_question_len=int(dataset_cfg.get("gsp_question_len", 128)),
        gsp_output_len=int(dataset_cfg.get("gsp_output_len", 256)),
        gsp_range_ratio=float(dataset_cfg.get("gsp_range_ratio", 1.0)),
        gsp_fast_prepare=bool(dataset_cfg.get("gsp_fast_prepare", False)),
        gsp_send_routing_key=bool(dataset_cfg.get("gsp_send_routing_key", False)),
        gsp_num_turns=int(dataset_cfg.get("gsp_num_turns", 1)),
        gsp_ordered=bool(dataset_cfg.get("gsp_ordered", False)),
        seed=int(dataset_cfg.get("seed", 1)),
    )
```
**EN:** This block uses `build_dataset_args` to build helper structures or requests. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `build_dataset_args` 来构建辅助结构或请求。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 772-773: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 774-785: load autobench rows function
```python
def load_autobench_rows(
    dataset_path: str,
    tokenizer_path: str,
    num_prompts: int = 0,
    output_len: Optional[int] = None,
) -> List[Any]:
    return sample_autobench_requests(
        dataset_path=dataset_path,
        num_requests=num_prompts,
        tokenizer=get_tokenizer(tokenizer_path),
        fixed_output_len=output_len,
    )
```
**EN:** This block uses `load_autobench_rows` to load resources or configuration. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `load_autobench_rows` 来加载资源或配置。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 786-787: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 788-820: prepare dataset function
```python
def prepare_dataset(
    dataset_cfg: Dict[str, Any],
    tokenizer_path: str,
    model: Optional[str],
    output_path: str,
) -> Tuple[str, List[Any], Dict[str, Any]]:
    dataset_cfg = normalize_dataset_cfg(dataset_cfg, {})
    if dataset_cfg["kind"] == "custom" and looks_like_autobench(
        dataset_cfg.get("path", "")
    ):
        rows = load_autobench_rows(
            dataset_path=dataset_cfg["path"],
            tokenizer_path=tokenizer_path,
            num_prompts=int(dataset_cfg.get("num_prompts", 0)),
            output_len=dataset_cfg.get("output_len"),
        )
    else:
        tokenizer = get_tokenizer(tokenizer_path)
        dataset_args = build_dataset_args(dataset_cfg, tokenizer_path, model)
        rows = get_dataset(dataset_args, tokenizer=tokenizer, model_id=model)

    if not rows:
        raise ValueError("Prepared dataset is empty.")

    write_autobench_jsonl(
        output_path,
        rows,
        metadata={
            "source_dataset_name": dataset_cfg["kind"],
            "source_dataset_path": dataset_cfg.get("path") or dataset_cfg["kind"],
        },
    )
    return output_path, rows, summarize_rows(rows)
```
**EN:** This block uses `prepare_dataset` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `prepare_dataset` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 821-822: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 823-844: infer total gpus function
```python
def infer_total_gpus(server_cfg: Dict[str, Any]) -> Optional[int]:
    parallel_cfg = server_cfg.get("parallel", {})
    for key in ("gpu_count",):
        value = parallel_cfg.get(key, server_cfg.get(key))
        if value is not None:
            return int(value)

    env = server_cfg.get("env", {})
    for key in (
        "CUDA_VISIBLE_DEVICES",
        "ROCR_VISIBLE_DEVICES",
        "HIP_VISIBLE_DEVICES",
        "NVIDIA_VISIBLE_DEVICES",
    ):
        value = env.get(key)
        if value is None:
            continue
        value = str(value).strip()
        if not value or value.lower() in {"all", "none", "void"}:
            continue
        return len([item for item in value.split(",") if item.strip()])
    return None
```
**EN:** This block uses `infer_total_gpus` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `infer_total_gpus` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 845-846: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 847-874: resolve parallelism function
```python
def resolve_parallelism(
    server_cfg: Dict[str, Any], flags: Dict[str, Any], parallel_requested: bool
) -> Dict[str, Any]:
    flags = canonicalize_flags(flags)
    if not parallel_requested:
        return flags

    tp_size = int(flags.get("tp_size", 1))
    pp_size = int(flags.get("pp_size", 1))
    if "dp_size" in flags:
        return flags

    total_gpus = infer_total_gpus(server_cfg)
    if total_gpus is None:
        raise ValueError(
            "Cannot infer total GPU count for parallel search. "
            "Set server.parallel.gpu_count or server.env.CUDA_VISIBLE_DEVICES."
        )

    shard_size = tp_size * pp_size
    if shard_size <= 0 or total_gpus % shard_size != 0:
        raise ValueError(
            f"Cannot derive dp_size: total_gpus={total_gpus}, "
            f"tp_size={tp_size}, pp_size={pp_size}."
        )

    flags["dp_size"] = total_gpus // shard_size
    return flags
```
**EN:** This block uses `resolve_parallelism` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `resolve_parallelism` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 875-876: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 877-903: build server candidates function
```python
def build_server_candidates(
    server_cfg: Dict[str, Any], tier: int, max_candidates: Optional[int]
) -> List[Dict[str, Any]]:
    base_flags = canonicalize_flags(deepcopy(server_cfg.get("base_flags", {})))
    search_space = canonicalize_flags(deepcopy(server_cfg.get("search_space", {})))
    parallel_cfg = canonicalize_flags(deepcopy(server_cfg.get("parallel", {})))
    parallel_requested = bool(parallel_cfg)
    for key, value in parallel_cfg.items():
        if key == "gpu_count":
            continue
        values = as_list(value)
        if values:
            base_flags.setdefault(key, values[0])
    search_space.update(
        {key: value for key, value in parallel_cfg.items() if key != "gpu_count"}
    )

    candidates = build_candidates(
        base_flags=base_flags,
        search_space=search_space,
        tier=tier,
        max_candidates=max_candidates,
    )
    return [
        resolve_parallelism(server_cfg, candidate, parallel_requested)
        for candidate in candidates
    ]
```
**EN:** This block uses `build_server_candidates` to build helper structures or requests. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `build_server_candidates` 来构建辅助结构或请求。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 904-905: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 906-955: build candidates function
```python
def build_candidates(
    base_flags: Dict[str, Any],
    search_space: Dict[str, Sequence[Any]],
    tier: int,
    max_candidates: Optional[int],
) -> List[Dict[str, Any]]:
    base_flags = canonicalize_flags(base_flags)
    search_space = canonicalize_flags(search_space)
    capability = detect_current_cuda_capability()
    items = [(key, as_list(values)) for key, values in search_space.items()]
    if tier == 1:
        items = [(k, v[:2]) for k, v in items[:6]]
    elif tier == 2:
        items = [(k, v[:3]) for k, v in items[:8]]

    candidates = [deepcopy(base_flags)]
    if tier == 1:
        for key, values in items:
            for value in values:
                candidates.append(deepcopy(base_flags) | {key: value})
    elif tier == 2 and items:
        head, tail = items[:3], items[3:]
        for combo in itertools.product(*[values for _, values in head]):
            candidate = deepcopy(base_flags)
            for (key, _), value in zip(head, combo):
                candidate[key] = value
            candidates.append(candidate)
        for key, values in tail:
            for value in values:
                candidates.append(deepcopy(base_flags) | {key: value})
    elif tier == 3 and items:
        for combo in itertools.product(*[values for _, values in items]):
            candidate = deepcopy(base_flags)
            for (key, _), value in zip(items, combo):
                candidate[key] = value
            candidates.append(candidate)

    deduped: List[Dict[str, Any]] = []
    seen = set()
    for candidate in candidates:
        if not is_candidate_supported_on_current_device(candidate, capability):
            continue
        key = json.dumps(candidate, sort_keys=True, ensure_ascii=False)
        if key in seen:
            continue
        seen.add(key)
        deduped.append(candidate)
        if max_candidates is not None and len(deduped) >= max_candidates:
            break
    return deduped
```
**EN:** This block uses `build_candidates` to build helper structures or requests. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `build_candidates` 来构建辅助结构或请求。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 956-957: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 958-975: build qps plan function
```python
def build_qps_plan(
    benchmark_cfg: Dict[str, Any],
) -> Tuple[str, List[float], float, int]:
    qps_cfg = benchmark_cfg.get("qps", benchmark_cfg.get("request_rate"))
    if isinstance(qps_cfg, (int, float)):
        return "fixed", [float(qps_cfg)], 0.0, 0
    if isinstance(qps_cfg, list):
        return "fixed", [float(value) for value in qps_cfg], 0.0, 0
    if isinstance(qps_cfg, dict) and "values" in qps_cfg:
        return "fixed", [float(value) for value in qps_cfg["values"]], 0.0, 0
    if isinstance(qps_cfg, dict) and {"lower", "upper"} <= set(qps_cfg):
        return (
            "search",
            [float(qps_cfg["lower"]), float(qps_cfg["upper"])],
            float(qps_cfg.get("tolerance", 0.1)),
            normalize_binary_search_rounds(qps_cfg.get("max_rounds")),
        )
    raise ValueError("benchmark.qps must be a list or a {lower, upper, tolerance} map.")
```
**EN:** This block uses `build_qps_plan` to build helper structures or requests. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `build_qps_plan` 来构建辅助结构或请求。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 976-977: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 978-995: trial key function
```python
def trial_key(
    stage_name: str,
    candidate_id: int,
    request_rate: float,
    max_concurrency: Optional[int],
    server_flags: Dict[str, Any],
) -> str:
    return json.dumps(
        {
            "stage": stage_name,
            "candidate_id": candidate_id,
            "requested_qps": request_rate,
            "max_concurrency": max_concurrency,
            "server_flags": canonicalize_flags(server_flags),
        },
        sort_keys=True,
        ensure_ascii=False,
    )
```
**EN:** This block uses `trial_key` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `trial_key` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 996-997: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 998-1005: record trial key function
```python
def record_trial_key(record: Dict[str, Any]) -> str:
    return trial_key(
        stage_name=str(record.get("stage", "")),
        candidate_id=int(record.get("candidate_id", 0)),
        request_rate=float(record.get("requested_qps", 0.0)),
        max_concurrency=record.get("max_concurrency"),
        server_flags=record.get("server_flags", {}),
    )
```
**EN:** This block uses `record_trial_key` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `record_trial_key` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1006-1007: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1008-1022: meets sla function
```python
def meets_sla(result: Dict[str, Any], benchmark_cfg: Dict[str, Any]) -> bool:
    sla = benchmark_cfg.get("sla", {})
    max_ttft_ms = sla.get("max_ttft_ms")
    max_tpot_ms = sla.get("max_tpot_ms")
    if (
        max_ttft_ms is not None
        and result.get("mean_ttft_ms", float("inf")) > max_ttft_ms
    ):
        return False
    if (
        max_tpot_ms is not None
        and result.get("mean_tpot_ms", float("inf")) > max_tpot_ms
    ):
        return False
    return True
```
**EN:** This block uses `meets_sla` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `meets_sla` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1023-1024: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1025-1032: result sort key function
```python
def result_sort_key(record: Dict[str, Any]) -> Tuple[Any, ...]:
    return (
        1 if record.get("sla_passed") else 0,
        record.get("requested_qps", 0.0),
        record.get("metrics", {}).get("output_throughput", 0.0),
        -record.get("metrics", {}).get("mean_ttft_ms", float("inf")),
        -record.get("metrics", {}).get("mean_tpot_ms", float("inf")),
    )
```
**EN:** This block uses `result_sort_key` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `result_sort_key` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1033-1034: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1035-1064: launch server function
```python
def launch_server(
    server_cfg: Dict[str, Any], server_flags: Dict[str, Any], log_path: str
) -> subprocess.Popen:
    command_prefix = server_cfg.get("command_prefix")
    if command_prefix is None:
        command = [sys.executable, "-m", "sglang.launch_server"]
    elif isinstance(command_prefix, str):
        command = shlex.split(command_prefix)
    else:
        command = [str(item) for item in command_prefix]

    command.extend(cli_args(server_flags))
    command.extend(str(item) for item in server_cfg.get("extra_args", []))

    env = os.environ.copy()
    env.update({key: str(value) for key, value in server_cfg.get("env", {}).items()})
    log_file = open(log_path, "w", encoding="utf-8")
    try:
        process = subprocess.Popen(
            command,
            stdout=log_file,
            stderr=subprocess.STDOUT,
            env=env,
            start_new_session=True,
        )
    except Exception:
        log_file.close()
        raise
    process._autobench_log_file = log_file  # type: ignore[attr-defined]
    return process
```
**EN:** This block uses `launch_server` to launch a service or execution flow. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `launch_server` 来启动服务或执行流程。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1065-1066: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1067-1081: stop server function
```python
def stop_server(process: Optional[subprocess.Popen]) -> None:
    if process is None:
        return
    try:
        os.killpg(process.pid, signal.SIGTERM)
        process.wait(timeout=20)
    except Exception:
        try:
            os.killpg(process.pid, signal.SIGKILL)
        except Exception:
            pass
    finally:
        log_file = getattr(process, "_autobench_log_file", None)
        if log_file is not None:
            log_file.close()
```
**EN:** This block uses `stop_server` to bridge this module with an external system. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `stop_server` 来将本模块与外部系统桥接起来。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1082-1083: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1084-1143: build bench command function (part 1/2)
```python
def build_bench_command(
    benchmark_cfg: Dict[str, Any],
    dataset_summary: Dict[str, Any],
    backend: str,
    base_url: str,
    dataset_path: str,
    tokenizer_path: str,
    request_rate: float,
    max_concurrency: Optional[int],
    output_file: str,
) -> List[str]:
    command = [
        sys.executable,
        "-m",
        "sglang.bench_serving",
        "--backend",
        backend,
        "--base-url",
        base_url,
        "--dataset-name",
        "autobench",
        "--dataset-path",
        dataset_path,
        "--tokenizer",
        tokenizer_path,
        "--num-prompts",
        str(dataset_summary["num_requests"]),
        "--request-rate",
        str(request_rate),
        "--output-file",
        output_file,
        "--seed",
        str(int(benchmark_cfg.get("seed", 1))),
        "--ready-check-timeout-sec",
        str(int(benchmark_cfg.get("ready_check_timeout_sec", 600))),
    ]
    if benchmark_cfg.get("model"):
        command.extend(["--model", str(benchmark_cfg["model"])])
    if benchmark_cfg.get("served_model_name"):
        command.extend(["--served-model-name", str(benchmark_cfg["served_model_name"])])
    if benchmark_cfg.get("disable_tqdm", True):
        command.append("--disable-tqdm")
    if benchmark_cfg.get("output_details"):
        command.append("--output-details")
    if benchmark_cfg.get("disable_stream"):
        command.append("--disable-stream")
    if benchmark_cfg.get("disable_ignore_eos"):
        command.append("--disable-ignore-eos")
    if benchmark_cfg.get("pd_separated"):
        command.append("--pd-separated")
    if benchmark_cfg.get("flush_cache"):
        command.append("--flush-cache")
    if benchmark_cfg.get("tag"):
        command.extend(["--tag", str(benchmark_cfg["tag"])])
    if max_concurrency is not None:
        command.extend(["--max-concurrency", str(max_concurrency)])
    if benchmark_cfg.get("warmup_requests") is not None:
        command.extend(
            ["--warmup-requests", str(int(benchmark_cfg["warmup_requests"]))]
        )
```
**EN:** This block uses `build_bench_command` to build helper structures or requests. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `build_bench_command` 来构建辅助结构或请求。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 1144-1151: build bench command function (part 2/2)
```python
    if benchmark_cfg.get("extra_request_body") is not None:
        command.extend(
            [
                "--extra-request-body",
                json.dumps(benchmark_cfg["extra_request_body"]),
            ]
        )
    return command
```
**EN:** This block uses `build_bench_command` to build helper structures or requests. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `build_bench_command` 来构建辅助结构或请求。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 1152-1153: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1154-1178: run bench command function
```python
def run_bench_command(
    command: List[str], timeout_sec: Optional[float] = None
) -> Dict[str, Any]:
    try:
        result = subprocess.run(
            command, capture_output=True, text=True, timeout=timeout_sec
        )
    except subprocess.TimeoutExpired as exc:
        raise SearchDeadlineExceeded(
            f"search budget expired while waiting for bench_serving: {exc.cmd}"
        ) from exc
    if result.returncode != 0:
        message = (result.stderr or result.stdout).strip()
        if len(message) > 4000:
            head = message[:2000].rstrip()
            tail = message[-2000:].lstrip()
            message = f"{head}\n...\n{tail}"
        raise RuntimeError(message)

    output_file = command[command.index("--output-file") + 1]
    with open(output_file, "r", encoding="utf-8") as f:
        lines = [line.strip() for line in f if line.strip()]
    if not lines:
        raise RuntimeError("bench_serving produced no JSONL output")
    return json.loads(lines[-1])
```
**EN:** This block uses `run_bench_command` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `run_bench_command` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1179-1180: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1181-1240: run trial function (part 1/2)
```python
def run_trial(
    stage_name: str,
    candidate_id: int,
    server_cfg: Dict[str, Any],
    benchmark_cfg: Dict[str, Any],
    dataset_summary: Dict[str, Any],
    backend: str,
    dataset_path: str,
    tokenizer_path: str,
    server_flags: Dict[str, Any],
    output_dir: str,
    request_rate: float,
    max_concurrency: Optional[int],
    search_deadline: Optional[float] = None,
    search_budget_hours: float = DEFAULT_SEARCH_DURATION_HOURS,
) -> Dict[str, Any]:
    process = None
    log_path = os.path.join(
        output_dir,
        f"server_{stage_name}_cand{candidate_id}_mc{max_concurrency}_q{request_rate}.log",
    )
    bench_path = os.path.join(
        output_dir,
        f"bench_{stage_name}_cand{candidate_id}_mc{max_concurrency}_q{request_rate}.jsonl",
    )
    host = server_cfg.get("host", "127.0.0.1")
    port = int(server_flags.get("port", server_cfg.get("port", 30000)))
    base_url = benchmark_cfg.get("base_url", f"http://{host}:{port}")
    record = {
        "stage": stage_name,
        "candidate_id": candidate_id,
        "requested_qps": request_rate,
        "max_concurrency": max_concurrency,
        "server_flags": deepcopy(server_flags),
        "sla_passed": False,
    }

    try:
        raise_if_search_deadline_reached(search_deadline, search_budget_hours)
        if server_cfg.get("launch", True):
            preclean_stale_server(port)
            process = launch_server(server_cfg, server_flags, log_path)
        metrics = run_bench_command(
            build_bench_command(
                benchmark_cfg=benchmark_cfg,
                dataset_summary=dataset_summary,
                backend=backend,
                base_url=base_url,
                dataset_path=dataset_path,
                tokenizer_path=tokenizer_path,
                request_rate=request_rate,
                max_concurrency=max_concurrency,
                output_file=bench_path,
            ),
            timeout_sec=remaining_search_seconds(search_deadline),
        )
        record["sla_passed"] = meets_sla(metrics, benchmark_cfg)
        record["metrics"] = metrics
    except SearchDeadlineExceeded:
        raise
```
**EN:** This block uses `run_trial` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `run_trial` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 1241-1252: run trial function (part 2/2)
```python
    except Exception as exc:  # noqa: BLE001
        record["error"] = repr(exc)
        diagnosis, hint = classify_failure(
            "\n".join(part for part in [repr(exc), tail_text(log_path)] if part)
        )
        if diagnosis:
            record["diagnosis"] = diagnosis
        if hint:
            record["hint"] = hint
    finally:
        stop_server(process)
    return record
```
**EN:** This block uses `run_trial` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `run_trial` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 1253-1254: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1255-1263: merge host port function
```python
def merge_host_port(
    server_cfg: Dict[str, Any], flags: Dict[str, Any]
) -> Dict[str, Any]:
    merged = canonicalize_flags(deepcopy(flags))
    if server_cfg.get("host") is not None and "host" not in merged:
        merged["host"] = server_cfg["host"]
    if server_cfg.get("port") is not None and "port" not in merged:
        merged["port"] = server_cfg["port"]
    return merged
```
**EN:** This block uses `merge_host_port` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `merge_host_port` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1264-1265: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1266-1325: run candidate function (part 1/3)
```python
def run_candidate(
    stage_name: str,
    candidate_id: int,
    server_cfg: Dict[str, Any],
    benchmark_cfg: Dict[str, Any],
    dataset_summary: Dict[str, Any],
    backend: str,
    dataset_path: str,
    tokenizer_path: str,
    server_flags: Dict[str, Any],
    output_dir: str,
    incumbent_record: Optional[Dict[str, Any]] = None,
    progress_callback: Optional[Callable[[Dict[str, Any]], None]] = None,
    record_callback: Optional[Callable[[Dict[str, Any]], None]] = None,
    existing_records: Optional[Sequence[Dict[str, Any]]] = None,
    search_deadline: Optional[float] = None,
    search_budget_hours: float = DEFAULT_SEARCH_DURATION_HOURS,
) -> List[Dict[str, Any]]:
    mode, values, tolerance, max_rounds = build_qps_plan(benchmark_cfg)
    max_concurrency_values = as_list(benchmark_cfg.get("max_concurrency", [None]))
    records: List[Dict[str, Any]] = []
    existing_by_key = {
        record_trial_key(record): deepcopy(record)
        for record in (existing_records or [])
    }

    def one_trial(
        request_rate: float, max_concurrency: Optional[int]
    ) -> Tuple[Dict[str, Any], bool]:
        key = trial_key(
            stage_name=stage_name,
            candidate_id=candidate_id,
            request_rate=request_rate,
            max_concurrency=max_concurrency,
            server_flags=server_flags,
        )
        if key in existing_by_key:
            return deepcopy(existing_by_key[key]), True
        return (
            run_trial(
                stage_name=stage_name,
                candidate_id=candidate_id,
                server_cfg=server_cfg,
                benchmark_cfg=benchmark_cfg,
                dataset_summary=dataset_summary,
                backend=backend,
                dataset_path=dataset_path,
                tokenizer_path=tokenizer_path,
                server_flags=server_flags,
                output_dir=output_dir,
                request_rate=request_rate,
                max_concurrency=max_concurrency,
                search_deadline=search_deadline,
                search_budget_hours=search_budget_hours,
            ),
            False,
        )

    for max_concurrency in max_concurrency_values:
        raise_if_search_deadline_reached(search_deadline, search_budget_hours)
```
**EN:** This block uses `run_candidate` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `run_candidate` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 1326-1385: run candidate function (part 2/3)
```python
        if mode == "fixed":
            incumbent_qps = None
            if (
                incumbent_record
                and incumbent_record.get("metrics")
                and incumbent_record.get("sla_passed")
            ):
                incumbent_qps = float(incumbent_record.get("requested_qps", 0.0))
            for qps in values:
                if incumbent_qps is not None and qps < incumbent_qps:
                    continue
                record, reused = one_trial(qps, max_concurrency)
                records.append(record)
                if record_callback is not None and not reused:
                    record_callback(record)
                if progress_callback is not None:
                    progress_callback(record)
            continue

        lower, upper = values
        best: Optional[Dict[str, Any]] = None
        incumbent_qps = None
        if (
            incumbent_record
            and incumbent_record.get("metrics")
            and incumbent_record.get("sla_passed")
        ):
            incumbent_qps = float(incumbent_record.get("requested_qps", 0.0))
        if incumbent_qps is not None and lower < incumbent_qps <= upper:
            probe_record, reused = one_trial(incumbent_qps, max_concurrency)
            records.append(probe_record)
            if record_callback is not None and not reused:
                record_callback(probe_record)
            if progress_callback is not None:
                progress_callback(probe_record)
            if probe_record.get("metrics") and probe_record["sla_passed"]:
                lower = max(lower, incumbent_qps)
                best = probe_record
            else:
                probe_record["heuristic_pruned"] = True
                probe_record["heuristic_reason"] = (
                    "Failed incumbent probe; skipped lower-QPS search because "
                    "it cannot beat the current best candidate."
                )
                log_line(
                    f"[{stage_name}] heuristic prune candidate={candidate_id} "
                    f"mc={max_concurrency} incumbent_qps={incumbent_qps:.4f}"
                )
                continue
        rounds_run = 0
        while upper - lower > tolerance and rounds_run < max_rounds:
            qps = pick_qps_midpoint(lower, upper)
            if qps <= lower or qps >= upper:
                break
            record, reused = one_trial(qps, max_concurrency)
            records.append(record)
            if record_callback is not None and not reused:
                record_callback(record)
            if progress_callback is not None:
                progress_callback(record)
```
**EN:** This block uses `run_candidate` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk continues the middle-stage logic and data flow of the routine.
**CN:** 该代码块通过 `run_candidate` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分继续展示例程中段的逻辑与数据流。

### Lines 1386-1395: run candidate function (part 3/3)
```python
            if record.get("metrics") and record["sla_passed"]:
                lower = qps
                best = record
            else:
                upper = qps
            rounds_run += 1
        if best is not None:
            best["best_for_candidate"] = True

    return records
```
**EN:** This block uses `run_candidate` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `run_candidate` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 1396-1397: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1398-1401: write jsonl function
```python
def write_jsonl(path: str, records: Iterable[Dict[str, Any]]) -> None:
    if os.path.exists(path):
        os.remove(path)
    append_jsonl(path, records)
```
**EN:** This block uses `write_jsonl` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `write_jsonl` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1402-1403: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1404-1412: write csv function
```python
def write_csv(path: str, records: Sequence[Dict[str, Any]]) -> None:
    if not records:
        return
    rows = [flatten(record) for record in records]
    headers = sorted({header for row in rows for header in row})
    with open(path, "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=headers)
        writer.writeheader()
        writer.writerows(rows)
```
**EN:** This block uses `write_csv` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `write_csv` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1413-1414: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1415-1417: best record function
```python
def best_record(records: Sequence[Dict[str, Any]]) -> Optional[Dict[str, Any]]:
    successful = [record for record in records if record.get("metrics")]
    return max(successful, key=result_sort_key) if successful else None
```
**EN:** This block uses `best_record` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `best_record` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1418-1419: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1420-1450: rendered launch command function
```python
def rendered_launch_command(
    server_cfg: Dict[str, Any], server_flags: Dict[str, Any]
) -> str:
    prefix = server_cfg.get("command_prefix")
    if prefix is None:
        command = ["python", "-m", "sglang.launch_server"]
    elif isinstance(prefix, str):
        command = shlex.split(prefix)
    else:
        command = [str(item) for item in prefix]
    command.extend(cli_args(server_flags))
    command.extend(str(item) for item in server_cfg.get("extra_args", []))

    env_parts = []
    for key, value in sorted(server_cfg.get("env", {}).items()):
        if any(marker in key.upper() for marker in SENSITIVE_ENV_MARKERS):
            continue
        env_parts.append(f"{key}={shlex.quote(str(value))}")
    parts: List[str] = env_parts
    i = 0
    while i < len(command):
        token = str(command[i])
        if token.startswith("--") and i + 1 < len(command):
            nxt = str(command[i + 1])
            if not nxt.startswith("--"):
                parts.append(f"{shlex.quote(token)} {shlex.quote(nxt)}")
                i += 2
                continue
        parts.append(shlex.quote(token))
        i += 1
    return " \\\n  ".join(parts)
```
**EN:** This block uses `rendered_launch_command` to launch a service or execution flow. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `rendered_launch_command` 来启动服务或执行流程。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1451-1452: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1453-1512: write markdown summary function (part 1/2)
```python
def write_markdown_summary(
    path: str,
    scenario: Dict[str, Any],
    dataset_cfg: Dict[str, Any],
    dataset_summary: Dict[str, Any],
    records: Sequence[Dict[str, Any]],
    best: Optional[Dict[str, Any]],
    server_cfg: Dict[str, Any],
    partial_reason: Optional[str] = None,
) -> None:
    lines = [f"# Auto Benchmark Summary: {scenario['display_name']}", ""]
    lines.append(f"- Dataset kind: `{dataset_cfg['kind']}`")
    lines.append(f"- Requests: `{dataset_summary['num_requests']}`")
    if partial_reason:
        lines.append(f"- Status: `partial` ({partial_reason})")
    if dataset_cfg["kind"] == "random":
        lines.append(
            f"- Random distribution: input `{dataset_cfg['random_input_len']}`, output `{dataset_cfg['random_output_len']}`"
        )
    lines.append("")

    if best is not None:
        lines.extend(["## Best Launch Command", "", "```bash"])
        lines.append(rendered_launch_command(server_cfg, best["server_flags"]))
        lines.extend(["```", ""])

    lines.extend(
        [
            "## Results",
            "",
            "| Candidate | Stage | QPS | Max Conc | Prefill | Decode | TP | EP | PP | Output tok/s | TTFT ms | TPOT ms | SLA | Note |",
            "|---|---:|---:|---:|---|---|---:|---:|---:|---:|---:|---:|---|---|",
        ]
    )
    for record in sorted(records, key=result_sort_key, reverse=True):
        flags = record["server_flags"]
        metrics = record.get("metrics", {})
        note = record.get("diagnosis") or record.get("hint") or record.get("error", "")
        note = note.splitlines()[0][:120] if note else ""
        lines.append(
            "| {candidate_id} | {stage} | {qps} | {mc} | {prefill} | {decode} | {tp} | {ep} | {pp} | {throughput} | {ttft} | {tpot} | {sla} | {note} |".format(
                candidate_id=record["candidate_id"],
                stage=record["stage"],
                qps=record["requested_qps"],
                mc=record["max_concurrency"],
                prefill=flags.get("prefill_attention_backend", ""),
                decode=flags.get("decode_attention_backend", ""),
                tp=flags.get("tp_size", 1),
                ep=flags.get("ep_size", ""),
                pp=flags.get("pp_size", 1),
                throughput=(
                    round(metrics.get("output_throughput", 0.0), 2) if metrics else ""
                ),
                ttft=round(metrics.get("mean_ttft_ms", 0.0), 2) if metrics else "",
                tpot=round(metrics.get("mean_tpot_ms", 0.0), 2) if metrics else "",
                sla="pass" if record.get("sla_passed") else "fail",
                note=note.replace("|", "/"),
            )
        )
```
**EN:** This block uses `write_markdown_summary` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `write_markdown_summary` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 1513-1514: write markdown summary function (part 2/2)
```python
    with open(path, "w", encoding="utf-8") as f:
        f.write("\n".join(lines) + "\n")
```
**EN:** This block uses `write_markdown_summary` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `write_markdown_summary` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 1515-1516: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1517-1576: render scenario summary markdown function (part 1/2)
```python
def render_scenario_summary_markdown(
    summary_rows: Sequence[Dict[str, Any]],
    run_partial_reason: Optional[str] = None,
) -> str:
    lines = ["# Scenario Summary", ""]
    if run_partial_reason:
        lines.extend([f"- Status: `partial` ({run_partial_reason})", ""])
    lines.extend(
        [
            "| Scenario | Status | QPS | Output tok/s | TTFT ms | TPOT ms | Summary |",
            "|---|---|---:|---:|---:|---:|---|",
        ]
    )

    for row in summary_rows:
        summary_path = os.path.join(row["scenario_dir"], "summary.md")
        lines.append(
            "| {name} | {status} | {qps} | {throughput} | {ttft} | {tpot} | `{path}` |".format(
                name=row["scenario_name"],
                status=row["status"],
                qps=row.get("requested_qps") or "",
                throughput=(
                    round(row.get("output_throughput", 0.0), 2)
                    if row.get("output_throughput") is not None
                    else ""
                ),
                ttft=(
                    round(row.get("mean_ttft_ms", 0.0), 2)
                    if row.get("mean_ttft_ms") is not None
                    else ""
                ),
                tpot=(
                    round(row.get("mean_tpot_ms", 0.0), 2)
                    if row.get("mean_tpot_ms") is not None
                    else ""
                ),
                path=summary_path,
            )
        )

    for row in summary_rows:
        if row.get("launch_command"):
            lines.extend(
                [
                    "",
                    f"## {row['scenario_name']}",
                    "",
                    "```bash",
                    row["launch_command"],
                    "```",
                ]
            )
        elif row["status"] == "no_successful_runs":
            lines.extend(
                [
                    "",
                    f"## {row['scenario_name']}",
                    "",
                    "No successful run with metrics was produced for this scenario.",
                ]
```
**EN:** This block uses `render_scenario_summary_markdown` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `render_scenario_summary_markdown` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 1577-1579: render scenario summary markdown function (part 2/2)
```python
            )

    return "\n".join(lines) + "\n"
```
**EN:** This block uses `render_scenario_summary_markdown` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `render_scenario_summary_markdown` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 1580-1581: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1582-1641: run stage function (part 1/2)
```python
def run_stage(
    scenario_name: str,
    stage_name: str,
    candidates: Sequence[Dict[str, Any]],
    server_cfg: Dict[str, Any],
    benchmark_cfg: Dict[str, Any],
    dataset_summary: Dict[str, Any],
    backend: str,
    dataset_path: str,
    tokenizer_path: str,
    output_dir: str,
    live_results_path: Optional[str] = None,
    existing_records: Optional[Sequence[Dict[str, Any]]] = None,
    search_deadline: Optional[float] = None,
    search_budget_hours: float = DEFAULT_SEARCH_DURATION_HOURS,
) -> Tuple[List[Dict[str, Any]], Optional[Dict[str, Any]]]:
    records: List[Dict[str, Any]] = []
    existing_stage_records = [
        deepcopy(record)
        for record in (existing_records or [])
        if record.get("stage") == stage_name
    ]
    current_best: Optional[Dict[str, Any]] = best_record(existing_stage_records)
    stage_label = f"{scenario_name} {stage_name}"
    candidate_pbar, candidate_started_at = make_progress_bar(
        desc=f"{stage_label} candidates",
        total=len(candidates),
        position=1,
        leave=True,
    )
    trial_pbar, trial_started_at = make_progress_bar(
        desc=f"{stage_label} trials",
        total=len(candidates) * estimate_trials_per_candidate(benchmark_cfg),
        position=2,
        leave=False,
    )
    try:
        for candidate_id, candidate_flags in enumerate(candidates):
            raise_if_search_deadline_reached(search_deadline, search_budget_hours)
            merged = merge_host_port(server_cfg, candidate_flags)
            log_line(
                f"[{stage_name}] scenario={scenario_name} "
                f"candidate {candidate_id + 1}/{len(candidates)}: "
                f"{json.dumps(merged, ensure_ascii=False)}"
            )

            def on_trial(record: Dict[str, Any]) -> None:
                nonlocal current_best
                if record.get("metrics") and (
                    current_best is None
                    or result_sort_key(record) > result_sort_key(current_best)
                ):
                    current_best = record
                advance_progress(trial_pbar, trial_started_at, best_record=current_best)
                refresh_progress_eta(
                    candidate_pbar, candidate_started_at, best_record=current_best
                )

            def on_record(record: Dict[str, Any]) -> None:
                if live_results_path is not None:
```
**EN:** This block uses `run_stage` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `run_stage` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 1642-1676: run stage function (part 2/2)
```python
                    append_jsonl(live_results_path, [record])

            candidate_records = run_candidate(
                stage_name=stage_name,
                candidate_id=candidate_id,
                server_cfg=server_cfg,
                benchmark_cfg=benchmark_cfg,
                dataset_summary=dataset_summary,
                backend=backend,
                dataset_path=dataset_path,
                tokenizer_path=tokenizer_path,
                server_flags=merged,
                output_dir=output_dir,
                incumbent_record=current_best,
                progress_callback=on_trial,
                record_callback=on_record,
                existing_records=existing_stage_records,
                search_deadline=search_deadline,
                search_budget_hours=search_budget_hours,
            )
            records.extend(candidate_records)

            advance_progress(
                candidate_pbar,
                candidate_started_at,
                best_record=current_best,
            )
    finally:
        if trial_pbar.total is not None and trial_pbar.n < trial_pbar.total:
            trial_pbar.total = trial_pbar.n
            refresh_progress_eta(trial_pbar, trial_started_at, current_best)
        candidate_pbar.close()
        trial_pbar.close()

    return records, current_best
```
**EN:** This block uses `run_stage` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `run_stage` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 1677-1678: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1679-1707: persist scenario outputs function
```python
def persist_scenario_outputs(
    scenario_output_dir: str,
    scenario: Dict[str, Any],
    scenario_cfg: Dict[str, Any],
    dataset_summary: Dict[str, Any],
    records: Sequence[Dict[str, Any]],
    server_cfg: Dict[str, Any],
    partial_reason: Optional[str] = None,
) -> Optional[Dict[str, Any]]:
    if not records:
        return None
    results_jsonl = os.path.join(scenario_output_dir, "results.jsonl")
    results_csv = os.path.join(scenario_output_dir, "results.csv")
    best = best_record(records)
    write_jsonl(results_jsonl, records)
    write_csv(results_csv, records)
    write_markdown_summary(
        path=os.path.join(scenario_output_dir, "summary.md"),
        scenario=scenario,
        dataset_cfg=scenario_cfg,
        dataset_summary=dataset_summary,
        records=records,
        best=best,
        server_cfg=server_cfg,
        partial_reason=partial_reason,
    )
    log_line(f"results_jsonl={results_jsonl}")
    log_line(f"results_csv={results_csv}")
    return best
```
**EN:** This block uses `persist_scenario_outputs` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `persist_scenario_outputs` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1708-1709: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1710-1769: run auto benchmark function (part 1/5)
```python
def run_auto_benchmark(config_path: str) -> str:
    config = load_yaml(config_path)
    server_cfg = config["server"]
    benchmark_cfg = config["benchmark"]
    search_cfg = config.get("search", {})

    timestamp = time.strftime("%Y%m%d-%H%M%S")
    output_dir = benchmark_cfg.get("output_dir") or os.path.join(
        os.getcwd(), "auto_benchmark_results", timestamp
    )
    os.makedirs(output_dir, exist_ok=True)

    tokenizer_path = benchmark_cfg.get("tokenizer") or server_cfg.get(
        "base_flags", {}
    ).get("model_path")
    model = benchmark_cfg.get("model") or server_cfg.get("base_flags", {}).get(
        "model_path"
    )
    if tokenizer_path is None:
        raise ValueError(
            "benchmark.tokenizer or server.base_flags.model_path is required."
        )

    dataset_cfg = normalize_dataset_cfg(config.get("dataset"), benchmark_cfg)
    scenarios = expand_dataset_scenarios(dataset_cfg)
    tier = int(search_cfg.get("tier", 2))
    max_candidates = resolve_max_candidates(search_cfg)
    resume_enabled = bool(search_cfg.get("resume", True))
    base_candidates = build_server_candidates(server_cfg, tier, max_candidates)
    search_budget_hours = resolve_search_budget_hours(search_cfg)
    search_deadline = time.time() + (search_budget_hours * 3600)
    scenario_records: List[Dict[str, Any]] = []
    interrupted = False
    run_partial_reason: Optional[str] = None
    print_run_plan(
        config_path=config_path,
        output_dir=output_dir,
        tier=tier,
        max_candidates=max_candidates,
        benchmark_cfg=benchmark_cfg,
        scenarios=scenarios,
        server_cfg=server_cfg,
        base_candidates=base_candidates,
        speculative_enabled=bool(config.get("speculative", {}).get("enabled")),
        search_budget_hours=search_budget_hours,
        search_deadline=search_deadline,
    )

    scenario_pbar, scenario_started_at = make_progress_bar(
        desc="scenarios",
        total=len(scenarios),
        position=0,
        leave=True,
    )
    previous_handlers = install_interrupt_handlers()
    try:
        for scenario in scenarios:
            raise_if_search_deadline_reached(search_deadline, search_budget_hours)
            scenario_output_dir = (
                output_dir
```
**EN:** This block uses `run_auto_benchmark` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `run_auto_benchmark` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 1770-1829: run auto benchmark function (part 2/5)
```python
                if len(scenarios) == 1
                else os.path.join(output_dir, scenario["name"])
            )
            os.makedirs(scenario_output_dir, exist_ok=True)
            live_results_path = os.path.join(scenario_output_dir, "live_results.jsonl")
            if os.path.exists(live_results_path) and not resume_enabled:
                os.remove(live_results_path)
            prepared_dataset_path = os.path.join(
                scenario_output_dir, "prepared_dataset.jsonl"
            )
            existing_records = read_jsonl(live_results_path)
            if resume_enabled and os.path.exists(prepared_dataset_path):
                rows = load_autobench_rows(
                    dataset_path=prepared_dataset_path,
                    tokenizer_path=tokenizer_path,
                    num_prompts=0,
                )
                dataset_summary = summarize_rows(rows)
            else:
                prepared_dataset_path, rows, dataset_summary = prepare_dataset(
                    dataset_cfg=scenario["cfg"],
                    tokenizer_path=tokenizer_path,
                    model=model,
                    output_path=prepared_dataset_path,
                )

            backend = infer_backend(benchmark_cfg.get("backend", "auto"), rows)
            log_line(f"scenario={scenario['display_name']}")
            log_line(f"prepared_dataset={prepared_dataset_path}")
            log_line(
                f"dataset_summary={json.dumps(dataset_summary, ensure_ascii=False)}"
            )
            log_line(f"selected_backend={backend}")
            if resume_enabled and existing_records:
                log_line(
                    f"resume=true loaded_records={len(existing_records)} "
                    f"scenario={scenario['display_name']}"
                )

            all_records: List[Dict[str, Any]] = []
            scenario_partial_reason: Optional[str] = None
            try:
                all_records, best_base = run_stage(
                    scenario_name=scenario["display_name"],
                    stage_name="base",
                    candidates=base_candidates,
                    server_cfg=server_cfg,
                    benchmark_cfg=benchmark_cfg,
                    dataset_summary=dataset_summary,
                    backend=backend,
                    dataset_path=prepared_dataset_path,
                    tokenizer_path=tokenizer_path,
                    output_dir=scenario_output_dir,
                    live_results_path=live_results_path,
                    existing_records=existing_records,
                    search_deadline=search_deadline,
                    search_budget_hours=search_budget_hours,
                )

                speculative_cfg = config.get("speculative", {})
```
**EN:** This block uses `run_auto_benchmark` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk continues the middle-stage logic and data flow of the routine.
**CN:** 该代码块通过 `run_auto_benchmark` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分继续展示例程中段的逻辑与数据流。

### Lines 1830-1889: run auto benchmark function (part 3/5)
```python
                if speculative_cfg.get("enabled"):
                    if best_base is None:
                        raise ValueError(
                            "Speculative search requires at least one successful base run."
                        )
                    if not speculative_cfg.get("draft_model_path"):
                        raise ValueError("speculative.draft_model_path is required.")

                    spec_base_flags = deepcopy(best_base["server_flags"])
                    spec_base_flags.update(
                        deepcopy(speculative_cfg.get("base_flags", {}))
                    )
                    spec_base_flags["speculative_algorithm"] = speculative_cfg.get(
                        "algorithm", "EAGLE"
                    )
                    spec_base_flags["speculative_draft_model_path"] = speculative_cfg[
                        "draft_model_path"
                    ]
                    spec_candidates = build_candidates(
                        base_flags=canonicalize_flags(spec_base_flags),
                        search_space=deepcopy(speculative_cfg.get("search_space", {})),
                        tier=tier,
                        max_candidates=max_candidates,
                    )
                    log_line(
                        f"Planned speculative candidates for scenario={scenario['display_name']}:"
                    )
                    for index, candidate in enumerate(spec_candidates, start=1):
                        log_line(
                            f"  [{index}/{len(spec_candidates)}] "
                            f"{json.dumps(merge_host_port(server_cfg, candidate), ensure_ascii=False)}"
                        )
                    spec_records, _ = run_stage(
                        scenario_name=scenario["display_name"],
                        stage_name="speculative",
                        candidates=spec_candidates,
                        server_cfg=server_cfg,
                        benchmark_cfg=benchmark_cfg,
                        dataset_summary=dataset_summary,
                        backend=backend,
                        dataset_path=prepared_dataset_path,
                        tokenizer_path=tokenizer_path,
                        output_dir=scenario_output_dir,
                        live_results_path=live_results_path,
                        existing_records=read_jsonl(live_results_path),
                        search_deadline=search_deadline,
                        search_budget_hours=search_budget_hours,
                    )
                    all_records.extend(spec_records)
            except SearchDeadlineExceeded as exc:
                interrupted = True
                scenario_partial_reason = str(exc)
                run_partial_reason = scenario_partial_reason
                log_line(
                    f"search_deadline_reached=true scenario={scenario['display_name']} "
                    f"detail={scenario_partial_reason}"
                )
            except KeyboardInterrupt:
                interrupted = True
                scenario_partial_reason = "interrupted before the full search completed"
```
**EN:** This block uses `run_auto_benchmark` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk continues the middle-stage logic and data flow of the routine.
**CN:** 该代码块通过 `run_auto_benchmark` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分继续展示例程中段的逻辑与数据流。

### Lines 1890-1949: run auto benchmark function (part 4/5)
```python
                run_partial_reason = scenario_partial_reason
                log_line(
                    f"interrupt_received=true scenario={scenario['display_name']} "
                    "saving partial results before exit"
                )
            finally:
                persisted_records = all_records
                live_records = read_jsonl(live_results_path)
                if len(live_records) > len(persisted_records):
                    persisted_records = live_records
                best = persist_scenario_outputs(
                    scenario_output_dir=scenario_output_dir,
                    scenario=scenario,
                    scenario_cfg=scenario["cfg"],
                    dataset_summary=dataset_summary,
                    records=persisted_records,
                    server_cfg=server_cfg,
                    partial_reason=scenario_partial_reason,
                )
                if persisted_records:
                    scenario_records.append(
                        {
                            "scenario_name": scenario["display_name"],
                            "scenario_dir": scenario_output_dir,
                            "best_record": best,
                            "has_records": True,
                        }
                    )
            if interrupted:
                break
            advance_progress(scenario_pbar, scenario_started_at)
    except SearchDeadlineExceeded as exc:
        interrupted = True
        run_partial_reason = str(exc)
        log_line(f"search_deadline_reached=true detail={run_partial_reason}")
    finally:
        scenario_pbar.close()
        restore_interrupt_handlers(previous_handlers)

    if scenario_records and len(scenarios) > 1:
        summary_rows = []
        for item in scenario_records:
            record = item["best_record"]
            metrics = record.get("metrics", {}) if record else {}
            summary_rows.append(
                {
                    "scenario_name": item["scenario_name"],
                    "scenario_dir": item["scenario_dir"],
                    "status": (
                        "ok"
                        if record and record.get("metrics")
                        else "no_successful_runs"
                    ),
                    "requested_qps": record.get("requested_qps") if record else None,
                    "mean_ttft_ms": metrics.get("mean_ttft_ms"),
                    "mean_tpot_ms": metrics.get("mean_tpot_ms"),
                    "output_throughput": metrics.get("output_throughput"),
                    "launch_command": (
                        rendered_launch_command(server_cfg, record["server_flags"])
                        if record
```
**EN:** This block uses `run_auto_benchmark` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk continues the middle-stage logic and data flow of the routine.
**CN:** 该代码块通过 `run_auto_benchmark` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分继续展示例程中段的逻辑与数据流。

### Lines 1950-1960: run auto benchmark function (part 5/5)
```python
                        else ""
                    ),
                }
            )
        write_jsonl(os.path.join(output_dir, "scenario_summary.jsonl"), summary_rows)
        write_csv(os.path.join(output_dir, "scenario_summary.csv"), summary_rows)
        with open(os.path.join(output_dir, "SUMMARY.md"), "w", encoding="utf-8") as f:
            f.write(render_scenario_summary_markdown(summary_rows, run_partial_reason))
    if interrupted:
        log_line(f"interrupted=true partial_output_dir={output_dir}")
    return output_dir
```
**EN:** This block uses `run_auto_benchmark` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `run_auto_benchmark` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 1961-1962: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1963-1980: convert dataset function
```python
def convert_dataset(args: argparse.Namespace) -> None:
    dataset_cfg = normalize_dataset_cfg(
        {
            key: value
            for key, value in vars(args).items()
            if key not in {"command", "output", "tokenizer", "model"}
        },
        {},
    )
    output_path, rows, summary = prepare_dataset(
        dataset_cfg=dataset_cfg,
        tokenizer_path=args.tokenizer,
        model=args.model,
        output_path=args.output,
    )
    print(f"prepared_dataset={output_path}")
    print(f"rows={len(rows)}")
    print(json.dumps(summary, ensure_ascii=False, indent=2))
```
**EN:** This block uses `convert_dataset` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `convert_dataset` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1981-1982: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1983-1985: validate dataset function
```python
def validate_dataset(args: argparse.Namespace) -> None:
    rows = load_autobench_rows(args.dataset_path, args.tokenizer, num_prompts=0)
    print(json.dumps(summarize_rows(rows), ensure_ascii=False, indent=2))
```
**EN:** This block uses `validate_dataset` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `validate_dataset` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Benchmarking workflows / 基准测试流程
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.benchmark.datasets`
- `sglang.benchmark.datasets.autobench`
- `sglang.benchmark.utils`
### External / 外部
- `signal`
- `torch`
- `tqdm`
- `yaml`
- `argparse` (stdlib)
- `copy` (stdlib)
- `csv` (stdlib)
- `itertools` (stdlib)
- `json` (stdlib)
- `os` (stdlib)
- `shlex` (stdlib)
- `subprocess` (stdlib)
- `sys` (stdlib)
- `time` (stdlib)
- `types` (stdlib)
- `typing` (stdlib)
