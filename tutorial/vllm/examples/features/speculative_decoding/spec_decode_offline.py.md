# spec_decode_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/features/speculative_decoding/spec_decode_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Demonstrates speculative decoding strategies. / 演示推测解码策略。

## Line-by-Line Analysis / 逐行分析
### Imports
```python
from transformers import AutoTokenizer

from vllm import LLM, SamplingParams
from vllm.benchmarks.datasets import add_dataset_parser, get_samples
from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.v1.metrics.reader import Counter, Vector
```
**EN:** This block loads helper libraries such as transformers and pulls in vLLM APIs like vllm, vllm.benchmarks.datasets, vllm.utils.argparse_utils, and vllm.v1.metrics.reader.
**CN:** 这一部分加载 transformers 等辅助库，并引入 vllm、vllm.benchmarks.datasets、vllm.utils.argparse_utils，以及 vllm.v1.metrics.reader 等 vLLM API。

### Top-level setup
```python
QUESTION = "What is the content of each image?"
IMAGE_URLS = [
    "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/duck.jpg",
    "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/lion.jpg",
    "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/flycatcher.jpeg",
    "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/somefish.jpg",
    "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/starfish.jpg",
    "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/snail.jpg",
    "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/thistle.jpg",
    "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/husky.jpg",
    "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/orangetabbycat.jpg",
    "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/guineapig.jpg",
    "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/rabbit.jpg",
    "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/horsepony.jpg",
]
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as QUESTION and IMAGE_URLS.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 QUESTION 和 IMAGE_URLS 等变量。

### Function: get_custom_mm_prompts
```python
def get_custom_mm_prompts(num_prompts):
    prompts = []
    for url in IMAGE_URLS:
        prompts.append(
            [
                {"type": "image_url", "image_url": {"url": url}},
                {"type": "text", "text": QUESTION},
            ]
        )
    if num_prompts > len(IMAGE_URLS):
        prompts = prompts * (num_prompts // len(IMAGE_URLS) + 1)

    return [[{"role": "user", "content": prompt}] for prompt in prompts[:num_prompts]]
```
**EN:** This function encapsulates a reusable step in the example. It works with parameters such as num_prompts. Key operations include len and prompts.append. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 num_prompts 等参数。关键操作包括 len 和 prompts.append。其返回值会继续传给示例管线的下一阶段。

### Function: parse_args
```python
def parse_args():
    parser = FlexibleArgumentParser()
    add_dataset_parser(parser)
    parser.add_argument("--test", action="store_true")
    parser.add_argument(
        "--method",
        type=str,
        default="eagle",
        choices=["ngram", "eagle", "eagle3", "mtp", "draft_model"],
    )
    parser.add_argument("--backend", type=str, default="openai")
    parser.add_argument("--num-spec-tokens", type=int, default=2)
    parser.add_argument("--prompt-lookup-max", type=int, default=5)
    parser.add_argument("--prompt-lookup-min", type=int, default=2)
    parser.add_argument("--tp", type=int, default=1)
    parser.add_argument("--enforce-eager", action="store_true")
    parser.add_argument("--enable-chunked-prefill", action="store_true")
    parser.add_argument("--max-model-len", type=int, default=16384)
    parser.add_argument("--temp", type=float, default=0)
    parser.add_argument("--top-p", type=float, default=1.0)
    parser.add_argument("--top-k", type=int, default=-1)
    parser.add_argument("--print-output", action="store_true")
    parser.add_argument("--output-len", type=int, default=256)
    parser.add_argument("--model-dir", type=str, default=None)
    parser.add_argument("--eagle-dir", type=str, default=None)
    parser.add_argument("--draft-model", type=str, default=None)
    parser.add_argument("--custom-mm-prompts", action="store_true")
    parser.add_argument("--gpu-memory-utilization", type=float, default=0.9)
    parser.add_argument("--disable-padded-drafter-batch", action="store_true")
    parser.add_argument("--max-num-seqs", type=int, default=None)
    parser.add_argument("--parallel-drafting", action="store_true")
    parser.add_argument("--allowed-local-media-path", type=str, default="")
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include parser.add_argument, FlexibleArgumentParser, add_dataset_parser, and parser.parse_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument、FlexibleArgumentParser、add_dataset_parser，以及 parser.parse_args。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args):
    model_dir = args.model_dir
    if args.model_dir is None:
        if args.custom_mm_prompts:
            raise ValueError(
                "custom_mm_prompts requires mm based models"
                "default llama3.1-8b-instruct is not mm based"
                "please specify model_dir to give a mm based model"
            )
        model_dir = "meta-llama/Llama-3.1-8B-Instruct"
    tokenizer = AutoTokenizer.from_pretrained(model_dir)

    if args.custom_mm_prompts:
        prompts = llm_prompts = get_custom_mm_prompts(args.num_prompts)
    else:
        prompts = get_samples(args, tokenizer)
        if args.enable_multimodal_chat:
            llm_prompts = [p.prompt for p in prompts]
        else:
            # add_special_tokens is False to avoid adding bos twice
    # ... key logic omitted for brevity ...
    print(f"total_num_output_tokens: {total_num_output_tokens}")
    print(f"num_drafts: {num_drafts}")
    print(f"num_draft_tokens: {num_draft_tokens}")
    print(f"num_accepted_tokens: {num_accepted_tokens}")
    acceptance_length = 1 + (num_accepted_tokens / num_drafts) if num_drafts > 0 else 1
    print(f"mean acceptance length: {acceptance_length:.2f}")
    print("-" * 50)

    # print acceptance at each token position
    for i in range(len(acceptance_counts)):
        acceptance_rate = acceptance_counts[i] / num_drafts if num_drafts > 0 else 0
        print(f"acceptance at token {i}: {acceptance_rate:.2f}")

    return acceptance_length
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include print, isinstance, len, range, and ValueError. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 print、isinstance、len、range，以及 ValueError。其返回值会继续传给示例管线的下一阶段。

### Entry point
```python
if __name__ == "__main__":
    args = parse_args()
    args.enable_multimodal_chat = args.backend == "openai-chat"

    acceptance_length = main(args)

    if args.test:
        # takes ~30s to run on 1xH100
        assert args.method in ["eagle", "eagle3"]
        assert args.tp == 1
        assert args.num_spec_tokens == 3
        assert args.dataset_name == "hf"
    # ... key logic omitted for brevity ...
            f"acceptance_length {acceptance_length} is not "
            f"within {rtol * 100}% of {expected_acceptance_length}"
        )

        print(
            f"Test passed! Expected AL: "
            f"{expected_acceptance_length}, got {acceptance_length}"
        )
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to parse_args, main, and print.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 parse_args、main，以及 print。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **OpenAI-compatible API usage / OpenAI 兼容 API 使用**: The example talks to an HTTP endpoint that follows the OpenAI schema. / 该示例通过遵循 OpenAI 协议的 HTTP 端点完成交互。
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。
- **Disaggregated execution / 解耦执行**: Prefill, decode, or cache components are split across services or processes. / 预填充、解码或缓存组件被拆分到不同服务或进程中。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.benchmarks.datasets`, `vllm.utils.argparse_utils`, `vllm.v1.metrics.reader` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `transformers` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `get_custom_mm_prompts`, `parse_args`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `len`, `prompts.append`, `parser.add_argument`, `FlexibleArgumentParser`, `add_dataset_parser`, `parser.parse_args`, `print`, `isinstance` reveal the main execution path / 这些调用体现了主要执行链路。
