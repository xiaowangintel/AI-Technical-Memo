# test_return_indexer_topk.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/8-gpu-models/test_return_indexer_topk.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on 8 gpu models return indexer topk in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 8 gpu models return indexer topk 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Import dependencies
```python
import asyncio
import logging
import unittest

import aiohttp
import numpy as np

from sglang.srt.state_capturer.indexer_topk import (
    extract_indexer_topk_from_meta_info,
)
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 20-20: Register CI metadata
```python
register_cuda_ci(est_time=270, stage="extra-b", runner_config="8-gpu-h200")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 22-22: Define module constants
```python
DEEPSEEK_V32_MODEL_PATH = "deepseek-ai/DeepSeek-V3.2"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 25-26: Define module constants
```python
NUM_INDEXER_LAYERS = 61
INDEX_TOPK = 2048
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 30-32: Define module constants
```python
INDEX_TOPK_FREQ = 2

logger = logging.getLogger(__name__)
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 35-35: Define class TestReturnIndexerTopk
```python
class TestReturnIndexerTopk(CustomTestCase):
```
**EN:** This declaration introduces the `TestReturnIndexerTopk` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestReturnIndexerTopk` 测试类，并说明它通过继承承担的职责。

### Lines 36-47: Document the class `TestReturnIndexerTopk`
```python
    """Indexer-topk capture e2e test for DSv3.2 (NSA).

    Single server with `--enable-return-indexer-topk` and `index_topk_freq=2`.
    Validates the native `/generate` endpoint only — OpenAI-protocol surface
    (`SglExt.indexer_topk`) not yet wired up; follow-up PR.

    Per response, validates:
      1. Captured tensor decodes to (seqlen-1, num_indexer_layers, index_topk).
      2. Indices are positional sentinels in [-1, +inf); -1 marks padding.
      3. With freq=2, layers L in {2,4,6,...} byte-equal layer L-1's slot —
         regression-protects the skip_topk capture path in forward_mla.py.
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestReturnIndexerTopk`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestReturnIndexerTopk`的设计意图。

### Lines 49-84: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.other_args = [
            "--trust-remote-code",
            "--tp",
            "8",
            "--dp",
            "8",
            "--enable-dp-attention",
            "--enable-return-indexer-topk",
            # Cap KV pool so the indexer-topk host buffer (488 KB / token for
            # V3.2) stays bounded; with the default ~600k tokens × 8 procs the
            # pinned allocation runs into TB-scale and OOMs the CI host.
            "--max-total-tokens",
            "32768",
            "--model-loader-extra-config",
            '{"enable_multithread_load": true, "num_threads": 64}',
            "--json-model-override-args",
            f'{{"index_topk_freq": {INDEX_TOPK_FREQ}}}',
        ]
        cls.sampling_args = {"temperature": 0, "max_new_tokens": 16}
        cls.texts = [
            "What is the capital of France?",
            "Solve: 2 + 3 = ?",
        ]
        cls.process = popen_launch_server(
            DEEPSEEK_V32_MODEL_PATH,
            DEFAULT_URL_FOR_TEST,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=cls.other_args,
        )
        try:
            cls.captured = asyncio.run(cls._collect_async())
        except Exception:
            kill_process_tree(cls.process.pid)
            raise
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 86-88: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 90-93: Run test: indexer topk generate
```python
    def test_indexer_topk_generate(self):
        for topk in self.captured:
            self._check_shape_and_range(topk)
            self._check_skip_topk_equality(topk)
```
**EN:** This test method exercises indexer topk generate and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 indexer topk generate 场景，并验证观测到的行为是否符合预期契约。

### Lines 95-102: Define helper: check shape and range
```python
    def _check_shape_and_range(self, topk: np.ndarray):
        self.assertEqual(topk.ndim, 3)
        seqlen_minus_1, num_layers, topk_size = topk.shape
        self.assertGreater(seqlen_minus_1, 0)
        self.assertEqual(num_layers, NUM_INDEXER_LAYERS)
        self.assertEqual(topk_size, INDEX_TOPK)
        # Indices are token positions; valid values are >= -1 (-1 = padding sentinel).
        self.assertTrue((topk >= -1).all(), f"min index {topk.min()} < -1")
```
**EN:** This helper function encapsulates reusable logic inside `TestReturnIndexerTopk` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestReturnIndexerTopk` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 104-116: Define helper: check skip topk equality
```python
    def _check_skip_topk_equality(self, topk: np.ndarray):
        """Layers L in {2, 4, 6, ...} must byte-equal layer L-1 with freq=2.

        With `skip_topk = max(layer_id - 1, 0) % freq != 0`, freq=2 yields
        skip=True for L >= 2 with L-1 odd → L even (>= 2). The forward_mla.py
        skip-path mirrors prev_topk_indices into layer L's slot.
        """
        for L in range(2, NUM_INDEXER_LAYERS, 2):
            np.testing.assert_array_equal(
                topk[:, L, :],
                topk[:, L - 1, :],
                err_msg=f"layer {L} should reuse layer {L - 1}'s topk (skip_topk path)",
            )
```
**EN:** This helper function encapsulates reusable logic inside `TestReturnIndexerTopk` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestReturnIndexerTopk` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 118-142: Define helper: collect async
```python
    @classmethod
    async def _collect_async(cls):
        async with aiohttp.ClientSession() as session:
            tasks = [
                asyncio.create_task(
                    make_request(
                        session,
                        f"{DEFAULT_URL_FOR_TEST}/generate",
                        {
                            "text": text,
                            "sampling_params": cls.sampling_args,
                            "return_indexer_topk": True,
                        },
                    )
                )
                for text in cls.texts
            ]
            http_results = await asyncio.gather(*tasks)
            # Reshape raw int32 bytes into (seqlen-1, num_indexer_layers, index_topk).
            return [
                extract_indexer_topk_from_meta_info(res).reshape(
                    -1, NUM_INDEXER_LAYERS, INDEX_TOPK
                )
                for res in http_results
            ]
```
**EN:** This helper function encapsulates reusable logic inside `TestReturnIndexerTopk` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestReturnIndexerTopk` 内部调用，从而让场景结构更清晰。

### Lines 145-147: Define helper: make request
```python
async def make_request(session, url, payload):
    async with session.post(url=url, json=payload) as response:
        return await response.json()
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 150-151: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Server lifecycle helpers start the target runtime before assertions begin. / 服务生命周期辅助函数会在断言开始前启动目标运行时。
- Explicit teardown avoids leaked child processes between runs. / 显式清理可以避免多次运行之间遗留子进程。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.state_capturer.indexer_topk`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `aiohttp`, `asyncio`, `logging`, `numpy`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
