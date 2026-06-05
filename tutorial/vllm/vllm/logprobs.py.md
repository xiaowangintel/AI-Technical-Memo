# logprobs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/logprobs.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: EN: Provides OpenAI-compatible logprob records plus a flattened container that reduces Python object overhead for long generations. / CN: 提供兼容 OpenAI 的 logprob 记录结构，以及一个可降低长序列 Python 对象开销的扁平化容器。

## Line-by-Line Analysis / 逐行分析

### Logprob record and aliases
```python
@dataclass
class Logprob:
    """Infos for supporting OpenAI compatible logprobs and token ranks.

    Attributes:
        logprob: The logprob of chosen token
        rank: The vocab rank of chosen token (>=1)
        decoded_token: The decoded chosen token index
    """

    logprob: float
    rank: int | None = None
    decoded_token: str | None = None


LogprobsOnePosition = dict[int, Logprob]
```
**EN:** `Logprob` is the leaf payload used in response objects: it stores the selected token's logprob, optional vocabulary rank, and decoded text. The aliases then distinguish a single position (`dict[int, Logprob]`) from multi-position prompt/sample containers.
**CN:** `Logprob` 是响应对象中的最小概率记录单元，保存选中 token 的对数概率、可选的词表排名以及解码后的文本。后面的类型别名则把“单个位置的映射”和“跨多个位置的容器”区分开来。

### Flattened storage layout
```python
@dataclass
class FlatLogprobs(MutableSequence[LogprobsOnePosition | None]):
    """
    Flat logprobs of a request into multiple primitive type lists.

    Compared to list[dict[int, Logprob]], this data structure reduced GC
    overhead significantly. As it flattened logprob information for
    all positions and ranks in to multiple primitive type lists (i.e.
    logprobs, token_ids, ranks per token_ids, decoded_tokens).
    So regardless of the sequence length and top_logprobs setup,
    FlatLogprobs would only introduce a constant amount of objects.

    As each position might contains different amount of ranks,
    start_indices_per_position would be used to access the logprob ranges
    for different positions.

    NOTE: To reduce the migration overhead and improve backward compatibility,
    we support the key Sequence APIs of list, so it could act as
    list[LogprobsOnePosition]
    """

    # Start / end indices to indicate the range of logprobs for each position.
    start_indices: list[int] = field(default_factory=list)
    end_indices: list[int] = field(default_factory=list)

    # Flatten Logprob information for (each position, rank).
    # For position <i>, the logprobs are ranged
    # from self.start_indices[i] to self.end_indices[i] (exclusive).
    token_ids: list[int] = field(default_factory=list)
    logprobs: list[float] = field(default_factory=list)
    ranks: list[int | None] = field(default_factory=list)
    decoded_tokens: list[str | None] = field(default_factory=list)

    def append(self, logprobs_one_position: LogprobsOnePosition | None) -> None:
        """Appends the container with logprobs for the next position"""
        self.start_indices.append(len(self.logprobs))
        if logprobs_one_position:
            for token_id, logprob in logprobs_one_position.items():
                self.token_ids.append(token_id)
                self.logprobs.append(logprob.logprob)
                self.ranks.append(logprob.rank)
                self.decoded_tokens.append(logprob.decoded_token)
        self.end_indices.append(len(self.logprobs))

    def append_fast(
        self,
        token_ids: list[int],
        logprobs: list[float],
        ranks: itertools.chain[int],
        decoded_tokens: Iterable[str | None],
    ) -> None:
        """
        Appends logprobs for the next position without creating
        the intermediate logprob dictionary.
        """
        self.start_indices.append(len(self.logprobs))
        for token_id, logprob, rank, decoded_token in zip(
            token_ids, logprobs, ranks, decoded_tokens
        ):
            self.token_ids.append(token_id)
            self.logprobs.append(logprob)
            self.ranks.append(rank)
            self.decoded_tokens.append(decoded_token)
        self.end_indices.append(len(self.logprobs))
```
**EN:** `FlatLogprobs` uses parallel primitive lists plus `start_indices`/`end_indices` to store all positions in one compact structure. This avoids constructing many short-lived dictionaries and `Logprob` objects, and `append_fast()` lets upstream code append already-vectorized data without building intermediate dicts.
**CN:** `FlatLogprobs` 用多条并行的基础类型列表，再配合 `start_indices`/`end_indices` 来描述每个位置的数据范围，从而把所有位置压缩进一个紧凑结构里。这能避免频繁创建大量短生命周期的字典和 `Logprob` 对象；`append_fast()` 还允许上游直接写入向量化后的结果，而不必先构造中间字典。

### Sequence-style view API
```python
    def extend(self, logprobs_multi_positions) -> None:
        """Extends the container with logprobs for the next multiple positions"""
        for logprobs_one_position in logprobs_multi_positions:
            self.append(logprobs_one_position)

    def __len__(self) -> int:
        """Gets number of positions stored in the container"""
        return len(self.start_indices)

    @overload
    def __getitem__(self, position: int) -> LogprobsOnePosition: ...

    @overload
    def __getitem__(self, s: slice, /) -> "FlatLogprobs": ...

    def __getitem__(self, index: int | slice):
        """Extracts logprobs of a given position or slice"""
        if isinstance(index, int):
            return {
                self.token_ids[i]: Logprob(
                    logprob=self.logprobs[i],
                    rank=self.ranks[i],
                    decoded_token=self.decoded_tokens[i],
                )
                for i in range(self.start_indices[index], self.end_indices[index])
            }
        elif isinstance(index, slice):
            min_index = self.start_indices[index][0]
            max_index = self.end_indices[index][-1]
            return FlatLogprobs(
                # Shift updated start_indices and end_indices to
                # be 0-indexed
                start_indices=[i - min_index for i in self.start_indices[index]],
                end_indices=[i - min_index for i in self.end_indices[index]],
                token_ids=self.token_ids[min_index:max_index],
                logprobs=self.logprobs[min_index:max_index],
                ranks=self.ranks[min_index:max_index],
                decoded_tokens=self.decoded_tokens[min_index:max_index],
            )
        else:
            raise TypeError(f"Invalid index type: {type(index)}")

    def __setitem__(self, item, value) -> None:
        raise TypeError("Cannot set logprobs in FlatLogprobs")

    def __delitem__(self, item) -> None:
        raise TypeError("Cannot delete logprobs from FlatLogprobs")

    def insert(self, index: int, value: dict[int, Logprob] | None) -> None:
        raise TypeError("Cannot insert logprobs to FlatLogprobs")

    def __iter__(self) -> Iterator[LogprobsOnePosition]:
        """
        Iterates the container and yields LogprobsOnePosition for
        each position.
        """
        for i in range(0, len(self.start_indices)):
            yield self.__getitem__(i)
```
**EN:** The class preserves the key read APIs of `MutableSequence` for backward compatibility. Integer indexing reconstructs one position's dictionary on demand, slicing rebases the flat offsets into a new `FlatLogprobs`, and all mutation methods intentionally raise `TypeError` so the structure stays append-only.
**CN:** 为了兼容旧接口，这个类保留了 `MutableSequence` 的关键读取能力。整数索引会按需重建某个位置的字典；切片会把扁平偏移量重新映射到一个新的 `FlatLogprobs`；所有修改型方法都显式抛出 `TypeError`，从而保证该结构只追加、不原地编辑。

### Factory helpers
```python
# {token_id -> logprob} per each sequence group. None if the corresponding
# sequence group doesn't require prompt logprob.
PromptLogprobs = FlatLogprobs | list[LogprobsOnePosition | None]
# {token_id -> logprob} for each sequence group.
SampleLogprobs = FlatLogprobs | list[LogprobsOnePosition]


def create_prompt_logprobs(flat_logprobs: bool) -> PromptLogprobs:
    """Creates a container to store prompt logprobs for a request"""
    logprobs: PromptLogprobs = FlatLogprobs() if flat_logprobs else []
    # NOTE: logprob of first prompt token is None.
    logprobs.append(None)
    return logprobs


def create_sample_logprobs(flat_logprobs: bool) -> SampleLogprobs:
    """Creates a container to store decode logprobs for a request"""
    return FlatLogprobs() if flat_logprobs else []
```
**EN:** `create_prompt_logprobs()` inserts a leading `None` because the first prompt token has no predecessor from which to compute probability. `create_sample_logprobs()` simply chooses between flat and legacy container forms for decode-time logprobs.
**CN:** `create_prompt_logprobs()` 会先插入一个 `None`，因为第一个 prompt token 没有前驱 token，无法计算条件概率。`create_sample_logprobs()` 则只是根据配置在扁平容器与旧式列表容器之间做选择，用于保存解码阶段的 logprobs。

### Appending next-token probabilities
```python
def append_logprobs_for_next_position(
    request_logprobs: PromptLogprobs | SampleLogprobs,
    token_ids: list[int],
    logprobs: list[float],
    decoded_tokens: Iterable[str | None],
    rank: int,
    num_logprobs: int,
) -> None:
    """Appends logprobs for the next position"""
    if num_logprobs == -1:
        num_logprobs = len(logprobs)
    # We do not need a special case for the sampled token
    # being in the topk, since inserting duplicated data
    # into a dictionary twice is the same as doing it once.
    topk_ranks = range(1, num_logprobs + 1)
    ranks = itertools.chain((rank,), topk_ranks)

    if isinstance(request_logprobs, FlatLogprobs):
        request_logprobs.append_fast(token_ids, logprobs, ranks, decoded_tokens)
    else:
        request_logprobs.append(
            {
                token_id: Logprob(
                    logprob=logprob,
                    rank=rank,
                    decoded_token=token,
                )
                for token_id, logprob, rank, token in zip(
                    token_ids, logprobs, ranks, decoded_tokens
                )
            }
        )
```
**EN:** This helper merges the sampled token rank with the requested top-k ranks for the next position. It also interprets `num_logprobs == -1` as “return everything” and writes through the most efficient container path depending on whether the request uses `FlatLogprobs` or the legacy list-of-dicts format.
**CN:** 这个辅助函数会把“采样得到的 token 排名”与“请求返回的 top-k 排名”合并成下一位置的概率记录。它还把 `num_logprobs == -1` 解释为“返回全部”，并根据容器类型选择最快的写入路径：扁平容器走 `append_fast()`，旧格式则重建字典。

## Key Concepts / 关键概念
- **Flattened representation** — EN: Parallel arrays trade read-time reconstruction for much lower allocation and GC pressure. / CN: 并行数组用读取时重建换取了更低的分配成本和 GC 压力。
- **Prompt vs sample logprobs** — EN: Prompt logprobs need a leading placeholder, while decode logprobs start at the first generated token. / CN: prompt logprobs 需要一个前置占位符，而解码 logprobs 则从首个生成 token 开始。
- **Compatibility layer** — EN: The container behaves like the old list-of-dicts API so higher layers do not need a full rewrite. / CN: 该容器对外仍表现得像旧的“字典列表”接口，因此上层代码无需整体重写。

## Dependencies / 依赖关系
- **itertools** — EN: Supplies the chained rank iterator used when appending sampled and top-k logprobs together. / CN: 提供链式 rank 迭代器，用于把采样 token 与 top-k 结果一起写入。
- **dataclasses** — EN: Used for compact record definitions such as `Logprob` and `FlatLogprobs`. / CN: 用于定义 `Logprob`、`FlatLogprobs` 这类紧凑的数据记录结构。
- **OpenAI-compatible responses** — EN: These types back the logprob fields returned through vLLM's API surfaces. / CN: 这些类型为 vLLM API 返回的 logprob 字段提供底层数据结构。
