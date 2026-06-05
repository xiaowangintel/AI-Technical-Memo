# support_new_models.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/supported_models/extending/support_new_models.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This document explains how to add support for new language models and multimodal large language models (MLLMs) in SGLang. It also covers how to test new models and register external implementations. / 该文档围绕 How to Support New 模型s 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This document explains how to add support for new language models and multimodal large language models (MLLMs) in SGLang. It also covers how to test new models and register external implementations.
**CN:** 本节围绕 Overview 展开，概述了 models, new, MLLMs, language 等要点，并说明相关配置、流程、示例或限制条件。

### Section: How to Support a New Language Model
**EN:** To support a new model in SGLang, you only need to add a single file under the SGLang Models Directory. You can learn from existing model implementations and create a new file for your model.
**CN:** 本节围绕 How to Support a New Language 模型 展开，概述了 model, new, Llama, add 等要点，并说明相关配置、流程、示例或限制条件。

### Section: How to Support a New Multimodal Large Language Model
**EN:** To support a new multimodal large language model (MLLM) in SGLang, there are several key components in addition to the standard LLM support: 1.
**CN:** 本节围绕 How to Support a New Multimodal Large Language 模型 展开，概述了 new, model, Register, multimodal 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Testing and Debugging
**EN:** Please note all your testing and benchmarking results in PR description.
**CN:** 本节围绕 Testing and Debugging 展开，概述了 Please, note, testing, results 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Interactive Debugging
**EN:** For interactive debugging, compare the outputs of Hugging Face/Transformers and SGLang.
**CN:** 本节围绕 Interactive Debugging 展开，概述了 model, output, new, text 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Add the Model to the Test Suite
**EN:** To ensure the new model is well maintained, add it to the test suite by including it in the ALL_OTHER_MODELS list in the test_generation_models.py file, test the new model on your local machine and report the results on demonstrative benchmarks (GSM8K, MMLU, MMMU, MMMU-Pro, etc.) in your PR.
**CN:** 本节围绕 Add the 模型 to the Test Suite 展开，概述了 test, new, model, ALL_OTHER_MODELS 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Benchmark
**EN:** **(Required) MMMU**: follow MMMU benchmark README.md to get SGLang vs. HF Transformer accuracy comparison.
**CN:** 本节围绕 Benchmark 展开，概述了 Transformer, MMMU, README.md, run 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Port a Model from vLLM to SGLang
**EN:** The vLLM Models Directory is a valuable resource, as vLLM covers many models. SGLang reuses vLLM’s interface and some layers, making it easier to port models from vLLM to SGLang.
**CN:** 本节围绕 Port a 模型 from vLLM to SGLang 展开，概述了 vLLM, Replace, Attention, RadixAttention 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Registering an External Model Implementation
**EN:** In addition to the methods above, you can register your new model with the ModelRegistry before launching the server. This allows you to integrate your model without modifying the source code.
**CN:** 本节围绕 Registering an External 模型 Implementation 展开，概述了 ModelRegistry, model, server, import 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example: Implementing and Serving a Llama Wrapper Model
**EN:** Below is an introductory, step-by-step walkthrough on how to implement a new model end-to-end in SGLang and then run it via the Offline Engine.
**CN:** 本节围绕 Example: Implementing and Serving a Llama Wrapper 模型 展开，概述了 Below, new, run, via 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Implementing Our Model
**EN:** To keep things simple, this new model will be a simple wrapper around Llama 3.1-8B-Instruct, and our goal will be just to bias the output logits for each forward call by taking the square root of each individual logit.
**CN:** 本节围绕 Implementing Our 模型 展开，概述了 LlamaForCausalLM, forward, model, import 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Serving Our Model Via SGLang's Offline Engine
**EN:** The next step of this walkthrough involves hosting our new model offline, so that it can be served locally and without an HTTP server. First, create a new file called run.py.
**CN:** 本节围绕 Serving Our 模型 Via SGLang's Offline Engine 展开，概述了 LlamaWrapper, model, import, Now 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Serving External Models via the Standard CLI
**EN:** The previous sections show how to register a model programmatically via ModelRegistry and serve it through the Offline Engine.
**CN:** 本节围绕 Serving External 模型s via the Standard CLI 展开，概述了 model, ModelRegistry, SGLANG_EXTERNAL_MODEL_PACKAGE, CLI 等要点，并说明相关配置、流程、示例或限制条件。

### Section: The EntryClass Variable
**EN:** When SGLang scans a model package, it looks for the variable EntryClass at the module level of your Python file. The model registry imports your file, checks for EntryClass, and registers the class assigned to it.
**CN:** 本节围绕 The EntryClass Variable 展开，概述了 EntryClass, model, end, Llama 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example: Text-Only Model
**EN:** Using the same Llama wrapper from the previous section, here is how to package and serve it via the CLI. Install your package Run this inside your sglang_custom_project directory to install your code into the active Python environment: `bash pip install -e .
**CN:** 本节围绕 Example: Text-Only 模型 展开，概述了 import, LlamaWrapper, CLI, Write 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example: Multimodal Model
**EN:** If you are working with multimodal models, setting SGLANG_EXTERNAL_MODEL_PACKAGE alone is not enough. SGLang also needs to recognize your architecture as multimodal to enable the image/video processing pipelines, and it needs a custom processor.
**CN:** 本节围绕 Example: Multimodal 模型 展开，概述了 CustomQwen2VL, processor, SGLANG_EXTERNAL_MODEL_PACKAGE, SGLANG_EXTERNAL_MM_MODEL_ARCH 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Documentation
**EN:** Add to table of supported models in generative_models.md or multimodal_language_models.md --- By following these guidelines, you can add support for new language models and multimodal large language models in SGLang and ensure they are thoroughly tested and easily integrated into the system.
**CN:** 本节围绕 Documentation 展开，概述了 models, Add, language, new 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** model / **CN:** 模型
- **EN:** LlamaWrapper / **CN:** LlamaWrapper
- **EN:** import / **CN:** import
- **EN:** new / **CN:** new
- **EN:** LlamaForCausalLM / **CN:** LlamaForCausalLM
- **EN:** EntryClass / **CN:** EntryClass
- **EN:** models / **CN:** 模型s
- **EN:** forward / **CN:** forward

## Dependencies / 依赖关系
- `test_vision_openai_server_{x}.py`
- `llama_wrapper.py`
- `run.py`
- `python run.py`
- `setup.py`
- `qwenvl_wrapper.py`
- `../text_generation/generative_models.md`
- `../text_generation/multimodal_language_models.md`
- `model_config.py`
- `//github.com/sgl-project/sglang/blob/0ab3f437aba729b348a683ab32b35b214456efc7/python/sglang/srt/configs/model_config.py`
- `conversation.py`
- `//github.com/sgl-project/sglang/blob/main/python/sglang/srt/parser/conversation.py`
