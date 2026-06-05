# test_no_bad_words.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/samplers/test_no_bad_words.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Make sure bad_words works. / 该文件主要围绕 No Bad Words 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-11)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Make sure bad_words works.

Run `pytest tests/samplers/test_no_bad_words.py`.

"""

from transformers import AutoTokenizer

from vllm import LLM, SamplingParams
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `transformers`, `vllm`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: _generate (lines 14-34)
```python
def _generate(
    llm: LLM,
    prompt: str,
    num_prompt_tokens: int,
    temperature: float = 0,
    bad_words: list[str] | None = None,
) -> list[int]:
    sampling_params = SamplingParams(
        temperature=temperature,
        bad_words=bad_words,
    )

    # [([output_token_ids, ], [output_text, ]), ]
    output = llm.generate([prompt], sampling_params=sampling_params)

    output_token_ids = output[0][0][0][num_prompt_tokens:]
    # [0] first (and only) request output
    # [0] token_ids (not text)
    # [0] first (and only) output completion

    return output_token_ids
```
**EN:** Implements a reusable helper for Generate, reducing duplication across related tests. It coordinates operations such as `SamplingParams`, `llm.generate`.
**CN:** 该辅助函数为 Generate 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `SamplingParams`, `llm.generate` 等操作。

### Class: TestOneTokenBadWord (lines 37-68)
```python
class TestOneTokenBadWord:
    MODEL = "hmellor/tiny-random-LlamaForCausalLM"

    PROMPT = "How old are "
    TARGET_TOKEN = "mn"

    def setup_method(self, method):
        self.tokenizer = AutoTokenizer.from_pretrained(self.MODEL)

        self.num_prompt_tokens = len(self._encode(self.PROMPT))
        self.target_token_id = self._encode(
            self.TARGET_TOKEN, add_special_tokens=False
        )[0]

    def test_one_token_bad_word(self, vllm_runner):
        with vllm_runner(self.MODEL) as llm:
            output_token_ids = self._generate(llm)
            assert output_token_ids[0] == self.target_token_id

            output_token_ids = self._generate(llm, bad_words=[self.TARGET_TOKEN])
            assert self.target_token_id not in output_token_ids

    def _generate(self, llm: LLM, bad_words: list[str] | None = None) -> list[int]:
        return _generate(
            llm=llm,
            prompt=self.PROMPT,
            num_prompt_tokens=self.num_prompt_tokens,
            bad_words=bad_words,
        )

    def _encode(self, prompt: str, add_special_tokens: bool = True) -> list[int]:
        return self.tokenizer(prompt, add_special_tokens=add_special_tokens).input_ids
```
**EN:** Groups related scenarios for Testonetokenbadword. The class contains 1 test method(s) and 3 helper/setup method(s).
**CN:** 该类把与 Testonetokenbadword 相关的场景组织在一起。 其中包含 1 个测试方法，以及 3 个辅助或初始化方法。

### Class: TestTwoTokenBadWord (lines 71-185)
```python
class TestTwoTokenBadWord:
    # Another model (with a different tokenizer behaviour)
    MODEL = "distilbert/distilgpt2"

    PROMPT = "How old are you? I am 10"
    TARGET_TOKEN1 = "years"
    TARGET_TOKEN2 = "old"
    NEIGHBOUR_TOKEN2 = "older"

    def setup_method(self, method):
        self.tokenizer = AutoTokenizer.from_pretrained(
            self.MODEL, add_prefix_space=True
        )

        self.num_prompt_tokens = len(self._encode(self.PROMPT))
        self.target_token_id1 = self._encode(
            self.TARGET_TOKEN1, add_special_tokens=False
        )[0]
        self.target_token_id2 = self._encode(
# ... omitted for brevity ...

            searched = True

            assert len(current_subsequence) == len(subsequence)

            if current_subsequence == subsequence:
                return True

        assert searched, "All subsequences did not match in length..."

        return False

    def _encode(self, prompt: str, add_special_tokens: bool = True) -> list[int]:
        return self.tokenizer(prompt, add_special_tokens=add_special_tokens).input_ids
```
**EN:** Groups related scenarios for Testtwotokenbadword. The class contains 1 test method(s) and 4 helper/setup method(s).
**CN:** 该类把与 Testtwotokenbadword 相关的场景组织在一起。 其中包含 1 个测试方法，以及 4 个辅助或初始化方法。

## Key Concepts / 关键概念
- **Tokenization / 分词与反分词**
  - **EN:** The logic checks how text and token IDs are converted, streamed, or reconstructed.
  - **CN:** 逻辑重点检查文本与 token ID 的转换、流式处理和重建过程。
- **Sampling behavior / 采样行为**
  - **EN:** The assertions target decoding controls and the shape of generated outputs.
  - **CN:** 这些断言关注解码控制参数以及生成结果的形态。
- **Assertions / 断言**
  - **EN:** The file relies on explicit assertions to pin down expected state transitions and outputs.
  - **CN:** 该文件通过显式断言固定预期的状态变化与输出结果。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `transformers`
- **vLLM internal / vLLM 内部依赖**: `vllm`
