# Exceptions.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/Exceptions.cpp`
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
17: #include "CoreClasses.h"
18: #include "Log.h"
19: 
20: #ifndef FBJNI_NO_EXCEPTION_PTR
21: #include <lyra/lyra.h>
22: #include <lyra/lyra_exceptions.h>
23: #endif
24: 
25: #include <cstdlib>
26: #include <ios>
27: #include <stdexcept>
28: #include <stdio.h>
29: #include <string>
30: #include <system_error>
31: 
32: #include <jni.h>
33: 
34: #ifndef _WIN32
35: #include <cxxabi.h>
36: #endif
37: 
38: namespace facebook {
39: namespace jni {
40: 
41: namespace {
42: class JRuntimeException : public JavaClass<JRuntimeException, JThrowable> {
43:  public:
44:   static auto constexpr kJavaDescriptor = "Ljava/lang/RuntimeException;";
45: 
46:   static local_ref<JRuntimeException> create(const char* str) {
47:     return newInstance(make_jstring(str));
48:   }
49: 
50:   static local_ref<JRuntimeException> create() {
51:     return newInstance();
52:   }
53: };
54: 
55: class JIOException : public JavaClass<JIOException, JThrowable> {
56:  public:
57:   static auto constexpr kJavaDescriptor = "Ljava/io/IOException;";
58: 
59:   static local_ref<JIOException> create(const char* str) {
60:     return newInstance(make_jstring(str));
````
- EN: Pulls in native headers such as `CoreClasses.h`, `Log.h`, `lyra/lyra.h`, `lyra/lyra_exceptions.h`.
- CN: 引入原生头文件，例如 `CoreClasses.h`, `Log.h`, `lyra/lyra.h`, `lyra/lyra_exceptions.h`。
- EN: Declares or extends types including `JRuntimeException`, `JIOException`.
- CN: 声明或扩展类型，包括 `JRuntimeException`, `JIOException`。
- EN: Implements callable logic such as `create`.
- CN: 实现可调用逻辑，例如 `create`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````cpp
 61:   }
 62: };
 63: 
 64: class JOutOfMemoryError : public JavaClass<JOutOfMemoryError, JThrowable> {
 65:  public:
 66:   static auto constexpr kJavaDescriptor = "Ljava/lang/OutOfMemoryError;";
 67: 
 68:   static local_ref<JOutOfMemoryError> create(const char* str) {
 69:     return newInstance(make_jstring(str));
 70:   }
 71: };
 72: 
 73: class JArrayIndexOutOfBoundsException : public JavaClass<JArrayIndexOutOfBoundsException, JThrowable> {
 74:  public:
 75:   static auto constexpr kJavaDescriptor = "Ljava/lang/ArrayIndexOutOfBoundsException;";
 76: 
 77:   static local_ref<JArrayIndexOutOfBoundsException> create(const char* str) {
 78:     return newInstance(make_jstring(str));
 79:   }
 80: };
 81: 
 82: class JUnknownCppException : public JavaClass<JUnknownCppException, JThrowable> {
 83:  public:
 84:   static auto constexpr kJavaDescriptor = "Lcom/facebook/jni/UnknownCppException;";
 85: 
 86:   static local_ref<JUnknownCppException> create() {
 87:     return newInstance();
 88:   }
 89: 
 90:   static local_ref<JUnknownCppException> create(const char* str) {
 91:     return newInstance(make_jstring(str));
 92:   }
 93: };
 94: 
 95: class JCppSystemErrorException : public JavaClass<JCppSystemErrorException, JThrowable> {
 96:  public:
 97:   static auto constexpr kJavaDescriptor = "Lcom/facebook/jni/CppSystemErrorException;";
 98: 
 99:   static local_ref<JCppSystemErrorException> create(const std::system_error& e) {
100:     return newInstance(make_jstring(e.what()), e.code().value());
101:   }
102: };
103: 
104: // Exception throwing & translating functions //////////////////////////////////////////////////////
105: 
106: // Functions that throw Java exceptions
107: 
108: void setJavaExceptionAndAbortOnFailure(alias_ref<JThrowable> throwable) {
109:   auto env = Environment::current();
110:   if (throwable) {
111:     env->Throw(throwable.get());
112:   }
113:   if (env->ExceptionCheck() != JNI_TRUE) {
114:     FBJNI_LOGF("Failed to set Java exception");
115:   }
116: }
117: 
118: }
119: 
120: // Functions that throw C++ exceptions
````
- EN: Declares or extends types including `JOutOfMemoryError`, `JArrayIndexOutOfBoundsException`, `JUnknownCppException`, `JCppSystemErrorException`.
- CN: 声明或扩展类型，包括 `JOutOfMemoryError`, `JArrayIndexOutOfBoundsException`, `JUnknownCppException`, `JCppSystemErrorException`。
- EN: Implements callable logic such as `create`, `setJavaExceptionAndAbortOnFailure`.
- CN: 实现可调用逻辑，例如 `create`, `setJavaExceptionAndAbortOnFailure`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````cpp
121: 
122: // TODO(T6618159) Inject the c++ stack into the exception's stack trace. One
123: // issue: when a java exception is created, it captures the full java stack
124: // across jni boundaries. lyra will only capture the c++ stack to the jni
125: // boundary. So, as we pass the java exception up to c++, we need to capture
126: // the c++ stack and then insert it into the correct place in the java stack
127: // trace. Then, as the exception propagates across the boundaries, we will
128: // slowly fill in the c++ parts of the trace.
129: void throwPendingJniExceptionAsCppException() {
130:   JNIEnv* env = Environment::current();
131:   if (env->ExceptionCheck() == JNI_FALSE) {
132:     return;
133:   }
134: 
135:   auto throwable = env->ExceptionOccurred();
136:   if (!throwable) {
137:     throw std::runtime_error("Unable to get pending JNI exception.");
138:   }
139:   env->ExceptionClear();
140: 
141:   throw JniException(adopt_local(throwable));
142: }
143: 
144: void throwCppExceptionIf(bool condition) {
145:   if (!condition) {
146:     return;
147:   }
148: 
149:   auto env = Environment::current();
150:   if (env->ExceptionCheck() == JNI_TRUE) {
151:     throwPendingJniExceptionAsCppException();
152:     return;
153:   }
154: 
155:   throw JniException();
156: }
157: 
158: void throwNewJavaException(jthrowable throwable) {
159:   throw JniException(wrap_alias(throwable));
160: }
161: 
162: void throwNewJavaException(const char* throwableName, const char* msg) {
163:   // If anything of the fbjni calls fail, an exception of a suitable
164:   // form will be thrown, which is what we want.
165:   auto throwableClass = findClassLocal(throwableName);
166:   auto throwable = throwableClass->newObject(
167:     throwableClass->getConstructor<jthrowable(jstring)>(),
168:     make_jstring(msg).release());
169:   throwNewJavaException(throwable.get());
170: }
171: 
172: // jthrowable //////////////////////////////////////////////////////////////////////////////////////
173: 
174: local_ref<JThrowable> JThrowable::initCause(alias_ref<JThrowable> cause) {
175:   static auto meth = javaClassStatic()->getMethod<javaobject(alias_ref<javaobject>)>("initCause");
176:   return meth(self(), cause);
177: }
178: 
179: auto JThrowable::getStackTrace() -> local_ref<JStackTrace> {
180:   static auto meth = javaClassStatic()->getMethod<JStackTrace::javaobject()>("getStackTrace");
````
- EN: Implements callable logic such as `TODO`, `throwCppExceptionIf`, `throwNewJavaException`, `JThrowable::initCause`.
- CN: 实现可调用逻辑，例如 `TODO`, `throwCppExceptionIf`, `throwNewJavaException`, `JThrowable::initCause`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````cpp
181:   return meth(self());
182: }
183: 
184: void JThrowable::setStackTrace(alias_ref<JStackTrace> stack) {
185:   static auto meth = javaClassStatic()->getMethod<void(alias_ref<JStackTrace>)>("setStackTrace");
186:   return meth(self(), stack);
187: }
188: 
189: auto JStackTraceElement::create(
190:     const std::string& declaringClass, const std::string& methodName, const std::string& file, int line)
191:     -> local_ref<javaobject> {
192:   return newInstance(declaringClass, methodName, file, line);
193: }
194: 
195: std::string JStackTraceElement::getClassName() const {
196:   static auto meth = javaClassStatic()->getMethod<local_ref<JString>()>("getClassName");
197:   return meth(self())->toStdString();
198: }
199: 
200: std::string JStackTraceElement::getMethodName() const {
201:   static auto meth = javaClassStatic()->getMethod<local_ref<JString>()>("getMethodName");
202:   return meth(self())->toStdString();
203: }
204: 
205: std::string JStackTraceElement::getFileName() const {
206:   static auto meth = javaClassStatic()->getMethod<local_ref<JString>()>("getFileName");
207:   return meth(self())->toStdString();
208: }
209: 
210: int JStackTraceElement::getLineNumber() const {
211:   static auto meth = javaClassStatic()->getMethod<jint()>("getLineNumber");
212:   return meth(self());
213: }
214: 
215: // Translate C++ to Java Exception
216: 
217: namespace {
218: 
219: // For each exception in the chain of the exception_ptr argument, func
220: // will be called with that exception (in reverse order, i.e. innermost first).
221: #ifndef FBJNI_NO_EXCEPTION_PTR
222: void denest(const std::function<void(std::exception_ptr)>& func, std::exception_ptr ptr) {
223:   FBJNI_ASSERT(ptr);
224:   try {
225:     std::rethrow_exception(ptr);
226:   } catch (const std::nested_exception& e) {
227:     denest(func, e.nested_ptr());
228:   } catch (...) {
229:     // ignored.
230:   }
231:   func(ptr);
232:   }
233: #endif
234: 
235: } // namespace
236: 
237: #ifndef FBJNI_NO_EXCEPTION_PTR
238: local_ref<JStackTraceElement> createJStackTraceElement(const lyra::StackTraceElement& cpp) {
239:   return JStackTraceElement::create(
240:       "|lyra|{" + cpp.libraryName() + "}", cpp.functionName(), cpp.buildId(), cpp.libraryOffset());
````
- EN: Implements callable logic such as `JThrowable::setStackTrace`, `JStackTraceElement::getClassName`, `JStackTraceElement::getMethodName`, `JStackTraceElement::getFileName`.
- CN: 实现可调用逻辑，例如 `JThrowable::setStackTrace`, `JStackTraceElement::getClassName`, `JStackTraceElement::getMethodName`, `JStackTraceElement::getFileName`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````cpp
241: }
242: 
243: void addCppStacktraceToJavaException(alias_ref<JThrowable> java, std::exception_ptr cpp) {
244:   auto cppStack = lyra::getStackTraceSymbols(
245:                     (cpp == nullptr) ?
246:                       lyra::getStackTrace()
247:                       : lyra::getExceptionTrace(cpp));
248: 
249:   auto javaStack = java->getStackTrace();
250:   auto newStack = JThrowable::JStackTrace::newArray(javaStack->size() + cppStack.size());
251:   size_t i = 0;
252:   for (size_t j = 0; j < cppStack.size(); j++, i++) {
253:     (*newStack)[i] = createJStackTraceElement(cppStack[j]);
254:   }
255:   for (size_t j = 0; j < javaStack->size(); j++, i++) {
256:     (*newStack)[i] = (*javaStack)[j];
257:   }
258:   java->setStackTrace(newStack);
259: }
260: 
261: local_ref<JThrowable> convertCppExceptionToJavaException(std::exception_ptr ptr) {
262:   FBJNI_ASSERT(ptr);
263:   local_ref<JThrowable> current;
264:   bool addCppStack = true;
265:   try {
266:     std::rethrow_exception(ptr);
267:     addCppStack = false;
268:   } catch (const JniException& ex) {
269:     current = ex.getThrowable();
270:   } catch (const std::ios_base::failure& ex) {
271:     current = JIOException::create(ex.what());
272:   } catch (const std::bad_alloc& ex) {
273:     current = JOutOfMemoryError::create(ex.what());
274:   } catch (const std::out_of_range& ex) {
275:     current = JArrayIndexOutOfBoundsException::create(ex.what());
276:   } catch (const std::system_error& ex) {
277:     current = JCppSystemErrorException::create(ex);
278:   } catch (const std::runtime_error& ex) {
279:     current = JRuntimeException::create(ex.what());
280:   } catch (const std::exception& ex) {
281:     current = JCppException::create(ex.what());
282:   } catch (const char* msg) {
283:     current = JUnknownCppException::create(msg);
284:   } catch (...) {
285: #ifdef _WIN32
286:     current = JUnknownCppException::create();
287: #else
288:     const std::type_info* tinfo = abi::__cxa_current_exception_type();
289:     if (tinfo) {
290:       std::string msg = std::string("Unknown: ") + tinfo->name();
291:       current = JUnknownCppException::create(msg.c_str());
292:     } else {
293:       current = JUnknownCppException::create();
294:     }
295: #endif
296:   }
297: 
298:   if (addCppStack) {
299:     addCppStacktraceToJavaException(current, ptr);
300:   }
````
- EN: Implements callable logic such as `addCppStacktraceToJavaException`, `convertCppExceptionToJavaException`.
- CN: 实现可调用逻辑，例如 `addCppStacktraceToJavaException`, `convertCppExceptionToJavaException`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 301-360
````cpp
301:   return current;
302:   }
303: #endif
304: 
305: local_ref<JThrowable> getJavaExceptionForCppBackTrace() {
306:   return getJavaExceptionForCppBackTrace(nullptr);
307: }
308: 
309: local_ref<JThrowable> getJavaExceptionForCppBackTrace(const char* msg) {
310:   local_ref<JThrowable> current =
311:       msg ? JUnknownCppException::create(msg) : JUnknownCppException::create();
312: #ifndef FBJNI_NO_EXCEPTION_PTR
313:   addCppStacktraceToJavaException(current, nullptr);
314: #endif
315:   return current;
316: }
317: 
318: 
319: #ifndef FBJNI_NO_EXCEPTION_PTR
320: local_ref<JThrowable> getJavaExceptionForCppException(std::exception_ptr ptr) {
321:   FBJNI_ASSERT(ptr);
322:   local_ref<JThrowable> previous;
323:   auto func = [&previous] (std::exception_ptr ptr) {
324:     auto current = convertCppExceptionToJavaException(ptr);
325:     if (previous) {
326:       current->initCause(previous);
327:     }
328:     previous = current;
329:   };
330:   denest(func, ptr);
331:   return previous;
332: }
333: #endif
334: 
335: void translatePendingCppExceptionToJavaException() {
336:   try {
337: #ifndef FBJNI_NO_EXCEPTION_PTR
338:     auto exc = getJavaExceptionForCppException(std::current_exception());
339: #else
340:     auto exc = JUnknownCppException::create();
341: #endif
342:     setJavaExceptionAndAbortOnFailure(exc);
343:   } catch (...) {
344: #ifndef FBJNI_NO_EXCEPTION_PTR
345:     FBJNI_LOGE(
346:         "Unexpected error in translatePendingCppExceptionToJavaException(): %s",
347:         lyra::toString(std::current_exception()).c_str());
348: #else
349:     FBJNI_LOGE(
350:       "Unexpected error in translatePendingCppExceptionToJavaException()");
351: #endif
352:     std::terminate();
353:   }
354: }
355: 
356: // JniException ////////////////////////////////////////////////////////////////////////////////////
357: 
358: const std::string JniException::kExceptionMessageFailure_ = "Unable to get exception message.";
359: 
360: JniException::JniException() : JniException(JRuntimeException::create()) { }
````
- EN: Implements callable logic such as `getJavaExceptionForCppBackTrace`, `getJavaExceptionForCppException`, `translatePendingCppExceptionToJavaException`, `JniException::JniException`.
- CN: 实现可调用逻辑，例如 `getJavaExceptionForCppBackTrace`, `getJavaExceptionForCppException`, `translatePendingCppExceptionToJavaException`, `JniException::JniException`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-413
````cpp
361: 
362: JniException::JniException(alias_ref<jthrowable> throwable) : isMessageExtracted_(false) {
363:   throwable_ = make_global(throwable);
364: }
365: 
366: JniException::JniException(JniException &&rhs)
367:     : throwable_(std::move(rhs.throwable_)),
368:       what_(std::move(rhs.what_)),
369:       isMessageExtracted_(rhs.isMessageExtracted_) {
370: }
371: 
372: JniException::JniException(const JniException &rhs)
373:     : what_(rhs.what_), isMessageExtracted_(rhs.isMessageExtracted_) {
374:   throwable_ = make_global(rhs.throwable_);
375: }
376: 
377: JniException::~JniException() {
378:   try {
379:     ThreadScope ts;
380:     throwable_.reset();
381:   } catch (...) {
382:     FBJNI_LOGE("Exception in ~JniException()");
383:     std::terminate();
384:   }
385: }
386: 
387: local_ref<JThrowable> JniException::getThrowable() const noexcept {
388:   return make_local(throwable_);
389: }
390: 
391: // TODO 6900503: consider making this thread-safe.
392: void JniException::populateWhat() const noexcept {
393:   try {
394:     ThreadScope ts;
395:     what_ = throwable_->toString();
396:     isMessageExtracted_ = true;
397:   } catch(...) {
398:     what_ = kExceptionMessageFailure_;
399:   }
400: }
401: 
402: const char* JniException::what() const noexcept {
403:   if (!isMessageExtracted_) {
404:     populateWhat();
405:   }
406:   return what_.c_str();
407: }
408: 
409: void JniException::setJavaException() const noexcept {
410:   setJavaExceptionAndAbortOnFailure(throwable_);
411: }
412: 
413: }}
````
- EN: Implements callable logic such as `JniException::JniException`, `JniException::~JniException`.
- CN: 实现可调用逻辑，例如 `JniException::JniException`, `JniException::~JniException`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `JRuntimeException` / 符号 `JRuntimeException`
- Symbol `JIOException` / 符号 `JIOException`
- Symbol `JOutOfMemoryError` / 符号 `JOutOfMemoryError`
- Symbol `JArrayIndexOutOfBoundsException` / 符号 `JArrayIndexOutOfBoundsException`

## Dependencies / 依赖关系
- C/C++ includes: `CoreClasses.h`, `Log.h`, `lyra/lyra.h`, `lyra/lyra_exceptions.h`, `cstdlib`, `ios`, `stdexcept`, `stdio.h`, `string`, `system_error`
- C/C++ 头文件: `CoreClasses.h`, `Log.h`, `lyra/lyra.h`, `lyra/lyra_exceptions.h`, `cstdlib`, `ios`, `stdexcept`, `stdio.h`, `string`, `system_error`
