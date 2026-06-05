# multimodal_embedding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/multimodal_embedding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example configures multimodal serving or embedding generation for a vision-language model. / 该示例用于配置视觉语言模型的多模态服务或多模态嵌入生成。

## Line-by-Line Analysis / 逐行分析
### Lines 4-4: Import dependencies and runtime symbols
````python
import requests
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 6-18: Set top-level configuration
````python
url = "http://127.0.0.1:30000"

text_input = "Represent this image in embedding space."
image_path = "https://huggingface.co/datasets/liuhaotian/llava-bench-in-the-wild/resolve/main/images/023.jpg"

payload = {
    "model": "gme-qwen2-vl",
    "input": [{"text": text_input}, {"image": image_path}],
}

response = requests.post(url + "/v1/embeddings", json=payload).json()

print("Embeddings:", [x.get("embedding") for x in response.get("data", [])])
````
**EN:** This block establishes constants or shared objects that the rest of the file relies on.
**CN:** 该代码块建立了后续逻辑依赖的常量或共享对象。

## Key Concepts / 关键概念
- **Multimodal inputs / 多模态输入**: The example mixes text with image or video inputs. / 该示例将文本与图像或视频输入结合起来。
- **Embedding generation / 向量嵌入生成**: The script requests dense representations instead of free-form text. / 脚本请求的是稠密向量表示，而不是自由文本。

## Dependencies / 依赖关系
- **Third-party / 第三方**: requests
