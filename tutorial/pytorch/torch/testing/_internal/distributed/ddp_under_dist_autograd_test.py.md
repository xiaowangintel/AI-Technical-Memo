# ddp_under_dist_autograd_test.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/distributed/ddp_under_dist_autograd_test.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for ddp under dist autograd test, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 ddp under dist autograd test 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9
```python
# mypy: allow-untyped-defs

import contextlib
import enum
import logging
import os
import threading
from typing import NamedTuple

```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `contextlib`, `enum`, `logging`, `os`, `threading`, `typing`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`contextlib`, `enum`, `logging`, `os`, `threading`, `typing`。

### Lines 10-23
```python
import torch
import torch.distributed as dist
import torch.distributed.autograd as dist_autograd
import torch.nn as nn
from torch.distributed import rpc
from torch.distributed.nn import RemoteModule
from torch.nn.parallel import DistributedDataParallel
from torch.testing._internal.common_distributed import (
    requires_gloo,
    requires_nccl,
    skip_if_lt_x_gpu,
    skip_if_rocm_multiprocess,
)
from torch.testing._internal.dist_utils import dist_init, INIT_METHOD_TEMPLATE
```
- EN: This block implements local helper logic for ddp under dist autograd test. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 ddp under dist autograd test 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 24-40
```python
from torch.testing._internal.distributed.rpc.rpc_agent_test_fixture import (
    RpcAgentTestFixture,
)


NUM_EM_ROW = 2
D_SPARSE = 3
D_DENSE = 2
D_HID = 3
D_OUT = 1
NUM_TRAINERS = 4
# Trainers + the master + the remote worker
WORLD_SIZE = NUM_TRAINERS + 2
TRAINER_RANKS = list(range(NUM_TRAINERS))
REMOTE_WORKER_RANK = TRAINER_RANKS[-1] + 1
MASTER_RANK = REMOTE_WORKER_RANK + 1

```
- EN: This block implements local helper logic for ddp under dist autograd test. Key symbols: `NUM_EM_ROW`, `D_SPARSE`, `D_DENSE`, `D_HID`, `D_OUT`, `NUM_TRAINERS`, `...`.
- CN: 该代码块实现与 ddp under dist autograd test 相关的局部辅助逻辑。关键符号：`NUM_EM_ROW`, `D_SPARSE`, `D_DENSE`, `D_HID`, `D_OUT`, `NUM_TRAINERS`, `...`。

### Lines 41-49
```python

class DdpMode(enum.Enum):
    # Don't apply DDP
    NONE = enum.auto()
    # Apply DDP to the top level nn.Module
    OUTSIDE = enum.auto()
    # Embed DDP inside the top level nn.Module
    INSIDE = enum.auto()

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `DdpMode`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`DdpMode`。

### Lines 50-64
```python

def init_logger():
    logger = logging.getLogger(__name__)
    level = logging.DEBUG if "debug" in os.environ else logging.INFO
    logger.setLevel(level)
    console = logging.StreamHandler()
    formatter = logging.Formatter(
        "%(asctime)s %(filename)s:%(lineno)s %(levelname)s p:%(processName)s t:%(threadName)s: %(message)s"
    )
    console.setFormatter(formatter)
    console.setLevel(level)
    # add the handlers to the logger
    logger.addHandler(console)
    logger.propagate = False
    return logger
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `init_logger`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`init_logger`。

### Lines 65-76
```python


gLogger = init_logger()


class FeatureSet(NamedTuple):
    """A feature set has 2 types of features"""

    dense_features: torch.Tensor
    sparse_features: torch.LongTensor
    values: torch.Tensor

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `FeatureSet`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`FeatureSet`。

### Lines 77-85
```python

def _call_method(method, rref, *args, **kwargs):
    return method(rref.local_value(), *args, **kwargs)


def _remote_method(method, rref, *args, **kwargs):
    args_tup = tuple([method, rref] + list(args))
    return rpc.rpc_sync(rref.owner(), _call_method, args=args_tup, kwargs=kwargs)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_call_method`, `_remote_method`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_call_method`, `_remote_method`。

### Lines 86-102
```python

def _remote_method_async(method, rref, *args, **kwargs):
    args_tup = tuple([method, rref] + list(args))
    return rpc.rpc_async(rref.owner(), _call_method, args=args_tup, kwargs=kwargs)


class RemoteEM(nn.Module):
    def __init__(self, num_embeddings: int, embedding_dim: int):
        gLogger.info("Initing RemoteEM with %s %s", num_embeddings, embedding_dim)
        super().__init__()
        init_em = [0.5] * embedding_dim
        self.em = nn.EmbeddingBag(
            num_embeddings,
            embedding_dim,
            _weight=torch.tensor([init_em] * num_embeddings),
        )

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `_remote_method_async`, `RemoteEM`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`_remote_method_async`, `RemoteEM`。

### Lines 103-115
```python
    def forward(self, input: torch.Tensor):
        gLogger.debug("Running RemoteEM.forward() on: %s", input)
        return self.em(input, offsets=torch.LongTensor(range(input.shape[0])))


# Return a linear module with predefined parameters.
def getLinear(d_in, d_out):
    l = nn.Linear(d_in, d_out, bias=False)
    w = torch.ones((d_out, d_in))
    w[0][0] = -1
    w.requires_grad_()
    l.weight.data = w
    return l
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `forward`, `getLinear`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`forward`, `getLinear`。

### Lines 116-124
```python


class RemoteNet(nn.Module):
    def __init__(self, d_in: int, d_out: int):
        gLogger.info("Initing RemoteNet with %s %s", d_in, d_out)
        super().__init__()
        self.fc = getLinear(d_in, d_out)
        self.relu = nn.ReLU()

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `RemoteNet`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`RemoteNet`。

### Lines 125-142
```python
    def forward(self, input: torch.Tensor):
        gLogger.debug("Running RemoteNet.forward() on: %s", input)
        return self.relu(self.fc(input))


class HybridModel(nn.Module):
    def __init__(
        self,
        remote_em_rref: rpc.RRef,
        remote_net_rref: rpc.RRef,
        process_group_for_ddp: dist.ProcessGroup = None,
    ):
        super().__init__()
        self.remote_em_rref = remote_em_rref
        self.remote_net_rref = remote_net_rref
        self.fc1 = getLinear(D_DENSE, D_DENSE)
        self.fc2 = getLinear(D_HID, D_OUT)

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `forward`, `HybridModel`, `__init__`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`forward`, `HybridModel`, `__init__`。

### Lines 143-157
```python
        self.non_ddp_params = tuple(self.fc1.parameters()) + tuple(
            self.fc2.parameters()
        )
        self.ddp_params = ()

        if process_group_for_ddp is not None:
            self.non_ddp_params, self.ddp_params = (
                tuple(self.fc1.parameters()),
                tuple(self.fc2.parameters()),
            )
            gLogger.info("Use DDP for the second local net.")
            self.fc2 = DistributedDataParallel(
                self.fc2, check_reduction=True, process_group=process_group_for_ddp
            )

```
- EN: This block checks invariants or expected outcomes; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 158-168
```python
        gLogger.info(
            "HybridModel has %s groups of parameters.", len(list(self.parameters()))
        )

    def forward(self, input: FeatureSet):
        gLogger.debug("Running HybridModel.forward on %s", input)
        sparse = _remote_method(
            RemoteEM.forward, self.remote_em_rref, input.sparse_features
        )
        # The same size of mini batch.
        if sparse.shape[0] != input.dense_features.shape[0]:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `forward`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`forward`。

### Lines 169-177
```python
            raise AssertionError(
                f"Expected sparse.shape[0] == input.dense_features.shape[0], "
                f"got {sparse.shape[0]} != {input.dense_features.shape[0]}"
            )
        dense = self.fc1(input.dense_features)
        x = torch.cat((dense, sparse), 1)
        gLogger.debug("Concatenated feature: %s", x)
        x = _remote_method(RemoteNet.forward, self.remote_net_rref, x)
        return self.fc2(x)
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 178-191
```python


class Trainer:
    def __init__(
        self,
        remote_em_rref: rpc.RRef,
        remote_net_rref: rpc.RRef,
        ddp_mode: DdpMode,
        rank: int,
    ):
        self.rank = rank
        self.trainer_group = (
            dist.new_group(TRAINER_RANKS)
            if ddp_mode in (DdpMode.INSIDE, DdpMode.OUTSIDE)
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `Trainer`, `__init__`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`Trainer`, `__init__`。

### Lines 192-205
```python
            else None
        )
        self.remote_em_rref = remote_em_rref
        self.remote_net_rref = remote_net_rref
        self.hybrid_module = HybridModel(
            self.remote_em_rref,
            self.remote_net_rref,
            self.trainer_group if ddp_mode == DdpMode.INSIDE else None,
        )
        self.ddp_params, self.non_ddp_params = (
            self.hybrid_module.ddp_params,
            self.hybrid_module.non_ddp_params,
        )
        if ddp_mode == DdpMode.OUTSIDE:
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 206-220
```python
            gLogger.info("Wrapping the whole hybrid module into DDP.")
            self.ddp_params += self.non_ddp_params
            self.non_ddp_params = ()
            self.hybrid_module = DistributedDataParallel(
                self.hybrid_module,
                check_reduction=True,
                process_group=self.trainer_group,
            )
        gLogger.info(
            "Succeeded in creating a HybridModel instance with "
            "%s ddp params and %s other local params.",
            len(self.ddp_params),
            len(self.non_ddp_params),
        )

```
- EN: This block checks invariants or expected outcomes. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果。关键符号：无明显局部符号。

### Lines 221-232
```python
    def destroy_pg(self):
        if self.trainer_group:
            dist.destroy_process_group(self.trainer_group)

    def train_batch(
        self,
        mini_batch: FeatureSet,
        trainer_has_less_inputs: bool,
        simulate_uneven_inputs: bool,
    ):
        grads_dict = None

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `destroy_pg`, `train_batch`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`destroy_pg`, `train_batch`。

### Lines 233-245
```python
        if not simulate_uneven_inputs:
            input_batches = [mini_batch]
        else:
            # Split into microbatches, and trim to simulate uneven inputs.
            dense_features = mini_batch.dense_features
            sparse_features = mini_batch.sparse_features
            values = mini_batch.values

            dense_microbatch = torch.split(dense_features, 2)
            sparse_microbatch = torch.split(sparse_features, 2)
            values_microbatch = torch.split(values, 2)
            batches = []
            for d, s, v in zip(
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 246-261
```python
                dense_microbatch, sparse_microbatch, values_microbatch, strict=True
            ):
                feature_set = FeatureSet(dense_features=d, sparse_features=s, values=v)
                batches.append(feature_set)

            if trainer_has_less_inputs:
                input_batches = batches[: len(batches) // 2]
                gLogger.info(
                    "Trainer reduced input patches from %s "
                    "to %s to simulate uneven inputs.",
                    len(batches),
                    len(input_batches),
                )
            else:
                input_batches = batches

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 262-279
```python
        with (
            self.hybrid_module.join()
            if simulate_uneven_inputs
            else contextlib.nullcontext()
        ):
            for b in input_batches:
                with dist_autograd.context() as context_id:
                    output = self.hybrid_module.forward(b)
                    loss = (output * mini_batch.values).sum()
                    dist_autograd.backward(context_id, [loss])
                    grads_dict = dist_autograd.get_gradients(context_id)
                    gLogger.info(
                        "Loss is %s for mini batch: %s. Grads dict has %s entries: %s",
                        loss,
                        mini_batch,
                        len(grads_dict),
                        grads_dict,
                    )
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 280-296
```python
        return (
            tuple(grads_dict[param] for param in self.ddp_params),
            tuple(grads_dict[param] for param in self.non_ddp_params),
        )


def get_training_examples():
    n = 16
    training_examples = FeatureSet(
        dense_features=torch.zeros((n, D_DENSE)),
        sparse_features=torch.zeros(n, dtype=torch.long),
        values=torch.zeros(n),
    )
    idx = 0
    # Every example has another one that has exactly the same features but an
    # opposite value. Therefore, their grads cancel each other in all-reduce.
    for value in (-1, 1):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_training_examples`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_training_examples`。

### Lines 297-306
```python
        for x in (-1.0 * value, 1.0 * value):
            for y in (1.0 * value, -1.0 * value):
                for z in (0, 1):
                    training_examples.dense_features[idx, :] = torch.tensor((x, y))
                    training_examples.sparse_features[idx] = z
                    training_examples.values[idx] = value
                    idx += 1

    # Split the examples among NUM_TRAINERS trainers
    if n % NUM_TRAINERS != 0:
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 307-321
```python
        raise AssertionError(
            f"Expected n % NUM_TRAINERS == 0, got {n} % {NUM_TRAINERS} = {n % NUM_TRAINERS}"
        )
    examples_per_trainer = int(n / NUM_TRAINERS)
    return [
        FeatureSet(
            dense_features=training_examples.dense_features[
                start : start + examples_per_trainer, :
            ],
            sparse_features=training_examples.sparse_features[
                start : start + examples_per_trainer
            ],
            values=training_examples.values[start : start + examples_per_trainer],
        )
        for start in range(0, n, examples_per_trainer)
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 322-332
```python
    ]


shutdown_signal = threading.Condition()


def set_shutdown_signal():
    global shutdown_signal
    with shutdown_signal:
        shutdown_signal.notify()

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `set_shutdown_signal`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`set_shutdown_signal`。

### Lines 333-341
```python

class DdpUnderDistAutogradTest(RpcAgentTestFixture):
    @property
    def world_size(self) -> int:
        return WORLD_SIZE

    def remote_worker_name(self) -> str:
        # The name has to be consistent with that in 'dist_init' decorator.
        return f"worker{REMOTE_WORKER_RANK}"
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `DdpUnderDistAutogradTest`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`DdpUnderDistAutogradTest`。

### Lines 342-355
```python

    def trainer_name(self, rank):
        # The name has to be consistent with that in 'dist_init' decorator.
        return f"worker{rank}"

    def _remote_worker_process(self, ddp_mode):
        gLogger.info("The remote worker is running.")
        dist.init_process_group(
            backend="gloo",
            init_method=INIT_METHOD_TEMPLATE.format(file_name=self.file_name),
            world_size=self.world_size,
            rank=self.rank,
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `trainer_name`, `_remote_worker_process`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`trainer_name`, `_remote_worker_process`。

### Lines 356-365
```python
        if ddp_mode in (DdpMode.INSIDE, DdpMode.OUTSIDE):
            # new_group needs to be called on ranks.
            dist.new_group(TRAINER_RANKS)

        global shutdown_signal
        with shutdown_signal:
            shutdown_signal.wait()
        gLogger.info("Exiting remote worker.")
        dist.destroy_process_group()

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 366-379
```python
    def _trainer_process(self, rank: int):
        gLogger.info("Running the trainer #%s...", rank)
        gLogger.info(
            "Initing trainer process group by trainer #%s with ranks %s",
            rank,
            TRAINER_RANKS,
        )
        dist.init_process_group(
            backend="gloo",
            init_method=INIT_METHOD_TEMPLATE.format(file_name=self.file_name),
            world_size=self.world_size,
            rank=self.rank,
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_trainer_process`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_trainer_process`。

### Lines 380-388
```python
        gLogger.info("Waiting for shutdown signal on trainer #%s...", rank)

        global shutdown_signal
        with shutdown_signal:
            shutdown_signal.wait()
        gLogger.info("Exiting the trainer #%s...", rank)
        dist.destroy_process_group()

    def _master_process(self, ddp_mode: DdpMode, simulate_uneven_inputs: bool):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_master_process`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_master_process`。

### Lines 389-406
```python
        gLogger.info("Running the master process...")
        dist.init_process_group(
            backend="gloo",
            init_method=INIT_METHOD_TEMPLATE.format(file_name=self.file_name),
            world_size=self.world_size,
            rank=self.rank,
        )

        remote_em_rref = rpc.remote(
            self.remote_worker_name(), RemoteEM, args=(NUM_EM_ROW, D_SPARSE)
        )
        remote_net_rref = rpc.remote(
            self.remote_worker_name(), RemoteNet, args=(D_DENSE + D_SPARSE, D_HID)
        )
        gLogger.info("Created remote rrefs on master")
        self.do_test_on_master(
            ddp_mode, simulate_uneven_inputs, remote_em_rref, remote_net_rref
        )
```
- EN: This block bridges to backend-specific execution artifacts. Key symbols: no dominant local symbols.
- CN: 该代码块桥接到特定后端的执行产物。关键符号：无明显局部符号。

### Lines 407-415
```python

    def do_test_on_master(
        self,
        ddp_mode: DdpMode,
        simulate_uneven_inputs: bool,
        remote_em_rref: rpc.RRef,
        remote_net_rref: rpc.RRef,
    ):
        if simulate_uneven_inputs:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `do_test_on_master`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`do_test_on_master`。

### Lines 416-430
```python
            gLogger.info(
                "Running DDP + RPC test with simulating uneven inputs across trainers."
            )

        trainer_rrefs = []
        for rank in TRAINER_RANKS:
            trainer = self.trainer_name(rank)
            trainer_rrefs.append(
                rpc.remote(
                    trainer,
                    Trainer,
                    args=(remote_em_rref, remote_net_rref, ddp_mode, rank),
                )
            )

```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 431-439
```python
        if ddp_mode in (DdpMode.INSIDE, DdpMode.OUTSIDE):
            # new_group needs to be called on ranks.
            dist.new_group(TRAINER_RANKS)

        training_examples = get_training_examples()
        for _ in range(3):
            futures = []
            num_trainers = len(trainer_rrefs)
            for idx, trainer_rref in enumerate(trainer_rrefs):
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 440-453
```python
                # Half the trainers will deplete inputs earlier than the rest.
                trainer_has_less_inputs = (
                    simulate_uneven_inputs and idx < num_trainers // 2
                )
                futures.append(
                    _remote_method_async(
                        Trainer.train_batch,
                        trainer_rref,
                        training_examples[idx],
                        trainer_has_less_inputs,
                        simulate_uneven_inputs,
                    )
                )

```
- EN: This block implements local helper logic for ddp under dist autograd test. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 ddp under dist autograd test 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 454-467
```python
            for future in futures:
                ddp_grads, non_ddp_grads = future.wait()
                # When there are uneven inputs, it is not necessary that grads
                # cancel each other out, since some trainers contribute 0 grad.
                if not simulate_uneven_inputs:
                    for grad in ddp_grads:
                        self.assertEqual(
                            grad,
                            torch.zeros_like(grad),
                            msg=f"The grad for any ddp parameter should be zeros, because "
                            "the training examples' grads cancel each other. Received "
                            f"gradient {grad}",
                        )
                for grad in non_ddp_grads:
```
- EN: This block checks invariants or expected outcomes; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 468-477
```python
                    self.assertNotEqual(
                        grad,
                        torch.zeros_like(grad),
                        msg="The grad for any non-ddp parameter shouldn't be zeros",
                    )

        # Destroy process groups
        for trainer_rref in trainer_rrefs:
            _remote_method_async(Trainer.destroy_pg, trainer_rref).wait()

```
- EN: This block checks invariants or expected outcomes; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 478-486
```python
        # Send shutdown signals.
        for rank in TRAINER_RANKS:
            trainer = self.trainer_name(rank)
            rpc.rpc_sync(trainer, set_shutdown_signal, args=())

        rpc.rpc_sync(self.remote_worker_name(), set_shutdown_signal, args=())

    def _do_test(self, ddp_mode, simulate_uneven_inputs=False):
        if self.rank == MASTER_RANK:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_do_test`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_do_test`。

### Lines 487-495
```python
            self._master_process(ddp_mode, simulate_uneven_inputs)
        elif self.rank == REMOTE_WORKER_RANK:
            self._remote_worker_process(ddp_mode)
        elif self.rank in TRAINER_RANKS:
            self._trainer_process(self.rank)
        else:
            raise RuntimeError(f"Unknown process rank: {self.rank}")

    @requires_gloo()
```
- EN: This block reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 496-504
```python
    @dist_init
    def test_backward_no_ddp(self):
        self._do_test(DdpMode.NONE)

    @requires_gloo()
    @dist_init
    def test_backward_ddp_outside(self):
        self._do_test(DdpMode.OUTSIDE)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `test_backward_no_ddp`, `test_backward_ddp_outside`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`test_backward_no_ddp`, `test_backward_ddp_outside`。

### Lines 505-514
```python
    @requires_gloo()
    @dist_init
    def test_backward_ddp_outside_uneven_inputs(self):
        self._do_test(DdpMode.OUTSIDE, simulate_uneven_inputs=True)

    @requires_gloo()
    @dist_init
    def test_backward_ddp_inside(self):
        self._do_test(DdpMode.INSIDE)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `test_backward_ddp_outside_uneven_inputs`, `test_backward_ddp_inside`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`test_backward_ddp_outside_uneven_inputs`, `test_backward_ddp_inside`。

### Lines 515-524
```python

# Common utils for both CPU and CUDA test suites
class CommonDdpComparisonTest(RpcAgentTestFixture):
    @property
    def world_size(self) -> int:
        return NUM_TRAINERS

    def trainer_name(self, rank):
        # The name has to be consistent with that in 'dist_init' decorator.
        return f"worker{rank}"
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `CommonDdpComparisonTest`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`CommonDdpComparisonTest`。

### Lines 525-542
```python

    @staticmethod
    def get_remote_grads(rref, context_id):
        return dist_autograd.get_gradients(context_id)[rref.local_value().weight]


class DdpComparisonTest(CommonDdpComparisonTest):
    def _run_test_ddp_comparision(self, simulate_uneven_inputs=False):
        gLogger.info("Running trainer rank: %s", self.rank)
        # Each trainer uses a different random seed. Otherwise, they are going
        # to have exactly the same initial model parameters, input, and
        # therefore grads. That means the grads will be the same before and
        # after DDP's all-reduce.
        torch.manual_seed(self.rank)
        dist.init_process_group(
            backend="gloo",
            # Postfix file_name with "pg" since file_name is also used by RPC agent
            init_method=INIT_METHOD_TEMPLATE.format(file_name=f"{self.file_name}_pg"),
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `get_remote_grads`, `DdpComparisonTest`, `_run_test_ddp_comparision`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`get_remote_grads`, `DdpComparisonTest`, `_run_test_ddp_comparision`。

### Lines 543-551
```python
            world_size=self.world_size,
            rank=self.rank,
        )
        net = nn.Linear(2, 3)
        ddp_net = DistributedDataParallel(net)

        # Odd ranks join early if simulate_uneven_inputs.
        num_inputs = 1
        if simulate_uneven_inputs:
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 552-560
```python
            if self.rank % 2 == 0:
                num_inputs += 2
        inputs_list = [torch.rand((3, 2)) for _ in range(num_inputs)]

        if simulate_uneven_inputs:
            gLogger.info(
                "Rank %s training with %s inputs.", self.rank, len(inputs_list)
            )

```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 561-570
```python
        # Use distributed autograd. The gradients will be in RPC context map.
        grads_dict = {}
        with ddp_net.join(simulate_uneven_inputs):
            for i, inputs in enumerate(inputs_list):
                with dist_autograd.context() as context_id:
                    loss = ddp_net(inputs).norm()
                    dist_autograd.backward(context_id, [loss])
                    grads_dict = dist_autograd.get_gradients(context_id)
                gLogger.info("Trainer #%s got grad dict: %s", self.rank, grads_dict)

```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 571-588
```python
                # Use local autograd. The gradients will be in each variable's '.grad'.
                ddp_net.zero_grad()
                loss = ddp_net(inputs).norm()
                loss.backward()

                # The gradients should be the same
                for param in net.parameters():
                    self.assertTrue(
                        param in grads_dict,
                        msg=f"Param {param} is not in dist_auto grad dict {grads_dict} for iteration {i}",
                    )
                    self.assertEqual(
                        grads_dict[param],
                        param.grad,
                        msg=f"The grads for param {param} are different under local "
                        f"and dist autograd: {param.grad} \n---\n {grads_dict[param]} for iteration {i}",
                    )
        dist.destroy_process_group()
```
- EN: This block checks invariants or expected outcomes; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 589-597
```python

    @requires_gloo()
    @dist_init
    def test_ddp_comparison(self):
        self._run_test_ddp_comparision()

    @requires_gloo()
    @dist_init
    def test_ddp_comparison_uneven_inputs(self):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `test_ddp_comparison`, `test_ddp_comparison_uneven_inputs`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`test_ddp_comparison`, `test_ddp_comparison_uneven_inputs`。

### Lines 598-615
```python
        # test with simulating uneven inputs in DDP
        self._run_test_ddp_comparision(simulate_uneven_inputs=True)

    @requires_gloo()
    @dist_init
    def test_ddp_dist_autograd_sparse_grads(self):
        # Each trainer uses a different random seed. Otherwise, they are going
        # to have exactly the same initial model parameters, input, and
        # therefore grads. That means the grads will be the same before and
        # after DDP's all-reduce.
        torch.manual_seed(self.rank)
        dist.init_process_group(
            backend="gloo",
            init_method=INIT_METHOD_TEMPLATE.format(file_name=self.file_name),
            world_size=self.world_size,
            rank=self.rank,
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `test_ddp_dist_autograd_sparse_grads`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`test_ddp_dist_autograd_sparse_grads`。

### Lines 616-626
```python
        model = nn.EmbeddingBag(10, 3, sparse=True)
        ddp_model = DistributedDataParallel(model)

        # Different inputs for each
        input = torch.LongTensor(10).random_(0, 10)
        offsets = torch.LongTensor([0, 4])

        # Run local.
        loss = ddp_model(input, offsets).sum()
        loss.backward()

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 627-635
```python
        with dist_autograd.context() as context_id:
            loss = ddp_model(input, offsets).sum()
            dist_autograd.backward(context_id, [loss])
            grads_dict = dist_autograd.get_gradients(context_id)
            self.assertEqual(1, len(grads_dict))
            self.assertEqual(model.weight.grad, grads_dict[model.weight])

    @requires_gloo()
    @dist_init
```
- EN: This block checks invariants or expected outcomes. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果。关键符号：无明显局部符号。

### Lines 636-648
```python
    def test_ddp_dist_autograd_local_vs_remote(self):
        # Each trainer uses a different random seed. Otherwise, they are going
        # to have exactly the same initial model parameters, input, and
        # therefore grads. That means the grads will be the same before and
        # after DDP's all-reduce.
        torch.manual_seed(self.rank)
        dist.init_process_group(
            backend="gloo",
            init_method=INIT_METHOD_TEMPLATE.format(file_name=self.file_name),
            world_size=self.world_size,
            rank=self.rank,
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `test_ddp_dist_autograd_local_vs_remote`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`test_ddp_dist_autograd_local_vs_remote`。

### Lines 649-658
```python
        # Use two different remote device input string, w/ and w/o the default
        # device string "cpu", respectively.
        for remote_device in ["worker0/cpu", "worker0"]:
            remote_layer1 = RemoteModule(
                remote_device=remote_device, module_cls=nn.Linear, args=(10, 5, False)
            )
            layer1 = nn.Linear(10, 5, False)
            # Start with the same parameters for remote and local
            layer1.weight = remote_layer1.module_rref.to_here().weight

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 659-676
```python
            # Run local case.
            layer2 = nn.Linear(5, 1)
            inputs = torch.rand((10, 10))
            ddp_model = DistributedDataParallel(layer2)
            loss = ddp_model(layer1(inputs)).sum()
            loss.backward()

            # Run remote case.
            with dist_autograd.context() as context_id:
                loss = ddp_model(remote_layer1(inputs)).sum()
                dist_autograd.backward(context_id, [loss])
                grads_dict = dist_autograd.get_gradients(context_id)
                dist.barrier()
                self.assertEqual(layer2.weight.grad, grads_dict[layer2.weight])
                self.assertEqual(
                    layer1.weight.grad,
                    rpc.rpc_sync(
                        "worker0",
```
- EN: This block checks invariants or expected outcomes. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果。关键符号：无明显局部符号。

### Lines 677-685
```python
                        CommonDdpComparisonTest.get_remote_grads,
                        args=(remote_layer1.module_rref, context_id),
                    ),
                )


class CudaDdpComparisonTest(CommonDdpComparisonTest):
    @skip_if_lt_x_gpu(NUM_TRAINERS)
    @requires_nccl()
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `CudaDdpComparisonTest`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`CudaDdpComparisonTest`。

### Lines 686-700
```python
    @dist_init
    @skip_if_rocm_multiprocess
    def test_ddp_dist_autograd_local_vs_remote_gpu(self):
        # Each trainer uses a different random seed. Otherwise, they are going
        # to have exactly the same initial model parameters, input, and
        # therefore grads. That means the grads will be the same before and
        # after DDP's all-reduce.
        torch.manual_seed(self.rank)
        dist.init_process_group(
            backend="gloo",
            init_method=INIT_METHOD_TEMPLATE.format(file_name=self.file_name),
            world_size=self.world_size,
            rank=self.rank,
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `test_ddp_dist_autograd_local_vs_remote_gpu`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`test_ddp_dist_autograd_local_vs_remote_gpu`。

### Lines 701-710
```python
        remote_layer1 = RemoteModule(
            remote_device="worker0/cpu", module_cls=nn.Linear, args=(10, 7, False)
        )
        layer1 = nn.Linear(10, 7, False)
        # Start with the same parameters for remote and local
        layer1.weight = remote_layer1.module_rref.to_here().weight

        layer2 = nn.Linear(7, 5).cuda(self.rank)
        ddp_layer2 = DistributedDataParallel(layer2, device_ids=[self.rank])

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 711-720
```python
        remote_layer3 = RemoteModule(
            remote_device="worker0/cpu", module_cls=nn.Linear, args=(5, 3, False)
        )
        layer3 = nn.Linear(5, 3, False)
        # Start with the same parameters for remote and local
        layer3.weight = remote_layer3.module_rref.to_here().weight

        layer4 = nn.Linear(3, 1).cuda(self.rank)
        ddp_layer4 = DistributedDataParallel(layer4, device_ids=[self.rank])

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 721-738
```python
        # Run local case.
        inputs = torch.rand((10, 10))
        loss = ddp_layer4(
            layer3(ddp_layer2(layer1(inputs).cuda(self.rank)).cpu()).cuda(self.rank)
        ).sum()
        loss.backward()

        # Run remote case.
        with dist_autograd.context() as context_id:
            loss = ddp_layer4(
                remote_layer3(
                    ddp_layer2(remote_layer1(inputs).cuda(self.rank)).cpu()
                ).cuda(self.rank)
            ).sum()
            dist_autograd.backward(context_id, [loss])
            grads_dict = dist_autograd.get_gradients(context_id)
            dist.barrier()
            self.assertEqual(
```
- EN: This block checks invariants or expected outcomes. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果。关键符号：无明显局部符号。

### Lines 739-755
```python
                layer1.weight.grad,
                rpc.rpc_sync(
                    "worker0",
                    CommonDdpComparisonTest.get_remote_grads,
                    args=(remote_layer1.module_rref, context_id),
                ),
            )
            self.assertEqual(layer2.weight.grad, grads_dict[layer2.weight])
            self.assertEqual(
                layer3.weight.grad,
                rpc.rpc_sync(
                    "worker0",
                    CommonDdpComparisonTest.get_remote_grads,
                    args=(remote_layer3.module_rref, context_id),
                ),
            )
            self.assertEqual(layer4.weight.grad, grads_dict[layer4.weight])
```
- EN: This block checks invariants or expected outcomes. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果。关键符号：无明显局部符号。


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
- Internal imports / 内部导入: `torch`, `torch.distributed`, `torch.distributed.autograd`, `torch.nn`, `torch.distributed.nn`, `torch.nn.parallel`, `torch.testing._internal.common_distributed`, `torch.testing._internal.dist_utils`, `torch.testing._internal.distributed.rpc.rpc_agent_test_fixture`
- External imports / 外部导入: `contextlib`, `enum`, `logging`, `os`, `threading`, `typing`
- Representative symbols / 代表性符号: `NUM_EM_ROW`, `D_SPARSE`, `D_DENSE`, `D_HID`, `D_OUT`, `NUM_TRAINERS`, `WORLD_SIZE`, `TRAINER_RANKS`, `REMOTE_WORKER_RANK`, `MASTER_RANK`, `...`
