# conn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/disaggregation/ascend/conn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements the ascend connection backend for disaggregated serving. It manages handshakes, transfer state, and KV movement for that specific transport or runtime environment. / 该文件实现了解耦式服务中的 ascend 连接后端，负责该特定传输/运行时环境下的握手、传输状态管理以及 KV 数据移动。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Imports and module setup
```python
import concurrent.futures
import logging
from typing import List, Tuple

import numpy as np
import numpy.typing as npt

from sglang.srt.disaggregation.ascend.transfer_engine import AscendTransferEngine
from sglang.srt.disaggregation.common.utils import group_concurrent_contiguous
from sglang.srt.disaggregation.mooncake.conn import (
    MooncakeKVBootstrapServer,
    MooncakeKVManager,
    MooncakeKVReceiver,
    MooncakeKVSender,
)
from sglang.srt.utils.network import get_local_ip_auto

logger = logging.getLogger(__name__)


```
**EN:** This block gathers the imports and module-level setup for ascend backend connection and KV transfer management. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与ascend 后端连接与 KV 传输管理相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 21-21: Class `AscendKVManager` declaration
```python
class AscendKVManager(MooncakeKVManager):
```
**EN:** This block declares the class `AscendKVManager` and establishes its responsibility inside ascend backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `AscendKVManager`.
**CN:** 这一段声明了类 `AscendKVManager`，并说明它在ascend 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `AscendKVManager`。

### Lines 22-29: Method `init_engine`
```python
    def init_engine(self):
        # TransferEngine initialized on ascend.
        local_ip = get_local_ip_auto()
        self.engine = AscendTransferEngine(
            hostname=local_ip,
            npu_id=self.kv_args.gpu_id,
            disaggregation_mode=self.disaggregation_mode,
        )
```
**EN:** This block defines the method `init_engine` on `AscendKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of ascend backend connection and KV transfer management. Definitions introduced here include `init_engine`. Notable operations include `get_local_ip_auto`, `AscendTransferEngine`.
**CN:** 这一段定义了method `init_engine`（属于 `AscendKVManager`），介绍了参数、初始化步骤，以及这部分ascend 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `init_engine`。 值得注意的操作包括 `get_local_ip_auto`、`AscendTransferEngine`。

### Lines 31-42: Method `register_buffer_to_engine`
```python
    def register_buffer_to_engine(self):
        self.engine.batch_register(self.kv_args.kv_data_ptrs, self.kv_args.kv_data_lens)
        # The Ascend backend optimize batch registration for small memory blocks.
        self.engine.batch_register(
            self.kv_args.aux_data_ptrs, self.kv_args.aux_data_lens
        )
        # Batch register state/extra pool data buffers
        for component_ptrs, component_lens in zip(
            self.kv_args.state_data_ptrs or [],
            self.kv_args.state_data_lens or [],
        ):
            self.engine.batch_register(component_ptrs, component_lens)
```
**EN:** This block defines the method `register_buffer_to_engine` on `AscendKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of ascend backend connection and KV transfer management. Definitions introduced here include `register_buffer_to_engine`. Notable operations include `batch_register`.
**CN:** 这一段定义了method `register_buffer_to_engine`（属于 `AscendKVManager`），介绍了参数、初始化步骤，以及这部分ascend 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `register_buffer_to_engine`。 值得注意的操作包括 `batch_register`。

### Lines 44-46: Method `get_mla_kv_ptrs_with_pp` signature and setup
```python
    def get_mla_kv_ptrs_with_pp(
        self, src_kv_ptrs: List[int], dst_kv_ptrs: List[int]
    ) -> Tuple[List[int], List[int], int]:
```
**EN:** This block defines the method `get_mla_kv_ptrs_with_pp` on `AscendKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of ascend backend connection and KV transfer management. Definitions introduced here include `get_mla_kv_ptrs_with_pp`.
**CN:** 这一段定义了method `get_mla_kv_ptrs_with_pp`（属于 `AscendKVManager`），介绍了参数、初始化步骤，以及这部分ascend 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `get_mla_kv_ptrs_with_pp`。

### Lines 47-70: Method `get_mla_kv_ptrs_with_pp` logic (part 1)
```python
        # src_kv_ptrs: k_data, v_data, index_k_data(optional)
        # dst_kv_ptrs: k_data, v_data, index_k_data(optional)
        start_layer = self.kv_args.prefill_start_layer
        kv_buf_groups = getattr(self.kv_args, "kv_buf_groups", 1)
        total_kv_layers = getattr(self.kv_args, "total_kv_layers", 0)
        src_layers = len(src_kv_ptrs) // kv_buf_groups
        # When only speculative-algorithm is enabled for decode
        # the KV has one more layer than prefill.
        # The draft layer needs to be skipped.
        dst_total_layers = (
            min(len(dst_kv_ptrs) // kv_buf_groups, total_kv_layers)
            if total_kv_layers
            else len(dst_kv_ptrs) // kv_buf_groups
        )
        end_layer = start_layer + src_layers
        if src_layers == dst_total_layers:
            sliced_dst_kv_ptrs = dst_kv_ptrs
        else:
            sliced_dst_kv_ptrs = []
            for i in range(kv_buf_groups):
                layer_offset = i * dst_total_layers
                sliced_dst_kv_ptrs.extend(
                    dst_kv_ptrs[layer_offset + start_layer : layer_offset + end_layer]
                )
```
**EN:** This block continues `get_mla_kv_ptrs_with_pp` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding ascend backend connection and KV transfer management workflow. Notable operations include `index_k_data`, `extend`.
**CN:** 这一段延续了 `get_mla_kv_ptrs_with_pp` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的ascend 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `index_k_data`、`extend`。

### Lines 71-72: Method `get_mla_kv_ptrs_with_pp` logic (part 2)
```python
        layers_current_pp_stage = len(src_kv_ptrs)
        return src_kv_ptrs, sliced_dst_kv_ptrs, layers_current_pp_stage
```
**EN:** This block continues `get_mla_kv_ptrs_with_pp` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding ascend backend connection and KV transfer management workflow.
**CN:** 这一段延续了 `get_mla_kv_ptrs_with_pp` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的ascend 后端连接与 KV 传输管理工作流。

### Lines 74-76: Method `send_kvcache` signature and setup
```python
    def send_kvcache(
        self,
        mooncake_session_id: str,
```
**EN:** This block defines the method `send_kvcache` on `AscendKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of ascend backend connection and KV transfer management. Definitions introduced here include `send_kvcache`.
**CN:** 这一段定义了method `send_kvcache`（属于 `AscendKVManager`），介绍了参数、初始化步骤，以及这部分ascend 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `send_kvcache`。

### Lines 77-107: Method `send_kvcache` logic (part 1)
```python
        prefill_kv_indices: npt.NDArray[np.int32],
        dst_kv_ptrs: list[int],
        dst_kv_indices: npt.NDArray[np.int32],
        executor: concurrent.futures.ThreadPoolExecutor,
    ):
        # Group by indices
        prefill_kv_blocks, dst_kv_blocks = group_concurrent_contiguous(
            prefill_kv_indices, dst_kv_indices
        )

        if self.pp_size > 1:
            if self.is_mla_backend:
                src_kv_ptrs, sliced_dst_kv_ptrs, layers_current_pp_stage = (
                    self.get_mla_kv_ptrs_with_pp(self.kv_args.kv_data_ptrs, dst_kv_ptrs)
                )
                layers_params = [
                    (
                        src_kv_ptrs[layer_id],
                        sliced_dst_kv_ptrs[layer_id],
                        self.kv_args.kv_item_lens[layer_id],
                    )
                    for layer_id in range(layers_current_pp_stage)
                ]
            else:
                (
                    src_k_ptrs,
                    src_v_ptrs,
                    dst_k_ptrs,
                    dst_v_ptrs,
                    layers_current_pp_stage,
                ) = self.get_mha_kv_ptrs_with_pp(self.kv_args.kv_data_ptrs, dst_kv_ptrs)
```
**EN:** This block continues `send_kvcache` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding ascend backend connection and KV transfer management workflow. Notable operations include `group_concurrent_contiguous`, `get_mla_kv_ptrs_with_pp`, `get_mha_kv_ptrs_with_pp`.
**CN:** 这一段延续了 `send_kvcache` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的ascend 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `group_concurrent_contiguous`、`get_mla_kv_ptrs_with_pp`、`get_mha_kv_ptrs_with_pp`。

### Lines 108-134: Method `send_kvcache` logic (part 2)
```python

                layers_params = [
                    (
                        src_k_ptrs[layer_id],
                        dst_k_ptrs[layer_id],
                        self.kv_args.kv_item_lens[layer_id],
                    )
                    for layer_id in range(layers_current_pp_stage)
                ] + [
                    (
                        src_v_ptrs[layer_id],
                        dst_v_ptrs[layer_id],
                        self.kv_args.kv_item_lens[layers_current_pp_stage + layer_id],
                    )
                    for layer_id in range(layers_current_pp_stage)
                ]
        else:
            num_layers = len(self.kv_args.kv_data_ptrs)
            layers_params = [
                (
                    self.kv_args.kv_data_ptrs[layer_id],
                    dst_kv_ptrs[layer_id],
                    self.kv_args.kv_item_lens[layer_id],
                )
                for layer_id in range(num_layers)
            ]

```
**EN:** This block continues `send_kvcache` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding ascend backend connection and KV transfer management workflow.
**CN:** 这一段延续了 `send_kvcache` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的ascend 后端连接与 KV 传输管理工作流。

### Lines 135-166: Method `send_kvcache` logic (part 3)
```python
        def set_transfer_blocks(
            src_ptr: int, dst_ptr: int, item_len: int
        ) -> List[Tuple[int, int, int]]:
            transfer_blocks = []
            for prefill_index, decode_index in zip(prefill_kv_blocks, dst_kv_blocks):
                src_addr = src_ptr + int(prefill_index[0]) * item_len
                dst_addr = dst_ptr + int(decode_index[0]) * item_len
                length = item_len * len(prefill_index)
                transfer_blocks.append((src_addr, dst_addr, length))
            return transfer_blocks

        # Worker function for processing a single layer
        def process_layer(src_ptr: int, dst_ptr: int, item_len: int) -> int:
            transfer_blocks = set_transfer_blocks(src_ptr, dst_ptr, item_len)
            return self._transfer_data(mooncake_session_id, transfer_blocks)

        # Worker function for processing all layers in a batch
        def process_layers(layers_params: List[Tuple[int, int, int]]) -> int:
            transfer_blocks = []
            for src_ptr, dst_ptr, item_len in layers_params:
                transfer_blocks.extend(set_transfer_blocks(src_ptr, dst_ptr, item_len))
            return self._transfer_data(mooncake_session_id, transfer_blocks)

        if self.enable_custom_mem_pool:
            futures = [
                executor.submit(
                    process_layer,
                    src_ptr,
                    dst_ptr,
                    item_len,
                )
                for (src_ptr, dst_ptr, item_len) in layers_params
```
**EN:** This block continues `send_kvcache` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding ascend backend connection and KV transfer management workflow. Definitions introduced here include `set_transfer_blocks`, `process_layer`, `process_layers`. Notable operations include `append`, `set_transfer_blocks`, `_transfer_data`, `extend`.
**CN:** 这一段延续了 `send_kvcache` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的ascend 后端连接与 KV 传输管理工作流。 此处引入的定义包括 `set_transfer_blocks`、`process_layer`、`process_layers`。 值得注意的操作包括 `append`、`set_transfer_blocks`、`_transfer_data`、`extend`。

### Lines 167-179: Method `send_kvcache` logic (part 4)
```python
            ]
            for future in concurrent.futures.as_completed(futures):
                status = future.result()
                if status != 0:
                    for f in futures:
                        f.cancel()
                    return status
        else:
            # Combining all layers' params in one batch transfer is more efficient
            # compared to using multiple threads
            return process_layers(layers_params)

        return 0
```
**EN:** This block continues `send_kvcache` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding ascend backend connection and KV transfer management workflow. Notable operations include `as_completed`, `result`, `cancel`, `process_layers`.
**CN:** 这一段延续了 `send_kvcache` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的ascend 后端连接与 KV 传输管理工作流。 值得注意的操作包括 `as_completed`、`result`、`cancel`、`process_layers`。

### Lines 182-182: Class `AscendKVSender` declaration
```python
class AscendKVSender(MooncakeKVSender):
```
**EN:** This block declares the class `AscendKVSender` and establishes its responsibility inside ascend backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `AscendKVSender`.
**CN:** 这一段声明了类 `AscendKVSender`，并说明它在ascend 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `AscendKVSender`。

### Lines 183-183: Supporting state inside `AscendKVSender`
```python
    pass
```
**EN:** This block adds supporting state or helper logic inside `AscendKVSender`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `AscendKVSender` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 186-186: Class `AscendKVReceiver` declaration
```python
class AscendKVReceiver(MooncakeKVReceiver):
```
**EN:** This block declares the class `AscendKVReceiver` and establishes its responsibility inside ascend backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `AscendKVReceiver`.
**CN:** 这一段声明了类 `AscendKVReceiver`，并说明它在ascend 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `AscendKVReceiver`。

### Lines 187-187: Supporting state inside `AscendKVReceiver`
```python
    pass
```
**EN:** This block adds supporting state or helper logic inside `AscendKVReceiver`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `AscendKVReceiver` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 190-190: Class `AscendKVBootstrapServer` declaration
```python
class AscendKVBootstrapServer(MooncakeKVBootstrapServer):
```
**EN:** This block declares the class `AscendKVBootstrapServer` and establishes its responsibility inside ascend backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `AscendKVBootstrapServer`.
**CN:** 这一段声明了类 `AscendKVBootstrapServer`，并说明它在ascend 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `AscendKVBootstrapServer`。

### Lines 191-191: Supporting state inside `AscendKVBootstrapServer`
```python
    pass
```
**EN:** This block adds supporting state or helper logic inside `AscendKVBootstrapServer`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `AscendKVBootstrapServer` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

## Key Concepts / 关键概念
- `AscendKVManager`: Class that encapsulates ascend kvmanager behavior in this module. / `AscendKVManager`：封装与“Ascendkvmanager”相关行为的类。
- `AscendKVSender`: Class that encapsulates ascend kvsender behavior in this module. / `AscendKVSender`：封装与“Ascendkvsender”相关行为的类。
- `AscendKVReceiver`: Class that encapsulates ascend kvreceiver behavior in this module. / `AscendKVReceiver`：封装与“Ascendkvreceiver”相关行为的类。
- `AscendKVBootstrapServer`: Class that encapsulates ascend kvbootstrap server behavior in this module. / `AscendKVBootstrapServer`：封装与“Ascendkvbootstrap服务端”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `concurrent`, `logging`, `typing`
- **External packages / 外部依赖**: `numpy`
- **Internal modules / 内部模块**: `sglang.srt.disaggregation.ascend.transfer_engine`, `sglang.srt.disaggregation.common.utils`, `sglang.srt.disaggregation.mooncake.conn`, `sglang.srt.utils.network`
