# Environment.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/Environment.cpp`
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
17: #include <fbjni/fbjni.h>
18: 
19: #ifdef __ANDROID__
20: #include <sys/prctl.h>
21: #endif // __ANDROID__
22: 
23: #include <vector>
24: #include <functional>
25: #ifndef _WIN32
26: #include <pthread.h>
27: #else
28: #include <windows.h>
29: #endif
30: 
31: namespace facebook {
32: namespace jni {
33: 
34: namespace {
35: 
36: JavaVM* g_vm = nullptr;
37: 
38: struct EnvironmentInitializer {
39:   EnvironmentInitializer(JavaVM* vm) {
40:       FBJNI_ASSERT(!g_vm);
41:       FBJNI_ASSERT(vm);
42:       g_vm = vm;
43:   }
44: };
45: 
46: int getEnv(JNIEnv** env) {
47:   FBJNI_ASSERT(g_vm);
48:   // g_vm->GetEnv() might not clear the env* in failure cases.
49:   *env = nullptr;
50:   jint ret = g_vm->GetEnv((void**)env, JNI_VERSION_1_6);
51:   // Other possibilites are that JNI_VERSION_1_6 is invalid, or some
52:   // unknown return was received.
53:   FBJNI_ASSERT(ret == JNI_OK || ret == JNI_EDETACHED);
54:   return ret;
55: }
56: 
57: // Some jni.h define the first arg to AttachCurrentThread as void**,
58: // and some as JNIEnv**.  This hack allows both to work.
59: 
60: template <typename>
````
- EN: Pulls in native headers such as `fbjni/fbjni.h`, `sys/prctl.h`, `vector`, `functional`.
- CN: 引入原生头文件，例如 `fbjni/fbjni.h`, `sys/prctl.h`, `vector`, `functional`。
- EN: Declares or extends types including `EnvironmentInitializer`.
- CN: 声明或扩展类型，包括 `EnvironmentInitializer`。
- EN: Implements callable logic such as `EnvironmentInitializer`, `getEnv`.
- CN: 实现可调用逻辑，例如 `EnvironmentInitializer`, `getEnv`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````cpp
 61: struct AttachTraits;
 62: 
 63: template <>
 64: struct AttachTraits<jint(JavaVM::*)(JNIEnv**, void*)> {
 65:   using EnvType = JNIEnv*;
 66: };
 67: 
 68: template <>
 69: struct AttachTraits<jint(JavaVM::*)(void**, void*)> {
 70:   using EnvType = void*;
 71: };
 72: 
 73: std::string getThreadName() {
 74: #ifdef _WIN32
 75:   return "";
 76: #else // _WIN32
 77:   constexpr int kMaxThreadNameSize = 100;
 78:   int ret = 0;
 79:   char threadName[kMaxThreadNameSize];
 80: #ifdef __ANDROID__
 81:   ret = prctl(PR_GET_NAME, threadName);
 82: #else
 83:   ret = pthread_getname_np(pthread_self(), threadName, sizeof(threadName));
 84: #endif
 85:   if (ret != 0) {
 86:     return "";
 87:   }
 88:   return threadName;
 89: #endif // _WIN32
 90: }
 91: 
 92: JNIEnv* attachCurrentThread() {
 93:   JavaVMAttachArgs args{JNI_VERSION_1_6, nullptr, nullptr};
 94: 
 95:   const auto threadName = getThreadName();
 96:   std::vector<char> nameBuf;
 97:   if (threadName.size()) {
 98:     const char* cName = threadName.c_str();
 99:     nameBuf.assign(cName, cName + threadName.size() + 1);
100:     args.name = nameBuf.data();
101:   }
102: 
103:   using AttachEnvType =
104:       typename AttachTraits<decltype(&JavaVM::AttachCurrentThread)>::EnvType;
105:   AttachEnvType env;
106:   auto result = g_vm->AttachCurrentThread(&env, &args);
107:   FBJNI_ASSERT(result == JNI_OK);
108:   return reinterpret_cast<JNIEnv*>(env);
109: }
110: 
111: }
112: 
113: /* static */
114: void Environment::initialize(JavaVM* vm) {
115:   static EnvironmentInitializer init(vm);
116: }
117: 
118: namespace {
119: 
120: #ifndef _WIN32
````
- EN: Declares or extends types including `AttachTraits`.
- CN: 声明或扩展类型，包括 `AttachTraits`。
- EN: Implements callable logic such as `getThreadName`, `attachCurrentThread`, `Environment::initialize`.
- CN: 实现可调用逻辑，例如 `getThreadName`, `attachCurrentThread`, `Environment::initialize`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````cpp
121: typedef pthread_key_t tls_key_t;
122: #else
123: typedef DWORD tls_key_t;
124: #endif
125: 
126: tls_key_t makeKey() {
127:   tls_key_t key;
128: #ifndef _WIN32
129:   int ret = pthread_key_create(&key, nullptr);
130:   if (ret != 0) {
131:     FBJNI_LOGF("pthread_key_create failed: %d", ret);
132:   }
133: #else
134:   key = TlsAlloc();
135:   if (key == TLS_OUT_OF_INDEXES) {
136:     FBJNI_LOGF("TlsAlloc failed");
137:   }
138: #endif
139:   return key;
140: }
141: 
142: tls_key_t getTLKey() {
143:   static tls_key_t key = makeKey();
144:   return key;
145: }
146: 
147: inline detail::TLData* getTLData(tls_key_t key) {
148: #ifndef _WIN32
149:   void* raw_data = pthread_getspecific(key);
150: #else
151:   LPVOID raw_data = TlsGetValue(key);
152:   // TODO: Maybe check for errors here?
153: #endif
154:   return reinterpret_cast<detail::TLData*>(raw_data);
155: }
156: 
157: inline void setTLData(tls_key_t key, detail::TLData* data) {
158: #ifndef _WIN32
159:   int ret = pthread_setspecific(key, data);
160:   if (ret != 0) {
161:     (void) ret;
162:     FBJNI_LOGF("pthread_setspecific failed: %d", ret);
163:   }
164: #else
165:   BOOL ret = TlsSetValue(key, data);
166:   if (!ret) {
167:     FBJNI_LOGF("TlsSetValue failed: %d", GetLastError());
168:   }
169: #endif
170: }
171: 
172: // This returns non-nullptr iff the env was cached from java.  So it
173: // can return nullptr for a thread which has been registered.
174: inline JNIEnv* cachedOrNull() {
175:   detail::TLData* pdata = getTLData(getTLKey());
176:   return (pdata ? pdata->env : nullptr);
177: }
178: 
179: }
180: 
````
- EN: Implements callable logic such as `makeKey`, `getTLKey`, `getTLData`, `setTLData`.
- CN: 实现可调用逻辑，例如 `makeKey`, `getTLKey`, `getTLData`, `setTLData`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````cpp
181: namespace detail {
182: 
183: // This will return a cached env if there is one, or get one from JNI
184: // if the thread has already been attached some other way.  If it
185: // returns nullptr, then the thread has never been registered, or the
186: // VM has never been set up for fbjni.
187: 
188: JNIEnv* currentOrNull() {
189:   if (!g_vm) {
190:     return nullptr;
191:   }
192: 
193:   detail::TLData* pdata = getTLData(getTLKey());
194:   if (pdata && pdata->env) {
195:     return pdata->env;
196:   }
197: 
198:   JNIEnv* env;
199:   if (getEnv(&env) != JNI_OK) {
200:     // If there's a ThreadScope on the stack, we should have gotten a
201:     // JNIEnv and not ended up here.
202:     FBJNI_ASSERT(!pdata || !pdata->attached);
203:   }
204:   return env;
205: }
206: 
207: // To understand JniEnvCacher and ThreadScope, it is helpful to
208: // realize that if a flagged JniEnvCacher is on the stack, then a
209: // flagged ThreadScope cannot be after it.  If a flagged ThreadCacher
210: // is on the stack, then a JniEnvCacher *can* be after it.  So,
211: // ThreadScope's setup and teardown can both assume they are the
212: // first/last interesting objects, but this is not true of
213: // JniEnvCacher.
214: 
215: JniEnvCacher::JniEnvCacher(JNIEnv* env)
216:   : thisCached_(false)
217: {
218:   FBJNI_ASSERT(env);
219: 
220:   tls_key_t key = getTLKey();
221:   detail::TLData* pdata = getTLData(key);
222:   if (pdata && pdata->env) {
223:     return;
224:   }
225: 
226:   if (!pdata) {
227:     pdata = &data_;
228:     setTLData(key, pdata);
229:     pdata->attached = false;
230:   } else {
231:     FBJNI_ASSERT(!pdata->env);
232:   }
233: 
234:   pdata->env = env;
235: 
236:   thisCached_ = true;
237: }
238: 
239: JniEnvCacher::~JniEnvCacher() {
240:   if (!thisCached_) {
````
- EN: Implements callable logic such as `currentOrNull`, `JniEnvCacher::JniEnvCacher`, `JniEnvCacher::~JniEnvCacher`.
- CN: 实现可调用逻辑，例如 `currentOrNull`, `JniEnvCacher::JniEnvCacher`, `JniEnvCacher::~JniEnvCacher`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````cpp
241:     return;
242:   }
243: 
244:   tls_key_t key = getTLKey();
245:   TLData* pdata = getTLData(key);
246:   FBJNI_ASSERT(pdata);
247:   FBJNI_ASSERT(pdata->env != nullptr);
248:   pdata->env = nullptr;
249:   if (!pdata->attached) {
250:     setTLData(key, nullptr);
251:   }
252: }
253: 
254: }
255: 
256: ThreadScope::ThreadScope()
257:   : thisAttached_(false)
258: {
259:   if (g_vm == nullptr) {
260:     throw std::runtime_error("fbjni is uninitialized; no thread can be attached.");
261:   }
262: 
263:   JNIEnv* env;
264: 
265:   // Check if the thread is attached somehow.
266:   auto result = getEnv(&env);
267:   if (result == JNI_OK) {
268:     return;
269:   }
270: 
271:   // At this point, it appears there's no thread attached and no env is
272:   // cached, or we would have returned already.  So there better not
273:   // be TLData.
274: 
275:   tls_key_t key = getTLKey();
276:   detail::TLData* pdata = getTLData(key);
277:   FBJNI_ASSERT(pdata == nullptr);
278:   setTLData(key, &data_);
279: 
280:   attachCurrentThread();
281: 
282:   data_.env = nullptr;
283:   data_.attached = true;
284: 
285:   thisAttached_ = true;
286: }
287: 
288: ThreadScope::~ThreadScope() {
289:   if (!thisAttached_) {
290:     return;
291:   }
292: 
293:   tls_key_t key = getTLKey();
294:   detail::TLData* pdata = getTLData(key);
295:   FBJNI_ASSERT(pdata);
296:   FBJNI_ASSERT(pdata->env == nullptr);
297:   FBJNI_ASSERT(pdata->attached);
298:   FBJNI_ASSERT(g_vm);
299:   g_vm->DetachCurrentThread();
300:   setTLData(key, nullptr);
````
- EN: Implements callable logic such as `ThreadScope::ThreadScope`, `ThreadScope::~ThreadScope`.
- CN: 实现可调用逻辑，例如 `ThreadScope::ThreadScope`, `ThreadScope::~ThreadScope`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````cpp
301: }
302: 
303: /* static */
304: JNIEnv* Environment::current() {
305:   FBJNI_ASSERT(g_vm);
306:   JNIEnv* env = detail::currentOrNull();
307:   if (env == nullptr) {
308:     throw std::runtime_error("Unable to retrieve jni environment. Is the thread attached?");
309:   }
310:   return env;
311: }
312: 
313: /* static */
314: JNIEnv* Environment::ensureCurrentThreadIsAttached() {
315:   FBJNI_ASSERT(g_vm);
316:   JNIEnv* env = detail::currentOrNull();
317:   if (env == nullptr) {
318:     env = attachCurrentThread();
319:     FBJNI_ASSERT(env);
320:   }
321:   return env;
322: }
323: 
324: /* static */
325: bool Environment::isGlobalJvmAvailable() {
326:   return g_vm != nullptr;
327: }
328: 
329: namespace {
330: struct JThreadScopeSupport : JavaClass<JThreadScopeSupport> {
331:   static auto constexpr kJavaDescriptor = "Lcom/facebook/jni/ThreadScopeSupport;";
332: 
333:   // These reinterpret_casts are a totally dangerous pattern. Don't use them. Use HybridData instead.
334:   static void runStdFunction(std::function<void()>&& func) {
335:     static const auto method = javaClassStatic()->getStaticMethod<void(jlong)>("runStdFunction");
336:     method(javaClassStatic(), reinterpret_cast<jlong>(&func));
337:   }
338: 
339:   static void runStdFunctionImpl(alias_ref<JClass>, jlong ptr) {
340:     (*reinterpret_cast<std::function<void()>*>(ptr))();
341:   }
342: 
343:   static void OnLoad() {
344:     // We need the javaClassStatic so that the class lookup is cached and that
345:     // runStdFunction can be called from a ThreadScope-attached thread.
346:     javaClassStatic()->registerNatives({
347:         makeNativeMethod("runStdFunctionImpl", runStdFunctionImpl),
348:       });
349:   }
350: };
351: }
352: 
353: /* static */
354: void ThreadScope::OnLoad() {
355:   // These classes are required for ScopeWithClassLoader. Ensure they are looked up when loading.
356:   JThreadScopeSupport::OnLoad();
357: }
358: 
359: /* static */
360: void ThreadScope::WithClassLoader(std::function<void()>&& runnable) {
````
- EN: Declares or extends types including `JThreadScopeSupport`, `lookup`.
- CN: 声明或扩展类型，包括 `JThreadScopeSupport`, `lookup`。
- EN: Implements callable logic such as `Environment::current`, `Environment::ensureCurrentThreadIsAttached`, `Environment::isGlobalJvmAvailable`, `runStdFunction`.
- CN: 实现可调用逻辑，例如 `Environment::current`, `Environment::ensureCurrentThreadIsAttached`, `Environment::isGlobalJvmAvailable`, `runStdFunction`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-369
````cpp
361:   if (cachedOrNull() == nullptr) {
362:     ThreadScope ts;
363:     JThreadScopeSupport::runStdFunction(std::move(runnable));
364:   } else {
365:     runnable();
366:   }
367: }
368: 
369: } }
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `EnvironmentInitializer` / 符号 `EnvironmentInitializer`
- Symbol `AttachTraits` / 符号 `AttachTraits`
- Symbol `JThreadScopeSupport` / 符号 `JThreadScopeSupport`
- Symbol `lookup` / 符号 `lookup`

## Dependencies / 依赖关系
- C/C++ includes: `fbjni/fbjni.h`, `sys/prctl.h`, `vector`, `functional`, `pthread.h`, `windows.h`
- C/C++ 头文件: `fbjni/fbjni.h`, `sys/prctl.h`, `vector`, `functional`, `pthread.h`, `windows.h`
