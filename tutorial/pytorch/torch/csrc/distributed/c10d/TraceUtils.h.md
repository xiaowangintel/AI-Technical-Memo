# TraceUtils.h — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/TraceUtils.h`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides utility helpers in the c10d distributed process-group subsystem. Key types include `TraceDebugEvent`.
- 用途 (CN): 该文件在c10d 分布式进程组子系统中提供工具辅助逻辑。 关键类型包括 `TraceDebugEvent`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
1: #pragma once
2: #include <c10/core/ScalarType.h>
3: #include <c10/util/ApproximateClock.h>
4: #include <c10/util/irange.h>
5: #include <torch/csrc/distributed/c10d/Store.hpp>
6: #include <torch/csrc/distributed/c10d/Types.hpp>
7: #include <torch/csrc/distributed/c10d/Utils.hpp>
8: #include <torch/csrc/jit/serialization/pickler.h>
9: #include <torch/csrc/profiler/combined_traceback.h>
10: 
11: #include <fmt/compile.h>
12: #include <fmt/core.h>
13: #include <fmt/ostream.h> // optional, for ostream fallback
14: #include <fmt/ranges.h> // for fmt::join
15: 
16: #include <sys/types.h>
```

- EN: Lines 1-16 pulls in the headers required by this translation unit or interface.
- CN: 第 1-16 行引入该实现单元或接口所需的头文件。

### Lines 17-32 / 第 17-32 行

```cpp
17: #include <cstdlib>
18: #include <cstring>
19: #include <iterator>
20: #include <string>
21: #include <vector>
22: 
23: namespace c10d {
24: 
25: inline std::string getTraceStartKey(const std::string& pgName, int rank) {
26:   return fmt::format(FMT_COMPILE("{}_{}_trace_start"), pgName, rank);
27: }
28: 
29: inline std::string getTraceEndKey(const std::string& pgName, int rank) {
30:   return fmt::format(FMT_COMPILE("{}_{}_trace_end"), pgName, rank);
31: }
32: 
```

- EN: Lines 17-32 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `getTraceStartKey`, `getTraceEndKey`.
- CN: 第 17-32 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `getTraceStartKey`、`getTraceEndKey` 等例程中引入具体执行逻辑。

### Lines 33-48 / 第 33-48 行

```cpp
33: inline bool traceUpdate(
34:     c10::intrusive_ptr<Store>& store,
35:     const std::string& key,
36:     uint64_t seq,
37:     const std::string& col) {
38:   std::vector<uint8_t> value(col.size() + sizeof(seq) + 1);
39:   std::memcpy(value.data(), &seq, sizeof(seq));
40:   std::memcpy(value.data() + sizeof(seq), col.data(), col.size());
41:   try {
42:     store->set(key, value);
43:     return true;
44:   } catch (...) {
45:     LOG(ERROR) << "Store is down while updating #" << seq << " with key "
46:                << key;
47:     return false;
48:   }
```

- EN: Lines 33-48 introduces executable logic in routines such as `traceUpdate`; returns computed state or forwards results to the surrounding caller.
- CN: 第 33-48 行在 `traceUpdate` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 49-64 / 第 49-64 行

```cpp
49:   return true;
50: }
51: 
52: enum TraceDebugEvent {
53:   kEventStart,
54:   kEventEnd,
55: };
56: // <seq, <rank, <col, start/end>>>
57: using TraceMap =
58:     std::map<uint64_t, std::map<int, std::pair<std::string, TraceDebugEvent>>>;
59: 
60: inline std::string ranksToString(const std::vector<int>& ranks) {
61:   return fmt::to_string(fmt::join(ranks, ", "));
62: }
63: 
64: inline std::string ranksFromTrace(
```

- EN: Lines 49-64 declares or defines types such as `TraceDebugEvent`; introduces executable logic in routines such as `ranksToString`.
- CN: 第 49-64 行声明或定义了 `TraceDebugEvent` 等类型；在 `ranksToString` 等例程中引入具体执行逻辑。

### Lines 65-80 / 第 65-80 行

```cpp
65:     const std::vector<std::pair<int, std::string>>& items) {
66:   fmt::memory_buffer buf;
67:   bool first = true;
68:   for (const auto& [rank, _] : items) {
69:     if (!first) {
70:       fmt::format_to(std::back_inserter(buf), ", ");
71:     }
72:     fmt::format_to(std::back_inserter(buf), "{}", rank);
73:     first = false;
74:   }
75:   return fmt::to_string(buf);
76: }
77: 
78: inline std::string analyzeMissingRanks(const std::vector<int>& missingRanks) {
79:   return c10::str(
80:       "\n\t - To our best knowledge, ranks [",
```

- EN: Lines 65-80 introduces executable logic in routines such as `analyzeMissingRanks`; returns computed state or forwards results to the surrounding caller.
- CN: 第 65-80 行在 `analyzeMissingRanks` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 81-96 / 第 81-96 行

```cpp
81:       ranksToString(missingRanks),
82:       "] are the lagging ranks that caused this timeout. "
83:       "They never joined any collectives");
84: }
85: 
86: inline std::string analyzeLaggingRanks(const TraceMap& traceMap) {
87:   uint64_t lagSeq = traceMap.begin()->first;
88:   std::vector<int> startRanks;
89:   std::vector<int> endRanks;
90:   for (auto& p : traceMap.begin()->second) {
91:     if (p.second.second == kEventStart) {
92:       startRanks.push_back(p.first);
93:     } else {
94:       endRanks.push_back(p.first);
95:     }
96:   }
```

- EN: Lines 81-96 introduces executable logic in routines such as `analyzeLaggingRanks`.
- CN: 第 81-96 行在 `analyzeLaggingRanks` 等例程中引入具体执行逻辑。

### Lines 97-112 / 第 97-112 行

```cpp
97:   std::string report =
98:       "\n\t - To our best knowledge, the lagging/dead/mismatched ranks "
99:       "that caused the desync are:";
100:   if (!startRanks.empty()) {
101:     report += c10::str(
102:         "\n\t   - [",
103:         ranksToString(startRanks),
104:         "] joined but didn't finish collective #",
105:         lagSeq,
106:         " (count from 1)");
107:   }
108:   if (!endRanks.empty()) {
109:     report += c10::str(
110:         "\n\t     [",
111:         ranksToString(endRanks),
112:         "] finished collective #",
```

- EN: Lines 97-112 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 97-112 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 113-128 / 第 113-128 行

```cpp
113:         lagSeq,
114:         ", but didn't join collective #",
115:         lagSeq + 1,
116:         " (count from 1)");
117:   }
118:   return report;
119: }
120: 
121: inline std::string dumpSnapshot(TraceMap& traceMap) {
122:   std::string report = "\n\t - Snapshot of ranks' latest states:";
123:   for (auto& tracePair : traceMap) {
124:     uint64_t seq = tracePair.first;
125:     std::map<int, std::pair<std::string, TraceDebugEvent>>& subMap =
126:         tracePair.second;
127: 
128:     std::unordered_map<std::string, std::vector<int>> collectivesStart;
```

- EN: Lines 113-128 introduces executable logic in routines such as `dumpSnapshot`; returns computed state or forwards results to the surrounding caller.
- CN: 第 113-128 行在 `dumpSnapshot` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 129-144 / 第 129-144 行

```cpp
129:     std::unordered_map<std::string, std::vector<int>> collectivesEnd;
130:     for (const auto& p : subMap) {
131:       int rank = p.first;
132:       const std::string& col = p.second.first;
133:       if (p.second.second == kEventStart) {
134:         collectivesStart[col].push_back(rank);
135:       } else {
136:         collectivesEnd[col].push_back(rank);
137:       }
138:     }
139: 
140:     if (!collectivesStart.empty()) {
141:       report += c10::str("\n\t   #", seq, " started ranks:");
142:       for (auto& mapPair : collectivesStart) {
143:         report += c10::str(
144:             "\n\t     [",
```

- EN: Lines 129-144 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 129-144 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 145-160 / 第 145-160 行

```cpp
145:             ranksToString(mapPair.second),
146:             "] started ",
147:             mapPair.first);
148:       }
149:     }
150:     if (!collectivesEnd.empty()) {
151:       report += c10::str("\n\t   #", seq, " finished ranks:");
152:       for (auto& mapPair : collectivesEnd) {
153:         report += c10::str(
154:             "\n\t     [",
155:             ranksToString(mapPair.second),
156:             "] finished ",
157:             mapPair.first);
158:       }
159:     }
160:   }
```

- EN: Lines 145-160 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 145-160 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 161-176 / 第 161-176 行

```cpp
161:   return report;
162: }
163: 
164: inline bool parseTraceValue(
165:     c10::intrusive_ptr<Store>& store,
166:     const std::string& key,
167:     uint64_t& seq,
168:     std::string& col) {
169:   try {
170:     std::vector<uint8_t> traceValue = store->get(key);
171:     std::memcpy(&seq, traceValue.data(), sizeof(seq));
172:     std::string colName((char*)traceValue.data() + sizeof(seq));
173:     col = colName;
174:     return true;
175:   } catch (...) {
176:     LOG(ERROR) << "Store is down while getting key " << key;
```

- EN: Lines 161-176 introduces executable logic in routines such as `parseTraceValue`; returns computed state or forwards results to the surrounding caller.
- CN: 第 161-176 行在 `parseTraceValue` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 177-192 / 第 177-192 行

```cpp
177:     return false;
178:   }
179:   return true;
180: }
181: 
182: inline std::string retrieveDesyncReport(
183:     c10::intrusive_ptr<Store>& store,
184:     const std::string& pgName,
185:     int myRank,
186:     int worldSize) {
187:   std::string report;
188: 
189:   uint64_t thisSeq = 0;
190:   std::string thisCol;
191: 
192:   std::vector<int> missingRanks;
```

- EN: Lines 177-192 introduces executable logic in routines such as `retrieveDesyncReport`; returns computed state or forwards results to the surrounding caller.
- CN: 第 177-192 行在 `retrieveDesyncReport` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 193-208 / 第 193-208 行

```cpp
193:   TraceMap traceMap;
194: 
195:   for (const auto rank : c10::irange(worldSize)) {
196:     // Build traceMapStart.
197:     uint64_t seqStart = 0;
198:     {
199:       std::string traceKeyStart = getTraceStartKey(pgName, rank);
200:       if (!store->check({traceKeyStart})) {
201:         missingRanks.push_back(rank);
202:         continue;
203:       }
204:       std::string col;
205:       if (!parseTraceValue(store, traceKeyStart, seqStart, col)) {
206:         return report;
207:       }
208:       traceMap[seqStart].emplace(rank, std::make_pair(col, kEventStart));
```

- EN: Lines 193-208 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 193-208 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 209-224 / 第 209-224 行

```cpp
209:       if (rank == myRank) {
210:         thisSeq = seqStart;
211:         thisCol = std::move(col);
212:       }
213:     }
214: 
215:     // Build traceMapEnd.
216:     {
217:       std::string traceKeyEnd = getTraceEndKey(pgName, rank);
218:       if (!store->check({traceKeyEnd})) {
219:         continue;
220:       }
221:       uint64_t seq = 0;
222:       std::string col;
223:       if (!parseTraceValue(store, traceKeyEnd, seq, col)) {
224:         return report;
```

- EN: Lines 209-224 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 209-224 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 225-240 / 第 225-240 行

```cpp
225:       }
226:       if (seq == seqStart) {
227:         traceMap[seq][rank].second = kEventEnd;
228:       }
229:     }
230:   }
231: 
232:   TORCH_INTERNAL_ASSERT(
233:       !missingRanks.empty() || !traceMap.empty(),
234:       "Trace shouldn't be empty while enabled GLOO_ASYNC_TIMEOUT_DEBUG");
235:   TORCH_INTERNAL_ASSERT(
236:       !thisCol.empty(),
237:       "Timeout rank [",
238:       myRank,
239:       "] must have collective tracking iteam in c10::Store trace");
240:   TORCH_INTERNAL_ASSERT(
```

- EN: Lines 225-240 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 225-240 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 241-256 / 第 241-256 行

```cpp
241:       traceMap[thisSeq][myRank].second == kEventStart,
242:       "Timeout rank [",
243:       myRank,
244:       "] last trace item must be kEventStart. thisSeq = ",
245:       thisSeq,
246:       ", col = ",
247:       thisCol);
248: 
249:   report += c10::str(
250:       "\n\t - [", myRank, "] Timeout at collective: ", thisCol, ", #", thisSeq);
251: 
252:   if (!missingRanks.empty()) {
253:     report += analyzeMissingRanks(missingRanks);
254:   } else {
255:     report += analyzeLaggingRanks(traceMap);
256:     report += dumpSnapshot(traceMap);
```

- EN: Lines 241-256 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 241-256 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 257-272 / 第 257-272 行

```cpp
257:   }
258: 
259:   return report;
260: }
261: 
262: inline std::string pickle_str(const c10::IValue& v) {
263:   std::vector<char> result;
264:   {
265:     auto writer = [&](const char* data, size_t size) {
266:       result.insert(result.end(), data, data + size);
267:     };
268:     torch::jit::Pickler pickler(
269:         writer, nullptr, nullptr, nullptr, nullptr, false);
270:     pickler.protocol();
271:     pickler.pushIValue(v);
272:     pickler.stop();
```

- EN: Lines 257-272 introduces executable logic in routines such as `pickle_str`; returns computed state or forwards results to the surrounding caller.
- CN: 第 257-272 行在 `pickle_str` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 273-288 / 第 273-288 行

```cpp
273:   }
274:   return std::string(result.begin(), result.end());
275: }
276: 
277: inline std::string get_python_cpp_trace() {
278:   // usage:
279:   // LOG(INFO) << "stacktrace: "
280:   //           << get_python_cpp_trace();
281:   // warn: might be slow in getting cpp traces
282:   // because of slow/broken addr2line
283:   // in different system libs
284:   std::shared_ptr<torch::CapturedTraceback> tb =
285:       torch::CapturedTraceback::gather(
286:           /*python=*/true, /*script=*/true, /*cpp=*/true);
287:   torch::SymbolizedTracebacks s_tbs = torch::symbolize({tb.get()});
288:   const auto& s_tb = s_tbs.tracebacks.at(0);
```

- EN: Lines 273-288 introduces executable logic in routines such as `get_python_cpp_trace`; returns computed state or forwards results to the surrounding caller.
- CN: 第 273-288 行在 `get_python_cpp_trace` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 289-304 / 第 289-304 行

```cpp
289:   constexpr auto TB_FMT_CSTR = FMT_COMPILE("#{} {} from {}:{}\n");
290:   fmt::memory_buffer buf;
291:   auto buf_iter = std::back_inserter(buf);
292:   for (auto idx : c10::irange(s_tb.size())) {
293:     auto frame_id = s_tb[idx];
294:     const auto& frame = s_tbs.all_frames.at(frame_id);
295:     fmt::format_to(
296:         buf_iter,
297:         TB_FMT_CSTR,
298:         idx,
299:         frame.funcname,
300:         frame.filename,
301:         frame.lineno);
302:   }
303:   return fmt::to_string(buf);
304: }
```

- EN: Lines 289-304 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 289-304 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 305-319 / 第 305-319 行

```cpp
305: 
306: inline c10::Dict<c10::IValue, c10::IValue> new_dict() {
307:   return c10::Dict<c10::IValue, c10::IValue>(
308:       c10::AnyType::get(), c10::AnyType::get());
309: }
310: 
311: inline c10::List<c10::IValue> new_list() {
312:   return c10::List<c10::IValue>(c10::AnyType::get());
313: }
314: 
315: inline std::string ranks_str(const std::vector<uint64_t>& ranks) {
316:   return fmt::format("[{}]", fmt::join(ranks, ", "));
317: }
318: 
319: } // namespace c10d
```

- EN: Lines 305-319 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `new_dict`, `new_list`, `ranks_str`.
- CN: 第 305-319 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `new_dict`、`new_list`、`ranks_str` 等例程中引入具体执行逻辑。

## Key Concepts / 关键概念

- EN: Subsystem: c10d distributed process-group subsystem.
- CN: 子系统：c10d 分布式进程组子系统。
- EN: File role: interface/definition.
- CN: 文件角色：接口/定义。
- EN: Primary symbols: `TraceDebugEvent`
- CN: 核心符号：`TraceDebugEvent`
- EN: Notable themes: store/state coordination, collective communication logic.
- CN: 值得关注的主题：存储/状态协调、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/Store.hpp`, `torch/csrc/distributed/c10d/Types.hpp`, `torch/csrc/distributed/c10d/Utils.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `c10/core/ScalarType.h`, `c10/util/ApproximateClock.h`, `c10/util/irange.h`, `torch/csrc/jit/serialization/pickler.h`, `torch/csrc/profiler/combined_traceback.h`
- External or system headers / 外部或系统头文件: `fmt/compile.h`, `fmt/core.h`, `fmt/ostream.h`, `fmt/ranges.h`, `sys/types.h`, `cstdlib`, `cstring`, `iterator`, `string`, `vector`
- Local symbols / 本地符号: `TraceDebugEvent`