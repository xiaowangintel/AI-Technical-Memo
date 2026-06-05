# object_storage.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/object_storage.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang supports direct loading of models from object storage (S3 and Google Cloud Storage) without requiring a full local download. / 该文档围绕 Loading 模型s from Object Storage 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang supports direct loading of models from object storage (S3 and Google Cloud Storage) without requiring a full local download.
**CN:** 本节围绕 Overview 展开，概述了 storage, local, runai_streamer, full 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Overview
**EN:** When loading models from object storage, SGLang uses a two-phase approach: 1. **Metadata Download** (once, before process launch): Configuration files and tokenizer files are downloaded to a local cache 2.
**CN:** 本节围绕 Overview 展开，概述了 Model, files, object, loading 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Storage Backends
**EN:** **Amazon S3**: s3://bucket-name/path/to/model/ 2. **Google Cloud Storage**: gs://bucket-name/path/to/model/ 3.
**CN:** 本节围绕 Supported Storage Backends 展开，概述了 s3://bucket-name/path/to/model/, Amazon, gs://bucket-name/path/to/model/, az://some-azure-container/path/ 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Basic Usage
**EN:** This content focuses on Basic Usage and highlights model-path, load-format, --model-path, --load-format.
**CN:** 本节围绕 Basic Usage 展开，概述了 model-path, load-format, --model-path, --load-format 等要点，并说明相关配置、流程、示例或限制条件。

### Section: With Tensor Parallelism
**EN:** ``bash python -m sglang.launch_server \ --model-path gs://my-bucket/models/llama-70b/ \ --tp 4 \ --model-loader-extra-config ' ' ``
**CN:** 本节围绕 With Tensor 并行策略 展开，概述了 --tp, model-path, distributed, --model-path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Load Format
**EN:** The runai_streamer load format is specifically designed for object storage, ssd and shared file systems ``bash python -m sglang.launch_server \ --model-path s3://bucket/model/ \ --load-format runai_streamer ``
**CN:** 本节围绕 Load Format 展开，概述了 runai_streamer, ssd, load, format 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Extended Configuration Parameters
**EN:** This section provides a comparison table for Extended Configuration Parameters, covering columns such as Parameter, Type, Description and examples such as distributed, concurrency, memory_limit.
**CN:** 本节围绕 Extended 配置 Parameters 展开，概述了 distributed, concurrency, memory_limit, --model-loader-extra-config 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Distributed Streaming
**EN:** For multi-GPU setups, enable distributed streaming to parallelize weight loading between the processes: ``bash python -m sglang.launch_server \ --model-path s3://bucket/model/ \ --tp 8 \ --model-loader-extra-config ' ' ``
**CN:** 本节围绕 Distributed Streaming 展开，概述了 distributed, GPU, --tp, setups 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Limitations
**EN:** **Supported Formats**: Currently only supports .safetensors weight format (recommended format) - **Supported Device**: Distributed streaming is supported on cuda alike devices. Otherwise fallback to non distributed streaming
**CN:** 本节围绕 Limitations 展开，概述了 format, Currently, Otherwise, Supported 等要点，并说明相关配置、流程、示例或限制条件。

### Section: See Also
**EN:** Runai model streamer documentation
**CN:** 本节围绕 See Also 展开，概述了 Runai, model, streamer 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** storage / **CN:** storage
- **EN:** distributed / **CN:** distributed
- **EN:** object / **CN:** object
- **EN:** model-path / **CN:** 模型-path
- **EN:** --model-path / **CN:** --模型-path
- **EN:** runai_streamer / **CN:** runai_streamer
- **EN:** sglang.launch_server / **CN:** sglang.launch_服务端
- **EN:** streaming / **CN:** streaming

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
