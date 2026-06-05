# pynccl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/distributed/device_communicators/pynccl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the distributed runtime layer. It centers on `PyNcclCommunicator`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于分布式运行时层。它围绕 `PyNcclCommunicator` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 8-23: module setup and imports / 模块初始化与导入
```python
import torch
import torch.distributed as dist
from torch.distributed import ProcessGroup, ReduceOp

from sglang.multimodal_gen.runtime.distributed.device_communicators.pynccl_wrapper import (
    NCCLLibrary,
    buffer_type,
    cudaStream_t,
    ncclComm_t,
    ncclDataTypeEnum,
    ncclRedOpTypeEnum,
    ncclUniqueId,
)
from sglang.multimodal_gen.runtime.distributed.utils import StatelessProcessGroup
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import current_stream
```
**EN:** This block establishes the module context and imports `torch`, `torch.distributed`, `sglang.multimodal_gen.runtime.distributed.device_communicators.pynccl_wrapper`, `sglang.multimodal_gen.runtime.distributed.utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`, and `sglang.multimodal_gen.utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `torch`、`torch.distributed`、`sglang.multimodal_gen.runtime.distributed.device_communicators.pynccl_wrapper`、`sglang.multimodal_gen.runtime.distributed.utils`、`sglang.multimodal_gen.runtime.utils.logging_utils` 和 `sglang.multimodal_gen.utils`。这些依赖为后续实现提供所需符号。

### Lines 25-25: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 28-29: `PyNcclCommunicator` class overview / `PyNcclCommunicator` 类概览
```python
class PyNcclCommunicator:
```
**EN:** This block defines class `PyNcclCommunicator`. It encapsulates py nccl communicator behavior.
**CN:** 该代码块定义了类 `PyNcclCommunicator`。 它用于封装 py nccl communicator 相关行为。

### Lines 30-118: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        group: ProcessGroup | StatelessProcessGroup,
        device: int | str | torch.device,
        library_path: str | None = None,
    ):
        """
        Args:
            group: the process group to work on. If None, it will use the
                default process group.
            device: the device to bind the PyNcclCommunicator to. If None,
                it will be bind to f"cuda:{local_rank}".
            library_path: the path to the NCCL library. If None, it will
                use the default library path.
        It is the caller's responsibility to make sure each communicator
        is bind to a unique device.
        """
        if not isinstance(group, StatelessProcessGroup):
            assert dist.is_initialized()
            assert (
                dist.get_backend(group) != dist.Backend.NCCL
            ), "PyNcclCommunicator should be attached to a non-NCCL group."
            # note: this rank is the rank in the group
            self.rank = dist.get_rank(group)
            self.world_size = dist.get_world_size(group)
        else:
            self.rank = group.rank
            self.world_size = group.world_size

        self.group = group

        # if world_size == 1, no need to create communicator
        if self.world_size == 1:
            self.available = False
            self.disabled = True
            return
        try:
            self.nccl = NCCLLibrary(library_path)
        except Exception:
            # disable because of missing NCCL library
            # e.g. in a non-GPU environment
            self.available = False
            self.disabled = True
            return

        self.available = True
        self.disabled = False

        logger.info("sglang-diffusion is using nccl==%s", self.nccl.ncclGetVersion())

        if self.rank == 0:
            # get the unique id from NCCL
            self.unique_id = self.nccl.ncclGetUniqueId()
        else:
            # construct an empty unique id
            self.unique_id = ncclUniqueId()

        if not isinstance(group, StatelessProcessGroup):
            tensor = torch.ByteTensor(list(self.unique_id.internal))
            ranks = dist.get_process_group_ranks(group)
            # arg `src` in `broadcast` is the global rank
            dist.broadcast(tensor, src=ranks[0], group=group)
            byte_list = tensor.tolist()
            for i, byte in enumerate(byte_list):
                self.unique_id.internal[i] = byte
        else:
            self.unique_id = group.broadcast_obj(self.unique_id, src=0)
        if isinstance(device, int):
            device = torch.device(f"cuda:{device}")
        elif isinstance(device, str):
            device = torch.device(device)
        # now `device` is a `torch.device` object
        assert isinstance(device, torch.device)
        self.device = device
        # nccl communicator and stream will use this device
        # `torch.cuda.device` is a context manager that changes the
        # current cuda device to the specified one
        with torch.cuda.device(device):
            self.comm: ncclComm_t = self.nccl.ncclCommInitRank(
                self.world_size, self.unique_id, self.rank
            )

            stream = current_stream()
            # A small all_reduce for warmup.
            data = torch.zeros(1, device=device)
            self.all_reduce(data)
            if stream is not None:
                stream.synchronize()
            del data
```
**EN:** This block defines method `__init__` on `PyNcclCommunicator`. Args: group: the process group to work on. If None, it will use the default process group. Key calls include `logger.info`, `isinstance`, `dist.is_initialized`, `dist.get_rank`, and `dist.get_world_size`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths, uses context-managed resources. Parameters such as `group`, `device`, and `library_path` drive the behavior in this section.
**CN:** 该代码块定义了 `PyNcclCommunicator` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `logger.info`、`isinstance`、`dist.is_initialized`、`dist.get_rank` 和 `dist.get_world_size`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径，使用上下文管理资源。 本段逻辑主要由 `group`、`device` 和 `library_path` 等参数驱动。

### Lines 120-146: `all_reduce` implementation / `all_reduce` 实现
```python
    def all_reduce(
        self, in_tensor: torch.Tensor, op: ReduceOp = ReduceOp.SUM, stream=None
    ) -> torch.Tensor:
        if self.disabled:
            return None
        # nccl communicator created on a specific device
        # will only work on tensors on the same device
        # otherwise it will cause "illegal memory access"
        assert in_tensor.device == self.device, (
            f"this nccl communicator is created to work on {self.device}, "
            f"but the input tensor is on {in_tensor.device}"
        )

        out_tensor = torch.empty_like(in_tensor)

        if stream is None:
            stream = current_stream()
        self.nccl.ncclAllReduce(
            buffer_type(in_tensor.data_ptr()),
            buffer_type(out_tensor.data_ptr()),
            in_tensor.numel(),
            ncclDataTypeEnum.from_torch(in_tensor.dtype),
            ncclRedOpTypeEnum.from_torch(op),
            self.comm,
            cudaStream_t(stream.cuda_stream),
        )
        return out_tensor
```
**EN:** This block defines method `all_reduce` on `PyNcclCommunicator`. It handles all reduce logic. Key calls include `torch.empty_like`, `self.nccl.ncclAllReduce`, `current_stream`, `buffer_type`, and `in_tensor.numel`. The implementation branches on conditions. Parameters such as `in_tensor`, `op`, and `stream` drive the behavior in this section.
**CN:** 该代码块定义了 `PyNcclCommunicator` 的方法 `all_reduce`。 它用于处理 all reduce 相关逻辑。 关键调用包括 `torch.empty_like`、`self.nccl.ncclAllReduce`、`current_stream`、`buffer_type` 和 `in_tensor.numel`。 实现中包含条件分支。 本段逻辑主要由 `in_tensor`、`op` 和 `stream` 等参数驱动。

### Lines 148-169: `all_gather` implementation / `all_gather` 实现
```python
    def all_gather(
        self, output_tensor: torch.Tensor, input_tensor: torch.Tensor, stream=None
    ):
        if self.disabled:
            return
        # nccl communicator created on a specific device
        # will only work on tensors on the same device
        # otherwise it will cause "illegal memory access"
        assert input_tensor.device == self.device, (
            f"this nccl communicator is created to work on {self.device}, "
            f"but the input tensor is on {input_tensor.device}"
        )
        if stream is None:
            stream = current_stream()
        self.nccl.ncclAllGather(
            buffer_type(input_tensor.data_ptr()),
            buffer_type(output_tensor.data_ptr()),
            input_tensor.numel(),
            ncclDataTypeEnum.from_torch(input_tensor.dtype),
            self.comm,
            cudaStream_t(stream.cuda_stream),
        )
```
**EN:** This block defines method `all_gather` on `PyNcclCommunicator`. It handles all gather logic. Key calls include `self.nccl.ncclAllGather`, `current_stream`, `buffer_type`, `input_tensor.numel`, and `ncclDataTypeEnum.from_torch`. The implementation branches on conditions. Parameters such as `output_tensor`, `input_tensor`, and `stream` drive the behavior in this section.
**CN:** 该代码块定义了 `PyNcclCommunicator` 的方法 `all_gather`。 它用于处理 all gather 相关逻辑。 关键调用包括 `self.nccl.ncclAllGather`、`current_stream`、`buffer_type`、`input_tensor.numel` 和 `ncclDataTypeEnum.from_torch`。 实现中包含条件分支。 本段逻辑主要由 `output_tensor`、`input_tensor` 和 `stream` 等参数驱动。

### Lines 171-197: `reduce_scatter` implementation / `reduce_scatter` 实现
```python
    def reduce_scatter(
        self,
        output_tensor: torch.Tensor,
        input_tensor: torch.Tensor,
        op: ReduceOp = ReduceOp.SUM,
        stream=None,
    ):
        if self.disabled:
            return
        # nccl communicator created on a specific device
        # will only work on tensors on the same device
        # otherwise it will cause "illegal memory access"
        assert input_tensor.device == self.device, (
            f"this nccl communicator is created to work on {self.device}, "
            f"but the input tensor is on {input_tensor.device}"
        )
        if stream is None:
            stream = current_stream()
        self.nccl.ncclReduceScatter(
            buffer_type(input_tensor.data_ptr()),
            buffer_type(output_tensor.data_ptr()),
            output_tensor.numel(),
            ncclDataTypeEnum.from_torch(input_tensor.dtype),
            ncclRedOpTypeEnum.from_torch(op),
            self.comm,
            cudaStream_t(stream.cuda_stream),
        )
```
**EN:** This block defines method `reduce_scatter` on `PyNcclCommunicator`. It handles reduce scatter logic. Key calls include `self.nccl.ncclReduceScatter`, `current_stream`, `buffer_type`, `output_tensor.numel`, and `ncclDataTypeEnum.from_torch`. The implementation branches on conditions. Parameters such as `output_tensor`, `input_tensor`, `op`, and `stream` drive the behavior in this section.
**CN:** 该代码块定义了 `PyNcclCommunicator` 的方法 `reduce_scatter`。 它用于处理 reduce scatter 相关逻辑。 关键调用包括 `self.nccl.ncclReduceScatter`、`current_stream`、`buffer_type`、`output_tensor.numel` 和 `ncclDataTypeEnum.from_torch`。 实现中包含条件分支。 本段逻辑主要由 `output_tensor`、`input_tensor`、`op` 和 `stream` 等参数驱动。

### Lines 199-215: `send` implementation / `send` 实现
```python
    def send(self, tensor: torch.Tensor, dst: int, stream=None):
        if self.disabled:
            return
        assert tensor.device == self.device, (
            f"this nccl communicator is created to work on {self.device}, "
            f"but the input tensor is on {tensor.device}"
        )
        if stream is None:
            stream = current_stream()
        self.nccl.ncclSend(
            buffer_type(tensor.data_ptr()),
            tensor.numel(),
            ncclDataTypeEnum.from_torch(tensor.dtype),
            dst,
            self.comm,
            cudaStream_t(stream.cuda_stream),
        )
```
**EN:** This block defines method `send` on `PyNcclCommunicator`. It handles send logic. Key calls include `self.nccl.ncclSend`, `current_stream`, `buffer_type`, `tensor.numel`, and `ncclDataTypeEnum.from_torch`. The implementation branches on conditions. Parameters such as `tensor`, `dst`, and `stream` drive the behavior in this section.
**CN:** 该代码块定义了 `PyNcclCommunicator` 的方法 `send`。 它用于处理 send 相关逻辑。 关键调用包括 `self.nccl.ncclSend`、`current_stream`、`buffer_type`、`tensor.numel` 和 `ncclDataTypeEnum.from_torch`。 实现中包含条件分支。 本段逻辑主要由 `tensor`、`dst` 和 `stream` 等参数驱动。

### Lines 217-233: `recv` implementation / `recv` 实现
```python
    def recv(self, tensor: torch.Tensor, src: int, stream=None):
        if self.disabled:
            return
        assert tensor.device == self.device, (
            f"this nccl communicator is created to work on {self.device}, "
            f"but the input tensor is on {tensor.device}"
        )
        if stream is None:
            stream = current_stream()
        self.nccl.ncclRecv(
            buffer_type(tensor.data_ptr()),
            tensor.numel(),
            ncclDataTypeEnum.from_torch(tensor.dtype),
            src,
            self.comm,
            cudaStream_t(stream.cuda_stream),
        )
```
**EN:** This block defines method `recv` on `PyNcclCommunicator`. It handles recv logic. Key calls include `self.nccl.ncclRecv`, `current_stream`, `buffer_type`, `tensor.numel`, and `ncclDataTypeEnum.from_torch`. The implementation branches on conditions. Parameters such as `tensor`, `src`, and `stream` drive the behavior in this section.
**CN:** 该代码块定义了 `PyNcclCommunicator` 的方法 `recv`。 它用于处理 recv 相关逻辑。 关键调用包括 `self.nccl.ncclRecv`、`current_stream`、`buffer_type`、`tensor.numel` 和 `ncclDataTypeEnum.from_torch`。 实现中包含条件分支。 本段逻辑主要由 `tensor`、`src` 和 `stream` 等参数驱动。

### Lines 235-259: `broadcast` implementation / `broadcast` 实现
```python
    def broadcast(self, tensor: torch.Tensor, src: int, stream=None):
        if self.disabled:
            return
        assert tensor.device == self.device, (
            f"this nccl communicator is created to work on {self.device}, "
            f"but the input tensor is on {tensor.device}"
        )
        if stream is None:
            stream = current_stream()
        if src == self.rank:
            sendbuff = buffer_type(tensor.data_ptr())
            # NCCL requires the sender also to have a receive buffer
            recvbuff = buffer_type(tensor.data_ptr())
        else:
            sendbuff = buffer_type()
            recvbuff = buffer_type(tensor.data_ptr())
        self.nccl.ncclBroadcast(
            sendbuff,
            recvbuff,
            tensor.numel(),
            ncclDataTypeEnum.from_torch(tensor.dtype),
            src,
            self.comm,
            cudaStream_t(stream.cuda_stream),
        )
```
**EN:** This block defines method `broadcast` on `PyNcclCommunicator`. It handles broadcast logic. Key calls include `self.nccl.ncclBroadcast`, `current_stream`, `buffer_type`, `tensor.numel`, and `ncclDataTypeEnum.from_torch`. The implementation branches on conditions. Parameters such as `tensor`, `src`, and `stream` drive the behavior in this section.
**CN:** 该代码块定义了 `PyNcclCommunicator` 的方法 `broadcast`。 它用于处理 broadcast 相关逻辑。 关键调用包括 `self.nccl.ncclBroadcast`、`current_stream`、`buffer_type`、`tensor.numel` 和 `ncclDataTypeEnum.from_torch`。 实现中包含条件分支。 本段逻辑主要由 `tensor`、`src` 和 `stream` 等参数驱动。

## Key Concepts / 关键概念
- `PyNcclCommunicator`: Primary class that encapsulates py nccl communicator behavior. / 核心类，用于封装 py nccl communicator 相关行为。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.distributed.device_communicators.pynccl_wrapper`, `sglang.multimodal_gen.runtime.distributed.utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.utils`

- **Total lines / 总行数**: 259
