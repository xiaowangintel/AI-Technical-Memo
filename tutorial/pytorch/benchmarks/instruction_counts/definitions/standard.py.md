# standard.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/instruction_counts/definitions/standard.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: """Default set of benchmarks.
 2: 
 3: Parser notes:
 4:     `parse_stmts`:
 5:         - Width for the left (Python) column MUST be 40 characters.
 6:         - The column separator is " | ", not "|". Whitespace matters.
 7: 
 8:     `GroupedVariants`:
 9:         - `Setup` and `Global_Setup` (case insensitive) are reserved keywords
10:           to populate `setup` and `global_setup` for every generated benchmark.
11:         - To set a label for the succeeding block, add `# @YOUR_LABEL` (Python)
12:           or `// @YOUR_LABEL` (C++).
13: """
14: # mypy: ignore-errors
15: 
16: from core.api import GroupedModules, GroupedStmts, GroupedVariants
17: from core.types import FlatIntermediateDefinition
18: from core.utils import flatten, parse_stmts
19: 
20: from definitions.setup import Setup
21: 
22: 
23: BENCHMARKS: FlatIntermediateDefinition = flatten(
24:     {
25:         "Empty": {
26:             "no allocation": GroupedStmts(
27:                 r"torch.empty(())",
28:                 r"torch::empty({0});",
29:             ),
30:             "with allocation": GroupedStmts(
31:                 r"torch.empty((1,))",
32:                 r"torch::empty({1});",
33:             ),
34:             "overloads": GroupedVariants(
35:                 cpp_block=r"""
36:                 // @Setup
37:                 auto options_empty = c10::TensorOptions();
38:                 auto options_full = c10::TensorOptions().dtype(at::kFloat).device(at::kCPU);
39:                 auto optional_float = std::make_optional(at::kFloat);
40: 
41:                 // @TensorOptions overload
42:                 at::empty({0}, options_empty);
43:                 at::empty({0}, options_full);
44:                 at::empty({0}, at::kFloat); // implicit conversion
45: 
46:                 // @Faithful overload
47:                 at::empty({0}, std::nullopt, std::nullopt, std::nullopt, std::nullopt, std::nullopt);
48:                 at::empty({0}, at::kFloat, std::nullopt, std::nullopt, std::nullopt, std::nullopt);
49:                 at::empty({0}, optional_float, std::nullopt, std::nullopt, std::nullopt, std::nullopt);
50:             """
51:             ),
52:         },
53:         "Pointwise": {
54:             "Math": GroupedVariants(
55:                 *parse_stmts(
56:                     r"""
57:             Python                                   | C++
58:             ---------------------------------------- | ----------------------------------------
59:             # @setup                                 | // @setup
60:             torch.manual_seed(138_10_23)             | torch::manual_seed(1381023);
````
- EN: Handles module imports such as `core.api`, `core.types`, `core.utils`, `definitions.setup`.
- CN: 处理模块导入，例如 `core.api`, `core.types`, `core.utils`, `definitions.setup`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61:             x = torch.rand((4, 4))                   | auto x = torch::rand({4, 4});
 62:             y_float = torch.ones((4, 4))             | auto y_float = torch::ones({4, 4});
 63:             y_vector = torch.ones((4, 1))            | auto y_vector = torch::ones({4, 1});
 64:             y_int = torch.ones(                      | auto y_int = torch::ones({4, 4}, at::kInt);
 65:                 (4, 4), dtype=torch.int32)           |
 66:                                                      |
 67:             # @add                                   | // @add
 68:             x += 1.0                                 | x += 1;
 69:             x += y_float                             | x += y_float;
 70:             x += y_vector                            | x += y_vector;
 71:             x += y_int                               | x += y_int;
 72:             x + y_float                              | x + y_float;
 73:             torch.add(x, y_float)                    | torch::add(x, y_float);
 74:             torch.add(x, y_float, out=x)             | torch::add_out(/*out=*/x, x, y_float);
 75:                                                      |
 76:             # @multiply                              | // @multiply
 77:             x *= 1.0                                 | x *= 1;
 78:             x *= y_float                             | x *= y_float;
 79:             x *= y_vector                            | x *= y_vector;
 80:             x *= y_int                               | x *= y_int;
 81:             x * y_float                              | x * y_float;
 82:             torch.mul(x, y_float)                    | torch::mul(x, y_float);
 83:             torch.mul(x, y_float, out=x)             | torch::mul_out(/*out=*/x, x, y_float);
 84:                                                      |
 85:             # @equality                              | // @equality
 86:             x == y_float                             | x == y_float;
 87:             x == 1.0                                 | x == 1.0;
 88:         """
 89:                 )
 90:             ),
 91:             "Data movement": GroupedVariants(
 92:                 *parse_stmts(
 93:                     r"""
 94:             Python                                   | C++
 95:             ---------------------------------------- | ----------------------------------------
 96:             # @setup                                 | // @setup
 97:             x = torch.ones((4, 4))                   | auto x = torch::ones({4, 4});
 98:             y = torch.ones((4, 4))                   | auto y = torch::ones({4, 4});
 99:             x_t = x.t()                              | auto x_t = x.t();
100:                                                      |
101:             # @contiguous (trivial)                  | // @contiguous (trivial)
102:             x.contiguous()                           | x.contiguous();
103:                                                      |
104:             # @contiguous (non-trivial)              | // @contiguous (non-trivial)
105:             x_t.contiguous()                         | x_t.contiguous();
106:                                                      |
107:             # @clone                                 | // @clone
108:             x.clone()                                | x.clone();
109:                                                      |
110:             # @copy_                                 | // @copy_
111:             x.copy_(y)                               | x.copy_(y);
112:                                                      |
113:             # @zero_                                 | // @zero_
114:             x.zero_()                                | x.zero_();
115:                                                      |
116:             # @RNG                                   | // @RNG
117:             x.uniform_()                             | x.uniform_();
118:         """
119:                 )
120:             ),
````
- EN: This range contributes implementation details for the file goal: Implements benchmark definitions, helpers, or runners for performance measurement.
- CN: 该范围为文件目标提供实现细节：实现用于性能测量的基准定义、辅助工具或运行器。

### Lines 121-180
````python
121:         },
122:         "Reduction": GroupedVariants(
123:             *parse_stmts(
124:                 r"""
125:         Python                                   | C++
126:         ---------------------------------------- | ----------------------------------------
127:         # @setup                                 | // @setup
128:         x = torch.ones((4, 4))                   | auto x = torch::ones({4, 4});
129:                                                  |
130:         # @max                                   | // @max
131:         x.max()                                  | x.max();
132:                                                  |
133:         # @sum                                   | // @sum
134:         x.sum()                                  | x.sum();
135:                                                  |
136:         # @variance                              | // @variance
137:         x.var(0)                                 | x.var(0);
138:     """
139:             )
140:         ),
141:         "Indexing": GroupedVariants(
142:             *parse_stmts(
143:                 r"""
144:         Python                                   | C++
145:         ---------------------------------------- | ----------------------------------------
146:         # @setup                                 | // @setup
147:                                                  | using namespace torch::indexing;
148:         torch.manual_seed(6626_10_34)            | torch::manual_seed(66261034);
149:                                                  |
150:         x = torch.randn(1, 1, 1)                 | auto x = torch::randn({1, 1, 1});
151:         y = torch.randn(1, 1, 1)                 | auto y = torch::randn({1, 1, 1});
152:                                                  |
153:         # @Tensor-Scalar                         | // @Tensor-Scalar
154:         x[0] = 1                                 | x.index_put_({0}, 1);
155:         x[0, 0] = 1                              | x.index_put_({0, 0}, 1);
156:         x[0, 0, 0] = 1                           | x.index_put_({0, 0, 0}, 1);
157:                                                  |
158:         # @Tensor-Scalar (Advanced)              | // @Tensor-Scalar (Advanced)
159:         x[...] = 1                               | x.index_put_({"..."}, 1);
160:         x[:] = 1                                 | x.index_put_({Slice(None, None, None)}, 1);
161:         x[None] = 1                              | x.index_put_({None}, 1);
162:         x[False] = 1                             | x.index_put_({false}, 1);
163:         x[True] = 1                              | x.index_put_({true}, 1);
164:                                                  |
165:         # @Tensor-Tensor                         | // @Tensor-Tensor
166:         x[0] = y[0]                              | x.index_put_({0}, y.index({0}));
167:         x[0, 0] = y[0, 0]                        | x.index_put_({0, 0}, y.index({0, 0}));
168:         x[0, 0, 0] = y[0, 0, 0]                  | x.index_put_({0, 0, 0}, y.index({0, 0, 0}));
169:                                                  |
170:         # @Tensor-Tensor (Advanced)              | // @Tensor-Tensor (Advanced)
171:         x[...] = y[...]                          | x.index_put_({"..."}, y.index({"..."}));
172:         x[:] = y[:]                              | x.index_put_({Slice(None, None, None)}, y.index({Slice(None, None, None)}));
173:         x[None] = y[None]                        | x.index_put_({None}, y.index({None}));
174:         x[False] = y[False]                      | x.index_put_({false}, y.index({false}));
175:         x[True] = y[True]                        | x.index_put_({true}, y.index({true}));
176:     """
177:             )
178:         ),
179:         "Metadata and views": GroupedVariants(
180:             *parse_stmts(
````
- EN: This range contributes implementation details for the file goal: Implements benchmark definitions, helpers, or runners for performance measurement.
- CN: 该范围为文件目标提供实现细节：实现用于性能测量的基准定义、辅助工具或运行器。

### Lines 181-240
````python
181:                 r"""
182:         Python                                   | C++
183:         ---------------------------------------- | ----------------------------------------
184:         # @setup                                 | // @setup
185:         x = torch.ones((4, 4))                   | auto x = torch::ones({4, 4});
186:                                                  |
187:         # @size                                  | // @size
188:         x.size()[0]                              | x.sizes()[0];
189:                                                  |
190:         # @stride                                | // @stride
191:         x.stride(0)                              | x.stride(0);
192:                                                  |
193:         # @as_strided                            | // @as_strided
194:         torch.as_strided(x, (2, 3), (4, 1), 2)   | torch::as_strided(x, {2, 3}, {4, 1}, 2);
195:                                                  |
196:         # @select                                | // @select
197:         x.select(1, 1)                           | x.select(1, 1);
198:                                                  |
199:         # @unsqueeze                             | // @unsqueeze
200:         x.unsqueeze(0)                           | x.unsqueeze(0);
201:                                                  |
202:         # @view                                  | // @view
203:         x.view(-1, 1)                            | x.view({-1, 1});
204:                                                  |
205:         # @transpose                             | // @transpose
206:         x.t()                                    | x.t();
207:                                                  |
208:         # @reshape                               | // @reshape
209:         x.reshape((16, 1))                       | x.reshape({16, 1});
210:     """
211:             )
212:         ),
213:         "nn Modules": {
214:             py_constructor.split("(")[0]: GroupedModules(
215:                 f"model = torch.nn.{py_constructor}",
216:                 f"auto model = torch::nn::{cpp_constructor};",
217:                 setup=setup.value,
218:                 signature="f(x) -> y",
219:                 torchscript=torchscript,
220:             )
221:             for setup, torchscript, (py_constructor, cpp_constructor) in (
222:                 (Setup.TRIVIAL_4D, True, ("BatchNorm2d(4)",) * 2),
223:                 (Setup.TRIVIAL_4D, True, ("GroupNorm(2, 4)",) * 2),
224:                 (
225:                     Setup.TRIVIAL_4D,
226:                     True,
227:                     ("LayerNorm(4)", "LayerNorm(torch::nn::LayerNormOptions({4}))"),
228:                 ),
229:                 (Setup.TRIVIAL_3D, True, ("Conv1d(4, 4, 1)",) * 2),
230:                 (Setup.TRIVIAL_4D, True, ("Conv2d(4, 4, 1)",) * 2),
231:                 (Setup.TRIVIAL_4D, True, ("MaxPool2d(2)",) * 2),
232:                 (Setup.TRIVIAL_2D, True, ("ReLU()",) * 2),
233:                 (Setup.TRIVIAL_2D, True, ("Sigmoid()",) * 2),
234:                 (Setup.TRIVIAL_4D, True, ("Linear(4, 2)",) * 2),
235:                 # TODO: LSTM can't be TorchScript'd
236:                 (Setup.TRIVIAL_3D, False, ("LSTM(4, 2)",) * 2),
237:             )
238:         },
239:         "training": {
240:             "simple": GroupedStmts(
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 241-295
````python
241:                 *parse_stmts(
242:                     r"""
243:                 Python                                   | C++
244:                 ---------------------------------------- | ----------------------------------------
245:                 a0 = torch.nn.functional.relu(x * w0)    | auto a0 = torch::nn::functional::relu(x * w0);
246:                 y = a0 * w1                              | auto y = a0 * w1;
247:             """
248:                 ),
249:                 Setup.TRAINING.value,
250:                 num_threads=(1, 2),
251:                 signature=r"f(x, w0, w1) -> y",
252:                 torchscript=True,
253:                 autograd=True,
254:             ),
255:             "ensemble": GroupedStmts(
256:                 *parse_stmts(
257:                     r"""
258:                 Python                                   | C++
259:                 ---------------------------------------- | ----------------------------------------
260:                 a0 = torch.nn.functional.gelu(x * w0)    | auto a0 = torch::nn::functional::gelu(x * w0);
261:                 a1 = torch.nn.functional.prelu(y, w1)    | auto a1 = torch::nn::functional::prelu(y, w1);
262:                 z = torch.nn.functional.normalize(       | auto z = torch::nn::functional::normalize(
263:                     torch.cat([a0, a1]),                 |     torch::cat({a0, a1}),
264:                     p=2.0, dim=0,                        |     torch::nn::functional::NormalizeFuncOptions().p(2).dim(0)
265:                 ).dot(w2)                                | ).dot(w2);
266:             """
267:                 ),
268:                 Setup.TRAINING.value,
269:                 num_threads=(1, 2),
270:                 signature=r"f(x, y, w0, w1, w2) -> z",
271:                 torchscript=True,
272:                 autograd=True,
273:             ),
274:         },
275:         "InferenceMode": GroupedVariants(
276:             # In general, the mixed input scenario is less common so its
277:             # perf can be less important than pure inference tensor inputs.
278:             cpp_block=r"""
279:             // @Setup
280:             auto s = torch::ones({3, 3});  // Normal Tensor
281:             c10::InferenceMode guard;
282:             auto x = torch::ones({3, 3});  // Inference Tensor
283: 
284:             // @View
285:             torch::Tensor y = x.view({9});
286: 
287:             // @Inplace
288:             torch::Tensor y = x.mul_(x);
289: 
290:             // @Mixed
291:             torch::Tensor y = x + s;
292:         """
293:         ),
294:     }
295: )
````
- EN: This range contributes implementation details for the file goal: Implements benchmark definitions, helpers, or runners for performance measurement.
- CN: 该范围为文件目标提供实现细节：实现用于性能测量的基准定义、辅助工具或运行器。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试

## Dependencies / 依赖关系
- Python imports: `core.api`, `core.types`, `core.utils`, `definitions.setup`
- Python 导入: `core.api`, `core.types`, `core.utils`, `definitions.setup`
