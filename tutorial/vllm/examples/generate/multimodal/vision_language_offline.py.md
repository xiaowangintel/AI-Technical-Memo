# vision_language_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/generate/multimodal/vision_language_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This example shows how to use vLLM for running offline inference with the correct prompt format on vision language models for text generation / 演示多模态生成与预处理流程。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
This example shows how to use vLLM for running offline inference with
the correct prompt format on vision language models for text generation.

For most models, the prompt format should follow corresponding examples
on HuggingFace model repository.
"""
```
**EN:** This example shows how to use vLLM for running offline inference with the correct prompt format on vision language models for text generation.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import os
import random
from contextlib import contextmanager
from typing import NamedTuple

from huggingface_hub import snapshot_download
from transformers import AutoProcessor, AutoTokenizer

from vllm import LLM, EngineArgs, SamplingParams
from vllm.assets.image import ImageAsset
from vllm.assets.video import VideoAsset
from vllm.lora.request import LoRARequest
from vllm.multimodal.image import convert_image_mode
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block loads helper libraries such as os, random, contextlib, typing, and huggingface_hub and pulls in vLLM APIs like vllm, vllm.assets.image, vllm.assets.video, vllm.lora.request, and vllm.multimodal.image.
**CN:** 这一部分加载 os、random、contextlib、typing，以及 huggingface_hub 等辅助库，并引入 vllm、vllm.assets.image、vllm.assets.video、vllm.lora.request，以及 vllm.multimodal.image 等 vLLM API。

### Function: run_qwen3_5_moe
```python
def run_qwen3_5_moe(questions: list[str], modality: str) -> ModelRequestData:
    model_name = "Qwen/Qwen3.5-35B-A3B"

    mm_limit = {"image": 1, "video": 1} if modality == "image+video" else {modality: 1}
    engine_args = EngineArgs(
        model=model_name,
        max_model_len=4096,
        max_num_seqs=5,
        mm_processor_kwargs={
            "min_pixels": 28 * 28,
            "max_pixels": 1280 * 28 * 28,
            "fps": 1,
        },
        limit_mm_per_prompt=mm_limit,
    )

    image_placeholder = "<|vision_start|><|image_pad|><|vision_end|>"
    video_placeholder = "<|vision_start|><|video_pad|><|vision_end|>"

    if modality == "image":
        placeholder = image_placeholder
    elif modality == "video":
        placeholder = video_placeholder
    elif modality == "image+video":
        placeholder = image_placeholder + video_placeholder

    prompts = [
        (
            "<|im_start|>system\nYou are a helpful assistant.<|im_end|>\n"
            f"<|im_start|>user\n{placeholder}"
            f"{question}<|im_end|>\n"
            "<|im_start|>assistant\n"
        )
        for question in questions
    ]

    return ModelRequestData(
        engine_args=engine_args,
        prompts=prompts,
    )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as questions and modality. Key operations include EngineArgs and ModelRequestData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 questions 和 modality 等参数。关键操作包括 EngineArgs 和 ModelRequestData。其返回值会继续传给示例管线的下一阶段。

### Function: run_r_vl
```python
def run_r_vl(questions: list[str], modality: str) -> ModelRequestData:
    assert modality == "image"
    model_name = "YannQi/R-4B"

    prompts = [
        f"<|im_start|>user <image>\n{question}<|im_end|><|im_start|>assistant\n"
        for question in questions
    ]

    engine_args = EngineArgs(
        model=model_name,
        max_model_len=16384,
        limit_mm_per_prompt={modality: 1},
    )

    return ModelRequestData(
        engine_args=engine_args,
        prompts=prompts,
    )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as questions and modality. Key operations include EngineArgs and ModelRequestData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 questions 和 modality 等参数。关键操作包括 EngineArgs 和 ModelRequestData。其返回值会继续传给示例管线的下一阶段。

### Function: run_skyworkr1v
```python
def run_skyworkr1v(questions: list[str], modality: str) -> ModelRequestData:
    assert modality == "image"

    model_name = "Skywork/Skywork-R1V-38B"

    engine_args = EngineArgs(
        model=model_name,
        trust_remote_code=True,
        max_model_len=4096,
        limit_mm_per_prompt={modality: 1},
    )

    tokenizer = AutoTokenizer.from_pretrained(model_name, trust_remote_code=True)
    messages = [
        [{"role": "user", "content": f"<image>\n{question}"}] for question in questions
    ]
    prompts = tokenizer.apply_chat_template(
        messages, tokenize=False, add_generation_prompt=True
    )

    # Stop tokens for SkyworkR1V
    # https://huggingface.co/Skywork/Skywork-R1V-38B/blob/main/conversation.py
    stop_tokens = ["<｜end▁of▁sentence｜>", "<|endoftext|>"]
    stop_token_ids = [tokenizer.convert_tokens_to_ids(i) for i in stop_tokens]

    return ModelRequestData(
        engine_args=engine_args,
        prompts=prompts,
        stop_token_ids=stop_token_ids,
    )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as questions and modality. Key operations include EngineArgs, AutoTokenizer.from_pretrained, tokenizer.apply_chat_template, ModelRequestData, and tokenizer.convert_tokens_to_ids. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 questions 和 modality 等参数。关键操作包括 EngineArgs、AutoTokenizer.from_pretrained、tokenizer.apply_chat_template、ModelRequestData，以及 tokenizer.convert_tokens_to_ids。其返回值会继续传给示例管线的下一阶段。

### Function: run_smolvlm
```python
def run_smolvlm(questions: list[str], modality: str) -> ModelRequestData:
    assert modality == "image"
    model_name = "HuggingFaceTB/SmolVLM2-2.2B-Instruct"

    engine_args = EngineArgs(
        model=model_name,
        max_model_len=8192,
        max_num_seqs=2,
        enforce_eager=True,
        mm_processor_kwargs={
            "max_image_size": {"longest_edge": 384},
        },
        limit_mm_per_prompt={modality: 1},
    )
    prompts = [
        (f"<|im_start|>User:<image>{question}<end_of_utterance>\nAssistant:")
        for question in questions
    ]

    return ModelRequestData(
        engine_args=engine_args,
        prompts=prompts,
    )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as questions and modality. Key operations include EngineArgs and ModelRequestData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 questions 和 modality 等参数。关键操作包括 EngineArgs 和 ModelRequestData。其返回值会继续传给示例管线的下一阶段。

### Function: run_step3
```python
def run_step3(questions: list[str], modality: str) -> ModelRequestData:
    assert modality == "image"

    model_name = "stepfun-ai/step3-fp8"

    # NOTE: Below are verified configurations for step3-fp8
    # on 8xH100 GPUs.
    engine_args = EngineArgs(
        model=model_name,
        max_num_batched_tokens=4096,
        gpu_memory_utilization=0.85,
        tensor_parallel_size=8,
        limit_mm_per_prompt={modality: 1},
        reasoning_parser="step3",
    )

    prompts = [
        "<｜begin▁of▁sentence｜> You are a helpful assistant. <|BOT|>user\n "
        f"<im_patch>{question} <|EOT|><|BOT|>assistant\n<think>\n"
        for question in questions
    ]

    return ModelRequestData(
        engine_args=engine_args,
        prompts=prompts,
    )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as questions and modality. Key operations include EngineArgs and ModelRequestData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 questions 和 modality 等参数。关键操作包括 EngineArgs 和 ModelRequestData。其返回值会继续传给示例管线的下一阶段。

### Function: run_step_vl
```python
def run_step_vl(questions: list[str], modality: str) -> ModelRequestData:
    assert modality == "image"

    model_name = "stepfun-ai/Step3-VL-10B"
    engine_args = EngineArgs(
        model=model_name,
        max_num_batched_tokens=4096,
        tensor_parallel_size=1,
        trust_remote_code=True,
        limit_mm_per_prompt={modality: 1},
        reasoning_parser="deepseek_r1",
    )

    prompts = [
        "<｜begin▁of▁sentence｜> You are a helpful assistant.<|BOT|>user\n "
        f"<im_patch>{question} <|EOT|><|BOT|>assistant\n<think>\n"
        for question in questions
    ]

    return ModelRequestData(
        engine_args=engine_args,
        prompts=prompts,
    )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as questions and modality. Key operations include EngineArgs and ModelRequestData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 questions 和 modality 等参数。关键操作包括 EngineArgs 和 ModelRequestData。其返回值会继续传给示例管线的下一阶段。

### Function: run_tarsier
```python
def run_tarsier(questions: list[str], modality: str) -> ModelRequestData:
    assert modality == "image"
    model_name = "omni-research/Tarsier-7b"

    engine_args = EngineArgs(
        model=model_name,
        trust_remote_code=True,
        max_model_len=4096,
        limit_mm_per_prompt={modality: 1},
    )
    prompts = [(f"USER: <image>\n{question} ASSISTANT:") for question in questions]

    return ModelRequestData(
        engine_args=engine_args,
        prompts=prompts,
    )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as questions and modality. Key operations include EngineArgs and ModelRequestData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 questions 和 modality 等参数。关键操作包括 EngineArgs 和 ModelRequestData。其返回值会继续传给示例管线的下一阶段。

### Function: run_tarsier2
```python
def run_tarsier2(questions: list[str], modality: str) -> ModelRequestData:
    model_name = "omni-research/Tarsier2-Recap-7b"

    mm_limit = {"image": 1, "video": 1} if modality == "image+video" else {modality: 1}
    engine_args = EngineArgs(
        model=model_name,
        max_model_len=4096,
        hf_overrides={
            "architectures": ["Tarsier2ForConditionalGeneration"],
            "model_type": "tarsier2",
        },
        limit_mm_per_prompt=mm_limit,
    )

    image_placeholder = "<|vision_start|><|image_pad|><|vision_end|>"
    video_placeholder = "<|vision_start|><|video_pad|><|vision_end|>"

    if modality == "image":
        placeholder = image_placeholder
    elif modality == "video":
        placeholder = video_placeholder
    elif modality == "image+video":
        placeholder = image_placeholder + video_placeholder

    prompts = [
        (
            "<|im_start|>system\nYou are a helpful assistant.<|im_end|>\n"
            f"<|im_start|>user\n{placeholder}"
            f"{question}<|im_end|>\n"
            "<|im_start|>assistant\n"
        )
        for question in questions
    ]

    return ModelRequestData(
        engine_args=engine_args,
        prompts=prompts,
    )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as questions and modality. Key operations include EngineArgs and ModelRequestData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 questions 和 modality 等参数。关键操作包括 EngineArgs 和 ModelRequestData。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args):
    model = args.model_type
    if model not in model_example_map:
        raise ValueError(f"Model type {model} is not supported.")

    if args.tensor_parallel_size is not None and args.tensor_parallel_size < 1:
        raise ValueError(
            f"tensor_parallel_size must be a positive integer, "
            f"got {args.tensor_parallel_size}"
        )

    modality = args.modality
    mm_input = get_multi_modal_input(args)
    data = mm_input["data"]
    questions = mm_input["questions"]

    req_data = model_example_map[model](questions, modality)

    # Disable other modalities to save memory
    default_limits = {"image": 0, "video": 0, "audio": 0, "vision_chunk": 0}
    # ... key logic omitted for brevity ...
                " and matching UUIDs."
            )
            outputs = llm.generate(
                inputs_with_empty_media,
                sampling_params=sampling_params,
                lora_request=lora_request,
            )
            print("-" * 50)
            for o in outputs:
                generated_text = o.outputs[0].text
                print(generated_text)
                print("-" * 50)
        except Exception as e:
            print(f"Failed to verify cache hits with UUIDs. Error: {e}")
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include print, _mm_uuid, ValueError, llm.generate, and _mm_data. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 print、_mm_uuid、ValueError、llm.generate，以及 _mm_data。其返回值会继续传给示例管线的下一阶段。

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
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。
- **Reasoning traces / 推理轨迹**: The example exposes or consumes explicit reasoning content alongside final answers. / 该示例会在最终答案之外展示或消费显式推理内容。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.assets.image`, `vllm.assets.video`, `vllm.lora.request`, `vllm.multimodal.image`, `vllm.utils.argparse_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `os`, `random`, `contextlib`, `typing`, `huggingface_hub`, `transformers` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `run_qwen3_5_moe`, `run_r_vl`, `run_skyworkr1v`, `run_smolvlm`, `run_step3`, `run_step_vl`, `run_tarsier`, `run_tarsier2` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `EngineArgs`, `ModelRequestData`, `AutoTokenizer.from_pretrained`, `tokenizer.apply_chat_template`, `tokenizer.convert_tokens_to_ids`, `print`, `_mm_uuid`, `ValueError` reveal the main execution path / 这些调用体现了主要执行链路。
