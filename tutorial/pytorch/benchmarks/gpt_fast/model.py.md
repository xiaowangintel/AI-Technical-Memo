# model.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/gpt_fast/model.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: # flake8: noqa: E266, C417
 2: from dataclasses import dataclass
 3: 
 4: import torch
 5: import torch.nn as nn
 6: from torch import Tensor
 7: from torch.nn import functional as F
 8: 
 9: 
10: def find_multiple(n: int, k: int) -> int:
11:     if n % k == 0:
12:         return n
13:     return n + k - (n % k)
14: 
15: 
16: @dataclass
17: class ModelArgs:
18:     block_size: int = 2048
19:     vocab_size: int = 32000
20:     n_layer: int = 32
21:     n_head: int = 32
22:     dim: int = 4096
23:     intermediate_size: int = None
24:     n_local_heads: int = -1
25:     head_dim: int = 64
26:     rope_base: float = 10000
27:     norm_eps: float = 1e-5
28: 
29:     def __post_init__(self):
30:         if self.n_local_heads == -1:
31:             self.n_local_heads = self.n_head
32:         if self.intermediate_size is None:
33:             hidden_dim = 4 * self.dim
34:             n_hidden = int(2 * hidden_dim / 3)
35:             self.intermediate_size = find_multiple(n_hidden, 256)
36:         self.head_dim = self.dim // self.n_head
37: 
38:     @classmethod
39:     def from_name(cls, name: str):
40:         if name in transformer_configs:
41:             return cls(**transformer_configs[name])
42:         # fuzzy search
43:         config = [
44:             config
45:             for config in transformer_configs
46:             if config in str(name).upper() or config in str(name)
47:         ]
48: 
49:         # We may have two or more configs matched (e.g. "7B" and "Mistral-7B"). Find the best config match,
50:         # take longer name (as it have more symbols matched)
51:         if len(config) > 1:
52:             config.sort(key=len, reverse=True)
53:             if len(config[0]) == len(config[1]):
54:                 raise AssertionError(
55:                     f"Ambiguous config match for '{name}': {config[0]} and {config[1]} have same length"
56:                 )  # make sure only one 'best' match
57: 
58:         return cls(**transformer_configs[config[0]])
59: 
60: 
````
- EN: Handles module imports such as `dataclasses`, `torch`, `torch.nn`.
- CN: 处理模块导入，例如 `dataclasses`, `torch`, `torch.nn`。
- EN: Declares or extends types including `ModelArgs`.
- CN: 声明或扩展类型，包括 `ModelArgs`。
- EN: Implements callable logic such as `find_multiple`, `__post_init__`, `from_name`.
- CN: 实现可调用逻辑，例如 `find_multiple`, `__post_init__`, `from_name`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61: transformer_configs = {
 62:     "CodeLlama-7b-Python-hf": dict(
 63:         block_size=16384, vocab_size=32000, n_layer=32, dim=4096, rope_base=1000000
 64:     ),
 65:     "7B": dict(n_layer=32, n_head=32, dim=4096),
 66:     "13B": dict(n_layer=40, n_head=40, dim=5120),
 67:     "30B": dict(n_layer=60, n_head=52, dim=6656),
 68:     "34B": dict(
 69:         n_layer=48,
 70:         n_head=64,
 71:         dim=8192,
 72:         vocab_size=32000,
 73:         n_local_heads=8,
 74:         intermediate_size=22016,
 75:         rope_base=1000000,
 76:     ),  # CodeLlama-34B-Python-hf
 77:     "70B": dict(
 78:         n_layer=80, n_head=64, dim=8192, n_local_heads=8, intermediate_size=28672
 79:     ),
 80:     "Mistral-7B": dict(
 81:         n_layer=32,
 82:         n_head=32,
 83:         n_local_heads=8,
 84:         dim=4096,
 85:         intermediate_size=14336,
 86:         vocab_size=32000,
 87:     ),
 88: }
 89: 
 90: 
 91: class KVCache(nn.Module):
 92:     def __init__(
 93:         self, max_batch_size, max_seq_length, n_heads, head_dim, dtype=torch.bfloat16
 94:     ):
 95:         super().__init__()
 96:         cache_shape = (max_batch_size, n_heads, max_seq_length, head_dim)
 97:         self.register_buffer("k_cache", torch.zeros(cache_shape, dtype=dtype))
 98:         self.register_buffer("v_cache", torch.zeros(cache_shape, dtype=dtype))
 99: 
100:     def update(self, input_pos, k_val, v_val):
101:         # input_pos: [S], k_val: [B, H, S, D]
102:         if input_pos.shape[0] != k_val.shape[2]:
103:             raise AssertionError(
104:                 f"input_pos.shape[0] ({input_pos.shape[0]}) must equal k_val.shape[2] ({k_val.shape[2]})"
105:             )
106: 
107:         k_out = self.k_cache
108:         v_out = self.v_cache
109:         k_out[:, :, input_pos] = k_val
110:         v_out[:, :, input_pos] = v_val
111: 
112:         return k_out, v_out
113: 
114: 
115: class Transformer(nn.Module):
116:     def __init__(self, config: ModelArgs) -> None:
117:         super().__init__()
118:         self.config = config
119: 
120:         self.tok_embeddings = nn.Embedding(config.vocab_size, config.dim)
````
- EN: Declares or extends types including `KVCache`, `Transformer`.
- CN: 声明或扩展类型，包括 `KVCache`, `Transformer`。
- EN: Implements callable logic such as `__init__`, `update`.
- CN: 实现可调用逻辑，例如 `__init__`, `update`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:         self.layers = nn.ModuleList(
122:             TransformerBlock(config) for _ in range(config.n_layer)
123:         )
124:         self.norm = RMSNorm(config.dim, eps=config.norm_eps)
125:         self.output = nn.Linear(config.dim, config.vocab_size, bias=False)
126: 
127:         self.freqs_cis: Tensor | None = None
128:         self.mask_cache: Tensor | None = None
129:         self.max_batch_size = -1
130:         self.max_seq_length = -1
131: 
132:     def setup_caches(self, max_batch_size, max_seq_length):
133:         if (
134:             self.max_seq_length >= max_seq_length
135:             and self.max_batch_size >= max_batch_size
136:         ):
137:             return
138:         head_dim = self.config.dim // self.config.n_head
139:         max_seq_length = find_multiple(max_seq_length, 8)
140:         self.max_seq_length = max_seq_length
141:         self.max_batch_size = max_batch_size
142:         for b in self.layers:
143:             b.attention.kv_cache = KVCache(
144:                 max_batch_size, max_seq_length, self.config.n_local_heads, head_dim
145:             )
146: 
147:         self.freqs_cis = precompute_freqs_cis(
148:             self.config.block_size,
149:             self.config.dim // self.config.n_head,
150:             self.config.rope_base,
151:         )
152:         self.causal_mask = torch.tril(
153:             torch.ones(self.max_seq_length, self.max_seq_length, dtype=torch.bool)
154:         )
155: 
156:     def forward(self, idx: Tensor, input_pos: Tensor | None = None) -> Tensor:
157:         if self.freqs_cis is None:
158:             raise AssertionError("Caches must be initialized first")
159:         mask = self.causal_mask[None, None, input_pos]
160:         freqs_cis = self.freqs_cis[input_pos]
161:         x = self.tok_embeddings(idx)
162: 
163:         for i, layer in enumerate(self.layers):
164:             x = layer(x, input_pos, freqs_cis, mask)
165:         x = self.norm(x)
166:         logits = self.output(x)
167:         return logits
168: 
169:     @classmethod
170:     def from_name(cls, name: str):
171:         return cls(ModelArgs.from_name(name))
172: 
173: 
174: class TransformerBlock(nn.Module):
175:     def __init__(self, config: ModelArgs) -> None:
176:         super().__init__()
177:         self.attention = Attention(config)
178:         self.feed_forward = FeedForward(config)
179:         self.ffn_norm = RMSNorm(config.dim, config.norm_eps)
180:         self.attention_norm = RMSNorm(config.dim, config.norm_eps)
````
- EN: Declares or extends types including `TransformerBlock`.
- CN: 声明或扩展类型，包括 `TransformerBlock`。
- EN: Implements callable logic such as `setup_caches`, `forward`, `from_name`, `__init__`.
- CN: 实现可调用逻辑，例如 `setup_caches`, `forward`, `from_name`, `__init__`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181: 
182:     def forward(
183:         self, x: Tensor, input_pos: Tensor, freqs_cis: Tensor, mask: Tensor
184:     ) -> Tensor:
185:         h = x + self.attention(self.attention_norm(x), freqs_cis, mask, input_pos)
186:         out = h + self.feed_forward(self.ffn_norm(h))
187:         return out
188: 
189: 
190: class Attention(nn.Module):
191:     def __init__(self, config: ModelArgs):
192:         super().__init__()
193:         if config.dim % config.n_head != 0:
194:             raise AssertionError(
195:                 f"config.dim ({config.dim}) must be divisible by config.n_head ({config.n_head})"
196:             )
197: 
198:         total_head_dim = (config.n_head + 2 * config.n_local_heads) * config.head_dim
199:         # key, query, value projections for all heads, but in a batch
200:         self.wqkv = nn.Linear(config.dim, total_head_dim, bias=False)
201:         self.wo = nn.Linear(config.dim, config.dim, bias=False)
202:         self.kv_cache = None
203: 
204:         self.n_head = config.n_head
205:         self.head_dim = config.head_dim
206:         self.n_local_heads = config.n_local_heads
207:         self.dim = config.dim
208:         self._register_load_state_dict_pre_hook(self.load_hook)
209: 
210:     def load_hook(self, state_dict, prefix, *args):
211:         if prefix + "wq.weight" in state_dict:
212:             wq = state_dict.pop(prefix + "wq.weight")
213:             wk = state_dict.pop(prefix + "wk.weight")
214:             wv = state_dict.pop(prefix + "wv.weight")
215:             state_dict[prefix + "wqkv.weight"] = torch.cat([wq, wk, wv])
216: 
217:     def forward(
218:         self,
219:         x: Tensor,
220:         freqs_cis: Tensor,
221:         mask: Tensor,
222:         input_pos: Tensor | None = None,
223:     ) -> Tensor:
224:         bsz, seqlen, _ = x.shape
225: 
226:         kv_size = self.n_local_heads * self.head_dim
227:         q, k, v = self.wqkv(x).split([self.dim, kv_size, kv_size], dim=-1)
228: 
229:         q = q.view(bsz, seqlen, self.n_head, self.head_dim)
230:         k = k.view(bsz, seqlen, self.n_local_heads, self.head_dim)
231:         v = v.view(bsz, seqlen, self.n_local_heads, self.head_dim)
232: 
233:         q = apply_rotary_emb(q, freqs_cis)
234:         k = apply_rotary_emb(k, freqs_cis)
235: 
236:         q, k, v = map(lambda x: x.transpose(1, 2), (q, k, v))
237: 
238:         if self.kv_cache is not None:
239:             k, v = self.kv_cache.update(input_pos, k, v)
240: 
````
- EN: Declares or extends types including `Attention`.
- CN: 声明或扩展类型，包括 `Attention`。
- EN: Implements callable logic such as `forward`, `__init__`, `load_hook`.
- CN: 实现可调用逻辑，例如 `forward`, `__init__`, `load_hook`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-299
````python
241:         k = k.repeat_interleave(self.n_head // self.n_local_heads, dim=1)
242:         v = v.repeat_interleave(self.n_head // self.n_local_heads, dim=1)
243:         y = F.scaled_dot_product_attention(q, k, v, attn_mask=mask, dropout_p=0.0)
244: 
245:         y = y.transpose(1, 2).contiguous().view(bsz, seqlen, self.dim)
246: 
247:         y = self.wo(y)
248:         return y
249: 
250: 
251: class FeedForward(nn.Module):
252:     def __init__(self, config: ModelArgs) -> None:
253:         super().__init__()
254:         self.w1 = nn.Linear(config.dim, config.intermediate_size, bias=False)
255:         self.w3 = nn.Linear(config.dim, config.intermediate_size, bias=False)
256:         self.w2 = nn.Linear(config.intermediate_size, config.dim, bias=False)
257: 
258:     def forward(self, x: Tensor) -> Tensor:
259:         return self.w2(F.silu(self.w1(x)) * self.w3(x))
260: 
261: 
262: class RMSNorm(nn.Module):
263:     def __init__(self, dim: int, eps: float = 1e-5):
264:         super().__init__()
265:         self.eps = eps
266:         self.weight = nn.Parameter(torch.ones(dim))
267: 
268:     def _norm(self, x):
269:         return x * torch.rsqrt(torch.mean(x * x, dim=-1, keepdim=True) + self.eps)
270: 
271:     def forward(self, x: Tensor) -> Tensor:
272:         output = self._norm(x.float()).type_as(x)
273:         return output * self.weight
274: 
275: 
276: def precompute_freqs_cis(seq_len: int, n_elem: int, base: int = 10000) -> Tensor:
277:     freqs = 1.0 / (
278:         base ** (torch.arange(0, n_elem, 2)[: (n_elem // 2)].float() / n_elem)
279:     )
280:     t = torch.arange(seq_len, device=freqs.device)
281:     freqs = torch.outer(t, freqs)
282:     freqs_cis = torch.polar(torch.ones_like(freqs), freqs)
283:     cache = torch.stack([freqs_cis.real, freqs_cis.imag], dim=-1)
284:     return cache.to(dtype=torch.bfloat16)
285: 
286: 
287: def apply_rotary_emb(x: Tensor, freqs_cis: Tensor) -> Tensor:
288:     xshaped = x.float().reshape(*x.shape[:-1], -1, 2)
289:     freqs_cis = freqs_cis.view(1, xshaped.size(1), 1, xshaped.size(3), 2)
290:     x_out2 = torch.stack(
291:         [
292:             xshaped[..., 0] * freqs_cis[..., 0] - xshaped[..., 1] * freqs_cis[..., 1],
293:             xshaped[..., 1] * freqs_cis[..., 0] + xshaped[..., 0] * freqs_cis[..., 1],
294:         ],
295:         -1,
296:     )
297: 
298:     x_out2 = x_out2.flatten(3)
299:     return x_out2.type_as(x)
````
- EN: Declares or extends types including `FeedForward`, `RMSNorm`.
- CN: 声明或扩展类型，包括 `FeedForward`, `RMSNorm`。
- EN: Implements callable logic such as `__init__`, `forward`, `_norm`, `precompute_freqs_cis`.
- CN: 实现可调用逻辑，例如 `__init__`, `forward`, `_norm`, `precompute_freqs_cis`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `find_multiple` / 符号 `find_multiple`
- Symbol `ModelArgs` / 符号 `ModelArgs`
- Symbol `__post_init__` / 符号 `__post_init__`
- Symbol `from_name` / 符号 `from_name`

## Dependencies / 依赖关系
- Python imports: `dataclasses`, `torch`, `torch.nn`
- Python 导入: `dataclasses`, `torch`, `torch.nn`
