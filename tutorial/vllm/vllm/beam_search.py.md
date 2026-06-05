# beam_search.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/beam_search.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: EN: Defines beam-search state objects, prompt reconstruction helpers, and the length-penalized scoring used to rank beams. / CN: 定义束搜索的状态对象、prompt 重建辅助函数，以及用于排序 beam 的长度惩罚评分逻辑。

## Line-by-Line Analysis / 逐行分析

### BeamSearchSequence state
```python
@dataclass
class BeamSearchSequence:
    """A sequence for beam search.
    It keeps track of the tokens and the log probability of the sequence.
    The text field is optional and will only be filled when the sequence is
    about to be returned to the user.
    """

    orig_prompt: TokensInput | MultiModalInput | EncoderDecoderInput

    # NOTE: Tokens represents decoder tokens in the encoder / decoder case
    tokens: list[int]
    logprobs: list[dict[int, Logprob]]
    lora_request: LoRARequest | None = None
    cum_logprob: float = 0.0
    text: str | None = None
    finish_reason: str | None = None
    stop_reason: int | str | None = None
```
**EN:** This dataclass ties the original request payload to mutable beam-local state: decoder token ids, per-position logprob dictionaries, optional LoRA routing, and completion metadata. The inline note is important: for encoder-decoder models, `tokens` refers only to the decoder side, not the encoder prompt.
**CN:** 这个 dataclass 把原始请求载荷与可变的 beam 局部状态绑定在一起：decoder token id、按位置保存的 logprob 字典、可选的 LoRA 请求以及结束元数据。代码中的注释很关键：对 encoder-decoder 模型来说，`tokens` 只表示 decoder 侧的扩展结果，不包含 encoder prompt。

### Prompt reconstruction for active beam
```python
    def get_prompt(self):
        prompt = self.orig_prompt

        if prompt["type"] == "enc_dec":
            return self._build_encoder_decoder_inputs(prompt)

        # Handle decoder-only inputs
        prompt_text = prompt.get("prompt")
        cache_salt = prompt.get("cache_salt")

        if prompt["type"] == "token":
            return tokens_input(
                self.tokens,
                prompt=prompt_text,
                cache_salt=cache_salt,
            )

        return mm_input(
            prompt_token_ids=self.tokens,
            mm_kwargs=prompt["mm_kwargs"],
            mm_hashes=prompt["mm_hashes"],
            mm_placeholders=prompt["mm_placeholders"],
            prompt=prompt_text,
            cache_salt=cache_salt,
        )
```
**EN:** `get_prompt()` converts the current beam back into an engine input object. It preserves prompt text and cache salt, dispatches between plain token and multimodal inputs, and swaps in the beam's latest token list as the prompt token ids.
**CN:** `get_prompt()` 会把当前 beam 重新编码成引擎可消费的输入对象。它保留原始的 prompt 文本和 cache salt，在纯 token 与 multimodal 输入之间分派，并把当前 beam 的 token 序列替换为新的 prompt token id。

### Encoder-decoder rebuild path
```python
    def _build_encoder_decoder_inputs(
        self, prompt: EncoderDecoderInput
    ) -> EncoderDecoderInput:
        """Rebuild the encoder-decoder inputs with the current beam search
        sequence's tokens.

        FIXME (alex) - the encoder multimodal cache is not properly wired up
        yet, which means that currently we are running the encoder on every
        new beam because num_computed_tokens is 0 on each new request. This
        will be fixed once the cache is correctly implemented.
        """
        dec_prompt = prompt["decoder_prompt"]

        # Rebuild decoder prompt with updated tokens,
        # but keep everything else the same.
        new_dec_prompt: DecoderOnlyEngineInput
        if dec_prompt["type"] == "multimodal":
            new_dec_prompt = mm_input(
                self.tokens,
                mm_kwargs=dec_prompt["mm_kwargs"],
                mm_hashes=dec_prompt["mm_hashes"],
                mm_placeholders=dec_prompt["mm_placeholders"],
                prompt=dec_prompt.get("prompt"),
                cache_salt=dec_prompt.get("cache_salt"),
            )
        else:
            new_dec_prompt = tokens_input(
                self.tokens,
                prompt=dec_prompt.get("prompt"),
                cache_salt=dec_prompt.get("cache_salt"),
            )

        return EncoderDecoderInput(
            type="enc_dec",
            encoder_prompt=prompt["encoder_prompt"],
            decoder_prompt=new_dec_prompt,
        )
```
**EN:** For encoder-decoder requests, only the decoder prompt is rebuilt while the encoder prompt is reused verbatim. The FIXME documents a current limitation: encoder-side multimodal cache state is not transferred to new beams yet, so the encoder may be recomputed for every branched beam.
**CN:** 对于 encoder-decoder 请求，这里只重建 decoder prompt，而 encoder prompt 会被原样复用。FIXME 说明了当前限制：encoder 侧的 multimodal cache 还不能随新 beam 传递，因此每个分叉 beam 可能都会重新执行一次 encoder。

### BeamSearchInstance initialization
```python
class BeamSearchInstance:
    def __init__(
        self,
        prompt: TokensInput | MultiModalInput | EncoderDecoderInput,
        lora_request: LoRARequest | None = None,
        logprobs: list[dict[int, Logprob]] | None = None,
        **kwargs,
    ):
        decoder_prompt = (
            prompt if prompt["type"] != "enc_dec" else prompt["decoder_prompt"]
        )
        initial_tokens = decoder_prompt["prompt_token_ids"]

        self.beams: list[BeamSearchSequence] = [
            BeamSearchSequence(
                orig_prompt=prompt,
                tokens=initial_tokens,
                logprobs=[] if logprobs is None else list(logprobs),
                lora_request=lora_request,
                **kwargs,
            )
        ]
        self.completed: list[BeamSearchSequence] = []
```
**EN:** A new search instance starts with exactly one beam seeded from the decoder prompt token ids. Any existing logprobs and LoRA request are copied into that initial sequence, while completed hypotheses are tracked separately in `self.completed`.
**CN:** 一次新的搜索会从一个初始 beam 开始，这个 beam 由 decoder prompt 的 token id 初始化。若外部已经提供 logprobs 或 LoRA 请求，这里会把它们复制到初始序列中；已经结束的候选序列则单独放在 `self.completed` 中管理。

### Length-penalized beam ranking
```python
def get_beam_search_score(
    tokens: list[int],
    cumulative_logprob: float,
    eos_token_id: int,
    length_penalty: float = 1.0,
) -> float:
    """Calculate the beam search score with length penalty.

    Adapted from

    https://github.com/huggingface/transformers/blob/ccb92be23def445f2afdea94c31286f84b89eb5b/src/transformers/generation/beam_search.py#L938
    """
    seq_len = len(tokens)
    if tokens[-1] == eos_token_id:
        seq_len -= 1

    return cumulative_logprob / (seq_len**length_penalty)


def create_sort_beams_key_function(eos_token_id: int, length_penalty: float):
    def sort_beams_key(x: BeamSearchSequence) -> float:
        return get_beam_search_score(
            x.tokens, x.cum_logprob, eos_token_id, length_penalty
        )

    return sort_beams_key
```
**EN:** Beam ranking follows the Hugging Face length-penalty convention: cumulative logprob is divided by `seq_len ** length_penalty`, and a terminal EOS token is excluded from the effective length so finished beams are not over-penalized. `create_sort_beams_key_function()` closes over the EOS id and penalty to return a ready-to-use sorting key.
**CN:** beam 排序采用 Hugging Face 常见的长度惩罚公式：累计对数概率除以 `seq_len ** length_penalty`，并且若末尾是 EOS，则不会把该 EOS 计入有效长度，从而避免已结束序列被额外惩罚。`create_sort_beams_key_function()` 把 EOS id 与惩罚系数封装成一个可直接用于排序的 key 函数。

## Key Concepts / 关键概念
- **Beam-local state** — EN: Each candidate beam carries its own decoded tokens, token-level logprobs, and finish metadata. / CN: 每个候选 beam 都维护独立的解码 token、逐位置 logprob 和结束元数据。
- **Prompt rebuilding** — EN: The file regenerates engine inputs from the evolving beam so the engine can continue decoding. / CN: 该文件会根据不断扩展的 beam 重新生成引擎输入，以便继续解码。
- **Length penalty** — EN: Ranking normalizes cumulative score by sequence length to balance short and long candidates. / CN: 排序时会用序列长度归一化累计分数，以平衡短序列与长序列。

## Dependencies / 依赖关系
- **vllm.inputs** — EN: Supplies `tokens_input`, `mm_input`, and `EncoderDecoderInput` constructors used to rebuild requests. / CN: 提供 `tokens_input`、`mm_input` 和 `EncoderDecoderInput` 构造器，用于重建请求。
- **vllm.logprobs.Logprob** — EN: Stores token-level probability annotations attached to each beam position. / CN: 保存 beam 各位置附带的 token 级概率信息。
- **vllm.lora.request.LoRARequest** — EN: Carries optional adapter selection through beam expansion. / CN: 在 beam 扩展过程中携带可选的 LoRA 适配器选择信息。
