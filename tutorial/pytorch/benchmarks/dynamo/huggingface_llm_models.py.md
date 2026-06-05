# huggingface_llm_models.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/huggingface_llm_models.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: import subprocess
 2: import sys
 3: 
 4: import torch
 5: 
 6: 
 7: def pip_install(package):
 8:     subprocess.check_call([sys.executable, "-m", "pip", "install", package])
 9: 
10: 
11: try:
12:     from transformers import (
13:         AutoModelForCausalLM,
14:         AutoTokenizer,
15:         WhisperForConditionalGeneration,
16:         WhisperProcessor,
17:     )
18: except ModuleNotFoundError:
19:     print("Installing HuggingFace Transformers...")
20:     pip_install("git+https://github.com/huggingface/transformers.git#egg=transformers")
21: finally:
22:     from transformers import (
23:         AutoModelForCausalLM,
24:         AutoTokenizer,
25:         WhisperForConditionalGeneration,
26:         WhisperProcessor,
27:     )
28: 
29: 
30: class Benchmark:
31:     @staticmethod
32:     def get_model_and_inputs(model_name, device):
33:         raise NotImplementedError("get_model_and_inputs() not implemented")
34: 
35: 
36: class WhisperBenchmark(Benchmark):
37:     SAMPLE_RATE = 16000
38:     DURATION = 30.0  # seconds
39: 
40:     @staticmethod
````
- EN: Handles module imports such as `subprocess`, `sys`, `torch`, `transformers`.
- CN: 处理模块导入，例如 `subprocess`, `sys`, `torch`, `transformers`。
- EN: Declares or extends types including `Benchmark`, `WhisperBenchmark`.
- CN: 声明或扩展类型，包括 `Benchmark`, `WhisperBenchmark`。
- EN: Implements callable logic such as `pip_install`, `get_model_and_inputs`.
- CN: 实现可调用逻辑，例如 `pip_install`, `get_model_and_inputs`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 41-80
````python
41:     def get_model_and_inputs(model_name, device):
42:         processor = WhisperProcessor.from_pretrained(model_name)
43:         model = WhisperForConditionalGeneration.from_pretrained(model_name).to(device)
44:         model.config.forced_decoder_ids = None
45: 
46:         model.generation_config.do_sample = False
47:         model.generation_config.temperature = 0.0
48: 
49:         num_samples = int(WhisperBenchmark.DURATION * WhisperBenchmark.SAMPLE_RATE)
50:         audio = torch.randn(num_samples) * 0.1
51:         inputs = dict(
52:             processor(
53:                 audio, sampling_rate=WhisperBenchmark.SAMPLE_RATE, return_tensors="pt"
54:             )
55:         )
56:         inputs["input_features"] = inputs["input_features"].to(device)
57: 
58:         decoder_start_token = model.config.decoder_start_token_id
59:         inputs["decoder_input_ids"] = torch.tensor(
60:             [[decoder_start_token]], device=device
61:         )
62: 
63:         return model, inputs
64: 
65: 
66: class TextGenerationBenchmark(Benchmark):
67:     INPUT_LENGTH = 1000
68:     OUTPUT_LENGTH = 2000
69: 
70:     @staticmethod
71:     def get_model_and_inputs(model_name, device):
72:         tokenizer = AutoTokenizer.from_pretrained(model_name)
73:         model = AutoModelForCausalLM.from_pretrained(model_name, device_map=device)
74:         model.eval()
75: 
76:         model.generation_config.do_sample = False
77:         model.generation_config.use_cache = True
78:         model.generation_config.cache_implementation = "static"
79:         model.generation_config.max_new_tokens = TextGenerationBenchmark.OUTPUT_LENGTH
80:         model.generation_config.pad_token_id = tokenizer.eos_token_id
````
- EN: Declares or extends types including `TextGenerationBenchmark`.
- CN: 声明或扩展类型，包括 `TextGenerationBenchmark`。
- EN: Implements callable logic such as `get_model_and_inputs`.
- CN: 实现可调用逻辑，例如 `get_model_and_inputs`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-104
````python
 81:         model.generation_config.temperature = 0.0
 82: 
 83:         vocab_size = tokenizer.vocab_size
 84:         input_ids = torch.randint(
 85:             low=0,
 86:             high=vocab_size,
 87:             size=(1, TextGenerationBenchmark.INPUT_LENGTH),
 88:             device=device,
 89:             dtype=torch.long,
 90:         )
 91:         example_inputs = {"input_ids": input_ids}
 92: 
 93:         return model, example_inputs
 94: 
 95: 
 96: HF_LLM_MODELS: dict[str, Benchmark] = {
 97:     "meta-llama/Llama-3.2-1B": TextGenerationBenchmark,
 98:     "google/gemma-2-2b": TextGenerationBenchmark,
 99:     "google/gemma-3-4b-it": TextGenerationBenchmark,
100:     "openai/whisper-tiny": WhisperBenchmark,
101:     "Qwen/Qwen3-0.6B": TextGenerationBenchmark,
102:     "mistralai/Mistral-7B-Instruct-v0.3": TextGenerationBenchmark,
103:     "openai/gpt-oss-20b": TextGenerationBenchmark,
104: }
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `pip_install` / 符号 `pip_install`
- Symbol `Benchmark` / 符号 `Benchmark`
- Symbol `get_model_and_inputs` / 符号 `get_model_and_inputs`
- Symbol `WhisperBenchmark` / 符号 `WhisperBenchmark`

## Dependencies / 依赖关系
- Python imports: `subprocess`, `sys`, `torch`, `transformers`
- Python 导入: `subprocess`, `sys`, `torch`, `transformers`
