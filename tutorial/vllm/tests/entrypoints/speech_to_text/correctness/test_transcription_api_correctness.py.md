# test_transcription_api_correctness.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/speech_to_text/correctness/test_transcription_api_correctness.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers speech-to-text serving. The file defines 1 test(s), 0 fixture(s), and 8 helper/class block(s) to validate this area. / [CN] 该文件覆盖语音转文本服务。它定义了 1 个测试、0 个 fixture，以及 8 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L11-L27)
```python
import asyncio
import io
import time
from statistics import mean, median

import pytest
import soundfile
import torch
from datasets import load_dataset
from evaluate import load
from transformers.models.whisper.english_normalizer import EnglishTextNormalizer

from vllm.multimodal.audio import get_audio_duration
from vllm.tokenizers import get_tokenizer

from ....models.registry import HF_EXAMPLE_MODELS
from ....utils import RemoteOpenAIServer
```
**EN:** Imports standard-library modules such as `asyncio`, `io`, `statistics.mean`, third-party packages like `datasets.load_dataset`, `evaluate.load`, `pytest`, project helpers such as `vllm.multimodal.audio.get_audio_duration`, `vllm.tokenizers.get_tokenizer`, `....models.registry.HF_EXAMPLE_MODELS`.
**CN:** 导入标准库模块（如 `asyncio`、`io`、`statistics.mean`）、第三方包（如 `datasets.load_dataset`、`evaluate.load`、`pytest`）、项目内辅助模块（如 `vllm.multimodal.audio.get_audio_duration`、`vllm.tokenizers.get_tokenizer`、`....models.registry.HF_EXAMPLE_MODELS`）。

### Module setup / 模块级配置: MAX_SEQS_FOR_TRANSCRIPTION_TEST, GPU_UTIL_FOR_TRANSCRIPTION_TEST (L30-L31)
```python
MAX_SEQS_FOR_TRANSCRIPTION_TEST = 8
GPU_UTIL_FOR_TRANSCRIPTION_TEST = 0.5
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MAX_SEQS_FOR_TRANSCRIPTION_TEST`, `GPU_UTIL_FOR_TRANSCRIPTION_TEST`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MAX_SEQS_FOR_TRANSCRIPTION_TEST`、`GPU_UTIL_FOR_TRANSCRIPTION_TEST`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: to_bytes (L34-L38)
```python
def to_bytes(y, sr):
    buffer = io.BytesIO()
    soundfile.write(buffer, y, sr, format="WAV")
    buffer.seek(0)
    return buffer
```
**EN:** This helper encapsulates reusable logic in `to_bytes`. Key inputs are `y`, `sr`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `to_bytes` 中。 关键输入包括 `y`、`sr`。 它把计算得到的状态或辅助对象返回给调用方。

### Module setup / 模块级配置: normalizer_model_info, normalizer_tokenizer, normalizer (L42-L48)
```python
normalizer_model_info = HF_EXAMPLE_MODELS.find_hf_info("openai/whisper-large-v3")
normalizer_tokenizer = get_tokenizer(
    "openai/whisper-large-v3",
    tokenizer_mode=normalizer_model_info.tokenizer_mode,
    trust_remote_code=normalizer_model_info.trust_remote_code,
)
normalizer = EnglishTextNormalizer(normalizer_tokenizer.english_spelling_normalizer)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `normalizer_model_info`, `normalizer_tokenizer`, `normalizer`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `normalizer_model_info`、`normalizer_tokenizer`、`normalizer`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: transcribe_audio (L51-L68)
```python
async def transcribe_audio(client, tokenizer, y, sr):
    # Send loaded audio directly instead of loading from disk,
    # don't account for that time though
    with to_bytes(y, sr) as f:
        start_time = time.perf_counter()
        transcription = await client.audio.transcriptions.create(
            file=f,
            model=tokenizer.name_or_path,
            language="en",
            temperature=0.0,
        )
        end_time = time.perf_counter()
        # NOTE there's no streaming in transcriptions, can't measure ttft
    latency = end_time - start_time
    num_output_tokens = len(
        tokenizer(transcription.text, add_special_tokens=False).input_ids
    )
    return latency, num_output_tokens, transcription.text
```
**EN:** This async helper encapsulates reusable logic in `transcribe_audio`. Key inputs are `client`, `tokenizer`, `y`, `sr`. It drives client-facing request creation through the API surface under test. It returns computed state or helper objects back to the caller.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `transcribe_audio` 中。 关键输入包括 `client`、`tokenizer`、`y`、`sr`。 它通过被测 API 表面触发面向客户端的请求创建流程。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: bound_transcribe (L71-L78)
```python
async def bound_transcribe(sem, client, tokenizer, audio, reference):
    # Use semaphore to limit concurrent requests.
    async with sem:
        result = await transcribe_audio(client, tokenizer, *audio)
        # Normalize *english* output/reference for evaluation.
        out = normalizer(result[2])
        ref = normalizer(reference)
        return result[:2] + (out, ref)
```
**EN:** This async helper encapsulates reusable logic in `bound_transcribe`. Key inputs are `sem`, `client`, `tokenizer`, `audio`, `reference`. It returns computed state or helper objects back to the caller.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `bound_transcribe` 中。 关键输入包括 `sem`、`client`、`tokenizer`、`audio`、`reference`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: process_dataset (L81-L102)
```python
async def process_dataset(model, client, data, concurrent_request):
    sem = asyncio.Semaphore(concurrent_request)

    model_info = HF_EXAMPLE_MODELS.find_hf_info(model)
    tokenizer = get_tokenizer(
        model,
        tokenizer_mode=model_info.tokenizer_mode,
        trust_remote_code=model_info.trust_remote_code,
    )

    # Warmup call as the first `load_audio` server-side is quite slow.
    audio, sr = data[0]["audio"]["array"], data[0]["audio"]["sampling_rate"]
    _ = await bound_transcribe(sem, client, tokenizer, (audio, sr), "")

    tasks: list[asyncio.Task] = []
    for sample in data:
        audio, sr = sample["audio"]["array"], sample["audio"]["sampling_rate"]
        task = asyncio.create_task(
            bound_transcribe(sem, client, tokenizer, (audio, sr), sample["text"])
        )
        tasks.append(task)
    return await asyncio.gather(*tasks)
```
**EN:** This async helper encapsulates reusable logic in `process_dataset`. Key inputs are `model`, `client`, `data`, `concurrent_request`. The body fans out concurrent work via `asyncio.gather`. It drives client-facing request creation through the API surface under test. It returns computed state or helper objects back to the caller.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `process_dataset` 中。 关键输入包括 `model`、`client`、`data`、`concurrent_request`。 函数体通过 `asyncio.gather` 并发展开工作负载。 它通过被测 API 表面触发面向客户端的请求创建流程。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: print_performance_metrics (L105-L120)
```python
def print_performance_metrics(results, total_time):
    latencies = [res[0] for res in results]
    total_tokens = sum([res[1] for res in results])

    total = len(results)
    print(f"Total Requests: {total}")
    print(f"Successful Requests: {len(latencies)}")
    print(f"Average Latency: {mean(latencies):.4f} seconds")
    print(f"Median Latency: {median(latencies):.4f} seconds")
    perc = sorted(latencies)[int(len(latencies) * 0.95) - 1]
    print(f"95th Percentile Latency: {perc:.4f} seconds")
    # Throughput
    req_throughput = len(latencies) / total_time
    print(f"Estimated req_Throughput: {req_throughput:.2f} requests/s")
    throughput = total_tokens / total_time
    print(f"Estimated Throughput: {throughput:.2f} tok/s")
```
**EN:** This helper encapsulates reusable logic in `print_performance_metrics`. Key inputs are `results`, `total_time`.
**CN:** 这个辅助函数将可复用逻辑封装在 `print_performance_metrics` 中。 关键输入包括 `results`、`total_time`。

### Helper / 辅助函数: add_duration (L123-L126)
```python
def add_duration(sample):
    y, sr = sample["audio"]["array"], sample["audio"]["sampling_rate"]
    sample["duration_ms"] = get_audio_duration(y=y, sr=sr) * 1000
    return sample
```
**EN:** This helper encapsulates reusable logic in `add_duration`. Key inputs are `sample`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `add_duration` 中。 关键输入包括 `sample`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: load_hf_dataset (L129-L138)
```python
def load_hf_dataset(dataset_repo: str, split="validation", **hf_kwargs):
    ## Load and filter the dataset
    dataset = load_dataset(dataset_repo, split=split, **hf_kwargs)
    if "duration_ms" not in dataset[0]:
        # compute duration to filter
        dataset = dataset.map(add_duration)

    # Whisper max supported duration
    dataset = dataset.filter(lambda example: example["duration_ms"] < 30000)
    return dataset
```
**EN:** This helper encapsulates reusable logic in `load_hf_dataset`. Key inputs are `dataset_repo`, `split`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `load_hf_dataset` 中。 关键输入包括 `dataset_repo`、`split`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: run_evaluation (L141-L164)
```python
def run_evaluation(
    model: str,
    client,
    dataset,
    max_concurrent_reqs: int,
    n_examples: int = -1,
    print_metrics: bool = True,
):
    if n_examples > 0:
        dataset = dataset.select(range(n_examples))
    start = time.perf_counter()
    results = asyncio.run(process_dataset(model, client, dataset, max_concurrent_reqs))
    end = time.perf_counter()
    total_time = end - start
    print(f"Total Test Time: {total_time:.4f} seconds")
    if print_metrics:
        print_performance_metrics(results, total_time)
    # Compute WER
    predictions = [res[2] for res in results]
    references = [res[3] for res in results]
    wer = load("wer")
    wer_score = 100 * wer.compute(references=references, predictions=predictions)
    print("WER:", wer_score)
    return wer_score
```
**EN:** This helper encapsulates reusable logic in `run_evaluation`. Key inputs are `model`, `client`, `dataset`, `max_concurrent_reqs`, `n_examples`, `print_metrics`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_evaluation` 中。 关键输入包括 `model`、`client`、`dataset`、`max_concurrent_reqs`、`n_examples`、`print_metrics`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_wer_correctness (L170-L218)
```python
@pytest.mark.parametrize(
    "model_config",
    [
        ("openai/whisper-large-v3", 12.744980),
        # CohereASR is used to test the variable encoder length code paths
        ("CohereLabs/cohere-transcribe-03-2026", 11.92),
    ],
)
# Original dataset is 20GB+ in size, hence we use a pre-filtered slice.
@pytest.mark.parametrize(
    "dataset_repo", ["D4nt3/esb-datasets-earnings22-validation-tiny-filtered"]
)
def test_wer_correctness(
    model_config, dataset_repo, n_examples=-1, max_concurrent_request=None
):
    model_name, expected_wer = model_config
    model_info = HF_EXAMPLE_MODELS.find_hf_info(model_name)
    # TODO refactor to use `ASRDataset`
# ... 23 lines omitted for brevity ...
            max_concurrent_request,
            n_examples,
        )

        print(f"Expected WER: {expected_wer}, Actual WER: {wer}")

        if expected_wer:
            torch.testing.assert_close(wer, expected_wer, atol=1e-1, rtol=1e-2)
```
**EN:** This test validates `test_wer_correctness`. It uses parameterization over `model_config`. Key inputs are `model_config`, `dataset_repo`, `n_examples`, `max_concurrent_request`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint.
**CN:** 这个测试验证 `test_wer_correctness`。 它通过参数化组合 `model_config`。 关键输入包括 `model_config`、`dataset_repo`、`n_examples`、`max_concurrent_request`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。

## Key Concepts / 关键概念
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The scenarios exercise speech recognition or transcription endpoints.
  **CN:** 这些场景覆盖语音识别或转录端点。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `asyncio`, `io`, `statistics.mean`, `statistics.median`, `time`
- **Third-party / 第三方**: `datasets.load_dataset`, `evaluate.load`, `pytest`, `soundfile`, `torch`, `transformers.models.whisper.english_normalizer.EnglishTextNormalizer`
- **Project / 项目内**: `vllm.multimodal.audio.get_audio_duration`, `vllm.tokenizers.get_tokenizer`
- **Local relative imports / 本地相对导入**: `....models.registry.HF_EXAMPLE_MODELS`, `....utils.RemoteOpenAIServer`
