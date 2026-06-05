# audio_language_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/generate/multimodal/audio_language_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This example shows how to use vLLM for running offline inference with the correct prompt format on audio language models / 演示多模态生成与预处理流程。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
This example shows how to use vLLM for running offline inference
with the correct prompt format on audio language models.

For most models, the prompt format should follow corresponding examples
on HuggingFace model repository.
"""
```
**EN:** This example shows how to use vLLM for running offline inference with the correct prompt format on audio language models.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import os
from typing import Any, NamedTuple

from huggingface_hub import snapshot_download
from transformers import AutoTokenizer

from vllm import LLM, EngineArgs, SamplingParams
from vllm.assets.audio import AudioAsset
from vllm.lora.request import LoRARequest
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block loads helper libraries such as os, typing, huggingface_hub, and transformers and pulls in vLLM APIs like vllm, vllm.assets.audio, vllm.lora.request, and vllm.utils.argparse_utils.
**CN:** 这一部分加载 os、typing、huggingface_hub，以及 transformers 等辅助库，并引入 vllm、vllm.assets.audio、vllm.lora.request，以及 vllm.utils.argparse_utils 等 vLLM API。

### Function: run_phi4mm
```python
def run_phi4mm(question: str, audio_count: int) -> ModelRequestData:
    """
    Phi-4-multimodal-instruct supports both image and audio inputs. Here, we
    show how to process audio inputs.
    """
    model_path = snapshot_download("microsoft/Phi-4-multimodal-instruct")
    # Since the vision-lora and speech-lora co-exist with the base model,
    # we have to manually specify the path of the lora weights.
    speech_lora_path = os.path.join(model_path, "speech-lora")
    placeholders = "".join([f"<|audio_{i + 1}|>" for i in range(audio_count)])

    prompts = f"<|user|>{placeholders}{question}<|end|><|assistant|>"

    engine_args = EngineArgs(
        model=model_path,
        trust_remote_code=True,
        max_model_len=12800,
        max_num_seqs=2,
        enable_lora=True,
        max_lora_rank=320,
        limit_mm_per_prompt={"audio": audio_count},
    )

    return ModelRequestData(
        engine_args=engine_args,
        prompt=prompts,
        lora_requests=[LoRARequest("speech", 1, speech_lora_path)],
    )
```
**EN:** Phi-4-multimodal-instruct supports both image and audio inputs.. It works with parameters such as question and audio_count. Key operations include snapshot_download, os.path.join, join, EngineArgs, and ModelRequestData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 question 和 audio_count 等参数。关键操作包括 snapshot_download、os.path.join、join、EngineArgs，以及 ModelRequestData。其返回值会继续传给示例管线的下一阶段。

### Function: run_qwen2_audio
```python
def run_qwen2_audio(question: str, audio_count: int) -> ModelRequestData:
    model_name = "Qwen/Qwen2-Audio-7B-Instruct"

    engine_args = EngineArgs(
        model=model_name,
        max_model_len=4096,
        max_num_seqs=5,
        limit_mm_per_prompt={"audio": audio_count},
    )

    audio_in_prompt = "".join(
        [
            f"Audio {idx + 1}: <|audio_bos|><|AUDIO|><|audio_eos|>\n"
            for idx in range(audio_count)
        ]
    )

    prompt = (
        "<|im_start|>system\nYou are a helpful assistant.<|im_end|>\n"
        "<|im_start|>user\n"
        f"{audio_in_prompt}{question}<|im_end|>\n"
        "<|im_start|>assistant\n"
    )

    return ModelRequestData(
        engine_args=engine_args,
        prompt=prompt,
    )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as question and audio_count. Key operations include EngineArgs, join, ModelRequestData, and range. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 question 和 audio_count 等参数。关键操作包括 EngineArgs、join、ModelRequestData，以及 range。其返回值会继续传给示例管线的下一阶段。

### Function: run_qwen2_5_omni
```python
def run_qwen2_5_omni(question: str, audio_count: int):
    model_name = "Qwen/Qwen2.5-Omni-7B"

    engine_args = EngineArgs(
        model=model_name,
        max_model_len=4096,
        max_num_seqs=5,
        limit_mm_per_prompt={"audio": audio_count},
    )

    audio_in_prompt = "".join(
        ["<|audio_bos|><|AUDIO|><|audio_eos|>\n" for idx in range(audio_count)]
    )

    default_system = (
        "You are Qwen, a virtual human developed by the Qwen Team, Alibaba "
        "Group, capable of perceiving auditory and visual inputs, as well as "
        "generating text and speech."
    )

    prompt = (
        f"<|im_start|>system\n{default_system}<|im_end|>\n"
        "<|im_start|>user\n"
        f"{audio_in_prompt}{question}<|im_end|>\n"
        "<|im_start|>assistant\n"
    )
    return ModelRequestData(
        engine_args=engine_args,
        prompt=prompt,
    )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as question and audio_count. Key operations include EngineArgs, join, ModelRequestData, and range. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 question 和 audio_count 等参数。关键操作包括 EngineArgs、join、ModelRequestData，以及 range。其返回值会继续传给示例管线的下一阶段。

### Function: run_qwen3_asr
```python
def run_qwen3_asr(question: str, audio_count: int) -> ModelRequestData:
    model_name = "Qwen/Qwen3-Asr-1.7B"

    audio_in_prompt = "<|audio_start|><|audio_pad|><|audio_end|>\n" * audio_count
    prompt = f"<|im_start|>user\n{audio_in_prompt}<|im_end|>\n<|im_start|>assistant\n"

    engine_args = EngineArgs(
        model=model_name,
        max_model_len=4096,
        max_num_seqs=5,
        limit_mm_per_prompt={"audio": audio_count},
    )

    return ModelRequestData(
        engine_args=engine_args,
        prompt=prompt,
    )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as question and audio_count. Key operations include EngineArgs and ModelRequestData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 question 和 audio_count 等参数。关键操作包括 EngineArgs 和 ModelRequestData。其返回值会继续传给示例管线的下一阶段。

### Function: run_ultravox
```python
def run_ultravox(question: str, audio_count: int) -> ModelRequestData:
    model_name = "fixie-ai/ultravox-v0_5-llama-3_2-1b"

    tokenizer = AutoTokenizer.from_pretrained(model_name)
    messages = [{"role": "user", "content": "<|audio|>\n" * audio_count + question}]
    prompt = tokenizer.apply_chat_template(
        messages, tokenize=False, add_generation_prompt=True
    )

    engine_args = EngineArgs(
        model=model_name,
        max_model_len=4096,
        max_num_seqs=5,
        trust_remote_code=True,
        limit_mm_per_prompt={"audio": audio_count},
    )

    return ModelRequestData(
        engine_args=engine_args,
        prompt=prompt,
    )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as question and audio_count. Key operations include AutoTokenizer.from_pretrained, tokenizer.apply_chat_template, EngineArgs, and ModelRequestData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 question 和 audio_count 等参数。关键操作包括 AutoTokenizer.from_pretrained、tokenizer.apply_chat_template、EngineArgs，以及 ModelRequestData。其返回值会继续传给示例管线的下一阶段。

### Function: run_voxtral
```python
def run_voxtral(question: str, audio_count: int) -> ModelRequestData:
    from mistral_common.audio import Audio
    from mistral_common.protocol.instruct.chunk import (
        AudioChunk,
        RawAudio,
        TextChunk,
    )
    from mistral_common.protocol.instruct.messages import (
        UserMessage,
    )
    from mistral_common.protocol.instruct.request import ChatCompletionRequest
    from mistral_common.tokens.tokenizers.mistral import MistralTokenizer

    model_name = "mistralai/Voxtral-Mini-3B-2507"
    tokenizer = MistralTokenizer.from_hf_hub(model_name)

    engine_args = EngineArgs(
        model=model_name,
        max_model_len=8192,
        max_num_seqs=2,
    # ... key logic omitted for brevity ...
    req = ChatCompletionRequest(messages=messages, model=model_name)

    tokens = tokenizer.encode_chat_completion(req)
    prompt_ids, audios = tokens.tokens, tokens.audios

    audios_and_sr = [(au.audio_array, au.sampling_rate) for au in audios]

    multi_modal_data = {"audio": audios_and_sr}

    return ModelRequestData(
        engine_args=engine_args,
        prompt_token_ids=prompt_ids,
        multi_modal_data=multi_modal_data,
    )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as question and audio_count. Key operations include MistralTokenizer.from_hf_hub, EngineArgs, TextChunk, ChatCompletionRequest, and tokenizer.encode_chat_completion. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 question 和 audio_count 等参数。关键操作包括 MistralTokenizer.from_hf_hub、EngineArgs、TextChunk、ChatCompletionRequest，以及 tokenizer.encode_chat_completion。其返回值会继续传给示例管线的下一阶段。

### Function: run_whisper
```python
def run_whisper(question: str, audio_count: int) -> ModelRequestData:
    assert audio_count == 1, "Whisper only support single audio input per prompt"
    model_name = "openai/whisper-large-v3-turbo"

    prompt = "<|startoftranscript|>"

    engine_args = EngineArgs(
        model=model_name,
        max_model_len=448,
        max_num_seqs=5,
        limit_mm_per_prompt={"audio": audio_count},
    )

    return ModelRequestData(
        engine_args=engine_args,
        prompt=prompt,
    )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as question and audio_count. Key operations include EngineArgs and ModelRequestData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 question 和 audio_count 等参数。关键操作包括 EngineArgs 和 ModelRequestData。其返回值会继续传给示例管线的下一阶段。

### Function: run_fireredlid
```python
def run_fireredlid(question: str, audio_count: int) -> ModelRequestData:
    assert audio_count == 1, "FireRedLID only supports single audio input per prompt"
    model_name = "PatchyTisa/FireRedLID-vllm"

    prompt = "<sos>"

    engine_args = EngineArgs(
        model=model_name,
        max_model_len=8,
        max_num_seqs=5,
        limit_mm_per_prompt={"audio": audio_count},
    )

    return ModelRequestData(
        engine_args=engine_args,
        prompt=prompt,
    )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as question and audio_count. Key operations include EngineArgs and ModelRequestData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 question 和 audio_count 等参数。关键操作包括 EngineArgs 和 ModelRequestData。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args):
    model = args.model_type
    if model not in model_example_map:
        raise ValueError(f"Model type {model} is not supported.")

    if model == "funaudiochat" and not args.model:
        raise ValueError("--model is required when --model-type=funaudiochat")

    if args.tensor_parallel_size is not None and args.tensor_parallel_size < 1:
        raise ValueError(
            f"tensor_parallel_size must be a positive integer, "
            f"got {args.tensor_parallel_size}"
        )

    audio_count = args.num_audios
    req_data = model_example_map[model](
        question_per_audio_count[audio_count], audio_count
    )
    if model == "funaudiochat":
        req_data.engine_args.model = args.model
    # ... key logic omitted for brevity ...
    # Add LoRA request if applicable
    lora_request = (
        req_data.lora_requests * args.num_prompts if req_data.lora_requests else None
    )

    outputs = llm.generate(
        inputs,
        sampling_params=sampling_params,
        lora_request=lora_request,
    )

    for o in outputs:
        generated_text = o.outputs[0].text
        print(generated_text)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include ValueError, get_input, LLM, SamplingParams, and llm.generate. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 ValueError、get_input、LLM、SamplingParams，以及 llm.generate。其返回值会继续传给示例管线的下一阶段。

### Entry point
```python
if __name__ == "__main__":
    args = parse_args()
    main(args)
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to parse_args and main.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 parse_args 和 main。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **OpenAI-compatible API usage / OpenAI 兼容 API 使用**: The example talks to an HTTP endpoint that follows the OpenAI schema. / 该示例通过遵循 OpenAI 协议的 HTTP 端点完成交互。
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。
- **Reasoning traces / 推理轨迹**: The example exposes or consumes explicit reasoning content alongside final answers. / 该示例会在最终答案之外展示或消费显式推理内容。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.assets.audio`, `vllm.lora.request`, `vllm.utils.argparse_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `os`, `typing`, `huggingface_hub`, `transformers` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `run_phi4mm`, `run_qwen2_audio`, `run_qwen2_5_omni`, `run_qwen3_asr`, `run_ultravox`, `run_voxtral`, `run_whisper`, `run_fireredlid` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `snapshot_download`, `os.path.join`, `join`, `EngineArgs`, `ModelRequestData`, `range`, `LoRARequest`, `AutoTokenizer.from_pretrained` reveal the main execution path / 这些调用体现了主要执行链路。
