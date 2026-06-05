# deploy_and_serve_endpoint.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/sagemaker/deploy_and_serve_endpoint.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This deployment example packages an SGLang image for SageMaker, creates an endpoint, and validates it with a sample request. / 该部署示例会为 SageMaker 打包 SGLang 镜像、创建端点，并通过示例请求进行验证。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Import dependencies and runtime symbols
````python
import json

import boto3
from sagemaker import serializers
from sagemaker.model import Model
from sagemaker.predictor import Predictor
````
**EN:** This block imports AWS SDK and SageMaker utilities needed to package, deploy, and test a managed endpoint.
**CN:** 这一部分导入了 AWS SDK 与 SageMaker 工具，用于打包、部署并测试托管端点。

### Lines 8-69: Define deployment configuration
````python
boto_session = boto3.session.Session()
sm_client = boto_session.client("sagemaker")
sm_role = boto_session.resource("iam").Role("SageMakerRole").arn

endpoint_name = "<YOUR_ENDPOINT_NAME>"
image_uri = "<YOUR_DOCKER_IMAGE_URI>"
model_id = (
    "<YOUR_MODEL_ID>"  # eg: Qwen/Qwen3-0.6B from https://huggingface.co/Qwen/Qwen3-0.6B
)
hf_token = "<YOUR_HUGGINGFACE_TOKEN>"
prompt = "<YOUR_ENDPOINT_PROMPT>"

model = Model(
    name=endpoint_name,
    image_uri=image_uri,
    role=sm_role,
    env={
        "SM_SGLANG_MODEL_PATH": model_id,
        "HF_TOKEN": hf_token,
    },
)
print("Model created successfully")
print("Starting endpoint deployment (this may take 10-15 minutes)...")

endpoint_config = model.deploy(
    instance_type="ml.g5.12xlarge",
    initial_instance_count=1,
    endpoint_name=endpoint_name,
    inference_ami_version="al2-ami-sagemaker-inference-gpu-3-1",
    wait=True,
)
print("Endpoint deployment completed successfully")


print(f"Creating predictor for endpoint: {endpoint_name}")
predictor = Predictor(
    endpoint_name=endpoint_name,
    serializer=serializers.JSONSerializer(),
)

payload = {
    "model": model_id,
    "messages": [{"role": "user", "content": prompt}],
    "max_tokens": 2400,
    "temperature": 0.01,
    "top_p": 0.9,
    "top_k": 50,
}
print(f"Sending inference request with prompt: '{prompt[:50]}...'")
response = predictor.predict(payload)
print("Inference request completed successfully")

if isinstance(response, bytes):
    response = response.decode("utf-8")

if isinstance(response, str):
    try:
        response = json.loads(response)
    except json.JSONDecodeError:
        print("Warning: Response is not valid JSON. Returning as string.")

print(f"Received model response: '{response}'")
````
**EN:** This block collects the static names, instance types, or identifiers needed for the remote deployment step.
**CN:** 该代码块整理了远程部署步骤所需的静态名称、实例类型或标识符。

## Key Concepts / 关键概念
- **Cloud deployment / 云端部署**: The workflow packages a model for managed remote serving. / 该流程会将模型打包到托管的远程服务中。

## Dependencies / 依赖关系
- **Standard library / 标准库**: json
- **Third-party / 第三方**: boto3, sagemaker.model.Model, sagemaker.predictor.Predictor, sagemaker.serializers
