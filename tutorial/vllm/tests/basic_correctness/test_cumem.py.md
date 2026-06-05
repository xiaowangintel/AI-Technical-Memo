# test_cumem.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/basic_correctness/test_cumem.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Cumem behavior in the Basic Correctness test area through focused pytest scenarios. It focuses on scenarios such as Python Error, Basic Cumem, Cumem With Cudagraph. / 该文件在 Basic Correctness 测试域中，通过有针对性的 pytest 场景验证 Cumem 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-16)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import asyncio

import pytest
import torch

from vllm import LLM, AsyncEngineArgs, AsyncLLMEngine, SamplingParams
from vllm.device_allocator.cumem import CuMemAllocator
from vllm.platforms import current_platform
from vllm.utils.mem_constants import GiB_bytes

from ..utils import create_new_process_for_each_test, requires_fp8

DEVICE_TYPE = current_platform.device_type
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `asyncio`, `pytest`, `torch`, `vllm`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Test: test_python_error (lines 19-42)
```python
@create_new_process_for_each_test("fork" if not current_platform.is_rocm() else "spawn")
def test_python_error():
    """
    Test if Python error occurs when there's low-level
    error happening from the C++ side.
    """
    allocator = CuMemAllocator.get_instance()
    total_bytes = torch.cuda.mem_get_info()[1]
    alloc_bytes = int(total_bytes * 0.7)
    tensors = []
    with allocator.use_memory_pool():
        # allocate 70% of the total memory
        x = torch.empty(alloc_bytes, dtype=torch.uint8, device=DEVICE_TYPE)
        tensors.append(x)
    # release the memory
    allocator.sleep()

    # allocate more memory than the total memory
    y = torch.empty(alloc_bytes, dtype=torch.uint8, device=DEVICE_TYPE)
    tensors.append(y)
    with pytest.raises(RuntimeError):
        # when the allocator is woken up, it should raise an error
        # because we don't have enough memory
        allocator.wake_up()
```
**EN:** Test if Python error occurs when there's low-level error happening from the C++ side. The body exercises logic via `create_new_process_for_each_test`, `CuMemAllocator.get_instance`, `int` before asserting the expected outcome.
**CN:** 该测试用例验证 Python Error 在特定场景下的行为。 函数体会先通过 `create_new_process_for_each_test`, `CuMemAllocator.get_instance`, `int` 驱动目标逻辑，再断言预期结果。

### Test: test_basic_cumem (lines 45-75)
```python
@create_new_process_for_each_test("fork" if not current_platform.is_rocm() else "spawn")
def test_basic_cumem():
    # some tensors from default memory pool
    shape = (1024, 1024)
    x = torch.empty(shape, device=DEVICE_TYPE)
    x.zero_()

    # some tensors from custom memory pool
    allocator = CuMemAllocator.get_instance()
    with allocator.use_memory_pool():
        # custom memory pool
        y = torch.empty(shape, device=DEVICE_TYPE)
        y.zero_()
        y += 1
        z = torch.empty(shape, device=DEVICE_TYPE)
        z.zero_()
        z += 2

    # they can be used together
    output = x + y + z
    assert torch.allclose(output, torch.ones_like(output) * 3)

    free_bytes = torch.cuda.mem_get_info()[0]
    allocator.sleep()
    free_bytes_after_sleep = torch.cuda.mem_get_info()[0]
    assert free_bytes_after_sleep > free_bytes
    allocator.wake_up()

    # they can be used together
    output = x + y + z
    assert torch.allclose(output, torch.ones_like(output) * 3)
```
**EN:** Checks Basic Cumem under a focused test scenario. The body exercises logic via `create_new_process_for_each_test`, `torch.empty`, `x.zero_` before asserting the expected outcome.
**CN:** 该测试用例验证 Basic Cumem 在特定场景下的行为。 函数体会先通过 `create_new_process_for_each_test`, `torch.empty`, `x.zero_` 驱动目标逻辑，再断言预期结果。

### Test: test_cumem_with_cudagraph (lines 78-120)
```python
@create_new_process_for_each_test("fork" if not current_platform.is_rocm() else "spawn")
def test_cumem_with_cudagraph():
    allocator = CuMemAllocator.get_instance()
    with allocator.use_memory_pool():
        weight = torch.eye(1024, device=DEVICE_TYPE)
    with allocator.use_memory_pool(tag="discard"):
        cache = torch.empty(1024, 1024, device=DEVICE_TYPE)

    def model(x):
        out = x @ weight
        cache[: out.size(0)].copy_(out)
        return out + 1

    x = torch.empty(128, 1024, device=DEVICE_TYPE)

    # warmup
    model(x)

    # capture cudagraph
# ... omitted for brevity ...
    # after waking up, the content in the weight tensor
    # should be restored, but the content in the cache tensor
    # should be discarded

    # this operation is also compatible with cudagraph

    x.random_()
    model_graph.replay()

    # cache content is as expected
    assert torch.allclose(x, cache[: x.size(0)])

    # output content is as expected
    assert torch.allclose(y, x + 1)
```
**EN:** Checks Cumem With Cudagraph under a focused test scenario. The body exercises logic via `create_new_process_for_each_test`, `CuMemAllocator.get_instance`, `torch.empty` before asserting the expected outcome.
**CN:** 该测试用例验证 Cumem With Cudagraph 在特定场景下的行为。 函数体会先通过 `create_new_process_for_each_test`, `CuMemAllocator.get_instance`, `torch.empty` 驱动目标逻辑，再断言预期结果。

### Test: test_end_to_end (lines 123-177)
```python
@create_new_process_for_each_test("fork" if not current_platform.is_rocm() else "spawn")
@pytest.mark.parametrize(
    "model",
    [
        # sleep mode with safetensors
        "hmellor/tiny-random-LlamaForCausalLM",
        # sleep mode with pytorch checkpoint
        "facebook/opt-125m",
    ],
)
def test_end_to_end(model: str):
    free, total = torch.cuda.mem_get_info()
    used_bytes_baseline = total - free  # in case other process is running
    llm = LLM(model, enable_sleep_mode=True)
    prompt = "How are you?"
    sampling_params = SamplingParams(temperature=0, max_tokens=10)
    output = llm.generate(prompt, sampling_params)

    # the benefit of `llm.sleep(level=2)` is mainly CPU memory usage,
# ... omitted for brevity ...
    llm.wake_up(tags=["weights"])

    free_gpu_bytes_wake_up_w, total = torch.cuda.mem_get_info()
    used_bytes = total - free_gpu_bytes_wake_up_w - used_bytes_baseline

    # should just reallocate memory for weights (1B model, ~2GiB weights)
    assert used_bytes < 10 * GiB_bytes

    # now allocate kv cache memory
    llm.wake_up(tags=["kv_cache"])
    output3 = llm.generate(prompt, sampling_params)

    # cmp output
    assert output[0].outputs[0].text == output3[0].outputs[0].text
```
**EN:** Checks End To End under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `create_new_process_for_each_test`, `pytest.mark.parametrize`, `torch.cuda.mem_get_info` before asserting the expected outcome.
**CN:** 该测试用例验证 End To End 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `create_new_process_for_each_test`, `pytest.mark.parametrize`, `torch.cuda.mem_get_info` 驱动目标逻辑，再断言预期结果。

### Test: test_deep_sleep (lines 180-208)
```python
@create_new_process_for_each_test()
def test_deep_sleep():
    model = "hmellor/tiny-random-LlamaForCausalLM"
    free, total = torch.cuda.mem_get_info()
    used_bytes_baseline = total - free  # in case other process is running
    llm = LLM(model, enable_sleep_mode=True)
    prompt = "How are you?"
    sampling_params = SamplingParams(temperature=0, max_tokens=10)
    output = llm.generate(prompt, sampling_params)

    # Put the engine to deep sleep
    llm.sleep(level=2)

    free_gpu_bytes_after_sleep, total = torch.cuda.mem_get_info()
    used_bytes = total - free_gpu_bytes_after_sleep - used_bytes_baseline
    assert used_bytes < 3 * GiB_bytes

    llm.wake_up(tags=["weights"])
    llm.collective_rpc("reload_weights")
    free_gpu_bytes_wake_up_w, total = torch.cuda.mem_get_info()
    used_bytes = total - free_gpu_bytes_wake_up_w - used_bytes_baseline
    assert used_bytes < 4 * GiB_bytes

    # now allocate kv cache and cuda graph memory
    llm.wake_up(tags=["kv_cache"])
    output2 = llm.generate(prompt, sampling_params)

    # cmp output
    assert output[0].outputs[0].text == output2[0].outputs[0].text
```
**EN:** Checks Deep Sleep under a focused test scenario. The body exercises logic via `create_new_process_for_each_test`, `torch.cuda.mem_get_info`, `LLM` before asserting the expected outcome.
**CN:** 该测试用例验证 Deep Sleep 在特定场景下的行为。 函数体会先通过 `create_new_process_for_each_test`, `torch.cuda.mem_get_info`, `LLM` 驱动目标逻辑，再断言预期结果。

### Test: test_deep_sleep_async (lines 211-247)
```python
@create_new_process_for_each_test()
def test_deep_sleep_async():
    async def test():
        model = "hmellor/tiny-random-LlamaForCausalLM"
        free, total = torch.cuda.mem_get_info()
        used_bytes_baseline = total - free  # in case other process is running
        engine_args = AsyncEngineArgs(
            model=model,
            enable_sleep_mode=True,
        )

        llm = AsyncLLMEngine.from_engine_args(engine_args)
        prompt = "How are you?"
        sampling_params = SamplingParams(temperature=0, max_tokens=10)
        outputs = llm.generate(prompt, sampling_params, request_id="test_request_id1")
        async for output in outputs:
            pass

        # Put the engine to deep sleep
# ... omitted for brevity ...
        free_gpu_bytes_wake_up_w, total = torch.cuda.mem_get_info()
        used_bytes = total - free_gpu_bytes_wake_up_w - used_bytes_baseline
        assert used_bytes < 4 * GiB_bytes

        # now allocate kv cache and cuda graph memory
        await llm.wake_up(tags=["kv_cache"])
        outputs2 = llm.generate(prompt, sampling_params, request_id="test_request_id2")
        async for output2 in outputs2:
            pass

        # cmp output
        assert output.outputs[0].text == output2.outputs[0].text

    asyncio.run(test())
```
**EN:** Checks Deep Sleep Async under a focused test scenario. The body exercises logic via `create_new_process_for_each_test`, `asyncio.run`, `torch.cuda.mem_get_info` before asserting the expected outcome.
**CN:** 该测试用例验证 Deep Sleep Async 在特定场景下的行为。 函数体会先通过 `create_new_process_for_each_test`, `asyncio.run`, `torch.cuda.mem_get_info` 驱动目标逻辑，再断言预期结果。

### Test: test_deep_sleep_fp8_kvcache (lines 250-282)
```python
@requires_fp8
def test_deep_sleep_fp8_kvcache():
    model = "Qwen/Qwen2-0.5B"
    used_bytes_baseline = current_platform.get_current_memory_usage()

    llm = LLM(model, enable_sleep_mode=True, kv_cache_dtype="fp8")
    prompt = "How are you?"
    sampling_params = SamplingParams(temperature=0, max_tokens=10)
    output = llm.generate(prompt, sampling_params)

    # Put the engine to deep sleep
    llm.sleep(level=2)

    used_bytes = current_platform.get_current_memory_usage() - used_bytes_baseline

    # Rocm uses more memory for CudaGraphs, so we add 2 GiB more for the threshold
    rocm_extra_mem_bytes = 2 * GiB_bytes if current_platform.is_rocm() else 0
    mem_threshold_after_sleep = 3 * GiB_bytes + rocm_extra_mem_bytes
    assert used_bytes < mem_threshold_after_sleep

    llm.wake_up(tags=["weights"])
    llm.collective_rpc("reload_weights")

    used_bytes = current_platform.get_current_memory_usage() - used_bytes_baseline
    mem_threshold_after_wake_up = 4 * GiB_bytes + rocm_extra_mem_bytes
    assert used_bytes < mem_threshold_after_wake_up

    # now allocate kv cache and cuda graph memory
    llm.wake_up(tags=["kv_cache"])
    output2 = llm.generate(prompt, sampling_params)

    # cmp output
    assert output[0].outputs[0].text == output2[0].outputs[0].text
```
**EN:** Checks Deep Sleep FP8 Kvcache under a focused test scenario. The body exercises logic via `current_platform.get_current_memory_usage`, `LLM`, `SamplingParams` before asserting the expected outcome.
**CN:** 该测试用例验证 Deep Sleep FP8 Kvcache 在特定场景下的行为。 函数体会先通过 `current_platform.get_current_memory_usage`, `LLM`, `SamplingParams` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Quantization / 量化**
  - **EN:** The file covers low-precision model paths, kernels, or configuration rules for quantized inference.
  - **CN:** 该文件覆盖低精度模型路径、相关内核或量化推理配置规则。
- **Sampling behavior / 采样行为**
  - **EN:** The assertions target decoding controls and the shape of generated outputs.
  - **CN:** 这些断言关注解码控制参数以及生成结果的形态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`
- **Third-party / 第三方依赖**: `pytest`, `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.device_allocator.cumem`, `vllm.platforms`, `vllm.utils.mem_constants`
- **Local test utilities / 本地测试辅助**: `..utils`
