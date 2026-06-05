# test_embedding_lora_support.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/lora/test_embedding_lora_support.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates embedding lora support behavior in SGLang's lora area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 lora 领域中与 embedding lora support 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: supporting source context / 辅助源码上下文
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 14-19: supporting statements / 辅助语句
```python
"""
Unit tests for LoRA support in embedding models.

Validates that EmbeddingReqInput correctly handles LoRA fields through
normalization, batching, and request splitting.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 21-32: module imports and dependencies / 模块导入与依赖
```python
import multiprocessing as mp
import unittest

import numpy as np
import torch

from sglang.srt.entrypoints.openai.protocol import EmbeddingRequest
from sglang.srt.managers.io_struct import EmbeddingReqInput, TokenizedEmbeddingReqInput
from sglang.srt.sampling.sampling_params import SamplingParams
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.runners import SRTRunner
from sglang.test.test_utils import DEFAULT_PORT_FOR_SRT_TEST_RUNNER, CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `multiprocessing`, `unittest`, `numpy`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `multiprocessing`, `unittest`, `numpy`, `torch`。

### Lines 33-34: supporting source context / 辅助源码上下文
```python

# Test configuration (same model/LoRA as test_lora_hf_sgl_logprob_diff.py)
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 35-43: CI registration and metadata / CI 注册与元数据
```python
MODEL_PATH = "meta-llama/Llama-2-7b-hf"
LORA_PATH = "yushengsu/sglang_lora_logprob_diff_without_tuning"
LORA_BACKEND = "triton"
SIMILARITY_THRESHOLD = 0.9999

register_cuda_ci(
    est_time=150,
    suite="nightly-1-gpu",
)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 46-46: class TestEmbeddingLoraSupport declaration / 类 TestEmbeddingLoraSupport 声明
```python
class TestEmbeddingLoraSupport(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 47-47: supporting statements / 辅助语句
```python
    """Test LoRA support in embedding request structures."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 49-65: test case engine encode validates enable lora / 测试用例 engine encode validates enable lora
```python
    def test_engine_encode_validates_enable_lora(self):
        """Test Engine.encode() validates enable_lora before processing lora_path."""
        # Use a simple non-gated model for this validation test
        with SRTRunner(
            MODEL_PATH,
            torch_dtype=torch.float16,
            model_type="embedding",
            port=DEFAULT_PORT_FOR_SRT_TEST_RUNNER,
        ) as runner:
            # Should raise ValueError because enable_lora was not set for the server
            with self.assertRaises(ValueError) as context:
                runner.engine.encode(prompt="Test", lora_path="fake-adapter")

            error_msg = str(context.exception)
            self.assertIn("not enabled", error_msg.lower())
            self.assertIn("--enable-lora", error_msg)
            self.assertIn("fake-adapter", error_msg)
```
**EN:** Test Engine.encode() validates enable_lora before processing lora_path. This test exercises `test_engine_encode_validates_enable_lora` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Engine.encode() validates enable_lora before processing lora_path. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_engine_encode_validates_enable_lora`。

### Lines 67-99: test case embedding lora fields / 测试用例 embedding lora fields
```python
    def test_embedding_lora_fields(self):
        """Test LoRA fields exist and work correctly across all embedding structures."""
        # EmbeddingReqInput: fields exist, normalization expands single to batch, indexing works
        req = EmbeddingReqInput(
            text=["Hello", "World"], lora_path="my-adapter", lora_id=["id1", "id2"]
        )
        self.assertIsNotNone(req.lora_path)
        req.normalize_batch_and_arguments()
        self.assertEqual(req.lora_path, ["my-adapter", "my-adapter"])
        self.assertEqual(req[0].lora_path, "my-adapter")
        self.assertEqual(req[1].lora_id, "id2")

        # EmbeddingReqInput: mismatched list length raises error
        req = EmbeddingReqInput(text=["Hello", "World", "Test"], lora_path=["adapter1"])
        with self.assertRaises(ValueError):
            req.normalize_batch_and_arguments()

        # TokenizedEmbeddingReqInput and EmbeddingRequest have lora fields
        tokenized = TokenizedEmbeddingReqInput(
            input_text="Hello",
            input_ids=[1, 2, 3],
            image_inputs={},
            token_type_ids=[],
            sampling_params=SamplingParams(),
            lora_id="my-lora-id",
        )
        self.assertEqual(tokenized.lora_id, "my-lora-id")
        self.assertEqual(
            EmbeddingRequest(
                input="Hello", model="test", lora_path="adapter"
            ).lora_path,
            "adapter",
        )
```
**EN:** Test LoRA fields exist and work correctly across all embedding structures. This test exercises `test_embedding_lora_fields` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test LoRA fields exist and work correctly across all embedding structures. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding_lora_fields`。

### Lines 102-102: class TestEmbeddingLoraHFComparison declaration / 类 TestEmbeddingLoraHFComparison 声明
```python
class TestEmbeddingLoraHFComparison(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 103-103: supporting statements / 辅助语句
```python
    """Compare HF+LoRA vs SGLang+LoRA embedding outputs."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 105-148: helper routine get hf embedding with lora / 辅助流程 get hf embedding with lora
```python
    @classmethod
    def get_hf_embedding_with_lora(cls, model_path, lora_path, texts, torch_dtype):
        """Get embeddings from HuggingFace model with LoRA adapter."""
        from peft import PeftModel
        from transformers import AutoModelForCausalLM, AutoTokenizer

        # Load base model as CausalLM to match adapter's expected structure
        base_model = AutoModelForCausalLM.from_pretrained(
            model_path,
            torch_dtype=torch_dtype,
            trust_remote_code=True,
        ).cuda()

        # Load LoRA adapter
        model = PeftModel.from_pretrained(base_model, lora_path)
        model.eval()

        tokenizer = AutoTokenizer.from_pretrained(model_path, trust_remote_code=True)
        if tokenizer.pad_token is None:
            tokenizer.pad_token = tokenizer.eos_token

        with torch.no_grad():
            inputs = tokenizer(
                texts, padding=True, truncation=True, return_tensors="pt"
            ).to("cuda")

            # Access the inner model (CausalLM wraps the base model)
            outputs = model.model(**inputs, output_hidden_states=True)
            hidden_states = outputs.hidden_states[-1]

            # Last token pooling with L2 normalization (matching SGLang)
            attention_mask = inputs["attention_mask"]
            last_token_indices = attention_mask.sum(dim=1) - 1
            batch_size = hidden_states.shape[0]
            embeddings = hidden_states[
                torch.arange(batch_size, device="cuda"), last_token_indices
            ]
            embeddings = embeddings / embeddings.norm(dim=1, keepdim=True)

        # Cleanup
        del model, base_model
        torch.cuda.empty_cache()

        return embeddings.cpu().numpy()
```
**EN:** Get embeddings from HuggingFace model with LoRA adapter. This helper encapsulates `get_hf_embedding_with_lora` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Get embeddings from HuggingFace model with LoRA adapter. 该辅助函数封装了 `get_hf_embedding_with_lora`，以便周围测试复用准备、执行或校验逻辑。

### Lines 150-170: helper routine get sglang embedding with lora / 辅助流程 get sglang embedding with lora
```python
    @classmethod
    def get_sglang_embedding_with_lora(cls, model_path, lora_path, texts, torch_dtype):
        """Get embeddings from SGLang with LoRA adapter."""
        with SRTRunner(
            model_path,
            torch_dtype=torch_dtype,
            model_type="embedding",
            lora_paths=[lora_path],
            lora_backend=LORA_BACKEND,
            port=DEFAULT_PORT_FOR_SRT_TEST_RUNNER,
            trust_remote_code=True,
            mem_fraction_static=0.88,
        ) as runner:
            # Call engine.encode directly with lora_path
            response = runner.engine.encode(prompt=texts, lora_path=lora_path)
            if isinstance(response, list):
                embeddings = [r["embedding"] for r in response]
            else:
                embeddings = [response["embedding"]]

        return np.array(embeddings)
```
**EN:** Get embeddings from SGLang with LoRA adapter. This helper encapsulates `get_sglang_embedding_with_lora` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Get embeddings from SGLang with LoRA adapter. 该辅助函数封装了 `get_sglang_embedding_with_lora`，以便周围测试复用准备、执行或校验逻辑。

### Lines 172-175: method cosine similarity / 方法 cosine similarity
```python
    @staticmethod
    def cosine_similarity(a, b):
        """Compute cosine similarity between vectors."""
        return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))
```
**EN:** Compute cosine similarity between vectors. This block implements `cosine_similarity` and captures one focused piece of the module's behavior.
**CN:** Compute cosine similarity between vectors. 该代码块实现 `cosine_similarity`，承担模块行为中的一个聚焦逻辑片段。

### Lines 177-224: test case embedding lora hf sglang similarity / 测试用例 embedding lora hf sglang similarity
```python
    def test_embedding_lora_hf_sglang_similarity(self):
        """Test that HF+LoRA and SGLang+LoRA produce similar embeddings."""
        test_texts = [
            "Hello world",
            "This is a test sentence for embedding comparison",
        ]

        print(f"\nModel: {MODEL_PATH}")
        print(f"LoRA: {LORA_PATH}")

        # Get SGLang embeddings first (before HF loads model into GPU)
        # This order matches test_lora_hf_sgl_logprob_diff.py and avoids OOM
        print("\nGetting SGLang embeddings...")
        sglang_embeddings = self.get_sglang_embedding_with_lora(
            MODEL_PATH, LORA_PATH, test_texts, torch.float16
        )

        # Clear GPU memory
        torch.cuda.empty_cache()

        # Get HF embeddings
        print("Getting HF embeddings...")
        hf_embeddings = self.get_hf_embedding_with_lora(
            MODEL_PATH, LORA_PATH, test_texts, torch.float16
        )

        # Compare embeddings
        print("\nHF vs SGLang LoRA Embedding Comparison:")
        similarities = []
        for i, (hf_emb, sgl_emb) in enumerate(zip(hf_embeddings, sglang_embeddings)):
            sim = self.cosine_similarity(hf_emb, sgl_emb)
            similarities.append(sim)
            print(f"  Text {i}: cosine similarity = {sim:.6f}")
            self.assertGreater(
                sim,
                SIMILARITY_THRESHOLD,
                f"Text {i} similarity {sim:.6f} below threshold {SIMILARITY_THRESHOLD}",
            )

        avg_similarity = np.mean(similarities)
        print(f"  Average similarity: {avg_similarity:.6f}")
        print(f"  Threshold: {SIMILARITY_THRESHOLD}")

        self.assertGreater(
            avg_similarity,
            SIMILARITY_THRESHOLD,
            f"Average similarity {avg_similarity:.4f} below threshold {SIMILARITY_THRESHOLD}",
        )
```
**EN:** Test that HF+LoRA and SGLang+LoRA produce similar embeddings. This test exercises `test_embedding_lora_hf_sglang_similarity` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that HF+LoRA and SGLang+LoRA produce similar embeddings. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding_lora_hf_sglang_similarity`。

### Lines 227-232: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    try:
        mp.set_start_method("spawn")
    except RuntimeError:
        pass
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestEmbeddingLoraSupport`: Test LoRA support in embedding request structures. / 用于组织相关测试、夹具或辅助方法。
- `TestEmbeddingLoraHFComparison`: Compare HF+LoRA vs SGLang+LoRA embedding outputs. / 用于组织相关测试、夹具或辅助方法。
- `TestEmbeddingLoraSupport.test_engine_encode_validates_enable_lora`: Test Engine.encode() validates enable_lora before processing lora_path. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_engine_encode_validates_enable_lora`。
- `TestEmbeddingLoraSupport.test_embedding_lora_fields`: Test LoRA fields exist and work correctly across all embedding structures. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding_lora_fields`。
- `TestEmbeddingLoraHFComparison.get_hf_embedding_with_lora`: Get embeddings from HuggingFace model with LoRA adapter. / 该辅助函数封装了 `get_hf_embedding_with_lora`，以便周围测试复用准备、执行或校验逻辑。
- `TestEmbeddingLoraHFComparison.get_sglang_embedding_with_lora`: Get embeddings from SGLang with LoRA adapter. / 该辅助函数封装了 `get_sglang_embedding_with_lora`，以便周围测试复用准备、执行或校验逻辑。
- `TestEmbeddingLoraHFComparison.cosine_similarity`: Compute cosine similarity between vectors. / 该代码块实现 `cosine_similarity`，承担模块行为中的一个聚焦逻辑片段。
- `TestEmbeddingLoraHFComparison.test_embedding_lora_hf_sglang_similarity`: Test that HF+LoRA and SGLang+LoRA produce similar embeddings. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding_lora_hf_sglang_similarity`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `multiprocessing`, `unittest`
- **Third-party modules / 第三方模块**: `numpy`, `torch`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.managers.io_struct`, `sglang.srt.sampling.sampling_params`, `sglang.test.ci.ci_register`, `sglang.test.runners`, `sglang.test.test_utils`

- **Total lines / 总行数**: 232
