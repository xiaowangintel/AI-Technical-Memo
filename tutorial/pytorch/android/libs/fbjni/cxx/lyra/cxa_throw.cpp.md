# cxa_throw.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/lyra/cxa_throw.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements JNI bridge abstractions used by Android-facing native code.
- 用途 (CN): 实现供 Android 原生代码使用的 JNI 桥接抽象。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````cpp
 1: /*
 2:  * Copyright (c) Facebook, Inc. and its affiliates.
 3:  *
 4:  * Licensed under the Apache License, Version 2.0 (the "License");
 5:  * you may not use this file except in compliance with the License.
 6:  * You may obtain a copy of the License at
 7:  *
 8:  *     http://www.apache.org/licenses/LICENSE-2.0
 9:  *
10:  * Unless required by applicable law or agreed to in writing, software
11:  * distributed under the License is distributed on an "AS IS" BASIS,
12:  * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
13:  * See the License for the specific language governing permissions and
14:  * limitations under the License.
15:  */
16: 
17: #include <atomic>
18: #include <mutex>
19: #include <stdexcept>
20: #include <unordered_map>
21: #include <cassert>
22: #ifndef _WIN32
23: #include <unwind.h>
24: #include <cxxabi.h>
25: #endif
26: 
27: #include <lyra/lyra_exceptions.h>
28: 
29: namespace facebook {
30: namespace lyra {
31: 
32: using namespace detail;
33: 
34: namespace {
35: std::atomic<bool> enableBacktraces{true};
36: 
37: const ExceptionTraceHolder* getExceptionTraceHolderInException(
38:     std::exception_ptr ptr) {
39:   try {
40:     std::rethrow_exception(ptr);
41:   } catch (const ExceptionTraceHolder& holder) {
42:     return &holder;
43:   } catch (...) {
44:     return nullptr;
45:   }
46: }
47: } // namespace
48: 
49: void enableCxaThrowHookBacktraces(bool enable) {
50:   enableBacktraces.store(enable, std::memory_order_relaxed);
51: }
52: 
53: [[gnu::noreturn]] void (*original_cxa_throw)(void*, const std::type_info*, void (*) (void *));
54: 
55: #if defined(_LIBCPP_VERSION)
56: 
57: // We want to attach stack traces to C++ exceptions. Our API contract is that
58: // calling lyra::getExceptionTrace on the exception_ptr for an exception should
59: // return the stack trace for that exception.
60: //
````
- EN: Pulls in native headers such as `atomic`, `mutex`, `stdexcept`, `unordered_map`.
- CN: 引入原生头文件，例如 `atomic`, `mutex`, `stdexcept`, `unordered_map`。
- EN: Implements callable logic such as `getExceptionTraceHolderInException`, `enableCxaThrowHookBacktraces`.
- CN: 实现可调用逻辑，例如 `getExceptionTraceHolderInException`, `enableCxaThrowHookBacktraces`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 61-120
````cpp
 61: // We accomplish this by providing a hook for __cxa_throw, which creates an
 62: // ExceptionTraceHolder object (which captures the stack trace for the
 63: // exception), and creates a mapping from the pointer to the exception object
 64: // (which is a parameter to __cxa_throw) to its ExceptionTraceHolder object.
 65: // This mapping can then be queried by lyra::getExceptionTrace to get the stack
 66: // trace for the exception. We have a custom exception destructor to destroy the
 67: // trace object and call the original destructor for the exception object, and
 68: // our __cxa_throw hook calls the original __cxa_throw to perform the actual
 69: // exception throwing and passes this custom destructor.
 70: //
 71: // This works because __cxa_throw is only called when creating a new exception
 72: // object (that has been freshly allocated via __cxa_allocate_exception), so at
 73: // that point, we're able to capture the original stack trace for the exception.
 74: // Even if that exception is later rethrown, we'll still maintain its original
 75: // stack trace, assuming that std::current_exception creates a reference to the
 76: // current exception instead of copying it (which is true for both libstdc++ and
 77: // libc++), such that we can still look up the exception object via pointer.
 78: //
 79: // We don't have to worry about any pointer adjustments for the exception object
 80: // (e.g. for converting to or from a base class subobject pointer), because a
 81: // std::exception_ptr will always capture the pointer to the exception object
 82: // itself and not any subobjects.
 83: //
 84: // Our map must be global, since exceptions can be transferred across threads.
 85: // Consequently, we must use a mutex to guard all map operations.
 86: 
 87: typedef void (*destructor_type)(void*);
 88: 
 89: namespace {
 90: struct ExceptionState {
 91:   ExceptionTraceHolder trace;
 92:   destructor_type destructor;
 93: };
 94: 
 95: // We create our map and mutex as function statics and leak them intentionally,
 96: // to ensure they've been initialized before any global constructors and are
 97: // also available to use inside any global destructors.
 98: std::unordered_map<void*, ExceptionState>* get_exception_state_map() {
 99:   static auto* exception_state_map =
100:       new std::unordered_map<void*, ExceptionState>();
101:   return exception_state_map;
102: }
103: 
104: std::mutex* get_exception_state_map_mutex() {
105:   static auto* exception_state_map_mutex = new std::mutex();
106:   return exception_state_map_mutex;
107: }
108: 
109: void trace_destructor(void* exception_obj) {
110:   destructor_type original_destructor = nullptr;
111: 
112:   {
113:     std::lock_guard<std::mutex> lock(*get_exception_state_map_mutex());
114:     auto* exception_state_map = get_exception_state_map();
115:     auto it = exception_state_map->find(exception_obj);
116:     if (it == exception_state_map->end()) {
117:       // This really shouldn't happen, but if it does, just leaking the trace
118:       // and exception object seems better than crashing.
119:       return;
120:     }
````
- EN: Declares or extends types including `subobject`, `ExceptionState`.
- CN: 声明或扩展类型，包括 `subobject`, `ExceptionState`。
- EN: Implements callable logic such as `get_exception_state_map`, `get_exception_state_map_mutex`, `trace_destructor`.
- CN: 实现可调用逻辑，例如 `get_exception_state_map`, `get_exception_state_map_mutex`, `trace_destructor`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````cpp
121: 
122:     original_destructor = it->second.destructor;
123:     exception_state_map->erase(it);
124:   }
125: 
126:   if (original_destructor) {
127:     original_destructor(exception_obj);
128:   }
129: }
130: } // namespace
131: 
132: [[noreturn]] void
133: cxa_throw(void* obj, const std::type_info* type, destructor_type destructor) {
134:   // TODO(T61689492): Re-enable Lyra stack traces for libc++ arm64
135: #if !defined(__aarch64__)
136:   if (enableBacktraces.load(std::memory_order_relaxed)) {
137:     std::lock_guard<std::mutex> lock(*get_exception_state_map_mutex());
138:     get_exception_state_map()->emplace(
139:         obj, ExceptionState{ExceptionTraceHolder(), destructor});
140:   }
141: #endif
142: 
143:   original_cxa_throw(obj, type, trace_destructor);
144: }
145: 
146: const ExceptionTraceHolder* detail::getExceptionTraceHolder(
147:     std::exception_ptr ptr) {
148:   {
149:     std::lock_guard<std::mutex> lock(*get_exception_state_map_mutex());
150:     // The exception object pointer isn't a public member of std::exception_ptr,
151:     // and there isn't any public method to get it. However, for both libstdc++
152:     // and libc++, it's the first pointer inside the exception_ptr, and we can
153:     // rely on the ABI of those libraries to remain stable, so we can just
154:     // access it directly.
155:     void* exception_obj = *reinterpret_cast<void**>(&ptr);
156:     auto* exception_state_map = get_exception_state_map();
157:     auto it = exception_state_map->find(exception_obj);
158:     if (it != exception_state_map->end()) {
159:       return &it->second.trace;
160:     }
161:   }
162: 
163:   // Fall back to attempting to retrieve the ExceptionTraceHolder directly from
164:   // the exception (to support e.g. fbthrow).
165:   return getExceptionTraceHolderInException(ptr);
166: }
167: #elif !defined(_WIN32)
168: 
169: namespace {
170: 
171: const auto traceHolderType =
172:   static_cast<const abi::__class_type_info*>(&typeid(ExceptionTraceHolder));
173: 
174: // lyra's __cxa_throw attaches stack trace information to thrown exceptions. It basically does:
175: //   1. capture stack trace
176: //   2. construct a new type_info struct that:
177: //     a. holds the ExceptionTraceHolder
178: //     b. supports upcasting to lyra::ExceptionTraceHolder* (by just returning the holder member)
179: //     c. acts like the original exception type_info otherwise
180: //   3. call original __cxa_throw() with original exception pointer, the
````
- EN: Declares or extends types including `that`.
- CN: 声明或扩展类型，包括 `that`。
- EN: Implements callable logic such as `cxa_throw`, `TODO`, `detail::getExceptionTraceHolder`.
- CN: 实现可调用逻辑，例如 `cxa_throw`, `TODO`, `detail::getExceptionTraceHolder`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````cpp
181: //      HijackedExceptionTypeInfo, and HijackedExceptionTypeInfo::destructor
182: //      (which will both delete the constructed type info and call the original
183: //      destructor).
184: struct HijackedExceptionTypeInfo : public abi::__class_type_info {
185:   HijackedExceptionTypeInfo(void* obj, const std::type_info* base, void(*destructor)(void*))
186:       : abi::__class_type_info{base->name()}, base_{base}, orig_dest_{destructor} {
187:   }
188: 
189:   bool __is_pointer_p() const override {
190:     return base_->__is_pointer_p();
191:   }
192: 
193:   bool __is_function_p() const override {
194:     return base_->__is_function_p();
195:   }
196: 
197:   bool __do_catch(const type_info *__thr_type, void **__thr_obj, unsigned __outer) const override {
198:     return base_->__do_catch(__thr_type, __thr_obj, __outer);
199:   }
200: 
201:   bool __do_upcast(const abi::__class_type_info *__target, void **__obj_ptr) const override {
202:     if (__target == traceHolderType) {
203:       *__obj_ptr = (void*)&stack_;
204:       return true;
205:     }
206:     return base_->__do_upcast(__target, __obj_ptr);
207:   }
208: 
209:   static void destructor(void* obj) {
210:     auto exc_ptr = reinterpret_cast<std::exception_ptr*>(&obj);
211:     auto info = reinterpret_cast<const::std::type_info*>(exc_ptr->__cxa_exception_type());
212:     auto mutable_info = static_cast<HijackedExceptionTypeInfo*>(const_cast<std::type_info*>(info));
213:     if (mutable_info->orig_dest_) {
214:       mutable_info->orig_dest_(obj);
215:     }
216:     delete mutable_info;
217:   }
218: 
219:  private:
220:   const std::type_info* base_;
221:   void (*orig_dest_)(void*);
222:   ExceptionTraceHolder stack_;
223: };
224: 
225: } // namespace
226: 
227: [[noreturn]] void cxa_throw(void* obj, const std::type_info* type, void (*destructor) (void *)) {
228:   if (enableBacktraces.load(std::memory_order_relaxed)) {
229:     if (!type->__do_upcast(traceHolderType, &obj)) {
230:       type = new HijackedExceptionTypeInfo(obj, type, destructor);
231:       destructor = HijackedExceptionTypeInfo::destructor;
232:     }
233:   }
234:   original_cxa_throw(obj, type, destructor);
235: }
236: 
237: const ExceptionTraceHolder* detail::getExceptionTraceHolder(
238:     std::exception_ptr ptr) {
239:   return getExceptionTraceHolderInException(ptr);
240: }
````
- EN: Declares or extends types including `HijackedExceptionTypeInfo`.
- CN: 声明或扩展类型，包括 `HijackedExceptionTypeInfo`。
- EN: Implements callable logic such as `__is_pointer_p`, `__is_function_p`, `__do_catch`, `__do_upcast`.
- CN: 实现可调用逻辑，例如 `__is_pointer_p`, `__is_function_p`, `__do_catch`, `__do_upcast`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-245
````cpp
241: 
242: #endif // libc++/Windows
243: 
244: } // namespace lyra
245: } // namespace facebook
````
- EN: This range contributes implementation details for the file goal: Implements JNI bridge abstractions used by Android-facing native code.
- CN: 该范围为文件目标提供实现细节：实现供 Android 原生代码使用的 JNI 桥接抽象。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `subobject` / 符号 `subobject`
- Symbol `ExceptionState` / 符号 `ExceptionState`
- Symbol `that` / 符号 `that`
- Symbol `HijackedExceptionTypeInfo` / 符号 `HijackedExceptionTypeInfo`

## Dependencies / 依赖关系
- C/C++ includes: `atomic`, `mutex`, `stdexcept`, `unordered_map`, `cassert`, `unwind.h`, `cxxabi.h`, `lyra/lyra_exceptions.h`
- C/C++ 头文件: `atomic`, `mutex`, `stdexcept`, `unordered_map`, `cassert`, `unwind.h`, `cxxabi.h`, `lyra/lyra_exceptions.h`
