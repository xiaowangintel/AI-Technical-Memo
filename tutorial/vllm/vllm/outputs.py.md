# outputs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/outputs.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: EN: Defines API-facing output containers for generation, pooling, embedding, classification, and scoring requests, including streaming merge behavior. / CN: 定义面向 API 的输出容器，覆盖生成、pooling、embedding、分类与打分请求，并包含流式结果的合并逻辑。

## Line-by-Line Analysis / 逐行分析

### CompletionOutput payload
```python
@dataclass
class CompletionOutput:
    """The output data of one completion output of a request.

    Args:
        index: The index of the output in the request.
        text: The generated output text.
        token_ids: The token IDs of the generated output text.
        cumulative_logprob: The cumulative log probability of the generated
            output text.
        logprobs: The log probabilities of the top probability words at each
            position if the logprobs are requested.
        finish_reason: The reason why the sequence is finished.
        stop_reason: The stop string or token id that caused the completion
            to stop, None if the completion finished for some other reason
            including encountering the EOS token.
        lora_request: The LoRA request that was used to generate the output.
    """

    index: int
    text: str
    token_ids: GenericSequence[int]
    cumulative_logprob: float | None
    logprobs: SampleLogprobs | None
    routed_experts: np.ndarray | None = None  # [seq_len,layer_num,topk]
    finish_reason: str | None = None
    stop_reason: int | str | None = None
    lora_request: LoRARequest | None = None

    def finished(self) -> bool:
        return self.finish_reason is not None

    def __repr__(self) -> str:
        return (
            f"CompletionOutput(index={self.index}, "
            f"text={self.text!r}, "
            f"token_ids={self.token_ids}, "
            f"routed_experts={self.routed_experts}, "
            f"cumulative_logprob={self.cumulative_logprob}, "
            f"logprobs={self.logprobs}, "
            f"finish_reason={self.finish_reason}, "
            f"stop_reason={self.stop_reason})"
        )
```
**EN:** `CompletionOutput` is the leaf object for one generated candidate. Besides text and token ids, it carries cumulative logprob, optional per-token logprob traces, routed-expert telemetry, stop metadata, and the LoRA request that produced the completion.
**CN:** `CompletionOutput` 是单个生成候选的基础输出对象。除了文本和 token id，它还包含累计对数概率、可选的逐 token logprob 轨迹、路由专家信息、停止原因，以及生成该结果时使用的 LoRA 请求。

### PoolingOutput tensor wrapper
```python
@dataclass
class PoolingOutput:
    """The output data of one pooling output of a request.

    Args:
        data: The extracted hidden states.
    """

    data: torch.Tensor

    def __repr__(self) -> str:
        return f"PoolingOutput(data={self.data})"

    def __eq__(self, other: object) -> bool:
        return isinstance(other, self.__class__) and bool(
            (self.data == other.data).all()
        )
```
**EN:** `PoolingOutput` is deliberately minimal: it wraps a tensor produced by a pooling model and defines equality by tensor contents rather than object identity. That makes it a useful shared base for higher-level embedding/classification/scoring adapters.
**CN:** `PoolingOutput` 设计得非常精简：它只包装 pooling 模型输出的张量，并把相等性定义为“张量内容相等”而不是“对象身份相等”。这使它可以作为 embedding / classification / scoring 等更高层适配器的共同基础。

### RequestOutput initialization
```python
class RequestOutput:
    """The output data of a completion request to the LLM.

    Args:
        request_id: The unique ID of the request.
        prompt: The prompt string of the request.
                For encoder/decoder models, this is the
                decoder input prompt.
        prompt_token_ids: The token IDs of the prompt.
                          For encoder/decoder models, this is the
                          decoder input prompt token ids.
        prompt_logprobs: The log probabilities to return per prompt token.
        outputs: The output sequences of the request.
        finished: Whether the whole request is finished.
        metrics: Metrics associated with the request.
        lora_request: The LoRA request that was used to generate the output.
        encoder_prompt: The encoder prompt string of the request.
                        None if decoder-only.
        encoder_prompt_token_ids: The token IDs of the encoder prompt.
                                  None if decoder-only.
        num_cached_tokens: The number of tokens with prefix cache hit.
        kv_transfer_params: The params for remote K/V transfer.
    """

    def __init__(
        self,
        request_id: str,
        prompt: str | None,
        prompt_token_ids: list[int] | None,
        prompt_logprobs: PromptLogprobs | None,
        outputs: list[CompletionOutput],
        finished: bool,
        metrics: RequestStateStats | None = None,
        lora_request: LoRARequest | None = None,
        encoder_prompt: str | None = None,
        encoder_prompt_token_ids: list[int] | None = None,
        num_cached_tokens: int | None = None,
        *,
        kv_transfer_params: dict[str, Any] | None = None,
        # Forward compatibility, code that uses args added in new release can
        # still run with older versions of vLLM without breaking.
        **kwargs: Any,
    ) -> None:
        if kwargs:
            logger.warning_once(
                "RequestOutput: Ignoring extra arguments: %s", str(kwargs)
            )
        self.request_id = request_id
        self.prompt = prompt
        self.prompt_token_ids = prompt_token_ids
        self.prompt_logprobs = prompt_logprobs
        self.outputs = outputs
        self.finished = finished
        self.metrics = metrics
        self.lora_request = lora_request
        self.encoder_prompt = encoder_prompt
        self.encoder_prompt_token_ids = encoder_prompt_token_ids
        self.num_cached_tokens = num_cached_tokens
        self.kv_transfer_params = kv_transfer_params
```
**EN:** `RequestOutput` aggregates request-wide metadata: prompt text and ids, optional prompt logprobs, all completion candidates, metrics, LoRA info, encoder-side fields for seq2seq models, cache-hit counts, and K/V transfer parameters. The `**kwargs` warning is a compatibility valve so newer servers can send extra fields without breaking older clients.
**CN:** `RequestOutput` 汇总了请求级别的元数据：prompt 文本与 token id、可选的 prompt logprobs、所有 completion 候选、性能指标、LoRA 信息、seq2seq 模型的 encoder 侧字段、cache 命中数量，以及 K/V 传输参数。`**kwargs` 的 warning 机制则是一个兼容性阀门，让新版本服务端即使多传字段，也不会直接把老版本客户端打崩。

### Streaming merge logic
```python
    def add(self, next_output: "RequestOutput", aggregate: bool) -> None:
        """Merge subsequent RequestOutput into this one"""

        self.finished |= next_output.finished
        self.kv_transfer_params = next_output.kv_transfer_params

        for next_completion in next_output.outputs:
            for i, completion in enumerate(self.outputs):
                if completion.index == next_completion.index:
                    if aggregate:
                        # Merge outputs with same index
                        completion.text += next_completion.text
                        if not isinstance(completion.token_ids, MutableSequence):
                            completion.token_ids = list(completion.token_ids)
                        completion.token_ids.extend(next_completion.token_ids)
                        if next_completion.logprobs:
                            assert completion.logprobs is not None
                            completion.logprobs.extend(next_completion.logprobs)  # type: ignore[arg-type]
                        completion.cumulative_logprob = (
                            next_completion.cumulative_logprob
                        )
                        completion.finish_reason = next_completion.finish_reason
                        completion.stop_reason = next_completion.stop_reason
                    else:
                        # Replace the output with the new one
                        self.outputs[i] = next_completion
                    break
            else:
                self.outputs.append(next_completion)
```
**EN:** `add()` merges another `RequestOutput` by completion index. In aggregate mode it appends streamed text, token ids, and logprobs into the existing completion snapshot and updates finish metadata; otherwise it simply replaces the older completion object with the newer one.
**CN:** `add()` 会按 completion 的 `index` 合并另一个 `RequestOutput`。在 aggregate 模式下，它会把流式增量文本、token id 和 logprobs 拼接到已有快照上，并更新结束信息；否则就直接用新的 completion 对象替换旧的。

### Stream sentinel and generic pooling request wrapper
```python
# Sentinel to indicate request is finished, used with streaming inputs.
STREAM_FINISHED = RequestOutput(
    request_id="",
    prompt=None,
    prompt_token_ids=None,
    prompt_logprobs=None,
    outputs=[],
    finished=True,
)

_O = TypeVar("_O", default=PoolingOutput)


class PoolingRequestOutput(Generic[_O]):
    """
    The output data of a pooling request to the LLM.

    Args:
        request_id (str): A unique identifier for the pooling request.
        outputs (PoolingOutput): The pooling results for the given input.
        prompt_token_ids (list[int]): A list of token IDs used in the prompt.
        num_cached_tokens: The number of tokens with prefix cache hit.
        finished (bool): A flag indicating whether the pooling is completed.
    """

    def __init__(
        self,
        request_id: str,
        outputs: _O,
        prompt_token_ids: list[int],
        num_cached_tokens: int,
        finished: bool,
    ):
        self.request_id = request_id
        self.prompt_token_ids = prompt_token_ids
        self.num_cached_tokens = num_cached_tokens
        self.finished = finished
        self.outputs = outputs

    def __repr__(self):
        return (
            f"{type(self).__name__}(request_id={self.request_id!r}, "
            f"outputs={self.outputs!r}, "
            f"prompt_token_ids={self.prompt_token_ids}, "
            f"num_cached_tokens={self.num_cached_tokens}, "
            f"finished={self.finished})"
        )
```
**EN:** `STREAM_FINISHED` is a sentinel `RequestOutput` used by streaming APIs to signal end-of-stream without inventing a separate control channel. `PoolingRequestOutput` is a generic request wrapper that carries shared request metadata for any pooling-derived output type.
**CN:** `STREAM_FINISHED` 是流式 API 使用的哨兵 `RequestOutput`，用于在不引入额外控制通道的前提下显式表示“流结束”。`PoolingRequestOutput` 则是一个泛型请求包装器，用于保存所有 pooling 派生输出共享的请求元数据。

### Embedding adapters
```python
@dataclass
class EmbeddingOutput:
    """The output data of one embedding output of a request.

    Args:
        embedding: The embedding vector, which is a list of floats.
            Its length depends on the hidden dimension of the model.
    """

    embedding: list[float]

    @staticmethod
    def from_base(pooling_output: PoolingOutput):
        pooled_data = pooling_output.data
        if pooled_data.ndim != 1:
            raise ValueError("pooled_data should be a 1-D embedding vector")

        return EmbeddingOutput(pooled_data.tolist())

    @property
    def hidden_size(self) -> int:
        return len(self.embedding)

    def __repr__(self) -> str:
        return f"EmbeddingOutput(hidden_size={self.hidden_size})"


class EmbeddingRequestOutput(PoolingRequestOutput[EmbeddingOutput]):
    @staticmethod
    def from_base(request_output: PoolingRequestOutput):
        return EmbeddingRequestOutput(
            request_id=request_output.request_id,
            outputs=EmbeddingOutput.from_base(request_output.outputs),
            prompt_token_ids=request_output.prompt_token_ids,
            num_cached_tokens=request_output.num_cached_tokens,
            finished=request_output.finished,
        )
```
**EN:** `EmbeddingOutput.from_base()` requires the pooled tensor to be one-dimensional and converts it into a plain Python list for serialization. `EmbeddingRequestOutput.from_base()` then lifts the generic pooling wrapper into an embedding-specific request output without altering request bookkeeping.
**CN:** `EmbeddingOutput.from_base()` 要求 pooled tensor 必须是一维向量，并把它转换成便于序列化的 Python 浮点列表。`EmbeddingRequestOutput.from_base()` 则在不改变请求级元数据的前提下，把通用的 pooling 包装器提升为 embedding 专用输出类型。

### Classification adapters
```python
@dataclass
class ClassificationOutput:
    """The output data of one classification output of a request.

    Args:
        probs: The probability vector, which is a list of floats.
            Its length depends on the number of classes.
    """

    probs: list[float]

    @staticmethod
    def from_base(pooling_output: PoolingOutput):
        # pooling_output shape: (num_classes)
        pooled_data = pooling_output.data
        if pooled_data.ndim != 1:
            raise ValueError("pooled_data should be a 1-D probability vector")

        return ClassificationOutput(pooled_data.tolist())

    @property
    def num_classes(self) -> int:
        return len(self.probs)

    def __repr__(self) -> str:
        return f"ClassificationOutput(num_classes={self.num_classes})"


class ClassificationRequestOutput(PoolingRequestOutput[ClassificationOutput]):
    @staticmethod
    def from_base(request_output: PoolingRequestOutput):
        return ClassificationRequestOutput(
            request_id=request_output.request_id,
            outputs=ClassificationOutput.from_base(request_output.outputs),
            prompt_token_ids=request_output.prompt_token_ids,
            num_cached_tokens=request_output.num_cached_tokens,
            finished=request_output.finished,
        )
```
**EN:** Classification conversion mirrors embedding conversion, but it interprets the 1-D pooled tensor as a probability vector. The convenience property `num_classes` exposes the vector width as API-friendly metadata.
**CN:** 分类输出的转换流程与 embedding 类似，但这里把一维 pooled tensor 解释为概率向量。辅助属性 `num_classes` 则把向量宽度暴露成更友好的 API 元数据。

### Scoring adapters
```python
@dataclass
class ScoringOutput:
    """The output data of one scoring output of a request.

    Args:
        score: The similarity score, which is a scalar value.
    """

    score: float

    @staticmethod
    def from_base(pooling_output: PoolingOutput):
        # pooling_output shape:
        #   classify task: (num_classes) num_classes == 1
        #   embed task: a scalar value
        pooled_data = pooling_output.data.squeeze()
        if pooled_data.ndim != 0:
            raise ValueError("pooled_data should be a scalar score")

        return ScoringOutput(pooled_data.item())

    def __repr__(self) -> str:
        return f"ScoringOutput(score={self.score})"


class ScoringRequestOutput(PoolingRequestOutput[ScoringOutput]):
    @staticmethod
    def from_base(request_output: PoolingRequestOutput):
        return ScoringRequestOutput(
            request_id=request_output.request_id,
            outputs=ScoringOutput.from_base(request_output.outputs),
            prompt_token_ids=request_output.prompt_token_ids,
            num_cached_tokens=request_output.num_cached_tokens,
            finished=request_output.finished,
        )
```
**EN:** `ScoringOutput.from_base()` squeezes the pooled tensor down to a scalar and rejects any non-scalar remainder, so scoring requests always expose a single numeric similarity or relevance score. The corresponding request wrapper preserves request ids, cache stats, and completion state exactly like the other pooling variants.
**CN:** `ScoringOutput.from_base()` 会把 pooled tensor 压缩成标量，并拒绝任何仍然不是标量的结果，因此打分请求最终只会暴露一个数值型相似度或相关性分数。对应的请求包装器则与其它 pooling 变体一样，完整保留 request id、cache 统计和完成状态。

## Key Concepts / 关键概念
- **Leaf vs request-level outputs** — EN: `CompletionOutput`/`PoolingOutput` describe one result, while `RequestOutput` and `PoolingRequestOutput` wrap request-wide metadata. / CN: `CompletionOutput` / `PoolingOutput` 描述单个结果，而 `RequestOutput` 与 `PoolingRequestOutput` 负责封装请求级元数据。
- **Streaming aggregation** — EN: Partial generation updates can be appended into a stable per-index completion record. / CN: 流式生成的增量结果可以按 index 追加到稳定的 completion 记录中。
- **Typed pooling adapters** — EN: Embedding, classification, and scoring share the same base tensor wrapper but expose task-specific semantics. / CN: embedding、classification 和 scoring 共享同一个基础张量包装器，但会暴露各自任务语义。

## Dependencies / 依赖关系
- **vllm.logprobs** — EN: Supplies the prompt/sample logprob container types attached to completion outputs. / CN: 提供挂接在 completion 输出上的 prompt/sample logprob 容器类型。
- **numpy and torch** — EN: Used for routed-expert telemetry arrays and pooling tensors. / CN: 用于表示 routed expert 遥测数组以及 pooling 结果张量。
- **vllm.v1.metrics.stats.RequestStateStats** — EN: Carries request-level timing and scheduling statistics. / CN: 保存请求级别的时序与调度统计信息。
