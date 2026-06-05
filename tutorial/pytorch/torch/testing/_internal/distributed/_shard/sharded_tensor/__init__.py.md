# __init__.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/distributed/_shard/sharded_tensor/__init__.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for init, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 init 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```python
# mypy: allow-untyped-defs

import sys
from functools import partial, wraps

```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `sys`, `functools`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`sys`, `functools`。

### Lines 6-14
```python
import torch
import torch.distributed as dist
from torch.distributed import rpc
from torch.testing._internal.common_distributed import (
    MultiProcessTestCase,
    TEST_SKIPS,
    tp_transports,
)

```
- EN: This block implements local helper logic for init. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 init 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 15-19
```python

TEST_GPU_NUM = 4


class ShardedTensorTestBase(MultiProcessTestCase):
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `ShardedTensorTestBase`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`ShardedTensorTestBase`。

### Lines 20-24
```python
    @property
    def world_size(self):
        return TEST_GPU_NUM

    def init_pg(self, backend="nccl"):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `world_size`, `init_pg`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`world_size`, `init_pg`。

### Lines 25-34
```python
        if backend not in ["nccl", "gloo", "mpi", "hccl"]:
            raise RuntimeError(f"Backend {backend} not supported!")

        dist.init_process_group(
            backend=backend,
            world_size=self.world_size,
            rank=self.rank,
            init_method=f"file://{self.file_name}",
        )

```
- EN: This block bridges to backend-specific execution artifacts; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块桥接到特定后端的执行产物；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 35-39
```python
        # set device for nccl pg for collectives
        if backend == "nccl":
            torch.cuda.set_device(self.rank)

    def init_rpc(self):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `init_rpc`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`init_rpc`。

### Lines 40-44
```python
        rpc_backend_options = rpc.TensorPipeRpcBackendOptions(
            _transports=tp_transports()
        )
        rpc_backend_options.init_method = f"file://{self.file_name}"
        for rank in range(self.world_size):
```
- EN: This block handles tensor metadata or sample values; bridges to backend-specific execution artifacts; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；桥接到特定后端的执行产物；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 45-54
```python
            rpc_backend_options.set_device_map(
                f"worker{rank}", {rank: self.rank, self.rank: rank}
            )

        rpc.init_rpc(
            name=f"worker{self.rank:d}",
            rank=self.rank,
            world_size=self.world_size,
            rpc_backend_options=rpc_backend_options,
        )
```
- EN: This block handles tensor metadata or sample values; bridges to backend-specific execution artifacts. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；桥接到特定后端的执行产物。关键符号：无明显局部符号。

### Lines 55-60
```python

    def init_comms(self, init_rpc=True, backend="nccl"):
        if init_rpc:
            self.init_rpc()
        self.init_pg(backend=backend)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `init_comms`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`init_comms`。

### Lines 61-65
```python
    def destroy_comms(self, destroy_rpc=True):
        # Wait for all ranks to reach here before starting shutdown.
        dist.barrier()

        if destroy_rpc:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `destroy_comms`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`destroy_comms`。

### Lines 66-72
```python
            rpc.shutdown()
        dist.destroy_process_group()

    def setUp(self) -> None:
        super().setUp()
        self._spawn_processes()

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `setUp`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`setUp`。

### Lines 73-77
```python
    def assert_sharded_tensor_equal(self, st1, st2):
        st1_local_shards = st1.local_shards()
        st2_local_shards = st2.local_shards()
        self.assertEqual(len(st1_local_shards), len(st2_local_shards))
        for i, st1_local_shard in enumerate(st1_local_shards):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `assert_sharded_tensor_equal`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`assert_sharded_tensor_equal`。

### Lines 78-84
```python
            self.assertEqual(st1_local_shard.tensor, st2_local_shards[i].tensor)
            self.assertEqual(st1_local_shard.metadata, st2_local_shards[i].metadata)

        self.assertEqual(st1.metadata(), st2.metadata())
        self.assertEqual(st1.sharding_spec(), st2.sharding_spec())
        self.assertEqual(len(st1.remote_shards()), len(st2.remote_shards()))

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 85-89
```python

# wrapper to initialize comms (processgroup + rpc)
def with_comms(func=None, init_rpc=True, backend="nccl"):
    if func is None:
        return partial(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `with_comms`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`with_comms`。

### Lines 90-94
```python
            with_comms,
            init_rpc=init_rpc,
            backend=backend,
        )

```
- EN: This block bridges to backend-specific execution artifacts. Key symbols: no dominant local symbols.
- CN: 该代码块桥接到特定后端的执行产物。关键符号：无明显局部符号。

### Lines 95-102
```python
    @wraps(func)
    def wrapper(self, *args, **kwargs):
        if backend == "nccl" and torch.cuda.device_count() < self.world_size:
            sys.exit(TEST_SKIPS[f"multi-gpu-{self.world_size}"].exit_code)
        self.init_comms(init_rpc=init_rpc, backend=backend)
        func(self, *args, **kwargs)
        self.destroy_comms(destroy_rpc=init_rpc)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `wrapper`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`wrapper`。

### Lines 103-103
```python
    return wrapper
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Distributed testing — the implementation coordinates multi-process or sharded training checks.
  CN: 分布式测试——该实现会协调多进程或分片训练相关的检查逻辑。
- EN: Backend coverage — the file contains backend-specific behavior or validation paths.
  CN: 后端覆盖——该文件包含特定后端的行为或校验路径。
- EN: Backend lowering — the code bridges higher-level programs to executable backend artifacts.
  CN: 后端降级——代码把更高层程序桥接到可执行的后端产物。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.distributed`, `torch.testing._internal.common_distributed`
- External imports / 外部导入: `sys`, `functools`
- Representative symbols / 代表性符号: `TEST_GPU_NUM`, `ShardedTensorTestBase`, `with_comms`
