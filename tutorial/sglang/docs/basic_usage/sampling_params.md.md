# sampling_params.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/basic_usage/sampling_params.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This doc describes the sampling parameters of the SGLang Runtime. It is the low-level endpoint of the runtime. / 该文档围绕 Sampling Parameters 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This doc describes the sampling parameters of the SGLang Runtime. It is the low-level endpoint of the runtime.
**CN:** 本节围绕 Overview 展开，概述了 endpoint, API, want, chat 等要点，并说明相关配置、流程、示例或限制条件。

### Section: /generate Endpoint
**EN:** This section provides a comparison table for /generate Endpoint, covering columns such as Argument, Type/Default, Description and examples such as text, input_ids, input_embeds, image_data.
**CN:** 本节围绕 /generate Endpoint 展开，概述了 List, Optional, Union, Whether 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Sampling parameters
**EN:** The object is defined at sampling_params.py::SamplingParams. You can also read the source code to find more arguments and docs.
**CN:** 本节围绕 Sampling parameters 展开，概述了 sampling_params.py::SamplingParams, read, code, find 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Note on defaults
**EN:** By default, SGLang initializes several sampling parameters from the model's generation_config.json (when the server is launched with --sampling-defaults model, which is the default).
**CN:** 本节围绕 Note on defaults 展开，概述了 MODEL, sampling-defaults, --sampling-defaults, defaults 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Core parameters
**EN:** This section provides a comparison table for Core parameters, covering columns such as Argument, Type/Default, Description and examples such as max_new_tokens, stop, stop_token_ids, stop_regex.
**CN:** 本节围绕 Core parameters 展开，概述了 List, Optional, stop, tokens 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Penalizers
**EN:** This section provides a comparison table for Penalizers, covering columns such as Argument, Type/Default, Description and examples such as frequency_penalty, presence_penalty, repetition_penalty, min_new_tokens.
**CN:** 本节围绕 Penalizers 展开，概述了 tokens, Must, Penalizes, float 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Constrained decoding
**EN:** This section provides a comparison table for Constrained decoding, covering columns such as Argument, Type/Default, Description and examples such as json_schema, regex, ebnf, structural_tag.
**CN:** 本节围绕 Constrained decoding 展开，概述了 Optional, str, outputs, structured 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Other options
**EN:** This section provides a comparison table for Other options, covering columns such as Argument, Type/Default, Description and examples such as n, ignore_eos, skip_special_tokens, spaces_between_special_tokens.
**CN:** 本节围绕 Other options 展开，概述了 Don, EOS, bool, Optional 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Normal
**EN:** Launch a server: ``bash python -m sglang.launch_server --model-path meta-llama/Meta-Llama-3-8B-Instruct --port 30000 ` Send a request: `python import requests response = requests.post( "http://localhost:30000/generate", json= , }, ) print(response.json()) `` Detailed example in send request.
**CN:** 本节围绕 Normal 展开，概述了 Send, Launch, France, request 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Streaming
**EN:** This content focuses on Streaming and highlights chunk, data, output, prev.
**CN:** 本节围绕 Streaming 展开，概述了 chunk, data, output, prev 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multimodal
**EN:** This section provides a comparison table for Multimodal, covering columns such as "text": "<, im_start, >system\nYou are a helpful assistant.< and examples such as "<, "<.
**CN:** 本节围绕 Multimodal 展开，概述了 image, im_start, URL, See 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Structured Outputs (JSON, Regex, EBNF)
**EN:** You can specify a JSON schema, regular expression or EBNF to constrain the model output. The model output will be guaranteed to follow the given constraints.
**CN:** 本节围绕 Structured Outputs (JSON, Regex, EBNF) 展开，概述了 XGrammar, Outlines, EBNF, json_schema 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Custom logit processor
**EN:** Launch a server with --enable-custom-logit-processor flag on.
**CN:** 本节围绕 Custom logit processor 展开，概述了 logits, DeterministicLogitProcessor, Send, import 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** List / **CN:** List
- **EN:** Optional / **CN:** Optional
- **EN:** Union / **CN:** Union
- **EN:** tokens / **CN:** 令牌s
- **EN:** str / **CN:** str
- **EN:** text / **CN:** text
- **EN:** int / **CN:** int
- **EN:** token / **CN:** 令牌

## Dependencies / 依赖关系
- `openai_api_completions.ipynb`
- `native_api.ipynb`
- `../advanced_features/structured_outputs.ipynb`
- `./send_request.ipynb`
- `openai_api_vision.ipynb`
- `io_struct.py`
- `sampling_params.py`
- `python/sglang/srt/utils.py`
- `//github.com/ggerganov/llama.cpp/blob/master/grammars/README.md`
