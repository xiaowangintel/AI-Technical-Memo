# jina_embeddings_v4_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/token_embed/jina_embeddings_v4_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Demonstrates token-level embedding extraction. / 演示词元级嵌入提取。

## Line-by-Line Analysis / 逐行分析
### Imports
```python
import torch

from vllm import LLM
from vllm.config import PoolerConfig
from vllm.inputs import TextPrompt
from vllm.multimodal.utils import fetch_image
```
**EN:** This block loads helper libraries such as torch and pulls in vLLM APIs like vllm, vllm.config, vllm.inputs, and vllm.multimodal.utils.
**CN:** 这一部分加载 torch 等辅助库，并引入 vllm、vllm.config、vllm.inputs，以及 vllm.multimodal.utils 等 vLLM API。

### Function: main
```python
def main():
    # Initialize model
    model = LLM(
        model="jinaai/jina-embeddings-v4-vllm-text-matching",
        pooler_config=PoolerConfig(task="token_embed"),
        runner="pooling",
        max_model_len=1024,
        gpu_memory_utilization=0.8,
    )

    # Create text prompts
    text1 = "Ein wunderschöner Sonnenuntergang am Strand"
    text1_prompt = TextPrompt(prompt=f"Query: {text1}")

    text2 = "浜辺に沈む美しい夕日"
    text2_prompt = TextPrompt(prompt=f"Query: {text2}")

    # Create image prompt
    image = fetch_image(
        "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/eskimo.jpg"  # noqa: E501
    # ... key logic omitted for brevity ...
                embeddings_tensor = output.outputs.data.detach().clone()

            # Pool and normalize embeddings
            pooled_output = (
                embeddings_tensor.sum(dim=0, dtype=torch.float32)
                / embeddings_tensor.shape[0]
            )
            embeddings.append(torch.nn.functional.normalize(pooled_output, dim=-1))
        return embeddings

    embeddings = get_embeddings(outputs)

    for embedding in embeddings:
        print(embedding.shape)
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include TextPrompt, clone, torch.where, output.outputs.data.detach, and torch.tensor. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。关键操作包括 TextPrompt、clone、torch.where、output.outputs.data.detach，以及 torch.tensor。其返回值会继续传给示例管线的下一阶段。

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
- **Representation extraction / 表示提取**: The output is a dense or token-level representation rather than free-form text. / 输出是稠密向量或词元级表示，而不是自由文本。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.config`, `vllm.inputs`, `vllm.multimodal.utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `torch` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `TextPrompt`, `clone`, `torch.where`, `output.outputs.data.detach`, `torch.tensor`, `LLM`, `fetch_image`, `model.encode` reveal the main execution path / 这些调用体现了主要执行链路。
