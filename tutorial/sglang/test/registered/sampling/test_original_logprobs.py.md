# test_original_logprobs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/sampling/test_original_logprobs.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates original logprobs behavior in SGLang's sampling area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 sampling 领域中与 original logprobs 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: supporting statements / 辅助语句
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
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 16-26: module imports and dependencies / 模块导入与依赖
```python
import os
import random
import unittest

import torch
import torch.nn.functional as F
from transformers import AutoModelForCausalLM, AutoTokenizer

import sglang as sgl
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import DEFAULT_SMALL_MODEL_NAME_FOR_TEST
```
**EN:** This block imports the modules needed by the rest of the file, including `os`, `random`, `unittest`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`, `random`, `unittest`, `torch`。

### Lines 28-49: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=45, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=60, suite="stage-b-test-1-gpu-small-amd")

# ------------------------- Configurable via env ------------------------- #
MODEL_ID = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
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
# ------------------------------------------------

torch.manual_seed(1234)
if torch.cuda.is_available():
    torch.cuda.manual_seed_all(1234)
    torch.backends.cuda.matmul.allow_tf32 = False
    torch.backends.cudnn.allow_tf32 = False
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci, manual_seed.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci, manual_seed 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 52-52: class TestOriginalLogprob declaration / 类 TestOriginalLogprob 声明
```python
class TestOriginalLogprob(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 53-66: setUp setup routine / setUp 初始化流程
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
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 67-71: supporting source context / 辅助源码上下文
```python

    # ---------------------------------------------------------------------
    # Helper: compare one SGLang block (token_logprobs / top_logprobs / ids_logprobs)
    #         against a reference HF log‑prob vector.
    # ---------------------------------------------------------------------
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 72-125: helper routine assert logprobs block equal / 辅助流程 assert logprobs block equal
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
**EN:** This helper encapsulates `assert_logprobs_block_equal` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `assert_logprobs_block_equal`，以便周围测试复用准备、执行或校验逻辑。

### Lines 127-195: test case logprob match / 测试用例 logprob match
```python
    def test_logprob_match(self):
        vocab_size = self.tokenizer.vocab_size

        for env_val in ["True", "False"]:
            with self.subTest(SGLANG_RETURN_ORIGINAL_LOGPROB=env_val):
                os.environ["SGLANG_RETURN_ORIGINAL_LOGPROB"] = env_val

                # ----- SGLang side -----
                sgl_engine = sgl.Engine(
                    model_path=MODEL_ID,
                    skip_tokenizer_init=True,
                    trust_remote_code=True,
                    mem_fraction_static=0.60,
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
**EN:** This test exercises `test_logprob_match` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_logprob_match`。

### Lines 198-199: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestOriginalLogprob`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestOriginalLogprob.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestOriginalLogprob.assert_logprobs_block_equal`: This helper encapsulates `assert_logprobs_block_equal` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `assert_logprobs_block_equal`，以便周围测试复用准备、执行或校验逻辑。
- `TestOriginalLogprob.test_logprob_match`: This test exercises `test_logprob_match` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_logprob_match`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `random`, `unittest`
- **Third-party modules / 第三方模块**: `torch`, `torch.nn.functional`, `transformers`
- **Internal modules / 内部模块**: `sglang`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 199
