# test_gritlm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling/test_gritlm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and language-model behavior. The file defines 5 test(s), 0 fixture(s), and 5 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与语言模型行为。它定义了 5 个测试、0 个 fixture，以及 5 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L11)
```python
import numpy as np
import pytest
from scipy.spatial.distance import cosine

from vllm import LLM, SamplingParams
from vllm.config import ModelConfig

from ....utils import ROCM_ENV_OVERRIDES, ROCM_EXTRA_ARGS, RemoteOpenAIServer
from .embed_utils import run_client_embeddings
```
**EN:** Imports third-party packages like `numpy`, `pytest`, `scipy.spatial.distance.cosine`, project helpers such as `vllm.LLM`, `vllm.SamplingParams`, `vllm.config.ModelConfig`.
**CN:** 导入第三方包（如 `numpy`、`pytest`、`scipy.spatial.distance.cosine`）、项目内辅助模块（如 `vllm.LLM`、`vllm.SamplingParams`、`vllm.config.ModelConfig`）。

### Module setup / 模块级配置: MODEL_NAME, MAX_MODEL_LEN, ATOL (L13-L15)
```python
MODEL_NAME = "parasail-ai/GritLM-7B-vllm"
MAX_MODEL_LEN = 4000
ATOL = 0.002
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `MAX_MODEL_LEN`, `ATOL`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`MAX_MODEL_LEN`、`ATOL`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _arr (L18-L22)
```python
def _arr(arr):
    """
    Convert a list of integers to an array of integers.
    """
    return np.array(arr)
```
**EN:** This helper encapsulates reusable logic in `_arr`. Key inputs are `arr`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_arr` 中。 关键输入包括 `arr`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_find_array (L25-L46)
```python
def test_find_array():
    from vllm.model_executor.models.gritlm import GritLMMeanPool

    model_config = ModelConfig(
        MODEL_NAME,
        runner="pooling",
        dtype="bfloat16",
        seed=0,
    )
    pooling = GritLMMeanPool(model_config=model_config)

    arr = _arr([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])

    assert pooling._find_array(arr, _arr([3, 4, 5]), start_idx=0) == 3
    assert pooling._find_array(arr, _arr([3, 4, 5]), start_idx=1) == 3
    assert pooling._find_array(arr, _arr([3, 4, 5]), start_idx=5) == -1
    assert pooling._find_array(arr, _arr([3, 4, 5]), end_idx=3) == -1
    assert pooling._find_array(arr, _arr([3, 4, 5]), end_idx=4) == 3
    assert pooling._find_array(arr, _arr([3, 5]), start_idx=0) == -1

    with pytest.raises(ValueError):
        pooling._find_array(arr, _arr([3, 4, 5]), start_idx=-1)
```
**EN:** This test validates `test_find_array`. It checks an expected failure path with `pytest.raises`. The main assertion is `pooling._find_array(arr, _arr([3, 4, 5]), start_idx=0) == 3` and `pooling._find_array(arr, _arr([3, 4, 5]), start_idx=1) == 3`.
**CN:** 这个测试验证 `test_find_array`。 它使用 `pytest.raises` 检查预期失败路径。 核心断言是 `pooling._find_array(arr, _arr([3, 4, 5]), start_idx=0) == 3` and `pooling._find_array(arr, _arr([3, 4, 5]), start_idx=1) == 3`。

### Helper / 辅助函数: run_llm_encode (L49-L55)
```python
def run_llm_encode(
    llm: LLM,
    queries: list[str],
    instruction: str,
) -> list[list[float]]:
    outputs = llm.embed([instruction + q for q in queries])
    return [output.outputs.embedding for output in outputs]
```
**EN:** This helper encapsulates reusable logic in `run_llm_encode`. Key inputs are `llm`, `queries`, `instruction`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_llm_encode` 中。 关键输入包括 `llm`、`queries`、`instruction`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: gritlm_instruction (L58-L61)
```python
def gritlm_instruction(instruction):
    return (
        "<|user|>\n" + instruction + "\n<|embed|>\n" if instruction else "<|embed|>\n"
    )
```
**EN:** This helper encapsulates reusable logic in `gritlm_instruction`. Key inputs are `instruction`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `gritlm_instruction` 中。 关键输入包括 `instruction`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: get_test_data (L64-L84)
```python
def get_test_data():
    """
    Grabbed this test data and the expected values from
    README.md in https://github.com/ContextualAI/gritlm
    """
    q_instruction = gritlm_instruction(
        "Given a scientific paper title, retrieve the paper's abstract",
    )
    queries = [
        "Bitcoin: A Peer-to-Peer Electronic Cash System",
        "Generative Representational Instruction Tuning",
    ]

    d_instruction = gritlm_instruction("")
    documents = [
        # ruff: noqa: E501
        "A purely peer-to-peer version of electronic cash would allow online payments to be sent directly from one party to another without going through a financial institution. Digital signatures provide part of the solution, but the main benefits are lost if a trusted third party is still required to prevent double-spending. We propose a solution to the double-spending problem using a peer-to-peer network. The network timestamps transactions by hashing them into an ongoing chain of hash-based proof-of-work, forming a record that cannot be changed without redoing the proof-of-work. The longest chain not only serves as proof of the sequence of events witnessed, but proof that it came from the largest pool of CPU power. As long as a majority of CPU power is controlled by nodes that are not cooperating to attack the network, they'll generate the longest chain and outpace attackers. The network itself requires minimal structure. Messages are broadcast on a best effort basis, and nodes can leave and rejoin the network at will, accepting the longest proof-of-work chain as proof of what happened while they were gone.",
        "All text-based language problems can be reduced to either generation or embedding. Current models only perform well at one or the other. We introduce generative representational instruction tuning (GRIT) whereby a large language model is trained to handle both generative and embedding tasks by distinguishing between them through instructions. Compared to other open models, our resulting GritLM 7B sets a new state of the art on the Massive Text Embedding Benchmark (MTEB) and outperforms all models up to its size on a range of generative tasks. By scaling up further, GritLM 8X7B outperforms all open generative language models that we tried while still being among the best embedding models. Notably, we find that GRIT matches training on only generative or embedding data, thus we can unify both at no performance loss. Among other benefits, the unification via GRIT speeds up Retrieval-Augmented Generation (RAG) by > 60% for long documents, by no longer requiring separate retrieval and generation models. Models, code, etc. are freely available at https://github.com/ContextualAI/gritlm.",
    ]

    return queries, q_instruction, documents, d_instruction
```
**EN:** This helper encapsulates reusable logic in `get_test_data`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_test_data` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: validate_embed_output (L87-L98)
```python
def validate_embed_output(q_rep: list[list[float]], d_rep: list[list[float]]):
    cosine_sim_q0_d0 = 1 - cosine(q_rep[0], d_rep[0])
    assert cosine_sim_q0_d0 == pytest.approx(0.609, abs=ATOL)

    cosine_sim_q0_d1 = 1 - cosine(q_rep[0], d_rep[1])
    assert cosine_sim_q0_d1 == pytest.approx(0.101, abs=ATOL)

    cosine_sim_q1_d0 = 1 - cosine(q_rep[1], d_rep[0])
    assert cosine_sim_q1_d0 == pytest.approx(0.120, abs=ATOL)

    cosine_sim_q1_d1 = 1 - cosine(q_rep[1], d_rep[1])
    assert cosine_sim_q1_d1 == pytest.approx(0.534, abs=ATOL)
```
**EN:** This helper encapsulates reusable logic in `validate_embed_output`. Key inputs are `q_rep`, `d_rep`. The main assertion is `cosine_sim_q0_d0 == pytest.approx(0.609, abs=ATOL)` and `cosine_sim_q0_d1 == pytest.approx(0.101, abs=ATOL)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `validate_embed_output` 中。 关键输入包括 `q_rep`、`d_rep`。 核心断言是 `cosine_sim_q0_d0 == pytest.approx(0.609, abs=ATOL)` and `cosine_sim_q0_d1 == pytest.approx(0.101, abs=ATOL)`。

### Test / 测试: test_gritlm_offline_embedding (L101-L122)
```python
def test_gritlm_offline_embedding(vllm_runner):
    queries, q_instruction, documents, d_instruction = get_test_data()

    with vllm_runner(
        MODEL_NAME,
        runner="pooling",
        max_model_len=MAX_MODEL_LEN,
    ) as vllm_model:
        llm = vllm_model.llm

        d_rep = run_llm_encode(
            llm,
            documents,
            d_instruction,
        )
        q_rep = run_llm_encode(
            llm,
            queries,
            q_instruction,
        )

    validate_embed_output(q_rep, d_rep)
```
**EN:** This test validates `test_gritlm_offline_embedding`. Key inputs are `vllm_runner`.
**CN:** 这个测试验证 `test_gritlm_offline_embedding`。 关键输入包括 `vllm_runner`。

### Test / 测试: test_gritlm_api_server_embedding (L125-L153)
```python
@pytest.mark.asyncio
async def test_gritlm_api_server_embedding():
    queries, q_instruction, documents, d_instruction = get_test_data()

    args = [
        "--runner",
        "pooling",
        "--max_model_len",
        str(MAX_MODEL_LEN),
        *ROCM_EXTRA_ARGS,
    ]

    with RemoteOpenAIServer(MODEL_NAME, args, env_dict=ROCM_ENV_OVERRIDES) as server:
        client_embedding = server.get_async_client()

        d_rep = await run_client_embeddings(
            client_embedding,
            MODEL_NAME,
            documents,
            d_instruction,
        )
        q_rep = await run_client_embeddings(
            client_embedding,
            MODEL_NAME,
            queries,
            q_instruction,
        )

    validate_embed_output(q_rep, d_rep)
```
**EN:** This async test validates `test_gritlm_api_server_embedding`. Relevant pytest markers include `asyncio`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint.
**CN:** 这个异步测试验证 `test_gritlm_api_server_embedding`。 相关的 pytest 标记包括 `asyncio`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。

### Test / 测试: test_gritlm_offline_generate (L156-L169)
```python
def test_gritlm_offline_generate(monkeypatch: pytest.MonkeyPatch, vllm_runner):
    input = "<|user|>\nWhat is the capital of France?\n<|assistant|>\n"

    with vllm_runner(
        MODEL_NAME,
        runner="generate",
        max_model_len=MAX_MODEL_LEN,
    ) as vllm_model:
        llm = vllm_model.llm

        sampling_params = SamplingParams(temperature=0.0, max_tokens=256)
        outputs = llm.generate(input, sampling_params=sampling_params)

    assert outputs[0].outputs[0].text == "The capital of France is Paris."
```
**EN:** This test validates `test_gritlm_offline_generate`. Key inputs are `monkeypatch`, `vllm_runner`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `outputs[0].outputs[0].text == 'The capital of France is Paris.'`.
**CN:** 这个测试验证 `test_gritlm_offline_generate`。 关键输入包括 `monkeypatch`、`vllm_runner`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `outputs[0].outputs[0].text == 'The capital of France is Paris.'`。

### Test / 测试: test_gritlm_api_server_generate (L172-L188)
```python
@pytest.mark.asyncio
async def test_gritlm_api_server_generate():
    input = "<|user|>\nWhat is the capital of France?\n<|assistant|>\n"

    args = ["--runner", "generate", "--max_model_len", str(MAX_MODEL_LEN)]

    with RemoteOpenAIServer(MODEL_NAME, args) as server:
        client_generate = server.get_async_client()

        outputs = await client_generate.completions.create(
            model=MODEL_NAME,
            prompt=input,
            max_tokens=256,
            temperature=0.0,
        )

    assert outputs.choices[0].text == "The capital of France is Paris."
```
**EN:** This async test validates `test_gritlm_api_server_generate`. Relevant pytest markers include `asyncio`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. It drives client-facing request creation through the API surface under test. The main assertion is `outputs.choices[0].text == 'The capital of France is Paris.'`.
**CN:** 这个异步测试验证 `test_gritlm_api_server_generate`。 相关的 pytest 标记包括 `asyncio`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `outputs.choices[0].text == 'The capital of France is Paris.'`。

## Key Concepts / 关键概念
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `numpy`, `pytest`, `scipy.spatial.distance.cosine`
- **Project / 项目内**: `vllm.LLM`, `vllm.SamplingParams`, `vllm.config.ModelConfig`, `vllm.model_executor.models.gritlm.GritLMMeanPool`
- **Local relative imports / 本地相对导入**: `....utils.ROCM_ENV_OVERRIDES`, `....utils.ROCM_EXTRA_ARGS`, `....utils.RemoteOpenAIServer`, `.embed_utils.run_client_embeddings`
