# vertex_predict.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/vertex_predict.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This script sends prediction traffic to a Vertex-hosted model endpoint and prints the returned result. / 该脚本会向 Vertex 托管的模型端点发送预测请求，并打印返回结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Module overview and usage
````python
"""
Usage:
python -m sglang.launch_server --model meta-llama/Llama-2-7b-hf --port 30000
python vertex_predict.py

This example shows the request and response formats of the prediction route for
Google Cloud Vertex AI Online Predictions.

Vertex AI SDK for Python is recommended for deploying models to Vertex AI
instead of a local server. After deploying the model to a Vertex AI Online
Prediction Endpoint, send requests via the Python SDK:

response = endpoint.predict(
    instances=[
        {"text": "The capital of France is"},
        {"text": "What is a car?"},
    ],
    parameters={"sampling_params": {"max_new_tokens": 16}},
)
print(response.predictions)

More details about get online predictions from Vertex AI can be found at
https://cloud.google.com/vertex-ai/docs/predictions/get-online-predictions.
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 26-29: Import dependencies and runtime symbols
````python
from dataclasses import dataclass
from typing import List, Optional

import requests
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 32-34: Define VertexPrediction class
````python
@dataclass
class VertexPrediction:
    predictions: List
````
**EN:** This class collects the state and helper methods required by this example.
**CN:** 该类汇集了此示例所需的状态以及辅助方法。

### Lines 37-37: Define LocalVertexEndpoint class
````python
class LocalVertexEndpoint:
````
**EN:** This class collects the state and helper methods required by this example.
**CN:** 该类汇集了此示例所需的状态以及辅助方法。

### Lines 38-39: Method: Initialize class state
````python
    def __init__(self) -> None:
        self.base_url = "http://127.0.0.1:30000"
````
**EN:** This method initializes lightweight dependencies and stores the object state needed by later methods.
**CN:** 该方法会初始化轻量级依赖，并保存后续方法需要使用的对象状态。

### Lines 41-49: Method: Deploy and validate the remote service
````python
    def predict(self, instances: List[dict], parameters: Optional[dict] = None):
        response = requests.post(
            self.base_url + "/vertex_generate",
            json={
                "instances": instances,
                "parameters": parameters,
            },
        )
        return VertexPrediction(predictions=response.json()["predictions"])
````
**EN:** This method prepares cloud-side resources, performs deployment or prediction calls, and usually includes a lightweight verification step.
**CN:** 该方法会准备云端资源、执行部署或预测调用，并通常包含一个轻量级验证步骤。

### Lines 52-66: Set top-level configuration
````python
endpoint = LocalVertexEndpoint()

# Predict with a single prompt.
response = endpoint.predict(instances=[{"text": "The capital of France is"}])
print(response.predictions)

# Predict with multiple prompts and parameters.
response = endpoint.predict(
    instances=[
        {"text": "The capital of France is"},
        {"text": "What is a car?"},
    ],
    parameters={"sampling_params": {"max_new_tokens": 16}},
)
print(response.predictions)
````
**EN:** This block establishes constants or shared objects that the rest of the file relies on.
**CN:** 该代码块建立了后续逻辑依赖的常量或共享对象。

## Key Concepts / 关键概念
- **Cloud deployment / 云端部署**: The workflow packages a model for managed remote serving. / 该流程会将模型打包到托管的远程服务中。

## Dependencies / 依赖关系
- **Standard library / 标准库**: dataclasses.dataclass, typing.List, typing.Optional
- **Third-party / 第三方**: requests
