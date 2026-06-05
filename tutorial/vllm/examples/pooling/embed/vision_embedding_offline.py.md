# vision_embedding_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/embed/vision_embedding_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This example shows how to use vLLM for running offline inference with the correct prompt format on vision language models for multimodal embedding / 演示使用池化模型生成嵌入。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
This example shows how to use vLLM for running offline inference with
the correct prompt format on vision language models for multimodal embedding.

For most models, the prompt format should follow corresponding examples
on HuggingFace model repository.
"""
```
**EN:** This example shows how to use vLLM for running offline inference with the correct prompt format on vision language models for multimodal embedding.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import argparse
from pathlib import Path

from PIL.Image import Image

from vllm import LLM
from vllm.multimodal.utils import fetch_image
from vllm.utils.print_utils import print_embeddings
```
**EN:** This block loads helper libraries such as argparse, pathlib, and PIL.Image and pulls in vLLM APIs like vllm, vllm.multimodal.utils, and vllm.utils.print_utils.
**CN:** 这一部分加载 argparse、pathlib，以及 PIL.Image 等辅助库，并引入 vllm、vllm.multimodal.utils，以及 vllm.utils.print_utils 等 vLLM API。

### Top-level setup
```python
ROOT_DIR = Path(__file__).parent.parent.parent
EMBED_TEMPLATE_DIR = ROOT_DIR / "pooling/embed/template/"

image_url = "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/cat_snow.jpg"
text = "A cat standing in the snow."
multi_modal_data = {"image": fetch_image(image_url)}
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as ROOT_DIR, EMBED_TEMPLATE_DIR, image_url, text, and multi_modal_data. It also performs early helper calls such as Path and fetch_image.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 ROOT_DIR、EMBED_TEMPLATE_DIR、image_url、text，以及 multi_modal_data 等变量。它还会提前执行 Path 和 fetch_image 等辅助调用。

### Function: run_clip
```python
def run_clip(seed: int):
    llm = LLM(
        model="openai/clip-vit-base-patch32",
        runner="pooling",
        limit_mm_per_prompt={"image": 1},
        seed=seed,
    )

    print("Text embedding output:")
    outputs = llm.embed(text, use_tqdm=False)
    print_embeddings(outputs[0].outputs.embedding)

    print("Image embedding output:")
    prompt = ""  # For image input, make sure that the prompt text is empty
    outputs = llm.embed(
        {
            "prompt": prompt,
            "multi_modal_data": multi_modal_data,
        },
        use_tqdm=False,
    )
    print_embeddings(outputs[0].outputs.embedding)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as seed. Key operations include print, llm.embed, print_embeddings, and LLM.
**CN:** 该函数编排端到端工作流。它会处理 seed 等参数。关键操作包括 print、llm.embed、print_embeddings，以及 LLM。

### Function: run_e5_v
```python
def run_e5_v(seed: int):
    llm = LLM(
        model="royokong/e5-v",
        runner="pooling",
        max_model_len=4096,
        limit_mm_per_prompt={"image": 1},
        seed=seed,
    )

    llama3_template = "<|start_header_id|>user<|end_header_id|>\n\n{}<|eot_id|><|start_header_id|>assistant<|end_header_id|>\n\n \n"  # noqa: E501

    print("Text embedding output:")
    prompt_text = llama3_template.format(
        f"{text}\nSummary above sentence in one word: "
    )
    outputs = llm.embed(prompt_text, use_tqdm=False)
    print_embeddings(outputs[0].outputs.embedding)

    print("Image embedding output:")
    prompt_image = llama3_template.format("<image>\nSummary above image in one word: ")
    outputs = llm.embed(
        {
            "prompt": prompt_image,
            "multi_modal_data": multi_modal_data,
        },
        use_tqdm=False,
    )
    print_embeddings(outputs[0].outputs.embedding)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as seed. Key operations include print, llama3_template.format, llm.embed, print_embeddings, and LLM.
**CN:** 该函数编排端到端工作流。它会处理 seed 等参数。关键操作包括 print、llama3_template.format、llm.embed、print_embeddings，以及 LLM。

### Function: run_qwen3_vl
```python
def run_qwen3_vl(seed: int):
    try:
        from qwen_vl_utils import smart_resize
    except ModuleNotFoundError:
        print(
            "WARNING: `qwen-vl-utils` not installed, input images will not "
            "be automatically resized. This can cause different results "
            "comparing with HF repo's example. "
            "You can enable this functionality by `pip install qwen-vl-utils`."
        )
        smart_resize = None

    if smart_resize is not None:

        def post_process_image(image: Image) -> Image:
            width, height = image.size
            resized_height, resized_width = smart_resize(
                height,
                width,
                factor=32,
    # ... key logic omitted for brevity ...
        },
        use_tqdm=False,
    )
    print_embeddings(outputs[0].outputs.embedding)

    print("Image+Text embedding output:")
    outputs = llm.embed(
        {
            "prompt": prompt_image_text,
            "multi_modal_data": multi_modal_data,
        },
        use_tqdm=False,
    )
    print_embeddings(outputs[0].outputs.embedding)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as seed. Key operations include print, llm.embed, print_embeddings, LLM, and post_process_image. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 seed 等参数。关键操作包括 print、llm.embed、print_embeddings、LLM，以及 post_process_image。其返回值会继续传给示例管线的下一阶段。

### Function: run_siglip
```python
def run_siglip(seed: int):
    llm = LLM(
        model="google/siglip-base-patch16-224",
        runner="pooling",
        limit_mm_per_prompt={"image": 1},
        seed=seed,
    )

    print("Text embedding output:")
    outputs = llm.embed(text, use_tqdm=False)
    print_embeddings(outputs[0].outputs.embedding)

    print("Image embedding output:")
    prompt = ""  # For image input, make sure that the prompt text is empty
    outputs = llm.embed(
        {
            "prompt": prompt,
            "multi_modal_data": multi_modal_data,
        },
        use_tqdm=False,
    )
    print_embeddings(outputs[0].outputs.embedding)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as seed. Key operations include print, llm.embed, print_embeddings, and LLM.
**CN:** 该函数编排端到端工作流。它会处理 seed 等参数。关键操作包括 print、llm.embed、print_embeddings，以及 LLM。

### Function: run_vlm2vec_phi3v
```python
def run_vlm2vec_phi3v(seed: int):
    llm = LLM(
        model="TIGER-Lab/VLM2Vec-Full",
        runner="pooling",
        max_model_len=4096,
        trust_remote_code=True,
        mm_processor_kwargs={"num_crops": 4},
        limit_mm_per_prompt={"image": 1},
        seed=seed,
    )
    image_token = "<|image_1|>"

    print("Text embedding output:")
    prompt_text = f"Find me an everyday image that matches the given caption: {text}"
    outputs = llm.embed(prompt_text, use_tqdm=False)
    print_embeddings(outputs[0].outputs.embedding)

    print("Image embedding output:")
    prompt_image = f"{image_token} Find a day-to-day image that looks similar to the provided image."  # noqa: E501
    outputs = llm.embed(
        {
            "prompt": prompt_image,
            "multi_modal_data": multi_modal_data,
        },
        use_tqdm=False,
    )
    print_embeddings(outputs[0].outputs.embedding)

    print("Image+Text embedding output:")
    prompt_image_text = (
        f"{image_token} Represent the given image with the following question: {text}"  # noqa: E501
    )
    outputs = llm.embed(
        {
            "prompt": prompt_image_text,
            "multi_modal_data": multi_modal_data,
        },
        use_tqdm=False,
    )
    print_embeddings(outputs[0].outputs.embedding)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as seed. Key operations include print, llm.embed, print_embeddings, and LLM.
**CN:** 该函数编排端到端工作流。它会处理 seed 等参数。关键操作包括 print、llm.embed、print_embeddings，以及 LLM。

### Function: run_vlm2vec_qwen2vl
```python
def run_vlm2vec_qwen2vl(seed: int):
    # vLLM does not support LoRA adapters on multi-modal encoder,
    # so we merge the weights first
    from huggingface_hub.constants import HF_HUB_CACHE
    from peft import PeftConfig, PeftModel
    from transformers import AutoModelForImageTextToText, AutoProcessor

    from vllm.entrypoints.chat_utils import load_chat_template

    model_id = "TIGER-Lab/VLM2Vec-Qwen2VL-2B"

    base_model = AutoModelForImageTextToText.from_pretrained(model_id)
    lora_model = PeftModel.from_pretrained(
        base_model,
        model_id,
        config=PeftConfig.from_pretrained(model_id),
    )
    model = lora_model.merge_and_unload().to(dtype=base_model.dtype)
    model._hf_peft_config_loaded = False  # Needed to save the merged model

    # ... key logic omitted for brevity ...
    print_embeddings(outputs[0].outputs.embedding)

    print("Image+Text embedding output:")
    prompt_image_text = (
        f"{image_token} Represent the given image with the following question: {text}"  # noqa: E501
    )
    outputs = llm.embed(
        {
            "prompt": prompt_image_text,
            "multi_modal_data": multi_modal_data,
        },
        use_tqdm=False,
    )
    print_embeddings(outputs[0].outputs.embedding)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as seed. Key operations include print, llm.embed, print_embeddings, AutoModelForImageTextToText.from_pretrained, and PeftModel.from_pretrained.
**CN:** 该函数编排端到端工作流。它会处理 seed 等参数。关键操作包括 print、llm.embed、print_embeddings、AutoModelForImageTextToText.from_pretrained，以及 PeftModel.from_pretrained。

### Top-level setup
```python
model_example_map = {
    "clip": run_clip,
    "e5_v": run_e5_v,
    "qwen3_vl": run_qwen3_vl,
    "siglip": run_siglip,
    "vlm2vec_phi3v": run_vlm2vec_phi3v,
    "vlm2vec_qwen2vl": run_vlm2vec_qwen2vl,
}
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as model_example_map.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 model_example_map 等变量。

### Function: parse_args
```python
def parse_args():
    parser = argparse.ArgumentParser(
        "Script to run a specified VLM through vLLM offline api."
    )
    parser.add_argument(
        "--model",
        "-m",
        type=str,
        default="vlm2vec_phi3v",
        choices=model_example_map.keys(),
        help="The name of the embedding model.",
    )
    parser.add_argument(
        "--seed",
        type=int,
        default=0,
        help="Set the seed when initializing `vllm.LLM`.",
    )
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include parser.add_argument, argparse.ArgumentParser, parser.parse_args, and model_example_map.keys. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument、argparse.ArgumentParser、parser.parse_args，以及 model_example_map.keys。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args):
    model_example_map[args.model](args.seed)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。

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
- **Representation extraction / 表示提取**: The output is a dense or token-level representation rather than free-form text. / 输出是稠密向量或词元级表示，而不是自由文本。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.multimodal.utils`, `vllm.utils.print_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `argparse`, `pathlib`, `PIL.Image` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `run_clip`, `run_e5_v`, `run_qwen3_vl`, `run_siglip`, `run_vlm2vec_phi3v`, `run_vlm2vec_qwen2vl`, `parse_args`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `Path`, `fetch_image`, `print`, `llm.embed`, `print_embeddings`, `LLM`, `llama3_template.format`, `post_process_image` reveal the main execution path / 这些调用体现了主要执行链路。
