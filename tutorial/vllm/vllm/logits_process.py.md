# logits_process.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/logits_process.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: EN: Implements bad-word suppression as logits processors that can be injected into the sampling loop. / CN: 实现“禁用词”抑制逻辑，并把它封装为可插入采样循环的 logits processor。

## Line-by-Line Analysis / 逐行分析

### LogitsProcessor callable contract
```python
LogitsProcessor: TypeAlias = (
    Callable[[list[int], torch.Tensor], torch.Tensor]
    | Callable[[list[int], list[int], torch.Tensor], torch.Tensor]
)
"""LogitsProcessor is a function that takes a list
of previously generated tokens, the logits tensor
for the next token and, optionally, prompt tokens as a
first argument, and returns a modified tensor of logits
to sample from."""
```
**EN:** The type alias intentionally accepts two callable signatures: one based only on generated history and one that also receives prompt tokens. That matches the processor shapes supported by vLLM's sampler stack.
**CN:** 这个类型别名允许两类可调用对象：一种只依赖已生成 token 历史，另一种还会接收 prompt token。它对应了 vLLM 采样器栈中支持的两种 logits processor 形态。

### Bad-word tokenization
```python
def get_bad_words_logits_processors(
    bad_words: list[str], tokenizer: TokenizerLike
) -> list[LogitsProcessor]:
    bad_words_ids: list[list[int]] = list()

    for bad_word in bad_words:
        # To prohibit words both at the beginning
        # and in the middle of text
        # (related to add_prefix_space tokenizer parameter)
        for add_prefix_space in [False, True]:
            prefix = " " if add_prefix_space else ""
            prompt = prefix + bad_word.lstrip()

            prompt_token_ids = tokenizer.encode(text=prompt, add_special_tokens=False)

            # If no space at the beginning
            # or if prefix space produces a new word token
            if (not add_prefix_space) or (
                add_prefix_space
                and prompt_token_ids[0] != bad_words_ids[-1][0]
                and len(prompt_token_ids) == len(bad_words_ids[-1])
            ):
                bad_words_ids.append(prompt_token_ids)

    return [NoBadWordsLogitsProcessor(bad_words_ids=bad_words_ids)]
```
**EN:** Each banned word is tokenized twice: once as-is and once with a leading space. This catches both sentence-initial and mid-sentence occurrences, while the deduplication condition avoids storing redundant variants when the tokenizer produces the same segmentation.
**CN:** 每个禁用词都会被分词两次：一次按原样分词，一次在前面加空格。这样既能拦截句首出现，也能拦截句中出现；而后面的去重条件则避免在 tokenizer 给出相同切分时重复保存变体。

### Runtime masking
```python
class NoBadWordsLogitsProcessor:
    _SMALLEST_LOGIT = float("-inf")
    _NEUTRAL_LOGIT = 0.0

    def __init__(self, bad_words_ids: list[list[int]]):
        self.bad_words_ids = bad_words_ids
        self.word_bias: torch.FloatTensor = None

    def __call__(
        self,
        past_tokens_ids: Sequence[int],
        logits: torch.FloatTensor,
    ) -> torch.Tensor:
        if self.word_bias is None:
            self._init_word_bias(logits=logits)

        last_token_bias = torch.zeros_like(logits)

        for bad_word_ids in self.bad_words_ids:
            if len(bad_word_ids) == 1:  # 1-token words already processed
                continue

            if len(bad_word_ids) > len(past_tokens_ids) + 1:
                continue

            prefix_length = len(bad_word_ids) - 1
            last_token_id = bad_word_ids[-1]
            actual_prefix = past_tokens_ids[-prefix_length:]
            expected_prefix = bad_word_ids[:prefix_length]

            assert len(actual_prefix) == len(expected_prefix)

            is_match = tuple(actual_prefix) == tuple(expected_prefix)
            last_token_bias[last_token_id] += (
                self._SMALLEST_LOGIT if is_match else self._NEUTRAL_LOGIT
            )

        logits = logits + self.word_bias + last_token_bias

        return logits
```
**EN:** The processor lazily initializes a reusable vocabulary-sized bias tensor the first time it sees logits. Single-token bans are encoded there permanently; multi-token bans are checked on each step by matching the recent token suffix against the banned prefix and applying `-inf` only to the final token that would complete the forbidden phrase.
**CN:** processor 第一次拿到 logits 时会延迟初始化一个可复用的、按词表大小分配的 bias 张量。单 token 禁用项会被直接固化到该张量里；多 token 禁用项则在每一步都检查最近生成的后缀是否匹配禁用前缀，只有在即将补全整段禁用词时才对最后一个 token 施加 `-inf`。

### Bias initialization and bounds checks
```python
    def _init_word_bias(self, logits: torch.FloatTensor) -> None:
        # Code based on NoBadWordsLogitsProcessor and SequenceBiasLogitsProcessor  # noqa: E501
        # from https://github.com/huggingface/transformers/blob/main/src/transformers/generation/logits_process.py

        vocab_size = logits.shape[-1]

        self._check_token_ids_bounds(vocab_size=vocab_size)

        self.word_bias = torch.zeros(
            (vocab_size,), dtype=torch.float, device=logits.device
        )

        for bad_word_ids in self.bad_words_ids:
            if len(bad_word_ids) == 1:
                bad_word_id = bad_word_ids[-1]
                self.word_bias[bad_word_id] = self._SMALLEST_LOGIT

    def _check_token_ids_bounds(self, vocab_size: int) -> None:
        invalid_token_ids = []

        for bad_word_ids in self.bad_words_ids:
            for token_id in bad_word_ids:
                if token_id < 0 or token_id >= vocab_size:
                    invalid_token_ids.append(token_id)

        if len(invalid_token_ids) > 0:
            raise ValueError(
                f"The model vocabulary size is {vocab_size},"
                f" but the following tokens"
                f" were specified as bad: {invalid_token_ids}."
                f" All token id values should be integers satisfying:"
                f" 0 <= token_id < {vocab_size}."
            )
```
**EN:** `_init_word_bias()` validates token ids against the current vocabulary size and allocates the bias tensor on the same device as the logits. `_check_token_ids_bounds()` fails with a detailed error message instead of silently producing out-of-range indexing bugs.
**CN:** `_init_word_bias()` 会先根据当前词表大小校验 token id，再在与 logits 相同的设备上分配 bias 张量。`_check_token_ids_bounds()` 则会显式抛出详细异常，而不是让越界索引悄悄变成难定位的问题。

## Key Concepts / 关键概念
- **Static vs dynamic masking** — EN: One-token bans are precomputed once, while multi-token bans are checked against runtime history. / CN: 单 token 禁用项会预计算；多 token 禁用项则依赖运行时历史逐步检查。
- **Tokenizer-aware matching** — EN: Prefix-space variants are necessary because token boundaries depend on surrounding whitespace. / CN: 由于 token 边界受空白字符影响，必须同时考虑带前导空格的分词结果。
- **Hard suppression** — EN: The processor uses `-inf` logits so forbidden continuations become impossible to sample. / CN: 该 processor 通过设置 `-inf` logits，使禁用续写在采样时变为不可能。

## Dependencies / 依赖关系
- **torch** — EN: Provides tensor allocation and arithmetic for per-token bias updates. / CN: 提供按 token 更新 bias 所需的张量分配与运算能力。
- **vllm.tokenizers.TokenizerLike** — EN: Supplies the `encode()` method and tokenizer metadata used to map bad words to ids. / CN: 提供 `encode()` 方法和 tokenizer 元数据，用于把禁用词映射成 token id。
- **Sampling loop** — EN: These processors are designed to plug into vLLM's sampler before token selection. / CN: 这些 processor 设计目标就是在 token 选择前接入 vLLM 的采样流程。
