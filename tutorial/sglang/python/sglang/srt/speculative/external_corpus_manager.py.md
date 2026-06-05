# external_corpus_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/external_corpus_manager.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Manages external SAM corpora for ngram speculative decoding. / 该文件实现了推测解码基础设施，例如 worker、元数据对象、CUDA Graph 运行器或共享工具函数。它通过协调草稿模型执行与目标模型校验来提升吞吐。

## Line-by-Line Analysis / 逐行分析

### Lines 1-22: Module header, imports, and shared constants
```python
"""Manages external SAM corpora for ngram speculative decoding.

Handles add/remove/list operations and async background loading.
Used by the Scheduler — not a mixin, a standalone manager object.
"""

import logging
import threading
from typing import Callable, Optional, Tuple

from sglang.srt.managers.io_struct import (
    AddExternalCorpusReqInput,
    AddExternalCorpusReqOutput,
    ListExternalCorporaReqInput,
    ListExternalCorporaReqOutput,
    RemoveExternalCorpusReqInput,
    RemoveExternalCorpusReqOutput,
)

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 23-32: ExternalCorpusManager class declaration
```python
class ExternalCorpusManager:
    """Manages external SAM corpus lifecycle for a single scheduler.

    Args:
        draft_worker: the NGRAMWorker instance (must have add_external_corpus,
            remove_external_corpus, list_external_corpora methods).
        send_response: callable(output, recv_req) to send deferred responses
            back to the tokenizer manager.
    """
```
**EN:** This block declares the `ExternalCorpusManager` class, which exists to organize shared runtime state. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `ExternalCorpusManager` 类，其职责是组织共享运行时状态。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 33-39: ExternalCorpusManager initializer
```python
    def __init__(self, draft_worker, send_response: Callable):
        self._worker = draft_worker
        self._send_response = send_response
        self._pending_load: Optional[
            Tuple[AddExternalCorpusReqInput, threading.Thread]
        ] = None
        self._load_result: Optional[AddExternalCorpusReqOutput] = None
```
**EN:** This block initializes the `ExternalCorpusManager` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `ExternalCorpusManager` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 41-54: ExternalCorpusManager.check_pending_load method
```python
    def check_pending_load(self):
        """Poll from the scheduler event loop. Sends response when done."""
        if self._pending_load is None:
            return
        recv_req, thread = self._pending_load
        if thread.is_alive():
            return
        self._pending_load = None
        thread.join()  # formal happens-before for _load_result visibility
        result = self._load_result
        self._load_result = None
        if result.success:
            self._worker.commit_corpus_load(result.corpus_id, result.loaded_token_count)
        self._send_response(result, recv_req)
```
**EN:** This block uses `ExternalCorpusManager.check_pending_load` to load resources or weights. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ExternalCorpusManager.check_pending_load` 来加载资源或权重。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 56-84: ExternalCorpusManager.add method
```python
    def add(
        self, recv_req: AddExternalCorpusReqInput
    ) -> Optional[AddExternalCorpusReqOutput]:
        if self._pending_load is not None:
            return AddExternalCorpusReqOutput(
                success=False,
                message="Another corpus load is already in progress.",
            )

        def _build():
            try:
                loaded = self._worker.add_external_corpus(
                    recv_req.corpus_id, recv_req.token_chunks
                )
                self._load_result = AddExternalCorpusReqOutput(
                    success=True,
                    corpus_id=recv_req.corpus_id,
                    message=f"Loaded corpus '{recv_req.corpus_id}' with {loaded} tokens.",
                    loaded_token_count=loaded,
                )
            except Exception as e:
                self._load_result = AddExternalCorpusReqOutput(
                    success=False, message=str(e)
                )

        thread = threading.Thread(target=_build, daemon=True)
        self._pending_load = (recv_req, thread)
        thread.start()
        return None  # response sent later by check_pending_load
```
**EN:** This block uses `ExternalCorpusManager.add` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ExternalCorpusManager.add` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 85-87: Class-level supporting statements
```python

    # FIXME(kpham-sgl): remove a corpus during a pending load is an undefined behaviour
    # and should be explicitly prevented.
```
**EN:** This block contains supporting statements for the `ExternalCorpusManager` class, such as constants, attributes, registrations, or small helper expressions that shape the surrounding control flow.
**CN:** 该代码块包含`ExternalCorpusManager` 类的配套语句，例如常量、属性、注册逻辑或小型辅助表达式，它们共同塑造了周边控制流程。

### Lines 88-98: ExternalCorpusManager.remove method
```python
    def remove(
        self, recv_req: RemoveExternalCorpusReqInput
    ) -> RemoveExternalCorpusReqOutput:
        try:
            self._worker.remove_external_corpus(recv_req.corpus_id)
            return RemoveExternalCorpusReqOutput(
                success=True,
                message=f"Removed corpus '{recv_req.corpus_id}'.",
            )
        except Exception as e:
            return RemoveExternalCorpusReqOutput(success=False, message=str(e))
```
**EN:** This block uses `ExternalCorpusManager.remove` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ExternalCorpusManager.remove` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 100-110: ExternalCorpusManager.list method
```python
    def list(
        self, recv_req: ListExternalCorporaReqInput
    ) -> ListExternalCorporaReqOutput:
        try:
            token_counts = self._worker.list_external_corpora()
            return ListExternalCorporaReqOutput(
                success=True,
                corpus_token_counts=token_counts,
            )
        except Exception as e:
            return ListExternalCorporaReqOutput(success=False, message=str(e))
```
**EN:** This block uses `ExternalCorpusManager.list` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ExternalCorpusManager.list` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Runtime orchestration / 运行时编排
- Speculative decoding / 推测解码
- Corpus management / 语料管理

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.managers.io_struct`
### External / 外部
- `logging` (stdlib)
- `threading` (stdlib)
- `typing` (stdlib)
