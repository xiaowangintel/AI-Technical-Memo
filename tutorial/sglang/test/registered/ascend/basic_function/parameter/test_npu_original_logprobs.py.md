# test_npu_original_logprobs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/basic_function/parameter/test_npu_original_logprobs.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on parameter npu original logprobs in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 parameter npu original logprobs 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Document the module
```python
"""Test original log probability alignment between SGLang and Hugging Face.

This test suite verifies the correctness of the `origin_logprobs` output (temperature=1)
and the `logprobs` output (temperature=0.5) in SGLang by comparing it against
raw logit-based probabilities computed directly from a reference Hugging Face model.

The test covers the following scenarios:
- Next-token prediction: Verifies that the log probability of the next token from
  SGLang matches the Hugging Face model.
- Top-k logprobs: Ensures that the top-k original logprobs returned by SGLang are
  consistent with Hugging Face outputs.
- Specified token IDs: Confirms that the original logprobs for specific token IDs
  match the values computed from Hugging Face logits.
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 16-26: Import dependencies
```python
import os
import random
import unittest

import torch
import torch.nn.functional as F
from transformers import AutoModelForCausalLM, AutoTokenizer

import sglang as sgl
from sglang.test.ascend.test_ascend_utils import LLAMA_3_2_1B_INSTRUCT_WEIGHTS_PATH
from sglang.test.ci.ci_register import register_npu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 29-40: Define module constants
```python
MODEL_ID = LLAMA_3_2_1B_INSTRUCT_WEIGHTS_PATH

PROMPTS = [
    "Hello, my name is",
    "The future of AI is",
    "The president of the United States is",
    "The capital of France is ",
]
TOP_LOGPROBS_NUM = 50
NUM_RANDOM_TOKEN_IDS = 10
RTOL = 0.20
ATOL = 0.00
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 43-43: Implement expr logic
```python
torch.manual_seed(1234)
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 44-47: Implement control flow
```python
if torch.cuda.is_available():
    torch.cuda.manual_seed_all(1234)
    torch.backends.cuda.matmul.allow_tf32 = False
    torch.backends.cudnn.allow_tf32 = False
```
**EN:** This block adds supporting control flow around the main test or helper logic.
**CN:** 该代码块围绕主要测试或辅助逻辑补充了控制流。

### Lines 49-49: Register CI metadata
```python
register_npu_ci(est_time=400, suite="nightly-1-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 52-52: Define class TestOriginalLogprob
```python
class TestOriginalLogprob(unittest.TestCase):
```
**EN:** This declaration introduces the `TestOriginalLogprob` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestOriginalLogprob` 测试类，并说明它通过继承承担的职责。

### Lines 53-58: Document the class `TestOriginalLogprob`
```python
    """Testcase: Verify the behavior and log probability alignment of SGLang under two configurations of the environment variable `SGLANG_RETURN_ORIGINAL_LOGPROB` (True/False),
        by comparing SGLang's output with reference values from Hugging Face.

    [Test Category] Parameter
    [Test Target] SGLANG_RETURN_ORIGINAL_LOGPROB
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestOriginalLogprob`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestOriginalLogprob`的设计意图。

### Lines 60-73: Prepare per-test state
```python
    def setUp(self):
        # ----- HF side (float32 weights) -----
        self.tokenizer = AutoTokenizer.from_pretrained(MODEL_ID, padding_side="right")
        self.hf_model = AutoModelForCausalLM.from_pretrained(
            MODEL_ID, torch_dtype=torch.float32, device_map="auto"
        )

        # Shared sampling parameters
        self.sampling_params = {
            "temperature": 0.5,  # SGLang uses 0.5, but original logprobs are used 1.0
            "top_p": 1.0,
            "top_k": 10,
            "max_new_tokens": 1,
        }
```
**EN:** This setup method initializes the state required before each individual test executes.
**CN:** 该初始化方法会在每个独立测试执行前准备所需状态。

### Lines 79-132: Define helper: assert logprobs block equal
```python
    def assert_logprobs_block_equal(
        self,
        hf_log_probs: torch.Tensor,  # [V]
        token_log_probs: list,
        top_log_probs: list,
        ids_log_probs: list,
        random_token_ids: list,
        tag: str = "",
    ):
        vals, idxs, _ = zip(*token_log_probs)
        sgl_vals = torch.tensor(vals, device=self.hf_model.device, dtype=torch.float32)
        sgl_idxs = torch.tensor(idxs, device=self.hf_model.device, dtype=torch.long)
        hf_vals = hf_log_probs[sgl_idxs]

        self.assertTrue(
            torch.allclose(hf_vals, sgl_vals, rtol=RTOL, atol=ATOL),
            msg=f"[{tag}] token‑level mismatch at indices {sgl_idxs.tolist()}",
        )

        hf_topk, _ = torch.topk(hf_log_probs, k=TOP_LOGPROBS_NUM, dim=-1)

        sgl_topk = torch.tensor(
            [float(t[0]) for t in top_log_probs[0] if t and t[0] is not None][
                :TOP_LOGPROBS_NUM
            ],
            dtype=torch.float32,
            device=self.hf_model.device,
        )

        k = min(hf_topk.numel(), sgl_topk.numel())
        self.assertTrue(
            torch.allclose(hf_topk[:k], sgl_topk[:k], rtol=RTOL, atol=ATOL),
            msg=f"[{tag}] top‑k mismatch",
        )

        indices = torch.tensor(
            random_token_ids, dtype=torch.long, device=hf_log_probs.device
        )

        hf_token_ids = hf_log_probs[indices]

        sgl_token_ids = torch.tensor(
            [v for v, _, _ in ids_log_probs[0]],
            device=self.hf_model.device,
            dtype=torch.float32,
        )
        self.assertTrue(
            torch.allclose(hf_token_ids, sgl_token_ids, rtol=RTOL, atol=ATOL),
            msg=f"[{tag}] token‑IDs mismatch",
        )

        # Optional: print max abs diff for quick diagnostics
        max_diff = torch.max(torch.abs(hf_vals - sgl_vals)).item()
        print(f"[{tag}] max|diff| token‑level = {max_diff:.4f}")
```
**EN:** This helper function encapsulates reusable logic inside `TestOriginalLogprob` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestOriginalLogprob` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 134-182: Run test: logprob match (part 1)
```python
    def test_logprob_match(self):
        vocab_size = self.tokenizer.vocab_size

        for env_val in ["True", "False"]:
            with self.subTest(return_original_logprob=env_val):
                os.environ["SGLANG_RETURN_ORIGINAL_LOGPROB"] = env_val

                # ----- SGLang side -----
                sgl_engine = sgl.Engine(
                    model_path=MODEL_ID,
                    skip_tokenizer_init=True,
                    trust_remote_code=True,
                    mem_fraction_static=0.60,
                    attention_backend="ascend",
                    disable_cuda_graph=True,
                )

                for prompt in PROMPTS:
                    random_token_ids = sorted(
                        random.sample(range(vocab_size), NUM_RANDOM_TOKEN_IDS)
                    )

                    enc = self.tokenizer(prompt, return_tensors="pt")
                    input_ids = enc["input_ids"].to(self.hf_model.device)
                    attn_mask = enc["attention_mask"].to(self.hf_model.device)

                    with torch.inference_mode():
                        hf_out = self.hf_model(
                            input_ids=input_ids,
                            attention_mask=attn_mask,
                            return_dict=True,
                        )
                    logits = hf_out.logits[:, -1, :]  # [1, V]
                    hf_log_probs = F.log_softmax(
                        logits.float() / self.sampling_params["temperature"], dim=-1
                    )[0]
                    hf_original_log_probs = F.log_softmax(logits.float(), dim=-1)[0]

                    outputs = sgl_engine.generate(
                        input_ids=input_ids[0].tolist(),
                        sampling_params=self.sampling_params,
                        return_logprob=True,
                        top_logprobs_num=TOP_LOGPROBS_NUM,
                        token_ids_logprob=random_token_ids,
                    )

                    if isinstance(outputs, list):
                        outputs = outputs[0]
                    meta = outputs["meta_info"]
```
**EN:** This test method exercises logprob match and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 logprob match 场景，并验证观测到的行为是否符合预期契约。

### Lines 184-204: Run test: logprob match (part 2)
```python
                    # Check original logprobs only if enabled
                    if env_val.lower() == "true":
                        self.assert_logprobs_block_equal(
                            hf_log_probs=hf_original_log_probs,
                            token_log_probs=meta["output_token_logprobs"],
                            top_log_probs=meta["output_top_logprobs"],
                            ids_log_probs=meta["output_token_ids_logprobs"],
                            random_token_ids=random_token_ids,
                            tag=f"Original logprobs SGLang vs HF: {prompt} ({env_val})",
                        )
                    else:
                        # Always check regular logprobs
                        self.assert_logprobs_block_equal(
                            hf_log_probs=hf_log_probs,
                            token_log_probs=meta["output_token_logprobs"],
                            top_log_probs=meta["output_top_logprobs"],
                            ids_log_probs=meta["output_token_ids_logprobs"],
                            random_token_ids=random_token_ids,
                            tag=f"logprobs SGLang vs HF: {prompt} ({env_val})",
                        )
                sgl_engine.shutdown()
```
**EN:** This test method exercises logprob match and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 logprob match 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 207-208: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang`, `sglang.test.ascend.test_ascend_utils`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `os`, `random`, `torch`, `torch.nn.functional`, `transformers`, `unittest`
- Notable symbols / 关键符号: `unittest.main`
