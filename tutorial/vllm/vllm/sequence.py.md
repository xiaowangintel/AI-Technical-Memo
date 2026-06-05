# sequence.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/sequence.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: EN: Defines the lightweight container used to shuttle intermediate tensors and optional KV-connector metadata between pipeline stages. / CN: 定义在流水线阶段之间传递中间张量及可选 KV-connector 元数据的轻量级容器。

## Line-by-Line Analysis / 逐行分析

### IntermediateTensors structure and manual init
```python
@dataclass
class IntermediateTensors:
    """For all pipeline stages except the last, we need to return the hidden
    states and residuals to be sent to the next stage. This data structure
    contains the hidden states and residuals for a request.

    Each stage also needs to handle its own kv_connector_output.
    """

    tensors: dict[str, torch.Tensor]
    kv_connector_output: KVConnectorOutput | None

    def __init__(
        self,
        tensors: dict[str, torch.Tensor],
        kv_connector_output: KVConnectorOutput | None = None,
    ) -> None:
        # manually define this function, so that
        # Dynamo knows `IntermediateTensors()` comes from this file.
        # Otherwise, dataclass will generate this function by evaluating
        # a string, and we will lose the information about the source file.
        self.tensors = tensors
        self.kv_connector_output = kv_connector_output
```
**EN:** `IntermediateTensors` packages a dictionary of named tensors plus optional `kv_connector_output` for multi-stage execution. The constructor is written manually instead of relying on dataclass-generated code so Torch Dynamo can attribute the callable to this source file rather than to a synthesized function body.
**CN:** `IntermediateTensors` 把一组具名张量和可选的 `kv_connector_output` 打包在一起，用于多阶段执行。这里没有直接使用 dataclass 自动生成的构造函数，而是手写 `__init__`，以便 Torch Dynamo 能把这个构造过程归因到当前源文件，而不是一个动态生成的函数体。

### Dictionary-style and slice-style access
```python
    def __getitem__(self, key: str | slice):
        if isinstance(key, str):
            return self.tensors[key]
        elif isinstance(key, slice):
            return self.__class__({k: v[key] for k, v in self.tensors.items()})
```
**EN:** String indexing simply returns one named tensor. Slice indexing is more interesting: it applies the same slice to every stored tensor and returns a new `IntermediateTensors`, which is a convenient way to split or narrow a batched pipeline payload consistently.
**CN:** 字符串索引会直接返回对应名字的张量。切片索引更有意思：它会把同一个切片应用到所有保存的张量上，并返回一个新的 `IntermediateTensors`，这让批量流水线载荷的切分与裁剪能够保持一致。

### Container helpers
```python
    def __setitem__(self, key: str, value: torch.Tensor):
        self.tensors[key] = value

    def items(self):
        return self.tensors.items()

    def __len__(self):
        return len(self.tensors)

    def __eq__(self, other: object):
        if not isinstance(other, self.__class__):
            return False
        if self.tensors.keys() != other.tensors.keys():
            return False
        return all(torch.equal(self.tensors[k], other.tensors[k]) for k in self.tensors)

    def __repr__(self) -> str:
        return f"IntermediateTensors(tensors={self.tensors})"

    @staticmethod
    def empty_like(
        intermediate_tensors: "IntermediateTensors",
    ) -> "IntermediateTensors":
        tensors = {
            k: torch.empty_like(v) for k, v in intermediate_tensors.tensors.items()
        }
        return IntermediateTensors(tensors)
```
**EN:** The rest of the class adds small but practical container utilities: assignment, iteration through `items()`, length, tensor-aware equality, readable repr, and `empty_like()` for allocating a shape-matched buffer for downstream stages without copying actual values.
**CN:** 类的剩余部分补充了一些非常实用的容器能力：赋值、通过 `items()` 遍历、长度查询、基于张量内容的相等性比较、可读的 `repr`，以及 `empty_like()`——后者可以为下游阶段分配一个形状匹配但不复制实际数值的缓冲区。

## Key Concepts / 关键概念
- **Pipeline handoff** — EN: The class is meant for stage-to-stage transfer in pipeline parallel execution. / CN: 这个类主要用于流水线并行执行中的阶段间数据交接。
- **Consistent slicing** — EN: Slicing every stored tensor together keeps batch alignment intact. / CN: 对所有保存的张量同时切片可以保持 batch 对齐。
- **Dynamo-friendly construction** — EN: A handwritten constructor preserves source attribution for Torch Dynamo. / CN: 手写构造函数有助于 Torch Dynamo 正确保留源文件归因。

## Dependencies / 依赖关系
- **torch** — EN: Supplies the tensor type and `empty_like()` allocation primitive. / CN: 提供张量类型和 `empty_like()` 分配原语。
- **KVConnectorOutput** — EN: Optionally carries KV-transfer metadata for connector-aware pipeline stages. / CN: 可选地携带支持 connector 的流水线阶段所需的 KV 传输元数据。
- **dataclasses** — EN: Used for the lightweight record declaration even though init is overridden manually. / CN: 即使手动重写了 init，仍然使用 dataclass 进行轻量级记录声明。
