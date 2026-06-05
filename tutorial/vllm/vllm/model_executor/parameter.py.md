# parameter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/parameter.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements vLLM-specific parameter subclasses and helpers that know how to load tensor-parallel, fused, packed, quantized, and shared weights into model layers. / [CN] 实现 vLLM 专用的参数子类与辅助函数，使模型层能够正确加载张量并行、融合、打包、量化以及共享权重。

## Line-by-Line Analysis / 逐行分析

### Module exports and distributed context
```python
from vllm.distributed import (
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
)

__all__ = [
    "BasevLLMParameter",
    "PackedvLLMParameter",
    "PerTensorScaleParameter",
    "ModelWeightParameter",
    "ChannelQuantScaleParameter",
    "GroupQuantScaleParameter",
    "PackedColumnParameter",
    "RowvLLMParameter",
]
```
**EN:** This file depends on tensor-parallel rank/size because parameter loading is topology-aware: each parameter object needs to know which slice of a checkpoint belongs to the local worker. The export list exposes the commonly reused parameter specializations while keeping lower-level helpers private.
**CN:** 该文件依赖张量并行的 rank/size，因为参数加载与并行拓扑直接相关：每个参数对象都必须知道当前 worker 应当从 checkpoint 中加载哪一片。`__all__` 对外暴露常用参数特化类型，同时隐藏较底层的辅助实现。

### `BasevLLMParameter`: store loader and TP metadata
```python
class BasevLLMParameter(Parameter):
    def __new__(cls, data: torch.Tensor | None, **kwargs):
        return super().__new__(cls, data=data, requires_grad=False)

    def __init__(self, data: torch.Tensor, weight_loader: Callable):
        from vllm.platforms import current_platform

        if current_platform.use_sync_weight_loader():
            weight_loader = current_platform.make_synced_weight_loader(weight_loader)

        self._weight_loader = weight_loader
        self.tp_rank = get_tensor_model_parallel_rank()
        self.tp_size = get_tensor_model_parallel_world_size()
```
**EN:** Every vLLM parameter is a non-trainable `torch.nn.Parameter` carrying two extra pieces of state: a loader callback and tensor-parallel metadata. The TPU-specific sync wrapper is a defensive fix against lazy propagation causing duplicated memory during staged weight copies.
**CN:** 每个 vLLM 参数本质上都是不可训练的 `torch.nn.Parameter`，但额外携带两类状态：权重加载回调和张量并行元数据。针对 TPU 的同步包装是一种防御性修复，用来避免分阶段拷贝权重时因惰性传播导致的额外内存占用。

### Loader property and basic copy helpers
```python
@property
def weight_loader(self) -> Callable:
    if self._weight_loader is None:
        raise AttributeError(
            f"{self.__class__.__name__} weight_loader attribute has been deleted"
        )
    return self._weight_loader

@weight_loader.setter
def weight_loader(self, value: Callable):
    self._weight_loader = value

@weight_loader.deleter
def weight_loader(self):
    self._weight_loader = None  # type: ignore[assignment]
```
**EN:** The property intentionally supports replacement and deletion because some models patch weight loading behavior after construction. Treating deletion as an explicit `None` state makes failures clearer than silently keeping a stale callable.
**CN:** 这里故意允许替换和删除 `weight_loader`，因为某些模型会在构造后重写加载逻辑。删除时显式设为 `None`，比悄悄保留旧回调更容易暴露错误。

### Shape validation and default load methods
```python
def _is_1d_and_scalar(self, loaded_weight: torch.Tensor):
    cond1 = self.data.ndim == 1 and self.data.numel() == 1
    cond2 = loaded_weight.ndim == 0 and loaded_weight.numel() == 1
    return cond1 and cond2

def _assert_and_load(self, loaded_weight: torch.Tensor):
    assert self.data.shape == loaded_weight.shape or self._is_1d_and_scalar(
        loaded_weight
    )
    self.data.copy_(loaded_weight)

def load_column_parallel_weight(self, loaded_weight: torch.Tensor):
    self._assert_and_load(loaded_weight)
```
**EN:** The base implementation is “copy after validating shape”, with one compatibility escape hatch: scalar checkpoint values may load into 1-element vectors. Subclasses override the load methods only when they must slice or remap checkpoint tensors.
**CN:** 基础加载逻辑就是“先校验形状，再拷贝”，只有一个兼容例外：checkpoint 中的标量允许加载进长度为 1 的向量。只有在需要切片或重映射 checkpoint 张量时，子类才会覆写这些加载方法。

### Shard-ID normalization
```python
def _shard_id_as_int(self, shard_id: str | int) -> int:
    if isinstance(shard_id, int):
        return shard_id

    qkv_idxs = {"q": 0, "k": 1, "v": 2}
    assert isinstance(shard_id, str)
    assert shard_id in qkv_idxs
    return qkv_idxs[shard_id]
```
**EN:** Fused attention checkpoints sometimes identify shards symbolically (`q`, `k`, `v`) instead of numerically. This helper normalizes both forms so later slicing logic can work uniformly with integer indexes.
**CN:** 融合注意力 checkpoint 有时会用符号形式（`q`、`k`、`v`）标识分片，而不是整数。这个辅助函数把两种写法统一成整数索引，方便后续切片逻辑复用。

### `_ColumnvLLMParameter`: column-parallel slicing
```python
class _ColumnvLLMParameter(BasevLLMParameter):
    def __init__(self, output_dim: int, **kwargs):
        self._output_dim = output_dim
        super().__init__(**kwargs)

    def load_column_parallel_weight(self, loaded_weight: torch.Tensor):
        shard_size = self.data.shape[self.output_dim]
        loaded_weight = loaded_weight.narrow(
            self.output_dim, self.tp_rank * shard_size, shard_size
        )
        assert self.data.shape == loaded_weight.shape
        self.data.copy_(loaded_weight)
```
**EN:** Column-parallel parameters shard along the output dimension, so the loader extracts the local rank’s contiguous slice with `narrow`. The implementation assumes the checkpoint tensor is laid out globally and each worker receives its own output-channel band.
**CN:** 列并行参数沿输出维分片，因此加载器用 `narrow` 取出当前 rank 对应的连续切片。这里假设 checkpoint 中保存的是全局张量，而每个 worker 只加载自己的输出通道区段。

### Loading fused column and QKV tensors
```python
def load_merged_column_weight(self, loaded_weight: torch.Tensor, **kwargs):
    shard_offset: int = kwargs["shard_offset"]
    shard_size: int = kwargs["shard_size"]

    if (
        isinstance(self, (PackedColumnParameter, PackedvLLMParameter))
        and self.packed_dim == self.output_dim
    ):
        shard_size, shard_offset = self.adjust_shard_indexes_for_packing(
            shard_offset=shard_offset, shard_size=shard_size
        )

    param_data = self.data.narrow(self.output_dim, shard_offset, shard_size)
    loaded_weight = loaded_weight.narrow(
        self.output_dim, self.tp_rank * shard_size, shard_size
    )
    param_data.copy_(loaded_weight)
```
**EN:** Fused MLP/QKV weights need two levels of slicing: first pick the logical submatrix inside the parameter (`shard_offset`, `shard_size`), then pick the local tensor-parallel slice from the checkpoint. Packed weights complicate this because packed storage shrinks the logical dimension, so offsets and lengths must be adjusted before slicing.
**CN:** 融合 MLP/QKV 权重需要两层切片：先在参数内部选中逻辑子矩阵（`shard_offset`、`shard_size`），再从 checkpoint 中取出当前张量并行 rank 对应的局部分片。打包权重会压缩逻辑维度，因此必须在切片前先修正偏移和长度。

### QKV-specific shard routing
```python
def load_qkv_weight(self, loaded_weight: torch.Tensor, **kwargs):
    shard_offset: int = kwargs["shard_offset"]
    shard_size: int = kwargs["shard_size"]
    shard_id: str = kwargs["shard_id"]
    num_heads: int = kwargs["num_heads"]

    if (
        isinstance(self, (PackedColumnParameter, PackedvLLMParameter))
        and self.output_dim == self.packed_dim
    ):
        shard_size, shard_offset = self.adjust_shard_indexes_for_packing(
            shard_offset=shard_offset, shard_size=shard_size
        )

    shard_id_int = self.tp_rank if shard_id == "q" else self.tp_rank // num_heads
```
**EN:** QKV routing is asymmetric: `q` often shards per rank, while `k`/`v` can be grouped by KV-head replication, so their source slice may use `tp_rank // num_heads`. This detail captures how multi-query and grouped-query attention distribute K/V weights differently from Q.
**CN:** QKV 的路由并不完全对称：`q` 通常按 rank 直接切分，而 `k`/`v` 可能依据 KV 头共享关系使用 `tp_rank // num_heads` 来定位来源切片。这个细节体现了多查询/分组查询注意力中 K/V 与 Q 不同的分布方式。

### `RowvLLMParameter`: row-parallel slicing
```python
class RowvLLMParameter(BasevLLMParameter):
    def __init__(self, input_dim: int, **kwargs):
        self._input_dim = input_dim
        super().__init__(**kwargs)

    def load_row_parallel_weight(self, loaded_weight: torch.Tensor):
        shard_size = self.data.shape[self.input_dim]
        loaded_weight = loaded_weight.narrow(
            self.input_dim, self.tp_rank * shard_size, shard_size
        )

        if len(loaded_weight.shape) == 0:
            loaded_weight = loaded_weight.reshape(1)

        self.data.copy_(loaded_weight)
```
**EN:** Row-parallel layers shard along the input dimension instead of the output dimension. The extra scalar-to-vector reshape mirrors the base-class compatibility rule and prevents 0-D tensors from breaking copy into 1-element parameter shards.
**CN:** 行并行层沿输入维分片，而不是输出维。这里额外的标量转向量处理与基类兼容规则一致，用来避免 0 维张量复制到单元素参数分片时出错。

### Semantic aliases for different weight roles
```python
class ModelWeightParameter(_ColumnvLLMParameter, RowvLLMParameter):
    pass

class GroupQuantScaleParameter(_ColumnvLLMParameter, RowvLLMParameter):
    pass

class ChannelQuantScaleParameter(_ColumnvLLMParameter):
    pass
```
**EN:** These classes do not add code, but they encode intent. A quantization config or layer implementation can request a parameter type matching its semantics—model weights, grouped scales, or channel scales—even when the actual loading behavior is inherited unchanged.
**CN:** 这些类本身没有新增实现，但它们表达了语义角色。量化配置或层实现可以根据含义选择参数类型——模型权重、分组 scale、通道 scale——即使底层加载行为仍复用父类实现。

### `PerTensorScaleParameter`: fused-scale placement
```python
class PerTensorScaleParameter(BasevLLMParameter):
    def load_merged_column_weight(self, *args, **kwargs):
        self._load_into_shard_id(*args, **kwargs)

    def load_qkv_weight(self, *args, **kwargs):
        self._load_into_shard_id(*args, **kwargs)

    def _load_into_shard_id(
        self, loaded_weight: torch.Tensor, shard_id: str | int, **kwargs
    ):
        param_data = self.data
        shard_id = self._shard_id_as_int(shard_id)

        if len(loaded_weight.shape) != 0:
            assert loaded_weight.shape[0] == 1
            loaded_weight = loaded_weight[0]

        param_data = param_data[shard_id]
        param_data.copy_(loaded_weight)
```
**EN:** Per-tensor quantization scales often map one scalar per logical submatrix instead of one scale per channel. This class therefore indexes into the destination tensor by logical shard ID and writes the scalar or length-1 value into the correct slot.
**CN:** 按张量量化的 scale 往往是“每个逻辑子矩阵一个标量”，而不是“每个通道一个 scale”。因此该类会按逻辑分片 ID 选择目标位置，并把标量或长度为 1 的值写入正确槽位。

### Packed parameters for quantized storage
```python
class PackedColumnParameter(_ColumnvLLMParameter):
    def __init__(
        self,
        packed_factor: int | Fraction,
        packed_dim: int,
        marlin_tile_size: int | None = None,
        **kwargs,
    ):
        self._packed_factor = packed_factor
        self._packed_dim = packed_dim
        self._marlin_tile_size = marlin_tile_size
        super().__init__(**kwargs)

    def adjust_shard_indexes_for_packing(self, shard_size, shard_offset):
        return _adjust_shard_indexes_for_packing(
            shard_size=shard_size,
            shard_offset=shard_offset,
            packed_factor=self.packed_factor,
            marlin_tile_size=self.marlin_tile_size,
        )
```
**EN:** Packed parameters represent compressed checkpoint layouts such as int4 values packed into int32 storage. `packed_factor` tells vLLM how logical matrix extents shrink in storage space, while `marlin_tile_size` handles additional layout constraints imposed by Marlin kernels.
**CN:** 打包参数用于表示压缩后的 checkpoint 布局，例如把 int4 权重打包进 int32 存储。`packed_factor` 告诉 vLLM 逻辑矩阵尺寸在存储层面缩小了多少，而 `marlin_tile_size` 则处理 Marlin kernel 额外要求的分块布局约束。

### `PackedvLLMParameter`: packed full weight tensors
```python
class PackedvLLMParameter(ModelWeightParameter):
    def __init__(
        self,
        packed_factor: int | Fraction,
        packed_dim: int,
        marlin_tile_size: int | None = None,
        **kwargs,
    ):
        self._packed_factor = packed_factor
        self._packed_dim = packed_dim
        self._marlin_tile_size = marlin_tile_size
        super().__init__(**kwargs)
```
**EN:** This is the packed analogue of the main model-weight parameter, combining both row/column-parallel behavior with packing metadata. The actual row/column load routines inherited earlier consult `adjust_shard_indexes_for_packing` when the packed dimension matches the sharded dimension.
**CN:** 这是主模型权重参数的打包版本，把行/列并行行为与打包元数据结合起来。前面继承而来的行/列加载逻辑会在“打包维恰好也是分片维”时调用 `adjust_shard_indexes_for_packing`。

### `SharedWeightParameter`: keep multiple partitions sharing storage
```python
class SharedWeightParameter(BasevLLMParameter):
    tensors_registry: WeakValueDictionary = WeakValueDictionary()

    def __init__(self, input_dim: int = 1, output_dim: int = 0, **kwargs):
        weight_loader: Callable = kwargs.get("weight_loader")
        super().__init__(data=None, weight_loader=weight_loader)

        self.local_tensors = set()
        self.partitions = {}
        self.kwargs = {
            "input_dim": input_dim,
            "output_dim": output_dim,
            "weight_loader": self._fake_weight_loader,
        }
```
**EN:** `SharedWeightParameter` is a container rather than a normal parameter tensor. It exists for cases where several logical partitions across the model must point to the same underlying storage, preserving weight sharing even after transformations such as gate/up splitting.
**CN:** `SharedWeightParameter` 更像一个容器，而不是普通参数张量。它用于那些模型中多个逻辑分区必须共享同一块底层存储的场景，从而在 gate/up 拆分等变换之后仍能保持权重共享关系。

### Creating and reusing shared partitions
```python
def add_partition(self, index: int, data_key: Hashable, *args, **kwargs):
    if data_key not in self.tensors_registry:
        data = torch.empty(*args, **kwargs)
        self.tensors_registry[data_key] = data
    else:
        data = self.tensors_registry[data_key]

    self.partitions[index] = ModelWeightParameter(data=data, **self.kwargs)
    self.local_tensors.add(data)
```
**EN:** `data_key` is the sharing identity. If another partition uses the same key, both `ModelWeightParameter` wrappers point at the same tensor. The extra strong reference in `local_tensors` compensates for weak-reference storage and PyTorch’s imperfect tensor ownership semantics.
**CN:** `data_key` 就是共享身份标识。若另一个分区使用相同 key，那么两个 `ModelWeightParameter` 包装对象会指向同一张量。`local_tensors` 中的强引用则用于弥补弱引用注册表以及 PyTorch 张量所有权语义不够可靠的问题。

### Delegating loads into individual shared partitions
```python
def load_merged_column_weight(self, loaded_weight: torch.Tensor, **kwargs):
    partition_id = kwargs.pop("shard_id")
    partition_id = self._shard_id_as_int(partition_id)
    partition = self.partitions[partition_id]

    input_dim = self.kwargs.get("input_dim")
    shard_size = partition.data.size(input_dim) // self.tp_size
    shard_offset = self.tp_rank * shard_size

    ModelWeightParameter.load_merged_column_weight(
        partition, loaded_weight, shard_offset=shard_offset, shard_size=shard_size
    )
```
**EN:** Instead of owning one tensor, the shared parameter forwards load requests to the appropriate child partition. It reconstructs the shard metadata expected by `ModelWeightParameter`, allowing the existing fused-load logic to be reused without duplicating slicing code.
**CN:** 共享参数并不直接持有单一张量，而是把加载请求转发给对应的子分区。它重新计算 `ModelWeightParameter` 所需的分片元数据，从而复用已有的融合加载逻辑，而无需重复实现切片代码。

### Finalizing shared partitions and blocking unsafe access
```python
def process_weights_after_loading(self):
    for key in self.partitions:
        self.partitions[key] = torch.nn.Parameter(
            data=self.partitions[key].data, requires_grad=False
        )

@property
def data(self):
    raise ValueError(
        "Accessing `data` of a `SharedWeightParameter` is not allowed. "
        "Instead, use `get_partition` to get the weight of "
        "the particular partition you want to access"
    )
```
**EN:** After loading, each partition is downgraded to an ordinary frozen `Parameter`, because the custom loader wrapper is no longer needed. Direct `data` access is forbidden since there is no single canonical tensor; callers must work through partition-level access.
**CN:** 加载完成后，每个分区都会被降级成普通的冻结 `Parameter`，因为自定义加载包装已不再需要。之所以禁止直接访问 `data`，是因为这里并不存在唯一的“主张量”，调用方必须按分区访问。

### `permute_param_layout_`: normalize tensor layout metadata and storage
```python
def permute_param_layout_(
    param: BasevLLMParameter, input_dim: int, output_dim: int, **kwargs
) -> BasevLLMParameter:
    curr_input_dim = getattr(param, "input_dim", None)
    curr_output_dim = getattr(param, "output_dim", None)

    if curr_input_dim is None or curr_output_dim is None:
        assert param.data.dim() == 2
```
**EN:** This helper mutates both the tensor storage order and the parameter’s metadata so later code can assume a known `(input_dim, output_dim)` convention. That is especially important when quantized or packed kernels expect a specific physical layout.
**CN:** 这个辅助函数会同时修改张量的物理维度顺序和参数对象记录的元数据，使后续代码可以假定统一的 `(input_dim, output_dim)` 约定。这在量化或打包 kernel 依赖特定物理布局时尤其重要。

### Permutation construction and packed-dimension safety check
```python
    perm = [
        i for i in range(param.data.dim()) if i not in [curr_input_dim, curr_output_dim]
    ]
    perm.insert(input_dim, curr_input_dim)
    perm.insert(output_dim, curr_output_dim)

    if "packed_dim" in kwargs:
        assert (
            hasattr(param, "packed_dim")
            and param.packed_dim == perm[kwargs["packed_dim"]]
        ), "permute_param_layout_ currently doesn't support repacking"

    param.data = param.data.permute(*perm)
```
**EN:** The function computes a general permutation that preserves all non-input/non-output axes in order. If a packed dimension is specified, it only allows layout changes that keep the same underlying packed axis; repacking compressed data would require a semantic transform, not just a `permute`.
**CN:** 该函数构造一个通用置换：除输入维和输出维外，其余轴保持相对顺序。如果指定了 `packed_dim`，则只允许“打包轴保持不变”的布局调整；因为压缩数据的重新打包不是单纯 `permute` 就能完成的语义变换。

### Packing index helpers
```python
def _adjust_shard_indexes_for_marlin(shard_size, shard_offset, marlin_tile_size):
    return shard_size * marlin_tile_size, shard_offset * marlin_tile_size


def _adjust_shard_indexes_for_packing(
    shard_size, shard_offset, packed_factor, marlin_tile_size
):
    shard_size = round(shard_size // packed_factor)
    shard_offset = round(shard_offset // packed_factor)
    if marlin_tile_size is not None:
        return _adjust_shard_indexes_for_marlin(
            shard_size=shard_size,
            shard_offset=shard_offset,
            marlin_tile_size=marlin_tile_size,
        )

    return shard_size, shard_offset
```
**EN:** These helpers convert logical shard extents into physical extents for packed storage. First divide by the packing ratio, then optionally multiply by Marlin tile size to account for kernel-specific blocked layouts.
**CN:** 这些辅助函数负责把“逻辑分片范围”转换成“打包存储下的物理范围”：先按打包比例缩小，再在需要时乘上 Marlin 的 tile 大小，以匹配该 kernel 的块状布局。

## Key Concepts / 关键概念
- **Tensor-parallel aware parameters**: EN: Each parameter object knows local rank and world size, so loading logic can slice checkpoint tensors correctly. CN: 每个参数对象都知道本地 rank 和 world size，因此能正确切分 checkpoint 张量。
- **Semantic parameter subclasses**: EN: Different subclasses encode how a weight or scale should be loaded, even when their storage type is still `Parameter`. CN: 不同子类通过语义描述“权重或 scale 应如何加载”，即使底层类型仍是 `Parameter`。
- **Packed/quantized layouts**: EN: Packing metadata bridges logical matrix shapes and compressed on-disk storage. CN: 打包元数据连接逻辑矩阵形状与磁盘上的压缩存储布局。
- **Shared tensors**: EN: `SharedWeightParameter` preserves aliasing relationships between logically separate partitions. CN: `SharedWeightParameter` 用于保持逻辑分区之间的底层张量别名关系。
- **Layout normalization**: EN: `permute_param_layout_` lets later kernels assume a canonical axis arrangement. CN: `permute_param_layout_` 使后续 kernel 可以依赖统一的轴布局约定。

## Dependencies / 依赖关系
- **`vllm.distributed`**: EN: Supplies tensor-parallel rank/world-size used in all shard calculations. CN: 提供所有分片计算所需的张量并行 rank/world-size。
- **`vllm.platforms.current_platform`**: EN: Wraps weight loaders on platforms that need synchronization during loading. CN: 在需要加载同步的平台上包装权重加载器。
- **PyTorch `Parameter` and tensor ops**: EN: Provide storage, `narrow`, `permute`, copy semantics, and weak-reference-managed tensors. CN: 提供参数存储、`narrow`、`permute`、复制语义以及弱引用管理的张量对象。
- **Quantization backends such as Marlin**: EN: Influence packing metadata and shard-index conversion rules. CN: 量化后端（如 Marlin）会影响打包元数据和分片索引换算规则。
