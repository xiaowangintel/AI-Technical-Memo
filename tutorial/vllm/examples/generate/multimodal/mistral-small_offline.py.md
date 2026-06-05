# mistral-small_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/generate/multimodal/mistral-small_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Demonstrates multimodal generation and preprocessing flows. / 演示多模态生成与预处理流程。

## Line-by-Line Analysis / 逐行分析
### Imports
```python
import argparse

from vllm import LLM
from vllm.sampling_params import SamplingParams
from vllm.assets.image import ImageAsset
from vllm.multimodal.utils import encode_image_url
```
**EN:** This block loads helper libraries such as argparse and pulls in vLLM APIs like vllm, vllm.sampling_params, vllm.assets.image, and vllm.multimodal.utils.
**CN:** 这一部分加载 argparse 等辅助库，并引入 vllm、vllm.sampling_params、vllm.assets.image，以及 vllm.multimodal.utils 等 vLLM API。

### Function: run_simple_demo
```python
def run_simple_demo(args: argparse.Namespace):
    model_name = "mistralai/Mistral-Small-3.1-24B-Instruct-2503"
    sampling_params = SamplingParams(max_tokens=8192)

    llm = LLM(
        model=model_name,
        tokenizer_mode="mistral" if args.format == "mistral" else "hf",
        config_format="mistral" if args.format == "mistral" else "hf",
        load_format="mistral" if args.format == "mistral" else "hf",
        limit_mm_per_prompt={"image": 1},
        max_model_len=4096,
        max_num_seqs=2,
        tensor_parallel_size=2,
        mm_processor_cache_gb=0 if args.disable_mm_processor_cache else 4,
    )

    prompt = "Describe this image in one sentence."

    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": prompt},
                {
                    "type": "image_url",
                    "image_url": {
                        "url": encode_image_url(ImageAsset("cherry_blossom").pil_image)
                    },
                },
            ],
        },
    ]
    outputs = llm.chat(messages, sampling_params=sampling_params)
    print("-" * 50)
    print(outputs[0].outputs[0].text)
    print("-" * 50)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include print, SamplingParams, LLM, llm.chat, and encode_image_url.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 print、SamplingParams、LLM、llm.chat，以及 encode_image_url。

### Function: run_advanced_demo
```python
def run_advanced_demo(args: argparse.Namespace):
    model_name = "mistralai/Mistral-Small-3.1-24B-Instruct-2503"
    max_img_per_msg = 3
    max_tokens_per_img = 4096

    sampling_params = SamplingParams(max_tokens=8192, temperature=0.7)
    llm = LLM(
        model=model_name,
        tokenizer_mode="mistral" if args.format == "mistral" else "hf",
        config_format="mistral" if args.format == "mistral" else "hf",
        load_format="mistral" if args.format == "mistral" else "hf",
        limit_mm_per_prompt={"image": max_img_per_msg},
        max_model_len=max_img_per_msg * max_tokens_per_img,
        tensor_parallel_size=2,
        mm_processor_cache_gb=0 if args.disable_mm_processor_cache else 4,
    )

    prompt = "Describe the following image."

    url_1 = "https://huggingface.co/datasets/patrickvonplaten/random_img/resolve/main/yosemite.png"
    # ... key logic omitted for brevity ...
            "content": "More details please and answer only in French!.",
        },
        {
            "role": "user",
            "content": [
                {"type": "image_url", "image_url": {"url": url_3}},
            ],
        },
    ]

    outputs = llm.chat(messages=messages, sampling_params=sampling_params)
    print("-" * 50)
    print(outputs[0].outputs[0].text)
    print("-" * 50)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include print, SamplingParams, LLM, and llm.chat.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 print、SamplingParams、LLM，以及 llm.chat。

### Function: parse_args
```python
def parse_args():
    parser = argparse.ArgumentParser(
        description="Run a demo in simple or advanced mode."
    )

    parser.add_argument(
        "mode",
        choices=["simple", "advanced"],
        help="Specify the demo mode: 'simple' or 'advanced'",
    )

    parser.add_argument(
        "--format",
        choices=["mistral", "hf"],
        default="mistral",
        help="Specify the format of the model to load.",
    )

    parser.add_argument(
        "--disable-mm-processor-cache",
        action="store_true",
        help="If True, disables caching of multi-modal processor.",
    )
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include parser.add_argument, argparse.ArgumentParser, and parser.parse_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument、argparse.ArgumentParser，以及 parser.parse_args。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main():
    args = parse_args()

    if args.mode == "simple":
        print("Running simple demo...")
        run_simple_demo(args)
    elif args.mode == "advanced":
        print("Running advanced demo...")
        run_advanced_demo(args)
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include print, parse_args, run_simple_demo, and run_advanced_demo.
**CN:** 该函数编排端到端工作流。关键操作包括 print、parse_args、run_simple_demo，以及 run_advanced_demo。

### Entry point
```python
if __name__ == "__main__":
    main()
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to main.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 main。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.sampling_params`, `vllm.assets.image`, `vllm.multimodal.utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `argparse` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `run_simple_demo`, `run_advanced_demo`, `parse_args`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `print`, `SamplingParams`, `LLM`, `llm.chat`, `encode_image_url`, `ImageAsset`, `parser.add_argument`, `argparse.ArgumentParser` reveal the main execution path / 这些调用体现了主要执行链路。
