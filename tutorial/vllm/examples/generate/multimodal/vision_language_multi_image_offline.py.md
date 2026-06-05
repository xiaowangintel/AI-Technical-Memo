# vision_language_multi_image_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/generate/multimodal/vision_language_multi_image_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This example shows how to use vLLM for running offline inference with multi-image input on vision language models for text generation, using the chat template defined by the model / 演示多模态生成与预处理流程。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
This example shows how to use vLLM for running offline inference with
multi-image input on vision language models for text generation,
using the chat template defined by the model.
"""
```
**EN:** This example shows how to use vLLM for running offline inference with multi-image input on vision language models for text generation, using the chat template defined by the model.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import os
from argparse import Namespace
from typing import NamedTuple

from huggingface_hub import snapshot_download
from PIL.Image import Image
from transformers import AutoProcessor, AutoTokenizer

from vllm import LLM, EngineArgs, SamplingParams
from vllm.lora.request import LoRARequest
from vllm.multimodal.utils import fetch_image
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block loads helper libraries such as os, argparse, typing, huggingface_hub, and PIL.Image and pulls in vLLM APIs like vllm, vllm.lora.request, vllm.multimodal.utils, and vllm.utils.argparse_utils.
**CN:** 这一部分加载 os、argparse、typing、huggingface_hub，以及 PIL.Image 等辅助库，并引入 vllm、vllm.lora.request、vllm.multimodal.utils，以及 vllm.utils.argparse_utils 等 vLLM API。

### Function: load_tarsier2
```python
def load_tarsier2(question: str, image_urls: list[str]) -> ModelRequestData:
    model_name = "omni-research/Tarsier2-Recap-7b"

    engine_args = EngineArgs(
        model=model_name,
        trust_remote_code=True,
        max_model_len=32768,
        limit_mm_per_prompt={"image": len(image_urls)},
        hf_overrides={
            "architectures": ["Tarsier2ForConditionalGeneration"],
            "model_type": "tarsier2",
        },
    )

    prompt = (
        "<|im_start|>system\nYou are a helpful assistant.<|im_end|>\n"
        f"<|im_start|>user\n<|vision_start|>{'<|image_pad|>' * len(image_urls)}"
        f"<|vision_end|>{question}<|im_end|>\n"
        "<|im_start|>assistant\n"
    )
    image_data = [fetch_image(url) for url in image_urls]

    return ModelRequestData(
        engine_args=engine_args,
        prompt=prompt,
        image_data=image_data,
    )
```
**EN:** This function loads input data and prepares it for later stages. It works with parameters such as question and image_urls. Key operations include len, EngineArgs, ModelRequestData, and fetch_image. The return value feeds the next stage of the example pipeline.
**CN:** 该函数加载输入数据并为后续阶段做准备。它会处理 question 和 image_urls 等参数。关键操作包括 len、EngineArgs、ModelRequestData，以及 fetch_image。其返回值会继续传给示例管线的下一阶段。

### Function: load_glm4_1v
```python
def load_glm4_1v(question: str, image_urls: list[str]) -> ModelRequestData:
    model_name = "zai-org/GLM-4.1V-9B-Thinking"

    engine_args = EngineArgs(
        model=model_name,
        max_model_len=45082,
        max_num_seqs=2,
        limit_mm_per_prompt={"image": len(image_urls)},
        enforce_eager=True,
    )

    placeholders = [{"type": "image", "image": url} for url in image_urls]
    messages = [
        {
            "role": "user",
            "content": [
                *placeholders,
                {"type": "text", "text": question},
            ],
        }
    ]

    processor = AutoProcessor.from_pretrained(model_name)
    prompt = processor.apply_chat_template(
        messages, tokenize=False, add_generation_prompt=True
    )
    image_data = [fetch_image(url) for url in image_urls]

    return ModelRequestData(
        engine_args=engine_args,
        prompt=prompt,
        image_data=image_data,
    )
```
**EN:** This function loads input data and prepares it for later stages. It works with parameters such as question and image_urls. Key operations include EngineArgs, AutoProcessor.from_pretrained, processor.apply_chat_template, ModelRequestData, and fetch_image. The return value feeds the next stage of the example pipeline.
**CN:** 该函数加载输入数据并为后续阶段做准备。它会处理 question 和 image_urls 等参数。关键操作包括 EngineArgs、AutoProcessor.from_pretrained、processor.apply_chat_template、ModelRequestData，以及 fetch_image。其返回值会继续传给示例管线的下一阶段。

### Function: load_glm4_5v
```python
def load_glm4_5v(question: str, image_urls: list[str]) -> ModelRequestData:
    model_name = "zai-org/GLM-4.5V"

    engine_args = EngineArgs(
        model=model_name,
        max_model_len=32768,
        max_num_seqs=2,
        limit_mm_per_prompt={"image": len(image_urls)},
        enforce_eager=True,
        tensor_parallel_size=4,
    )
    placeholders = [{"type": "image", "image": url} for url in image_urls]
    messages = [
        {
            "role": "user",
            "content": [
                *placeholders,
                {"type": "text", "text": question},
            ],
        }
    ]
    processor = AutoProcessor.from_pretrained(model_name)
    prompt = processor.apply_chat_template(
        messages, tokenize=False, add_generation_prompt=True
    )
    image_data = [fetch_image(url) for url in image_urls]

    return ModelRequestData(
        engine_args=engine_args,
        prompt=prompt,
        image_data=image_data,
    )
```
**EN:** This function loads input data and prepares it for later stages. It works with parameters such as question and image_urls. Key operations include EngineArgs, AutoProcessor.from_pretrained, processor.apply_chat_template, ModelRequestData, and fetch_image. The return value feeds the next stage of the example pipeline.
**CN:** 该函数加载输入数据并为后续阶段做准备。它会处理 question 和 image_urls 等参数。关键操作包括 EngineArgs、AutoProcessor.from_pretrained、processor.apply_chat_template、ModelRequestData，以及 fetch_image。其返回值会继续传给示例管线的下一阶段。

### Function: load_glm4_5v_fp8
```python
def load_glm4_5v_fp8(question: str, image_urls: list[str]) -> ModelRequestData:
    model_name = "zai-org/GLM-4.5V-FP8"

    engine_args = EngineArgs(
        model=model_name,
        max_model_len=32768,
        max_num_seqs=2,
        limit_mm_per_prompt={"image": len(image_urls)},
        enforce_eager=True,
        tensor_parallel_size=4,
    )
    placeholders = [{"type": "image", "image": url} for url in image_urls]
    messages = [
        {
            "role": "user",
            "content": [
                *placeholders,
                {"type": "text", "text": question},
            ],
        }
    ]
    processor = AutoProcessor.from_pretrained(model_name)
    prompt = processor.apply_chat_template(
        messages, tokenize=False, add_generation_prompt=True
    )
    image_data = [fetch_image(url) for url in image_urls]

    return ModelRequestData(
        engine_args=engine_args,
        prompt=prompt,
        image_data=image_data,
    )
```
**EN:** This function loads input data and prepares it for later stages. It works with parameters such as question and image_urls. Key operations include EngineArgs, AutoProcessor.from_pretrained, processor.apply_chat_template, ModelRequestData, and fetch_image. The return value feeds the next stage of the example pipeline.
**CN:** 该函数加载输入数据并为后续阶段做准备。它会处理 question 和 image_urls 等参数。关键操作包括 EngineArgs、AutoProcessor.from_pretrained、processor.apply_chat_template、ModelRequestData，以及 fetch_image。其返回值会继续传给示例管线的下一阶段。

### Function: load_molmo2
```python
def load_molmo2(question: str, image_urls: list[str]) -> ModelRequestData:
    model_name = "allenai/Molmo2-8B"

    engine_args = EngineArgs(
        model=model_name,
        trust_remote_code=True,
        dtype="bfloat16",
        limit_mm_per_prompt={"image": len(image_urls)},
        max_num_batched_tokens=36864,
    )

    placeholders = [{"type": "image", "image": url} for url in image_urls]
    messages = [
        {
            "role": "user",
            "content": [
                *placeholders,
                {"type": "text", "text": question},
            ],
        },
    ]

    processor = AutoProcessor.from_pretrained(model_name)

    prompt = processor.apply_chat_template(
        messages, tokenize=False, add_generation_prompt=True
    )

    image_data = [fetch_image(url) for url in image_urls]

    return ModelRequestData(
        engine_args=engine_args,
        prompt=prompt,
        image_data=image_data,
    )
```
**EN:** This function loads input data and prepares it for later stages. It works with parameters such as question and image_urls. Key operations include EngineArgs, AutoProcessor.from_pretrained, processor.apply_chat_template, ModelRequestData, and fetch_image. The return value feeds the next stage of the example pipeline.
**CN:** 该函数加载输入数据并为后续阶段做准备。它会处理 question 和 image_urls 等参数。关键操作包括 EngineArgs、AutoProcessor.from_pretrained、processor.apply_chat_template、ModelRequestData，以及 fetch_image。其返回值会继续传给示例管线的下一阶段。

### Function: run_generate
```python
def run_generate(
    model,
    question: str,
    image_urls: list[str],
    seed: int,
    tensor_parallel_size: int | None,
):
    req_data = model_example_map[model](question, image_urls)

    engine_args = req_data.engine_args
    engine_args.seed = seed
    if tensor_parallel_size is not None:
        engine_args.tensor_parallel_size = tensor_parallel_size
    llm = LLM.from_engine_args(engine_args)

    sampling_params = SamplingParams(
        temperature=0.0, max_tokens=256, stop_token_ids=req_data.stop_token_ids
    )

    outputs = llm.generate(
        {
            "prompt": req_data.prompt,
            "multi_modal_data": {"image": req_data.image_data},
        },
        sampling_params=sampling_params,
        lora_request=req_data.lora_requests,
    )

    print("-" * 50)
    for o in outputs:
        generated_text = o.outputs[0].text
        print(generated_text)
        print("-" * 50)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as model, question, image_urls, seed, and tensor_parallel_size. Key operations include print, LLM.from_engine_args, SamplingParams, and llm.generate.
**CN:** 该函数编排端到端工作流。它会处理 model、question、image_urls、seed，以及 tensor_parallel_size 等参数。关键操作包括 print、LLM.from_engine_args、SamplingParams，以及 llm.generate。

### Function: run_chat
```python
def run_chat(
    model: str,
    question: str,
    image_urls: list[str],
    seed: int,
    tensor_parallel_size: int | None,
):
    req_data = model_example_map[model](question, image_urls)

    # Disable other modalities to save memory
    default_limits = {"image": 0, "video": 0, "audio": 0}
    req_data.engine_args.limit_mm_per_prompt = default_limits | dict(
        req_data.engine_args.limit_mm_per_prompt or {}
    )

    engine_args = req_data.engine_args
    engine_args.seed = seed
    if tensor_parallel_size is not None:
        engine_args.tensor_parallel_size = tensor_parallel_size
    llm = LLM.from_engine_args(engine_args)
    # ... key logic omitted for brevity ...
                    ),
                ],
            }
        ],
        sampling_params=sampling_params,
        chat_template=req_data.chat_template,
        lora_request=req_data.lora_requests,
    )

    print("-" * 50)
    for o in outputs:
        generated_text = o.outputs[0].text
        print(generated_text)
        print("-" * 50)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as model, question, image_urls, seed, and tensor_parallel_size. Key operations include print, LLM.from_engine_args, llm.chat, dict, and SamplingParams.
**CN:** 该函数编排端到端工作流。它会处理 model、question、image_urls、seed，以及 tensor_parallel_size 等参数。关键操作包括 print、LLM.from_engine_args、llm.chat、dict，以及 SamplingParams。

### Function: parse_args
```python
def parse_args():
    parser = FlexibleArgumentParser(
        description="Demo on using vLLM for offline inference with "
        "vision language models that support multi-image input for text "
        "generation"
    )
    parser.add_argument(
        "--model-type",
        "-m",
        type=str,
        default="phi3_v",
        choices=model_example_map.keys(),
        help='Huggingface "model_type".',
    )
    parser.add_argument(
        "--method",
        type=str,
        default="generate",
        choices=["generate", "chat"],
        help="The method to run in `vllm.LLM`.",
    )
    parser.add_argument(
        "--seed",
        type=int,
        default=0,
        help="Set the seed when initializing `vllm.LLM`.",
    )
    parser.add_argument(
        "--num-images",
        "-n",
        type=int,
        choices=list(range(1, len(IMAGE_URLS) + 1)),  # the max number of images
        default=2,
        help="Number of images to use for the demo.",
    )
    parser.add_argument(
        "--tensor-parallel-size",
        "-tp",
        type=int,
        default=None,
        help="Tensor parallel size to override the model's default setting. ",
    )
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include parser.add_argument, FlexibleArgumentParser, parser.parse_args, model_example_map.keys, and list. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument、FlexibleArgumentParser、parser.parse_args、model_example_map.keys，以及 list。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args: Namespace):
    model = args.model_type
    method = args.method
    seed = args.seed
    tensor_parallel_size = args.tensor_parallel_size

    if tensor_parallel_size is not None and tensor_parallel_size < 1:
        raise ValueError(
            f"tensor_parallel_size must be a positive integer, "
            f"got {tensor_parallel_size}"
        )

    image_urls = IMAGE_URLS[: args.num_images]

    if method == "generate":
        run_generate(model, QUESTION, image_urls, seed, tensor_parallel_size)
    elif method == "chat":
        run_chat(model, QUESTION, image_urls, seed, tensor_parallel_size)
    else:
        raise ValueError(f"Invalid method: {method}")
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include ValueError, run_generate, and run_chat.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 ValueError、run_generate，以及 run_chat。

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
- **vLLM APIs / vLLM API**: `vllm`, `vllm.lora.request`, `vllm.multimodal.utils`, `vllm.utils.argparse_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `os`, `argparse`, `typing`, `huggingface_hub`, `PIL.Image`, `transformers` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `load_tarsier2`, `load_glm4_1v`, `load_glm4_5v`, `load_glm4_5v_fp8`, `load_molmo2`, `run_generate`, `run_chat`, `parse_args` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `len`, `EngineArgs`, `ModelRequestData`, `fetch_image`, `AutoProcessor.from_pretrained`, `processor.apply_chat_template`, `print`, `LLM.from_engine_args` reveal the main execution path / 这些调用体现了主要执行链路。
