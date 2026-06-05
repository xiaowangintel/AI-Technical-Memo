# compile_benchmark.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/diffusion/compile_benchmark.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: """
 2: Compile benchmarks for diffusion model transformers.
 3: 
 4: These models have large numbers of repeated transformer blocks, leading to high
 5: cold-start compile times under full-model compilation. This makes them good
 6: candidates for regional/hierarchical compilation strategies that reduce compile
 7: time by compiling individual blocks instead of the full model.
 8: 
 9: But if you are looking to reduce the cold start time for full models, these are
10: excellent candidates.
11: 
12: Example:
13:     python compile_benchmark.py --model auroflow --mode full
14:     python compile_benchmark.py --model flux --mode full --backend eager
15: 
16: If you see issues downloading models, try: pip uninstall hf_xet
17: """
18: 
19: import argparse
20: import time
21: 
22: import diffusers
23: from diffusers import (
24:     AuraFlowPipeline,
25:     AuraFlowTransformer2DModel,
26:     GGUFQuantizationConfig,
27: )
28: 
29: import torch
30: from torch._higher_order_ops.invoke_subgraph import mark_compile_region
31: 
32: 
33: def compile_model(model, mode, block_types, backend="inductor"):
34:     if mode == "full":
35:         model.compile(backend=backend, fullgraph=True, mode="reduce-overhead")
36:     elif mode == "regional":
37:         for submod in model.modules():
38:             if isinstance(submod, block_types):
39:                 print("Compiling", submod.__class__)
40:                 submod.compile(backend=backend, fullgraph=True)
````
- EN: Handles module imports such as `argparse`, `time`, `diffusers`, `torch`.
- CN: 处理模块导入，例如 `argparse`, `time`, `diffusers`, `torch`。
- EN: Implements callable logic such as `compile_model`.
- CN: 实现可调用逻辑，例如 `compile_model`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````python
41:     elif mode == "hierarchical":
42:         for submod in model.modules():
43:             if isinstance(submod, block_types):
44:                 submod.__class__.forward = mark_compile_region(submod.__class__.forward)
45:         model.compile(backend=backend, fullgraph=True)
46: 
47: 
48: def bench(run_fn, warmup_steps=1, bench_steps=50):
49:     run_fn(warmup_steps)
50:     t0 = time.perf_counter()
51:     run_fn(bench_steps)
52:     t1 = time.perf_counter()
53:     print(f"{t1 - t0:.3f}s")
54: 
55: 
56: def auroflow_benchmark(mode, backend="inductor"):
57:     transformer = AuraFlowTransformer2DModel.from_single_file(
58:         "https://huggingface.co/city96/AuraFlow-v0.3-gguf/blob/main/aura_flow_0.3-Q2_K.gguf",
59:         quantization_config=GGUFQuantizationConfig(compute_dtype=torch.bfloat16),
60:         torch_dtype=torch.bfloat16,
61:     )
62:     pipe = AuraFlowPipeline.from_pretrained(
63:         "fal/AuraFlow-v0.3",
64:         torch_dtype=torch.bfloat16,
65:         transformer=transformer,
66:     ).to("cuda")
67: 
68:     block_types = (
69:         diffusers.models.transformers.auraflow_transformer_2d.AuraFlowSingleTransformerBlock,
70:         diffusers.models.transformers.auraflow_transformer_2d.AuraFlowJointTransformerBlock,
71:     )
72:     compile_model(pipe.transformer, mode, block_types, backend)
73: 
74:     def run(steps):
75:         pipe("A cute pony", width=512, height=512, num_inference_steps=steps)
76: 
77:     bench(run)
78: 
79: 
80: def wan_benchmark(mode, backend="inductor"):
````
- EN: Implements callable logic such as `bench`, `auroflow_benchmark`, `run`, `wan_benchmark`.
- CN: 实现可调用逻辑，例如 `bench`, `auroflow_benchmark`, `run`, `wan_benchmark`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-120
````python
 81:     import numpy as np
 82:     from diffusers import AutoencoderKLWan, WanImageToVideoPipeline
 83:     from diffusers.utils import load_image
 84:     from transformers import CLIPVisionModel
 85: 
 86:     model_id = "Wan-AI/Wan2.1-I2V-14B-480P-Diffusers"
 87:     image_encoder = CLIPVisionModel.from_pretrained(
 88:         model_id, subfolder="image_encoder", torch_dtype=torch.float32
 89:     )
 90:     vae = AutoencoderKLWan.from_pretrained(
 91:         model_id, subfolder="vae", torch_dtype=torch.float32
 92:     )
 93:     pipe = WanImageToVideoPipeline.from_pretrained(
 94:         model_id, vae=vae, image_encoder=image_encoder, torch_dtype=torch.bfloat16
 95:     ).to("cuda")
 96: 
 97:     image = load_image(
 98:         "https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/diffusers/astronaut.jpg"
 99:     )
100:     max_area = 480 * 832
101:     aspect_ratio = image.height / image.width
102:     mod_value = pipe.vae_scale_factor_spatial * pipe.transformer.config.patch_size[1]
103:     height = round(np.sqrt(max_area * aspect_ratio)) // mod_value * mod_value
104:     width = round(np.sqrt(max_area / aspect_ratio)) // mod_value * mod_value
105:     image = image.resize((width, height))
106: 
107:     prompt = (
108:         "An astronaut hatching from an egg, on the surface of the moon, the darkness and depth of space realised in "
109:         "the background. High quality, ultrarealistic detail and breath-taking movie-like camera shot."
110:     )
111:     negative_prompt = "Bright tones, overexposed, static, blurred details, subtitles, style, works, paintings, images, static, overall gray, worst quality, low quality, JPEG compression residue, ugly, incomplete, extra fingers, poorly drawn hands, poorly drawn faces, deformed, disfigured, misshapen limbs, fused fingers, still picture, messy background, three legs, many people in the background, walking backwards"
112: 
113:     block_types = (diffusers.models.transformers.transformer_wan.WanTransformerBlock,)
114:     compile_model(pipe.transformer, mode, block_types, backend)
115: 
116:     def run(steps):
117:         pipe(
118:             image=image,
119:             prompt=prompt,
120:             negative_prompt=negative_prompt,
````
- EN: Handles module imports such as `numpy`, `diffusers`, `diffusers.utils`, `transformers`.
- CN: 处理模块导入，例如 `numpy`, `diffusers`, `diffusers.utils`, `transformers`。
- EN: Implements callable logic such as `run`.
- CN: 实现可调用逻辑，例如 `run`。

### Lines 121-160
````python
121:             height=height,
122:             width=width,
123:             num_frames=33,
124:             num_inference_steps=steps,
125:             guidance_scale=5.0,
126:         )
127: 
128:     bench(run)
129: 
130: 
131: def ltx_benchmark(mode, backend="inductor"):
132:     from diffusers import LTXConditionPipeline
133: 
134:     pipe = LTXConditionPipeline.from_pretrained(
135:         "Lightricks/LTX-Video-0.9.7-dev", torch_dtype=torch.bfloat16
136:     ).to("cuda")
137:     pipe.vae.enable_tiling()
138: 
139:     height = 512 - (512 % pipe.vae_spatial_compression_ratio)
140:     width = 704 - (704 % pipe.vae_spatial_compression_ratio)
141: 
142:     prompt = "The video depicts a winding mountain road covered in snow, with a single vehicle traveling along it. The road is flanked by steep, rocky cliffs and sparse vegetation. The landscape is characterized by rugged terrain and a river visible in the distance. The scene captures the solitude and beauty of a winter drive through a mountainous region."
143:     negative_prompt = "worst quality, inconsistent motion, blurry, jittery, distorted"
144: 
145:     block_types = (
146:         diffusers.models.transformers.transformer_ltx.LTXVideoTransformerBlock,
147:     )
148:     compile_model(pipe.transformer, mode, block_types, backend)
149: 
150:     def run(steps):
151:         pipe(
152:             conditions=None,
153:             prompt=prompt,
154:             negative_prompt=negative_prompt,
155:             width=width,
156:             height=height,
157:             num_frames=121,
158:             num_inference_steps=steps,
159:             generator=torch.Generator().manual_seed(0),
160:             output_type="latent",
````
- EN: Handles module imports such as `diffusers`.
- CN: 处理模块导入，例如 `diffusers`。
- EN: Implements callable logic such as `ltx_benchmark`, `run`.
- CN: 实现可调用逻辑，例如 `ltx_benchmark`, `run`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 161-200
````python
161:         )
162: 
163:     bench(run)
164: 
165: 
166: def flux_benchmark(mode, backend="inductor"):
167:     from diffusers import FluxPipeline
168: 
169:     pipe = FluxPipeline.from_pretrained(
170:         "black-forest-labs/FLUX.1-dev",
171:         torch_dtype=torch.bfloat16,
172:     ).to("cuda")
173: 
174:     prompt = "A cat holding a sign that says hello world"
175: 
176:     block_types = (
177:         diffusers.models.transformers.transformer_flux.FluxTransformerBlock,
178:         diffusers.models.transformers.transformer_flux.FluxSingleTransformerBlock,
179:     )
180:     compile_model(pipe.transformer, mode, block_types, backend)
181: 
182:     def run(steps):
183:         pipe(
184:             prompt,
185:             height=1024,
186:             width=1024,
187:             guidance_scale=3.5,
188:             num_inference_steps=steps,
189:             max_sequence_length=512,
190:         )
191: 
192:     bench(run)
193: 
194: 
195: BENCHMARKS = {
196:     "auroflow": auroflow_benchmark,
197:     "wan": wan_benchmark,
198:     "ltx": ltx_benchmark,
199:     "flux": flux_benchmark,
200: }
````
- EN: Handles module imports such as `diffusers`.
- CN: 处理模块导入，例如 `diffusers`。
- EN: Implements callable logic such as `flux_benchmark`, `run`.
- CN: 实现可调用逻辑，例如 `flux_benchmark`, `run`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 201-210
````python
201: 
202: MODES = ("eager", "full", "regional", "hierarchical")
203: 
204: if __name__ == "__main__":
205:     parser = argparse.ArgumentParser(description=__doc__)
206:     parser.add_argument("--model", choices=BENCHMARKS, required=True)
207:     parser.add_argument("--mode", choices=MODES, required=True)
208:     parser.add_argument("--backend", default="inductor")
209:     args = parser.parse_args()
210:     BENCHMARKS[args.model](args.mode, args.backend)
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `compile_model` / 符号 `compile_model`
- Symbol `bench` / 符号 `bench`
- Symbol `auroflow_benchmark` / 符号 `auroflow_benchmark`
- Symbol `run` / 符号 `run`

## Dependencies / 依赖关系
- Python imports: `argparse`, `time`, `diffusers`, `torch`, `torch._higher_order_ops.invoke_subgraph`, `numpy`, `diffusers.utils`, `transformers`
- Python 导入: `argparse`, `time`, `diffusers`, `torch`, `torch._higher_order_ops.invoke_subgraph`, `numpy`, `diffusers.utils`, `transformers`
