# model_container.h — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/inductor/aoti_runtime/model_container.h`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Implements low-level AOTInductor runtime helpers, ABI shims, and execution support code.
- 目的 (CN): 实现底层 AOTInductor 运行时辅助逻辑、ABI 适配层与执行支持代码。
- Lines: 827
- Language: C++ / C++

## Line-by-Line Analysis / 逐行分析

### Lines 1-16

```cpp
 1: #pragma once
 2: 
 3: #include <algorithm>
 4: #include <condition_variable>
 5: #include <deque>
 6: #include <mutex>
 7: #include <shared_mutex>
 8: 
 9: // WARNING: Be careful when adding new includes here. This header will be used
10: // in model.so, and should not refer to any aten/c10 headers except the stable
11: // C ABI defined in torch/csrc/inductor/aoti_torch/c/shim.h. The same rule
12: // applies to other files under torch/csrc/inductor/aoti_runtime/.
13: #include <torch/csrc/inductor/aoti_runtime/model.h>
14: 
15: namespace torch::aot_inductor {
16: // The state transition is done by:
```

- EN: These lines pull in dependencies such as `algorithm`, `condition_variable`, `deque`, establishing the headers needed by the implementation. They also open, refine, or close the relevant C++ namespace scope. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这些行引入了依赖，例如 `algorithm`, `condition_variable`, `deque`，为后续实现建立所需的头文件基础。 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 17-32

```cpp
17: // (1) NONE state: The default state when created. This state should only exist
18: // when model_container is created and no constants are being loaded or updated.
19: // (2) INITIALIZED state: This state get set whenever we load the constants into
20: // the buffer. This could be done by load_constants or update_constants_buffer.
21: // (3) FOLDED state: This state should transition from INITIALIZED after
22: // const_fold is being invoked.
23: enum class ConstantState : uint8_t { NONE, INITIALIZED, FOLDED, UNKNOWN };
24: 
25: inline std::string toStringConstantState(ConstantState state) {
26:   switch (state) {
27:     case ConstantState::NONE:
28:       return "ConstantState::NONE";
29:     case ConstantState::INITIALIZED:
30:       return "ConstantState::INITIALIZED";
31:     case ConstantState::FOLDED:
32:       return "ConstantState::FOLDED";
```

- EN: This range declares or shapes types such as `ConstantState`. The main execution path in this span is carried by `toStringConstantState`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段声明或塑造了 ``ConstantState`` 等类型。 这一段的主要执行路径由 `toStringConstantState` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 33-48

```cpp
33:     case ConstantState::UNKNOWN:
34:       return "ConstantState::UNKNOWN";
35:     default:
36:       return "Unknown enum class state for ConstantState";
37:   }
38: }
39: 
40: class AOTInductorModelContainer {
41:  public:
42:   AOTInductorModelContainer(
43:       size_t num_models,
44:       const std::string& device_str,
45:       const std::optional<std::string>& cubin_dir = std::nullopt) {
46:     constants_map_ = std::make_shared<ConstantMap>();
47:     constants_array_ = std::make_shared<std::vector<ConstantHandle>>();
48: 
```

- EN: This range declares or shapes types such as `AOTInductorModelContainer`. The main execution path in this span is carried by `AOTInductorModelContainer`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段声明或塑造了 ``AOTInductorModelContainer`` 等类型。 这一段的主要执行路径由 `AOTInductorModelContainer` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 49-64

```cpp
49:     models_.reserve(num_models);
50:     available_models_.reserve(num_models);
51:     for (size_t i = 0; i < num_models; ++i) {
52:       models_.push_back(AOTInductorModel::Create(
53:           constants_map_, constants_array_, device_str, cubin_dir));
54:       available_models_.push_back(models_.back().get());
55:     }
56: 
57:     // Note that the all following fields (input_names_, output_names,
58:     // etc) can be filled in by the AOT
59:     // codegen. However, we choose to query such information from
60:     // the owned AOTInductorModel for a couple of reasons:
61:     //   * simplify the codegen templates
62:     //   * reduce information fragmentation and duplication
63:     //   * the initialization process below is done only once when the container
64:     //     is constructed, so it would have little performance impact
```

- EN: The main execution path in this span is carried by `fields`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `fields` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 65-80

```cpp
65:     auto* model = available_models_[0];
66:     size_t num_inputs = model->num_inputs();
67:     input_names_.reserve(num_inputs);
68:     for (size_t i = 0; i < num_inputs; i++) {
69:       input_names_.emplace_back(model->input_name(static_cast<int64_t>(i)));
70:     }
71: 
72:     size_t num_outputs = model->num_outputs();
73:     output_names_.reserve(num_outputs);
74:     for (size_t i = 0; i < num_outputs; i++) {
75:       output_names_.emplace_back(model->output_name(static_cast<int64_t>(i)));
76:     }
77:     model->load_constants();
78:     constant_blob_ = model->release_constant_blob();
79:     constants_internal_offset_.resize(
80:         model->num_constants() - model->num_folded_constants());
```

- EN: Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 81-96

```cpp
81:     secondary_cpu_constants_internal_offset_.resize(
82:         model->num_constants() - model->num_folded_constants());
83:     model->compute_constant_blob(
84:         blob_size_,
85:         constants_internal_offset_,
86:         secondary_cpu_blob_size_,
87:         secondary_cpu_constants_internal_offset_);
88:     constant_folded_ = ConstantState::INITIALIZED;
89: 
90:     for (auto& model : models_) {
91:       model->update_constants_map(constants_map_);
92:     }
93: 
94:     in_spec_ = model->get_in_spec();
95:     out_spec_ = model->get_out_spec();
96:   }
```

- EN: Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 97-112

```cpp
 97: 
 98:   void run(
 99:       AtenTensorHandle*
100:           input_handles, // array of input AtenTensorHandle; handles
101:                          // are stolen; the array itself is borrowed
102:       AtenTensorHandle*
103:           output_handles, // array for writing output AtenTensorHandle; handles
104:                           // will be stolen by the caller; the array itself is
105:                           // borrowed
106:       DeviceStreamType stream,
107:       AOTIProxyExecutorHandle proxy_executor) {
108:     std::shared_lock model_lk(model_exec_mutex_);
109:     auto* model = get_available_model();
110: 
111:     ConstantState& const_folded =
112:         use_secondary_ ? constant_folded_secondary_ : constant_folded_;
```

- EN: The main execution path in this span is carried by `run`, `model_lk`, `get_available_model`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `run`, `model_lk`, `get_available_model` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 113-128

```cpp
113:     if (const_folded == ConstantState::INITIALIZED) {
114:       // At this point, constant is not ready yet. We need to call constant
115:       // folding before we execute the model. We obtain a unique lock at this
116:       // point to make sure constant is ready for all.
117:       model_lk.unlock();
118:       std::unique_lock constants_folding_lk(model_exec_mutex_);
119:       // Double locking to make sure constant folding is only ran once.
120:       if (const_folded == ConstantState::INITIALIZED) {
121:         auto folded_const_map = model->run_const_fold(
122:             stream, proxy_executor, /* initialization = */ true);
123:         update_constant_buffer(
124:             std::move(folded_const_map),
125:             /* use_inactive = */ false,
126:             /* validate_full_update = */ false);
127:         const_folded = ConstantState::FOLDED;
128:       }
```

- EN: The main execution path in this span is carried by `constants_folding_lk`, `update_constant_buffer`, `move`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `constants_folding_lk`, `update_constant_buffer`, `move` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 129-144

```cpp
129:       constants_folding_lk.unlock();
130:       model_lk.lock();
131:     } else if (const_folded != ConstantState::FOLDED) {
132:       throw std::runtime_error(
133:           "Unknown constant state: " + toStringConstantState(constant_folded_));
134:     }
135: 
136:     try {
137:       model->run(input_handles, output_handles, stream, proxy_executor);
138:     } catch (...) {
139:       std::lock_guard lk(models_mutex_);
140:       available_models_.push_back(model);
141:       throw;
142:     }
143: 
144:     {
```

- EN: The main execution path in this span is carried by `runtime_error`, `toStringConstantState`, `lk`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `runtime_error`, `toStringConstantState`, `lk` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 145-160

```cpp
145:       std::lock_guard lk(models_mutex_);
146:       pending_models_.push_back(model);
147:     }
148:     pending_models_available_.notify_one();
149:   }
150: 
151:   // Non-thread-aware variant of run(). Obviously unsafe to use in a threaded
152:   // environment :)
153:   void run_single_threaded(
154:       AtenTensorHandle*
155:           input_handles, // array of input AtenTensorHandle; handles
156:                          // are stolen; the array itself is borrowed
157:       AtenTensorHandle*
158:           output_handles, // array for writing output AtenTensorHandle; handles
159:                           // will be stolen by the caller; the array itself is
160:                           // borrowed
```

- EN: The main execution path in this span is carried by `lk`, `run`, `run_single_threaded`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `lk`, `run`, `run_single_threaded` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 161-176

```cpp
161:       DeviceStreamType stream,
162:       AOTIProxyExecutorHandle proxy_executor) {
163:     auto* model = available_models_[0];
164: 
165:     ConstantState& const_folded =
166:         use_secondary_ ? constant_folded_secondary_ : constant_folded_;
167:     if (const_folded == ConstantState::INITIALIZED) {
168:       auto folded_const_map = model->run_const_fold(
169:           stream, proxy_executor, /* initialization = */ true);
170:       update_constant_buffer(
171:           std::move(folded_const_map),
172:           /* use_inactive = */ false,
173:           /* validate_full_update = */ false);
174:       const_folded = ConstantState::FOLDED;
175:     } else if (const_folded != ConstantState::FOLDED) {
176:       throw std::runtime_error(
```

- EN: The main execution path in this span is carried by `update_constant_buffer`, `move`, `runtime_error`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `update_constant_buffer`, `move`, `runtime_error` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 177-192

```cpp
177:           "Unknown constant state: " + toStringConstantState(const_folded));
178:     }
179: 
180:     model->run_single_threaded(
181:         input_handles, output_handles, stream, proxy_executor);
182:   }
183: 
184:   const std::unordered_map<std::string, AtenTensorHandle> extract_constants_map(
185:       bool use_inactive) const {
186:     size_t n_consts = this->num_constants();
187:     std::unordered_map<std::string, AtenTensorHandle> ret;
188:     ret.reserve(n_consts);
189: 
190:     std::shared_ptr<ConstantMap> extract_map = constants_map_;
191:     // Essentially a XOR
192:     if (use_inactive != use_secondary_) {
```

- EN: The main execution path in this span is carried by `toStringConstantState`, `extract_constants_map`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `toStringConstantState`, `extract_constants_map` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 193-208

```cpp
193:       extract_map = constants_map_secondary_;
194:     }
195:     for (size_t idx = 0; idx < n_consts; idx++) {
196:       if (this->constant_from_folded(idx)) {
197:         continue;
198:       }
199: 
200:       auto it = extract_map->find(this->constant_name(idx));
201:       if (it != extract_map->end()) {
202:         ret.emplace(this->constant_original_fqn(idx), it->second);
203:         continue;
204:       }
205:     }
206: 
207:     return ret;
208:   }
```

- EN: Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 209-224

```cpp
209: 
210:   size_t num_constants() const {
211:     if (this->num_models() == 0) {
212:       throw std::runtime_error("No available models in container!");
213:     }
214:     return models_[0]->num_constants();
215:   }
216: 
217:   // retrieve the constant name of constants_info_[idx]
218:   const char* constant_name(size_t idx) const {
219:     if (this->num_models() == 0) {
220:       throw std::runtime_error("No available models in container!");
221:     }
222:     return models_[0]->constant_name(static_cast<int64_t>(idx));
223:   }
224: 
```

- EN: The main execution path in this span is carried by `num_constants`, `runtime_error`, `constant_name`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `num_constants`, `runtime_error`, `constant_name` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 225-240

```cpp
225:   // retrieve original FQN of constants_info_[idx]
226:   const char* constant_original_fqn(size_t idx) const {
227:     if (this->num_models() == 0) {
228:       throw std::runtime_error("No available models in container!");
229:     }
230:     return models_[0]->constant_original_fqn(static_cast<int64_t>(idx));
231:   }
232: 
233:   // retrieve whether constant is from folded of constants_info_[idx]
234:   bool constant_from_folded(size_t idx) const {
235:     if (this->num_models() == 0) {
236:       throw std::runtime_error("No available models in container!");
237:     }
238:     return models_[0]->constant_from_folded(static_cast<int64_t>(idx));
239:   }
240: 
```

- EN: The main execution path in this span is carried by `constant_original_fqn`, `runtime_error`, `constant_from_folded`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `constant_original_fqn`, `runtime_error`, `constant_from_folded` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 241-256

```cpp
241:   size_t constant_data_size(size_t idx) const {
242:     if (this->num_models() == 0) {
243:       throw std::runtime_error("No available models in container!");
244:     }
245:     return models_[0]->constant_data_size(static_cast<int64_t>(idx));
246:   }
247: 
248:   // retrieve type of constants_info_[idx]
249:   int32_t constant_type(size_t idx) const {
250:     if (this->num_models() == 0) {
251:       throw std::runtime_error("No available models in container!");
252:     }
253:     return models_[0]->constant_type(static_cast<int64_t>(idx));
254:   }
255: 
256:   // retrieve dtype of constants_info_[idx]
```

- EN: The main execution path in this span is carried by `constant_data_size`, `runtime_error`, `constant_type`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `constant_data_size`, `runtime_error`, `constant_type` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 257-272

```cpp
257:   int32_t constant_dtype(size_t idx) const {
258:     if (this->num_models() == 0) {
259:       throw std::runtime_error("No available models in container!");
260:     }
261:     return models_[0]->constant_dtype(static_cast<int64_t>(idx));
262:   }
263: 
264:   uint64_t constant_blob_size() const {
265:     if (this->num_models() == 0) {
266:       throw std::runtime_error("No available models in container!");
267:     }
268:     return models_[0]->constant_blob_size();
269:   }
270: 
271:   void update_constants_from_blob(const uint8_t* weight_blob_ptr) {
272:     if (this->num_models() == 0) {
```

- EN: The main execution path in this span is carried by `constant_dtype`, `runtime_error`, `constant_blob_size`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `constant_dtype`, `runtime_error`, `constant_blob_size` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 273-288

```cpp
273:       throw std::runtime_error("No available models in container!");
274:     }
275:     return models_[0]->update_constants_from_blob(weight_blob_ptr);
276:   }
277: 
278:   void run_const_fold(
279:       bool inactive_buffer,
280:       DeviceStreamType stream,
281:       AOTIProxyExecutorHandle proxy_executor) {
282:     AOTInductorModel* model;
283:     ConstantState& const_folded = inactive_buffer == use_secondary_
284:         ? constant_folded_
285:         : constant_folded_secondary_;
286:     if (!inactive_buffer) {
287:       // We would need to acquire a unique lock if we want to run constant
288:       // folding on the active buffer.
```

- EN: The main execution path in this span is carried by `runtime_error`, `run_const_fold`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `runtime_error`, `run_const_fold` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 289-304

```cpp
289:       std::unique_lock constants_folding_lk(model_exec_mutex_);
290:       model = get_available_model();
291:       try {
292:         auto folded_const_map = model->run_const_fold(stream, proxy_executor);
293:         update_constant_buffer(
294:             std::move(folded_const_map),
295:             /* use_inactive = */ false,
296:             /* validate_full_update = */ false);
297:         const_folded = ConstantState::FOLDED;
298:       } catch (...) {
299:         std::lock_guard lk(models_mutex_);
300:         available_models_.push_back(model);
301:         throw;
302:       }
303:     } else {
304:       std::shared_lock model_lk(model_exec_mutex_);
```

- EN: The main execution path in this span is carried by `constants_folding_lk`, `get_available_model`, `update_constant_buffer`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `constants_folding_lk`, `get_available_model`, `update_constant_buffer` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 305-320

```cpp
305:       model = get_available_model();
306: 
307:       // We swap the constant mapping to the inactive buffer in the model to run
308:       // const run.
309:       auto constants_map = get_constants_map(/* get_inactive= */ true);
310:       auto constants_array = get_constants_array(/* get_inactive= */ true);
311: 
312:       try {
313:         model->update_constants_map(
314:             constants_map, /* remap_constants_array= */ false);
315:         model->update_constants_array(constants_array);
316: 
317:         auto folded_const_map = model->run_const_fold(stream, proxy_executor);
318:         update_constant_buffer(
319:             std::move(folded_const_map),
320:             /* use_inactive = */ true,
```

- EN: The main execution path in this span is carried by `get_available_model`, `get_constants_map`, `get_constants_array`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `get_available_model`, `get_constants_map`, `get_constants_array` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 321-336

```cpp
321:             /* validate_full_update = */ false);
322: 
323:         // Swap back the model's constants mapping
324:         constants_map = get_constants_map(/* get_inactive= */ false);
325:         constants_array = get_constants_array(/* get_inactive= */ false);
326:         model->update_constants_map(
327:             constants_map, /* remap_constants_array= */ false);
328:         model->update_constants_array(constants_array);
329:         const_folded = ConstantState::FOLDED;
330:       } catch (...) {
331:         std::lock_guard lk(models_mutex_);
332:         available_models_.push_back(model);
333:         throw;
334:       }
335:     }
336: 
```

- EN: The main execution path in this span is carried by `get_constants_map`, `get_constants_array`, `lk`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `get_constants_map`, `get_constants_array`, `lk` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 337-352

```cpp
337:     {
338:       std::lock_guard lk(models_mutex_);
339:       pending_models_.push_back(model);
340:     }
341:     pending_models_available_.notify_one();
342:   }
343: 
344:   bool _is_tensor_constant_type(const size_t idx) const {
345:     auto constant_type = models_[0]->constant_type(static_cast<int64_t>(idx));
346:     // We should skip constants
347:     return constant_type == ConstantType::TensorConstant;
348:   }
349: 
350:   bool _is_buffer_type(const size_t idx) const {
351:     auto constant_type = models_[0]->constant_type(static_cast<int64_t>(idx));
352:     // Buffer can be optionally skipped, so if it not provided by upstream
```

- EN: The main execution path in this span is carried by `lk`, `_is_tensor_constant_type`, `_is_buffer_type`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `lk`, `_is_tensor_constant_type`, `_is_buffer_type` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 353-368

```cpp
353:     // services, it is OK to relax the check.
354:     return constant_type == ConstantType::Buffer;
355:   }
356: 
357:   bool _is_empty_parameter_type(const size_t idx) const {
358:     auto constant_type = models_[0]->constant_type(static_cast<int64_t>(idx));
359:     auto constant_data_size =
360:         models_[0]->constant_data_size(static_cast<int64_t>(idx));
361:     // Empty parameters are skipped and not provided by the upstream services,
362:     // it is OK to skip.
363:     return constant_type == ConstantType::Parameter && constant_data_size == 0;
364:   }
365: 
366:   bool _is_tensor_constant_or_buffer_type_or_empty_parameter(
367:       const size_t idx) const {
368:     return _is_tensor_constant_type(idx) || _is_buffer_type(idx) ||
```

- EN: The main execution path in this span is carried by `_is_empty_parameter_type`, `_is_tensor_constant_or_buffer_type_or_empty_parameter`, `_is_tensor_constant_type`. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `_is_empty_parameter_type`, `_is_tensor_constant_or_buffer_type_or_empty_parameter`, `_is_tensor_constant_type` 等函数/方法承载。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 369-384

```cpp
369:         _is_empty_parameter_type(idx);
370:   }
371: 
372:   void assert_all_constants(
373:       const std::unordered_map<std::string, AtenTensorHandle>& constants_map) {
374:     auto num_constants = models_[0]->num_constants();
375:     for (size_t idx = 0; idx < num_constants; idx++) {
376:       if (models_[0]->constant_from_folded(static_cast<int64_t>(idx))) {
377:         continue;
378:       }
379: 
380:       auto constant_name =
381:           std::string(models_[0]->constant_name(static_cast<int64_t>(idx)));
382:       auto it = constants_map.find(constant_name);
383:       if (it == constants_map.end()) {
384:         if (_is_tensor_constant_or_buffer_type_or_empty_parameter(idx)) {
```

- EN: The main execution path in this span is carried by `_is_empty_parameter_type`, `assert_all_constants`, `string`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `_is_empty_parameter_type`, `assert_all_constants`, `string` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 385-400

```cpp
385:           // tracing sometimes creates tensors that are non-existent in
386:           // original graph. We could skip those and do a direct copy.
387:           std::cerr << "[WARNING] Found constant or module state buffer or "
388:                     << "empty module state parameter " << constant_name
389:                     << " in model, but not provided by user!\n";
390:           continue;
391:         }
392:         throw std::runtime_error(
393:             std::string("Cannot find constants ") + constant_name +
394:             std::string(" in constants_map!"));
395:       }
396:     }
397:   }
398: 
399:   // We directly take ownership from AtenTensorHandle if constants are moved.
400:   void update_constant_buffer(
```

- EN: The main execution path in this span is carried by `runtime_error`, `string`, `update_constant_buffer`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `runtime_error`, `string`, `update_constant_buffer` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 401-416

```cpp
401:       std::unordered_map<std::string, AtenTensorHandle>&& constants_map,
402:       bool use_inactive,
403:       bool validate_full_update) {
404:     if (this->num_models() == 0) {
405:       throw std::runtime_error("No model available in container!");
406:     }
407:     if (validate_full_update) {
408:       assert_all_constants(constants_map);
409:     }
410: 
411:     ConstantState& const_folded = use_inactive == use_secondary_
412:         ? constant_folded_
413:         : constant_folded_secondary_;
414:     const_folded = ConstantState::INITIALIZED;
415: 
416:     auto original_constants_map = get_constants_map(!use_inactive);
```

- EN: The main execution path in this span is carried by `runtime_error`, `assert_all_constants`, `get_constants_map`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `runtime_error`, `assert_all_constants`, `get_constants_map` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 417-432

```cpp
417:     auto constants_map_to_update = get_constants_map(use_inactive);
418: 
419:     auto num_constants = models_[0]->num_constants();
420:     for (size_t idx = 0; idx < num_constants; idx++) {
421:       auto constant_name =
422:           std::string(models_[0]->constant_name(static_cast<int64_t>(idx)));
423:       auto it = constants_map.find(constant_name);
424:       if (it == constants_map.end() &&
425:           !(use_inactive && _is_tensor_constant_type(idx))) {
426:         continue;
427:       }
428: 
429:       AtenTensorHandle tensor;
430:       if (it == constants_map.end()) {
431:         aoti_torch_clone(
432:             original_constants_map->find(constant_name)->second.get(), &tensor);
```

- EN: The main execution path in this span is carried by `get_constants_map`, `string`, `aoti_torch_clone`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `get_constants_map`, `string`, `aoti_torch_clone` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 433-448

```cpp
433:       } else {
434:         tensor = it->second;
435:       }
436: 
437:       constants_map_to_update->insert_or_assign(
438:           constant_name, RAIIAtenTensorHandle(tensor));
439:     }
440:     // Update the inactive constant array.
441:     update_array_from_map(
442:         get_constants_array(use_inactive), constants_map_to_update);
443:   }
444: 
445:   // This function updates the buffer for storing constants.
446:   // It will update the buffer, the mapping and the array mapping.
447:   void update_constant_buffer(
448:       const std::unordered_map<std::string, AtenTensorHandle>& constants_map,
```

- EN: The main execution path in this span is carried by `RAIIAtenTensorHandle`, `update_array_from_map`, `get_constants_array`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `RAIIAtenTensorHandle`, `update_array_from_map`, `get_constants_array` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 449-464

```cpp
449:       bool use_inactive,
450:       bool validate_full_update,
451:       bool user_managed = false) {
452:     if (this->num_models() == 0) {
453:       throw std::runtime_error("No model available in container!");
454:     }
455:     if (validate_full_update) {
456:       assert_all_constants(constants_map);
457:     }
458: 
459:     // update_constant_buffer does not support mixed CPU/CUDA constants
460:     int32_t model_device_type = models_[0]->get_device_type();
461:     for (const auto& kv : constants_map) {
462:       int32_t tensor_device_type = 0;
463:       aoti_torch_get_device_type(kv.second, &tensor_device_type);
464:       if (tensor_device_type != model_device_type) {
```

- EN: The main execution path in this span is carried by `runtime_error`, `assert_all_constants`, `aoti_torch_get_device_type`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `runtime_error`, `assert_all_constants`, `aoti_torch_get_device_type` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 465-480

```cpp
465:         throw std::runtime_error(
466:             "update_constant_buffer does not support mixed device constants. "
467:             "Constant '" +
468:             kv.first + "' has device type " +
469:             std::to_string(tensor_device_type) +
470:             " but model expects device type " +
471:             std::to_string(model_device_type));
472:       }
473:     }
474: 
475:     ConstantState& const_folded = use_inactive == use_secondary_
476:         ? constant_folded_
477:         : constant_folded_secondary_;
478:     const_folded = ConstantState::INITIALIZED;
479: 
480:     auto original_constants_map = get_constants_map(!use_inactive);
```

- EN: The main execution path in this span is carried by `runtime_error`, `to_string`, `get_constants_map`.
- CN: 这一段的主要执行路径由 `runtime_error`, `to_string`, `get_constants_map` 等函数/方法承载。
### Lines 481-496

```cpp
481:     auto constants_map_to_update = get_constants_map(use_inactive);
482: 
483:     auto num_constants = models_[0]->num_constants();
484:     for (size_t idx = 0; idx < num_constants; idx++) {
485:       auto constant_name =
486:           std::string(models_[0]->constant_name(static_cast<int64_t>(idx)));
487:       auto it = constants_map.find(constant_name);
488:       if (it == constants_map.end() &&
489:           !(use_inactive &&
490:             _is_tensor_constant_or_buffer_type_or_empty_parameter(idx))) {
491:         continue;
492:       }
493: 
494:       AtenTensorHandle tensor;
495:       if (it == constants_map.end()) {
496:         tensor = original_constants_map->find(constant_name)->second.get();
```

- EN: The main execution path in this span is carried by `get_constants_map`, `string`, `_is_tensor_constant_or_buffer_type_or_empty_parameter`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `get_constants_map`, `string`, `_is_tensor_constant_or_buffer_type_or_empty_parameter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 497-512

```cpp
497:       } else {
498:         tensor = it->second;
499:       }
500: 
501:       if (user_managed) {
502:         // If user managed, we pass in the pointer directly, and skip the
503:         // copy.
504:         constants_map_to_update->insert_or_assign(
505:             constant_name,
506:             MaybeOwningAtenTensorHandle(tensor, /* user_managed = */ true));
507:         continue;
508:       }
509: 
510:       auto* constants_blob_ptr =
511:           static_cast<uint8_t*>(get_constant_blob_ptr(use_inactive));
512: 
```

- EN: The main execution path in this span is carried by `MaybeOwningAtenTensorHandle`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `MaybeOwningAtenTensorHandle` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 513-528

```cpp
513:       // Move the data to container handled blob.
514:       uint8_t* internal_constants_ptr =
515:           constants_blob_ptr + constants_internal_offset_[idx];
516:       void* user_constant_ptr;
517:       int64_t constant_size;
518:       int64_t* stride;
519:       int64_t offset;
520:       aoti_torch_get_data_ptr(tensor, &user_constant_ptr);
521:       aoti_torch_get_storage_size(tensor, &constant_size);
522:       AOTI_TORCH_ERROR_CODE_CHECK(aoti_torch_get_strides(tensor, &stride));
523:       AOTI_TORCH_ERROR_CODE_CHECK(
524:           aoti_torch_get_storage_offset(tensor, &offset));
525:       auto dtype = models_[0]->constant_dtype(idx);
526: 
527: #ifdef USE_XPU
528:       sycl::queue* queue_ptr = nullptr;
```

- EN: The main execution path in this span is carried by `aoti_torch_get_data_ptr`, `aoti_torch_get_storage_size`, `AOTI_TORCH_ERROR_CODE_CHECK`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_get_data_ptr`, `aoti_torch_get_storage_size`, `AOTI_TORCH_ERROR_CODE_CHECK` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 529-544

```cpp
529:       aoti_torch_get_current_sycl_queue((void**)&queue_ptr);
530:       queue_ptr
531:           ->memcpy(internal_constants_ptr, user_constant_ptr, constant_size)
532:           .wait();
533: #elif USE_MPS
534:       internal_constants_ptr = constants_blob_ptr;
535:       aoti_torch_mps_copy_buffer(
536:           user_constant_ptr,
537:           constants_blob_ptr,
538:           constant_size,
539:           offset,
540:           constants_internal_offset_[idx]);
541:       // For mps tensors, all constants are stored in one buffer, with the
542:       // offset being where the constant starts. So we want to change the
543:       // constant tensor's offset to point to constants_internal_offset_[idx]
544:       offset = constants_internal_offset_[idx] /
```

- EN: The main execution path in this span is carried by `aoti_torch_get_current_sycl_queue`, `aoti_torch_mps_copy_buffer`.
- CN: 这一段的主要执行路径由 `aoti_torch_get_current_sycl_queue`, `aoti_torch_mps_copy_buffer` 等函数/方法承载。
### Lines 545-560

```cpp
545:           aoti_torch_dtype_element_size(dtype);
546: #elif USE_CUDA
547:       AOTI_RUNTIME_CUDA_CHECK(cudaMemcpy(
548:           internal_constants_ptr,
549:           user_constant_ptr,
550:           constant_size,
551:           cudaMemcpyDefault));
552: #else
553:       memcpy(internal_constants_ptr, user_constant_ptr, constant_size);
554: #endif
555:       // Generate Tensor from container handled blob.
556:       // We extract stride and offset from provided Tensor since we do not
557:       // guarantee that the tensor is contiguous.
558:       AtenTensorHandle tensor_handle;
559:       int device_type = models_[0]->get_device_type();
560:       int device_idx = models_[0]->get_device_idx();
```

- EN: The main execution path in this span is carried by `aoti_torch_dtype_element_size`, `AOTI_RUNTIME_CUDA_CHECK`, `memcpy`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_dtype_element_size`, `AOTI_RUNTIME_CUDA_CHECK`, `memcpy` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 561-576

```cpp
561:       AOTI_TORCH_ERROR_CODE_CHECK(aoti_torch_create_tensor_from_blob(
562:           internal_constants_ptr,
563:           models_[0]->constant_ndim(idx),
564:           models_[0]->constant_shape(idx),
565:           stride,
566:           offset,
567:           dtype,
568:           device_type,
569:           device_idx,
570:           &tensor_handle));
571: 
572:       // Now place the tensor to constants_map. Note at this point the
573:       // ownership of the tensor_handle will be taken over.
574:       constants_map_to_update->insert_or_assign(
575:           constant_name, RAIIAtenTensorHandle(tensor_handle));
576:     }
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_ERROR_CODE_CHECK`, `RAIIAtenTensorHandle`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_ERROR_CODE_CHECK`, `RAIIAtenTensorHandle` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 577-592

```cpp
577:     // Update the inactive constant array.
578:     update_array_from_map(
579:         get_constants_array(use_inactive), constants_map_to_update);
580:   }
581: 
582:   void update_array_from_map(
583:       const std::shared_ptr<std::vector<ConstantHandle>>& constants_array,
584:       const std::shared_ptr<ConstantMap>& constants_map) {
585:     auto num_constants = models_[0]->num_constants();
586:     for (size_t idx = 0; idx < num_constants; idx++) {
587:       if (constants_map->find(models_[0]->constant_name(
588:               static_cast<int64_t>(idx))) != constants_map->end()) {
589:         constants_array->at(idx) = ConstantHandle(
590:             constants_map
591:                 ->find(models_[0]->constant_name(static_cast<int64_t>(idx)))
592:                 ->second);
```

- EN: The main execution path in this span is carried by `update_array_from_map`, `get_constants_array`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `update_array_from_map`, `get_constants_array` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 593-608

```cpp
593:       }
594:     }
595:   }
596: 
597:   void swap_constant_buffer() {
598:     std::lock_guard unique_lk(model_exec_mutex_);
599: 
600:     auto constants_map = get_constants_map(/* get_inactive= */ true);
601:     auto constants_array = get_constants_array(/* get_inactive= */ true);
602: 
603:     for (auto& model : models_) {
604:       model->update_constants_map(
605:           constants_map, /* remap_constants_array = */ false);
606:       model->update_constants_array(constants_array);
607:     }
608: 
```

- EN: The main execution path in this span is carried by `swap_constant_buffer`, `unique_lk`, `get_constants_map`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `swap_constant_buffer`, `unique_lk`, `get_constants_map` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 609-624

```cpp
609:     use_secondary_ = !use_secondary_;
610:   }
611: 
612:   void free_inactive_constant_buffer() {
613:     if (use_secondary_) {
614:       constant_folded_ = ConstantState::NONE;
615:       constant_blob_.reset();
616:     } else {
617:       constant_folded_secondary_ = ConstantState::NONE;
618:       constant_blob_secondary_.reset();
619:     }
620:     // Free the internally held constants
621:     int num_constants = static_cast<int>(models_[0]->num_constants());
622:     std::shared_ptr<ConstantMap> to_free_map =
623:         use_secondary_ ? constants_map_ : constants_map_secondary_;
624: 
```

- EN: The main execution path in this span is carried by `free_inactive_constant_buffer`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `free_inactive_constant_buffer` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 625-640

```cpp
625:     for (int i = 0; i < num_constants; i++) {
626:       if (models_[0]->constant_from_folded(i)) {
627:         auto it = to_free_map->find(models_[0]->constant_name(i));
628:         if (it != to_free_map->end()) {
629:           it->second.reset();
630:         }
631:       }
632:     }
633:   }
634: 
635:   size_t num_inputs() const {
636:     return input_names_.size();
637:   }
638: 
639:   size_t num_outputs() const {
640:     return output_names_.size();
```

- EN: The main execution path in this span is carried by `num_inputs`, `num_outputs`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `num_inputs`, `num_outputs` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 641-656

```cpp
641:   }
642: 
643:   const char* input_name(size_t idx) const {
644:     return input_names_.at(idx).c_str();
645:   }
646: 
647:   const char* output_name(size_t idx) const {
648:     return output_names_.at(idx).c_str();
649:   }
650: 
651:   size_t num_models() const {
652:     return models_.size();
653:   }
654: 
655:   const char* get_in_spec() const {
656:     return in_spec_;
```

- EN: The main execution path in this span is carried by `input_name`, `output_name`, `num_models`. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `input_name`, `output_name`, `num_models` 等函数/方法承载。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 657-672

```cpp
657:   }
658: 
659:   const char* get_out_spec() const {
660:     return out_spec_;
661:   }
662: 
663:  private:
664:   std::vector<std::string> input_names_;
665:   std::vector<std::string> output_names_;
666:   const char* in_spec_;
667:   const char* out_spec_;
668: 
669:   // Holds the blob storage for constants' at::Tensor within the container.
670:   // This blob of memory will be managed by the container.
671:   RAIIDataPtr constant_blob_;
672:   RAIIDataPtr constant_blob_secondary_;
```

- EN: The main execution path in this span is carried by `get_out_spec`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `get_out_spec` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 673-688

```cpp
673: 
674:   size_t blob_size_;
675:   std::vector<size_t> constants_internal_offset_;
676:   size_t secondary_cpu_blob_size_;
677:   std::vector<size_t> secondary_cpu_constants_internal_offset_;
678: 
679:   // Determine which constants is being used for the model.
680:   // If true,
681:   // constants_map_secondary/constant_blob_secondary/constants_array_secondary
682:   // is being used.
683:   bool use_secondary_{false};
684: 
685:   // Determine whether we have ran constant folding
686:   ConstantState constant_folded_{ConstantState::NONE};
687:   ConstantState constant_folded_secondary_{ConstantState::NONE};
688: 
```

- EN: Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 689-704

```cpp
689:   // Holds the mapping of constants to at::Tensor.
690:   // The underlying data of at::Tensor is in either constant_blob_ (for CUDA).
691:   // or _binary_constants_bin_start (for CPU).
692:   std::shared_ptr<ConstantMap> constants_map_;
693:   std::shared_ptr<ConstantMap> constants_map_secondary_;
694: 
695:   // Holds the indexed array of constant for faster lookup during runtime.
696:   std::shared_ptr<std::vector<ConstantHandle>> constants_array_;
697:   std::shared_ptr<std::vector<ConstantHandle>> constants_array_secondary_;
698: 
699:   // Holds all the AOTInductorModel instances owned by this container.
700:   std::vector<std::unique_ptr<AOTInductorModel>> models_;
701: 
702:   // Holds the AOTInductorModel instances available for inference.
703:   std::vector<AOTInductorModel*> available_models_;
704: 
```

- EN: The main execution path in this span is carried by `constant_blob_`, `_binary_constants_bin_start`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `constant_blob_`, `_binary_constants_bin_start` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 705-720

```cpp
705:   // Holds the AOTInductorModel instances that have started running
706:   // inference and can be placed onto available_models_ upon their
707:   // completion.
708:   std::deque<AOTInductorModel*> pending_models_;
709: 
710:   // Protects available_models_ and pending_models_.
711:   std::mutex models_mutex_;
712: 
713:   // Notified whenever a model is placed onto pending_models_.
714:   std::condition_variable pending_models_available_;
715: 
716:   AOTInductorModel* get_available_model() {
717:     std::unique_lock lk(models_mutex_);
718:     if (available_models_.empty()) {
719:       reclaim_finished_models(lk);
720:     }
```

- EN: The main execution path in this span is carried by `get_available_model`, `lk`, `reclaim_finished_models`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `get_available_model`, `lk`, `reclaim_finished_models` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 721-736

```cpp
721:     auto* result = available_models_.back();
722:     available_models_.pop_back();
723:     return result;
724:   }
725: 
726:   // This mutex is used to protect execution of model.
727:   // We acquire the mutex in shared mode if we allow concurrent execution.
728:   // We acquire the mutex in unique mode when we want exclusive access of the
729:   // model. One such case is when we want to do a weight swapping. We want to
730:   // make sure no one is executing the model.
731:   std::shared_mutex model_exec_mutex_;
732: 
733:   RAIIDataPtr allocate_constant_blob() {
734: #if defined(USE_CUDA) || defined(USE_XPU) || defined(USE_MPS)
735:     return RAII_gpuMalloc(blob_size_);
736: #else
```

- EN: The main execution path in this span is carried by `allocate_constant_blob`, `RAII_gpuMalloc`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `allocate_constant_blob`, `RAII_gpuMalloc` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 737-752

```cpp
737:     return RAII_cpuMalloc(blob_size_);
738: #endif // USE_CUDA
739:   }
740: 
741:   void* get_constant_blob_ptr(bool get_inactive) {
742:     if ((get_inactive && use_secondary_) ||
743:         (!get_inactive && !use_secondary_)) {
744:       if (!constant_blob_) {
745:         constant_blob_ = allocate_constant_blob();
746:       }
747:       return constant_blob_.get();
748:     } else {
749:       if (!constant_blob_secondary_) {
750:         constant_blob_secondary_ = allocate_constant_blob();
751:       }
752:       return constant_blob_secondary_.get();
```

- EN: The main execution path in this span is carried by `RAII_cpuMalloc`, `get_constant_blob_ptr`, `allocate_constant_blob`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `RAII_cpuMalloc`, `get_constant_blob_ptr`, `allocate_constant_blob` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 753-768

```cpp
753:     }
754:   }
755: 
756:   std::shared_ptr<ConstantMap> get_constants_map(bool get_inactive) {
757:     if ((get_inactive && use_secondary_) ||
758:         (!get_inactive && !use_secondary_)) {
759:       return constants_map_;
760:     } else {
761:       if (!constants_map_secondary_) {
762:         constants_map_secondary_ = std::make_shared<ConstantMap>();
763:       }
764:       return constants_map_secondary_;
765:     }
766:   }
767: 
768:   std::shared_ptr<std::vector<ConstantHandle>> get_constants_array(
```

- EN: The main execution path in this span is carried by `get_constants_map`, `get_constants_array`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `get_constants_map`, `get_constants_array` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 769-784

```cpp
769:       bool get_inactive) {
770:     if ((get_inactive && use_secondary_) ||
771:         (!get_inactive && !use_secondary_)) {
772:       return constants_array_;
773:     } else {
774:       if (!constants_array_secondary_) {
775:         constants_array_secondary_ =
776:             std::make_shared<std::vector<ConstantHandle>>(
777:                 models_[0]->num_constants());
778:       }
779:       return constants_array_secondary_;
780:     }
781:   }
782: 
783:   void reclaim_finished_models(std::unique_lock<std::mutex>& lk) {
784: #ifdef __aarch64__
```

- EN: The main execution path in this span is carried by `reclaim_finished_models`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `reclaim_finished_models` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 785-800

```cpp
785:     // push finished model instances to the end of pending_models_
786:     auto it = std::partition(
787:         pending_models_.begin(),
788:         pending_models_.end(),
789:         [](AOTInductorModel* m) { return !m->is_finished(); });
790: #else
791:     // push finished model instances to the end of pending_models_
792:     auto it = std::stable_partition(
793:         pending_models_.begin(),
794:         pending_models_.end(),
795:         [](AOTInductorModel* m) { return !m->is_finished(); });
796: #endif
797: 
798:     if (it != pending_models_.end()) {
799:       // We have finished model instances that can be pushed into
800:       // available_models_ so that we don't have to be blocked on waiting
```

- EN: The main execution path in this span is carried by `partition`, `stable_partition`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `partition`, `stable_partition` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 801-816

```cpp
801:       // the pending_models_available_ condition.
802:       available_models_.insert(
803:           available_models_.end(), it, pending_models_.end());
804:       pending_models_.erase(it, pending_models_.end());
805:       return;
806:     }
807: 
808:     pending_models_available_.wait(
809:         lk, [this]() { return !pending_models_.empty(); });
810:     // Let's make the schedule simple first. We always wait on the first
811:     // pending_models_ to be complete.
812:     auto* model = pending_models_.front();
813:     pending_models_.pop_front();
814:     lk.unlock();
815:     try {
816:       model->wait_for_completion();
```

- EN: Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 817-827

```cpp
817:     } catch (...) {
818:       lk.lock();
819:       available_models_.push_back(model);
820:       throw;
821:     }
822:     lk.lock();
823:     available_models_.push_back(model);
824:   }
825: };
826: 
827: } // namespace torch::aot_inductor
```

- EN: They also open, refine, or close the relevant C++ namespace scope. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。

## Key Concepts / 关键概念
- Inductor / AOTInductor runtime integration / Inductor / AOTInductor 运行时集成
- Tensor/value representation management / 张量/数值表示管理
- State coordination and lifetime management / 状态协调与生命周期管理
- Stable ABI / C interface exposure / 稳定 ABI / C 接口暴露
- Primary symbol `ConstantState` / 核心符号 `ConstantState`
- Primary symbol `AOTInductorModelContainer` / 核心符号 `AOTInductorModelContainer`

## Dependencies / 依赖关系
- Direct includes / 直接包含: `algorithm`, `condition_variable`, `deque`, `mutex`, `shared_mutex`, `torch/csrc/inductor/aoti_runtime/model.h`
- Include roots / 头文件根模块: `torch`
- Key symbols / 关键符号: `ConstantState`, `AOTInductorModelContainer`, `toStringConstantState`, `model_lk`, `constants_folding_lk`, `lk`, `num_constants`, `constant_name`, `constant_original_fqn`, `constant_from_folded`
- Related subsystems / 相关子系统: ATen operator layer / ATen 算子层, Inductor runtime / Inductor 运行时
