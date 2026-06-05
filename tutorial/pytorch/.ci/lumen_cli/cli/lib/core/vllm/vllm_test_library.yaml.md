# vllm_test_library.yaml — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `.ci/lumen_cli/cli/lib/core/vllm/vllm_test_library.yaml`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides declarative configuration data that drives builds, workflows, or packaging behavior. The opening comment frames the file as: "VLLM Test Library Configuration This file defines the test plans for vllm CI development See https://github.com/vllm-project/vllm/blob/main/.buildkite/test-pipeline.yaml."
- **Purpose (CN)**: 提供驱动构建、工作流或打包行为的声明式配置数据。 开头注释将该文件概括为：“VLLM Test Library Configuration This file defines the test plans for vllm CI development See https://github.com/vllm-project/vllm/blob/main/.buildkite/test-pipeline.yaml”。

## Content Analysis / 内容分析

### Lines 1-11 / 第 1-11 行

```yaml
# VLLM Test Library Configuration
# This file defines the test plans for vllm CI development
# See https://github.com/vllm-project/vllm/blob/main/.buildkite/test-pipeline.yaml

vllm_basic_correctness_test:
  title: Basic Correctness Test
  id: vllm_basic_correctness_test
  env_vars:
    VLLM_WORKER_MULTIPROC_METHOD: spawn
  steps:
    - pytest -v -s basic_correctness/test_cumem.py
```

- **EN:** This chunk introduces sections such as VLLM Test Library Configuration, This file defines the test plans for vllm CI development, See https://github.com/vllm-project/vllm/blob/main/.buildkite/test-pipeline.yaml, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 VLLM Test Library Configuration、This file defines the test plans for vllm CI development、See https://github.com/vllm-project/vllm/blob/main/.buildkite/test-pipeline.yaml 等标题组织周边说明或配置。
- **EN:** Top-level or nested keys such as vllm_basic_correctness_test, env_vars, steps define the declarative structure of the file.
- **CN:** vllm_basic_correctness_test、env_vars、steps 等顶层或嵌套键定义了该文件的声明式结构。

### Lines 12-19 / 第 12-19 行

```yaml
    - pytest -v -s basic_correctness/test_basic_correctness.py
    - pytest -v -s basic_correctness/test_cpu_offload.py

vllm_basic_models_test:
  title: Basic models test
  id: vllm_basic_models_test
  steps:
    - pytest -v -s models/test_transformers.py
```

- **EN:** Top-level or nested keys such as vllm_basic_models_test, steps define the declarative structure of the file.
- **CN:** vllm_basic_models_test、steps 等顶层或嵌套键定义了该文件的声明式结构。

### Lines 20-31 / 第 20-31 行

```yaml
    - pytest -v -s models/test_registry.py
    - pytest -v -s models/test_utils.py
    - pytest -v -s models/test_vision.py
    - HF_DATASETS_OFFLINE=0 TRANSFORMERS_OFFLINE=0 pytest -v -s models/test_initialization.py

vllm_entrypoints_test:
  title: Entrypoints Test
  id: vllm_entrypoints_test
  env_vars:
    VLLM_WORKER_MULTIPROC_METHOD: spawn
  steps:
    - pytest -v -s entrypoints/llm --ignore=entrypoints/llm/test_generate.py --ignore=entrypoints/llm/test_collective_rpc.py
```

- **EN:** Top-level or nested keys such as vllm_entrypoints_test, env_vars, steps define the declarative structure of the file.
- **CN:** vllm_entrypoints_test、env_vars、steps 等顶层或嵌套键定义了该文件的声明式结构。

### Lines 32-39 / 第 32-39 行

```yaml
    - pytest -v -s entrypoints/llm/test_generate.py
    - pytest -v -s entrypoints/offline_mode

vllm_regression_test:
  title: Regression Test
  id: vllm_regression_test
  package_install:
    - modelscope
```

- **EN:** Top-level or nested keys such as vllm_regression_test, package_install define the declarative structure of the file.
- **CN:** vllm_regression_test、package_install 等顶层或嵌套键定义了该文件的声明式结构。

### Lines 40-51 / 第 40-51 行

```yaml
  steps:
    - pytest -v -s test_regression.py -k 'not test_model_from_modelscope'
    - HF_DATASETS_OFFLINE=0 TRANSFORMERS_OFFLINE=0 pytest -v -s test_regression.py -k test_model_from_modelscope

vllm_lora_tp_test_distributed:
  title: LoRA TP Test (Distributed)
  id: vllm_lora_tp_test_distributed
  env_vars:
    VLLM_WORKER_MULTIPROC_METHOD: spawn
  num_gpus: 4
  steps:
    - pytest -v -s -x lora/test_chatglm3_tp.py
```

- **EN:** Top-level or nested keys such as steps, vllm_lora_tp_test_distributed, env_vars, steps define the declarative structure of the file.
- **CN:** steps、vllm_lora_tp_test_distributed、env_vars、steps 等顶层或嵌套键定义了该文件的声明式结构。

### Lines 52-63 / 第 52-63 行

```yaml
    - pytest -v -s -x lora/test_llama_tp.py -k 'not test_tp2_serialize_and_deserialize_lora'
    - HF_DATASETS_OFFLINE=0 TRANSFORMERS_OFFLINE=0 pytest -v -s -x lora/test_llama_tp.py -k test_tp2_serialize_and_deserialize_lora
    - pytest -v -s -x lora/test_llm_with_multi_loras.py

vllm_distributed_test_28_failure_test:
  title: Distributed Tests (2 GPUs) pytorch 2.8 release failure
  id: vllm_distributed_test_28_failure_test
  env_vars:
    VLLM_WORKER_MULTIPROC_METHOD: spawn
  num_gpus: 4
  steps:
    - pytest -v -s compile/correctness_e2e/test_sequence_parallel.py
```

- **EN:** Top-level or nested keys such as vllm_distributed_test_28_failure_test, env_vars, steps define the declarative structure of the file.
- **CN:** vllm_distributed_test_28_failure_test、env_vars、steps 等顶层或嵌套键定义了该文件的声明式结构。

### Lines 64-75 / 第 64-75 行

```yaml

vllm_lora_28_failure_test:
  title: LoRA pytorch 2.8 failure test
  id: vllm_lora_28_failure_test
  steps:
    - pytest -v lora/test_quant_model.py

vllm_multi_model_processor_test:
  title: Multi-Modal Processor Test
  id: vllm_multi_model_processor_test
  package_install:
    - git+https://github.com/TIGER-AI-Lab/Mantis.git
```

- **EN:** Top-level or nested keys such as vllm_lora_28_failure_test, steps, vllm_multi_model_processor_test, package_install define the declarative structure of the file.
- **CN:** vllm_lora_28_failure_test、steps、vllm_multi_model_processor_test、package_install 等顶层或嵌套键定义了该文件的声明式结构。

### Lines 76-86 / 第 76-86 行

```yaml
  steps:
    - pytest -v -s models/multimodal/processing --ignore models/multimodal/processing/test_tensor_schema.py --ignore models/multimodal/processing/test_common.py --ignore models/multimodal/processing/test_glm4_1v.py
    - pytest -v -s models/multimodal/processing/test_common.py -k 'not mistralai'
    - HF_DATASETS_OFFLINE=0 TRANSFORMERS_OFFLINE=0 pytest -v -s models/multimodal/processing/test_common.py -k mistralai
    - HF_DATASETS_OFFLINE=0 TRANSFORMERS_OFFLINE=0 pytest -v -s models/multimodal/processing/test_glm4_1v.py

vllm_multi_model_test_28_failure_test:
  title: Multi-Model Test (Failed 2.8 release)
  id: vllm_multi_model_test_28_failure_test
  package_install:
    - git+https://github.com/TIGER-AI-Lab/Mantis.git
```

- **EN:** Top-level or nested keys such as steps, vllm_multi_model_test_28_failure_test, package_install define the declarative structure of the file.
- **CN:** steps、vllm_multi_model_test_28_failure_test、package_install 等顶层或嵌套键定义了该文件的声明式结构。

### Lines 87-95 / 第 87-95 行

```yaml
  steps:
    - HF_DATASETS_OFFLINE=0 TRANSFORMERS_OFFLINE=0 pytest -v -s models/multimodal/generation/test_voxtral.py
    - pytest -v -s models/multimodal/pooling

vllm_pytorch_compilation_unit_tests:
  title: PyTorch Compilation Unit Tests
  id: vllm_pytorch_compilation_unit_tests
  steps:
    - pytest -v -s compile/passes/test_pass_manager.py
```

- **EN:** Top-level or nested keys such as steps, vllm_pytorch_compilation_unit_tests, steps define the declarative structure of the file.
- **CN:** steps、vllm_pytorch_compilation_unit_tests、steps 等顶层或嵌套键定义了该文件的声明式结构。

### Lines 96-103 / 第 96-103 行

```yaml
    - pytest -v -s compile/passes/test_fusion.py
    - pytest -v -s compile/passes/test_fusion_attn.py
    - pytest -v -s compile/passes/test_silu_mul_quant_fusion.py
    - pytest -v -s compile/passes/distributed/test_sequence_parallelism.py
    - pytest -v -s compile/passes/distributed/test_async_tp.py
    - pytest -v -s compile/passes/distributed/test_fusion_all_reduce.py
    - pytest -v -s compile/test_decorator.py

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 104-111 / 第 104-111 行

```yaml
vllm_language_model_test_extended_generation_28_failure_test:
  title: Language Models Test (Extended Generation) 2.8 release failure
  id: vllm_language_model_test_extended_generation_28_failure_test
  package_install:
    - --no-build-isolation
    - git+https://github.com/Dao-AILab/causal-conv1d@v1.6.0
  steps:
    - pytest -v -s models/language/generation/test_mistral.py
```

- **EN:** Top-level or nested keys such as vllm_language_model_test_extended_generation_28_failure_test, package_install, steps define the declarative structure of the file.
- **CN:** vllm_language_model_test_extended_generation_28_failure_test、package_install、steps 等顶层或嵌套键定义了该文件的声明式结构。

### Lines 112-120 / 第 112-120 行

```yaml

vllm_distributed_test_2_gpu_28_failure_test:
  title: Distributed Tests (2 GPUs) pytorch 2.8 release failure
  id: vllm_distributed_test_2_gpu_28_failure_test
  env_vars:
    VLLM_WORKER_MULTIPROC_METHOD: spawn
  num_gpus: 4
  steps:
    - pytest -v -s compile/correctness_e2e/test_sequence_parallel.py
```

- **EN:** Top-level or nested keys such as vllm_distributed_test_2_gpu_28_failure_test, env_vars, steps define the declarative structure of the file.
- **CN:** vllm_distributed_test_2_gpu_28_failure_test、env_vars、steps 等顶层或嵌套键定义了该文件的声明式结构。

### Lines 121-127 / 第 121-127 行

```yaml

vllm_lora_test:
  title: LoRA Test %N
  id: lora_test
  parallelism: 4
  steps:
    - pytest -v -s lora --shard-id=$$BUILDKITE_PARALLEL_JOB --num-shards=$$BUILDKITE_PARALLEL_JOB_COUNT --ignore=lora/test_chatglm3_tp.py --ignore=lora/test_llama_tp.py --ignore=lora/test_llm_with_multi_loras.py --ignore=lora/test_olmoe_tp.py --ignore=lora/test_deepseekv2_tp.py --ignore=lora/test_gptoss_tp.py --ignore=lora/test_qwen3moe_tp.py --ignore=lora/test_qwen35_densemodel_lora.py
```

- **EN:** Top-level or nested keys such as vllm_lora_test, steps define the declarative structure of the file.
- **CN:** vllm_lora_test、steps 等顶层或嵌套键定义了该文件的声明式结构。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **Repository bootstrap** — 与仓库状态、子模块或源码检出流程交互。
- **Declarative configuration** — 以声明式格式表示构建或工作流设置。
- **Representative symbols: vllm_basic_correctness_test, env_vars, steps, vllm_basic_models_test, vllm_entrypoints_test, vllm_regression_test, package_install, vllm_lora_tp_test_distributed** — 代表性符号：vllm_basic_correctness_test、env_vars、steps、vllm_basic_models_test、vllm_entrypoints_test、vllm_regression_test、package_install、vllm_lora_tp_test_distributed

## Dependencies / 依赖关系

- No prominent dependency reference detected. / 未检测到明显的依赖引用。
