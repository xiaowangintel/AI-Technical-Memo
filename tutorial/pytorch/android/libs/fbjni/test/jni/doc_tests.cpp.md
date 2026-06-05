# doc_tests.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/test/jni/doc_tests.cpp`
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
17: #include <cassert>
18: #include <cstring>
19: #include <type_traits>
20: #include <stdexcept>
21: #include <vector>
22: 
23: // SECTION registration
24: #include <fbjni/fbjni.h>
25: using namespace facebook::jni;
26: // END
27: 
28: // SECTION byte_buffer
29: #include <fbjni/ByteBuffer.h>
30: // END
31: 
32: // We can put all of our code in an anonymous namespace if
33: // it is not used from any other C++ code.
34: namespace {
35: 
36: 
37: // SECTION inheritance
38: struct JMyBaseClass : JavaClass<JMyBaseClass> {
39:   static constexpr auto kJavaDescriptor = "Lcom/facebook/jni/MyBaseClass;";
40: };
41: 
42: struct JMyDerivedClass : JavaClass<JMyDerivedClass, JMyBaseClass> {
43:   static constexpr auto kJavaDescriptor = "Lcom/facebook/jni/MyDerivedClass;";
44: };
45: 
46: /* MARKDOWN
47: This will allow implicit casts from Derived to Base and explicit downcasts.
48: When no base class is given, JObject will be used as the base.
49: // END
50: */
51: 
52: // SECTION nested_class
53: struct JNested : JavaClass<JNested> {
54:   static constexpr auto kJavaDescriptor = "Lcom/facebook/jni/Outer$Nested;";
55:   static local_ref<JNested> create() {
56:     return newInstance();
57:   }
58: };
59: // END
60: 
````
- EN: Pulls in native headers such as `cassert`, `cstring`, `type_traits`, `stdexcept`.
- CN: 引入原生头文件，例如 `cassert`, `cstring`, `type_traits`, `stdexcept`。
- EN: Declares or extends types including `JMyBaseClass`, `JMyDerivedClass`, `is`, `JNested`.
- CN: 声明或扩展类型，包括 `JMyBaseClass`, `JMyDerivedClass`, `is`, `JNested`。
- EN: Implements callable logic such as `create`.
- CN: 实现可调用逻辑，例如 `create`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````cpp
 61: // SECTION constructor
 62: struct JDataHolder : JavaClass<JDataHolder> {
 63:   static constexpr auto kJavaDescriptor = "Lcom/facebook/jni/DataHolder;";
 64:   // newInstance should be wrapped to ensure compile-time checking of call sites.
 65:   static local_ref<JDataHolder> create(int i, std::string const& s) {
 66:     // Constructor is looked up by argument types at *runtime*.
 67:     return newInstance(i, s);
 68:   }
 69:   // END
 70: 
 71:   // SECTION fields
 72:   void getAndSetFields() {
 73:     static const auto cls = javaClassStatic();
 74:     // Primitive fields.
 75:     static const auto iField = cls->getField<jint>("i");
 76:     jint i = this->getFieldValue(iField);
 77:     this->setFieldValue(iField, i + 1);
 78:     // Object fields work for standard classes and your own JavaObject classes.
 79:     static const auto sField = cls->getField<JString>("s");
 80:     // Object are returned as local refs ...
 81:     local_ref<JString> s = this->getFieldValue(sField);
 82:     // and can be set from any ref.
 83:     this->setFieldValue(sField, make_jstring(s->toStdString() + "1").get());
 84:     // Static fields work the same, but getStaticField, getStaticFieldValue,
 85:     // and setStaticFieldValue must all be called on the class object.
 86:     static const auto someInstanceField = cls->getStaticField<JDataHolder>("someInstance");
 87:     auto inst = cls->getStaticFieldValue(someInstanceField);
 88:     if (!inst) {
 89:       // NOTE: Can't use cls here because it is declared const.
 90:       getClass()->setStaticFieldValue(someInstanceField, self());
 91:     }
 92:   }
 93:   // END
 94: 
 95:   local_ref<JString> getStr() {
 96:     static const auto cls = javaClassStatic();
 97:     static const auto sField = cls->getField<JString>("s");
 98:     return getFieldValue(sField);
 99:   }
100: };
101: 
102: 
103: // SECTION registration
104: // Standard declaration for a normal class (no C++ fields).
105: struct DocTests : JavaClass<DocTests> {
106:   static constexpr auto kJavaDescriptor = "Lcom/facebook/jni/DocTests;";
107:   // END
108: 
109:   // SECTION constructor
110:   // Call-site in another file.
111:   static local_ref<JDataHolder> runConstructor(
112:       alias_ref<JClass> clazz) {
113:     // Call to ordinatry C++ function is checked at *compile time*.
114:     return JDataHolder::create(1, "hi");
115:   }
116:   // END
117: 
118:   // SECTION basic_methods
119:  public:
120:   // Java methods should usually be wrapped by C++ methods for ease-of-use.
````
- EN: Declares or extends types including `JDataHolder`, `object`, `DocTests`.
- CN: 声明或扩展类型，包括 `JDataHolder`, `object`, `DocTests`。
- EN: Implements callable logic such as `create`, `getAndSetFields`, `getStr`, `runConstructor`.
- CN: 实现可调用逻辑，例如 `create`, `getAndSetFields`, `getStr`, `runConstructor`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````cpp
121:   // (Most other examples in this document will inline these for brevity.)
122:   void callVoidMethod() {
123:     static const auto method = getClass()->getMethod<void()>("voidMethod");
124:     // self() returns the raw JNI reference to this object.
125:     method(self());
126:   }
127:   static void callStaticVoidMethod() {
128:     static const auto cls = javaClassStatic();
129:     static const auto method = cls->getStaticMethod<void()>("staticVoidMethod");
130:     method(cls);
131:   }
132: 
133:   // Native implementations of Java methods can be private.
134:  private:
135:   // For non-Hybrid objects, all JNI methods must be static on the C++ side
136:   // because only Hybrid objects can have C++ state.
137:   static void nativeVoidMethod(
138:       // All non-static methods receive "this" as a first argument.
139:       alias_ref<DocTests> thiz) {
140:     // Make sure we got the right object.
141:     assert(thiz->toString() == "instance of DocTests");
142:     thiz->callVoidMethod();
143:   }
144:   static void staticNativeVoidMethod(
145:       // All static methods receive the class as a first argument.
146:       alias_ref<JClass> clazz) {
147:     assert(clazz->toString() == "class com.facebook.jni.DocTests");
148:     DocTests::callStaticVoidMethod();
149:   }
150:   // END
151: 
152:   // SECTION primitives
153:   static jlong addSomeNumbers(alias_ref<JClass> clazz, jbyte b, jshort s, jint i) {
154:     static const auto doubler = clazz->getStaticMethod<jlong(jint)>("doubler");
155:     jlong l = doubler(clazz, 4);
156:     return b + s + i + l;
157:   }
158: 
159:   /* MARKDOWN
160:   Argument and return types can be in either JNI style or C++ style.
161: 
162:   | Java type | JNI types |
163:   | --- | --- |
164:   | `boolean` | `jboolean`, `bool` |
165:   | `byte` | `jbyte`, `int8_t` |
166:   | `char` | `jchar` |
167:   | `short` | `jshort`, `short`, `int16_t` |
168:   | `int` | `jint`, `int`, `int32_t` |
169:   | `long` | `jlong`, `int64_t` |
170:   | `float` | `jfloat`, `float` |
171:   | `double` | `jdouble`, `double` |
172:   // END
173:   */
174: 
175:   // SECTION strings
176:   static std::string fancyCat(
177:       alias_ref<JClass> clazz,
178:       // Native methods can receive strings as JString (direct JNI reference) ...
179:       alias_ref<JString> s1,
180:       // or as std::string (converted to real UTF-8).
````
- EN: Declares or extends types including `as`, `com`.
- CN: 声明或扩展类型，包括 `as`, `com`。
- EN: Implements callable logic such as `callVoidMethod`, `callStaticVoidMethod`, `nativeVoidMethod`, `staticNativeVoidMethod`.
- CN: 实现可调用逻辑，例如 `callVoidMethod`, `callStaticVoidMethod`, `nativeVoidMethod`, `staticNativeVoidMethod`。
- EN: Includes validation, assertions, or defensive checks.
- CN: 包含校验、断言或防御性检查。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 181-240
````cpp
181:       std::string s2) {
182:     // Convert JString to std::string.
183:     std::string result = s1->toStdString();
184:     // Java methods can receive and return JString ...
185:     static const auto doubler_java = clazz->getStaticMethod<JString(JString)>("doubler");
186:     result += doubler_java(clazz, *s1)->toStdString();
187:     // and also std::string (converted from real UTF-8).
188:     static const auto doubler_std = clazz->getStaticMethod<std::string(std::string)>("doubler");
189:     result += doubler_std(clazz, s2)->toStdString();
190:     // They can also receive const char*, but not return it.
191:     static const auto doubler_char = clazz->getStaticMethod<std::string(const char*)>("doubler");
192:     result += doubler_char(clazz, s2.c_str())->toStdString();
193:     // All 3 formats can be returned (std::string shown here, const char* below).
194:     return result;
195:   }
196: 
197:   static const char* getCString(alias_ref<JClass>) {
198:     // This string is converted to JString *after* getCString returns.
199:     // Watch your memory lifetimes.
200:     return "Watch your memory.";
201:   }
202:   // END
203: 
204:   // SECTION primitive_arrays
205:   static local_ref<JArrayInt> primitiveArrays(
206:       alias_ref<JClass> clazz,
207:       // JArrayX is available for all primitives.
208:       alias_ref<JArrayInt> arr) {
209:     size_t size = arr->size();
210:     std::vector<jint> buffer(size + 1L);
211:     // Copy elements into native memory.
212:     arr->getRegion(0, size, buffer.data());
213:     // Copy elements into fresh memory (returns unique_ptr<int[]>).
214:     auto elements = arr->getRegion(0, size);
215:     // Pin can eliminate the need for a copy.
216:     {
217:       auto pin = arr->pin();
218:       for (size_t i = 0; i < pin.size(); i++) {
219:         // Can read and/or write pin[i].
220:         buffer[size] += pin[i];
221:       }
222:     }
223:     // Allocating a new array and copying data in.
224:     // (Data can also be assigned by writing to a pin.)
225:     auto ret = JArrayInt::newArray(size + 1);
226:     ret->setRegion(0, size + 1, buffer.data());
227:     return ret;
228:   }
229:   // END
230: 
231:   // SECTION class_arrays
232:   static local_ref<JArrayClass<JString>> classArrays(
233:       alias_ref<JClass> clazz,
234:       alias_ref<JArrayClass<JDataHolder>> arr) {
235:     size_t size = arr->size();
236:     local_ref<JArrayClass<JString>> ret = JArrayClass<JString>::newArray(size);
237:     for (int i = 0; i < size; ++i) {
238:       local_ref<JString> str = arr->getElement(i)->getStr();
239:       ret->setElement(i, *str);
240:     }
````
- EN: Implements callable logic such as `getCString`, `primitiveArrays`, `classArrays`.
- CN: 实现可调用逻辑，例如 `getCString`, `primitiveArrays`, `classArrays`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````cpp
241:     return ret;
242:   }
243:   // END
244: 
245:   // SECTION references
246:   /* MARKDOWN
247: 
248:   ### `alias_ref<JFoo>`
249:   `alias_ref` is a non-owning reference, like a bare pointer.
250:   It is used almost exclusively for function arguments.
251: 
252:   ### `local_ref<JFoo>`
253:   `local_ref` is a ref-counted thread-specific pointer that is invalidated upon returning to Java.
254:   For variables used within a function, use `local_ref`.
255:   Most functions should return `local_ref` (and let the caller convert to a `global_ref` if necessary).
256: 
257:   ### `global_ref<JFoo>`
258:   `global_ref` is a ref-counted pointer.
259:   Use this for storing a reference to a Java object that may
260:   outlive the current call from Java into C++
261:   (e.g. class member fields are usually global refs).
262:   You can create a new `global_ref` (from an `alias_ref`/`local_ref`) by calling `make_global`.
263:   // END
264:   */
265:   // SECTION references
266:   static local_ref<JObject> convertReferences(
267:       alias_ref<JClass> clazz,
268:       alias_ref<JMyDerivedClass> derived) {
269:     local_ref<JMyDerivedClass> local_derived = make_local(derived);
270:     global_ref<JMyDerivedClass> global_derived = make_global(derived);
271:     // Store global_derived somewhere.
272:     return local_derived;
273:   }
274:   // END
275: 
276:   // SECTION inheritance
277:   static void castReferences(
278:       alias_ref<JClass> clazz,
279:       alias_ref<JMyBaseClass> base) {
280:     // Just like raw pointers, upcasting is implicit.
281:     alias_ref<JObject> obj = base;
282:     // static_ref_cast is like C++ static_cast.  No runtime checking is done.
283:     alias_ref<JMyDerivedClass> derived_1 = static_ref_cast<JMyDerivedClass>(base);
284:     // dynamic_ref_cast is like C++ dynamic_cast.
285:     // It will check that the runtime Java type is actually derived from the target type.
286:     try {
287:       alias_ref<JMyDerivedClass> derived_2 = dynamic_ref_cast<JMyDerivedClass>(base);
288:       (void)derived_2;
289:     } catch (const JniException& exn) {
290:       // Throws ClassCastException if the cast fails.
291:       throw;
292:     }
293:     // END
294:     // Supress warnings.
295:     (void)obj;
296:     (void)derived_1;
297:   }
298: 
299:   static void callGetAndSetFields(
300:       alias_ref<JClass> clazz,
````
- EN: Declares or extends types including `member`.
- CN: 声明或扩展类型，包括 `member`。
- EN: Implements callable logic such as `convertReferences`, `castReferences`.
- CN: 实现可调用逻辑，例如 `convertReferences`, `castReferences`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````cpp
301:       alias_ref<JDataHolder> data) {
302:     data->getAndSetFields();
303:   }
304: 
305:   // SECTION jobject_jclass
306:   static std::string showJObject(
307:       alias_ref<JClass> clazz,
308:       // JObject is the base class of all fbjni types.  It corresponds to java.lang.Object.
309:       alias_ref<JObject> obj,
310:       alias_ref<JDataHolder> data) {
311:     local_ref<JClass> objClass = obj->getClass();
312:     local_ref<JClass> dataClass = data->getClass();
313:     local_ref<JClass> parent = dataClass->getSuperclass();
314:     assert(isSameObject(parent, objClass));
315:     assert(data->isInstanceOf(parent));
316:     assert(objClass->isAssignableFrom(clazz));
317:     std::string str = "data=";
318:     {
319:       // Acquires the object lock until this object goes out of scope.
320:       auto lock = data->lock();
321:       // Calls Object.toString and converts to std::string.
322:       str += data->toString();
323:     }
324:     // All JavaClass types have a `javaobject` typedef, which is their raw JNI type.
325:     static_assert(std::is_same<JObject::javaobject, jobject>::value, "");
326:     static_assert(std::is_same<JClass::javaobject, jclass>::value, "");
327:     static_assert(!std::is_same<JDataHolder::javaobject, jobject>::value, "");
328:     static_assert(std::is_convertible<JDataHolder::javaobject, jobject>::value, "");
329:     return str;
330:   }
331:   // END
332: 
333:   // SECTION simple_exceptions
334:   static void catchAndThrow(
335:       alias_ref<JClass> clazz) {
336:     try {
337:       clazz->getStaticMethod<void()>("doesNotExist");
338:       assert(!"Exception wasn't thrown.");
339:     } catch (JniException& exn) {
340:       // JniException extends std::exception, so "catch (std::exception& exn)" also works.
341:       local_ref<JThrowable> underlying = exn.getThrowable();
342:       const char* msg = exn.what();
343:       // Throwing exceptions from C++ is fine.
344:       // They will be translated to an appropriate Java exception type.
345:       throw std::runtime_error(std::string() + "Caught '" + msg + "'");
346:     }
347:   }
348:   // END
349: 
350:   // SECTION boxed
351:   static local_ref<JDouble> scaleUp(
352:       alias_ref<JClass> clazz,
353:       alias_ref<JInteger> number) {
354:     // Boxed types exist for all Java primitive types.
355:     // Unbox with ->value() or ->intValue.
356:     jint unboxed = number->value();
357:     jdouble scaled = unboxed * 1.5;
358:     // Box with autobox() or JDouble::valueOf.
359:     local_ref<JDouble> ret = autobox(scaled);
360:     return ret;
````
- EN: Declares or extends types including `of`.
- CN: 声明或扩展类型，包括 `of`。
- EN: Implements callable logic such as `showJObject`, `catchAndThrow`, `scaleUp`.
- CN: 实现可调用逻辑，例如 `showJObject`, `catchAndThrow`, `scaleUp`。
- EN: Includes validation, assertions, or defensive checks.
- CN: 包含校验、断言或防御性检查。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 361-420
````cpp
361:   }
362:   // END
363: 
364:   // SECTION iterables
365:   static std::string concatMatches(
366:       alias_ref<JClass> clazz,
367:       // Note that generic types are *not* checked against Java declarations.
368:       alias_ref<JList<JInteger>> values,
369:       alias_ref<JMap<JString, JInteger>> names) {
370:     int sum = 0;
371:     std::string ret;
372:     // Iterator and Iterable support C++ iteration.
373:     // Collection, List, and Set support iteration and ->size().
374:     for (const auto& elem : *values) {
375:       sum += elem->value();
376:     }
377:     // Maps iterate like C++ maps.
378:     for (const auto& entry : *names) {
379:       if (entry.second->value() == sum) {
380:         ret += entry.first->toStdString();
381:       }
382:     }
383:     // This works if you build with C++17.
384:     // for (const auto& [key, value] : *names) {
385:     return ret;
386:   }
387:   // END
388: 
389:   // SECTION byte_buffer
390:   static local_ref<JByteBuffer> transformBuffer(
391:       alias_ref<JClass> clazz,
392:       alias_ref<JByteBuffer> data) {
393:     // Direct ByteBuffers are an efficient way to transfer bulk data between Java and C++.
394:     if (!data->isDirect()) {
395:       throw std::runtime_error("Argument is not a direct buffer.");
396:     }
397:     // Transform data into a local buffer.
398:     std::vector<uint8_t> buffer(data->getDirectSize());
399:     uint8_t* raw_data = data->getDirectBytes();
400:     for (size_t i = 0; i < buffer.size(); ++i) {
401:       buffer[i] = raw_data[i] + 1;
402:     }
403:     // Wrap our data in a buffer and pass to Java.
404:     // Note that the buffer *directly* references our memory.
405:     local_ref<JByteBuffer> wrapper = JByteBuffer::wrapBytes(buffer.data(), buffer.size());
406:     static const auto receiver = clazz->getStaticMethod<void(alias_ref<JByteBuffer>)>("receiveBuffer");
407:     receiver(clazz, wrapper);
408:     // We can create a new buffer that owns its own memory and safely return it.
409:     local_ref<JByteBuffer> ret = JByteBuffer::allocateDirect(buffer.size());
410:     std::memcpy(ret->getDirectBytes(), buffer.data(), buffer.size());
411:     return ret;
412:   }
413:   // END
414: 
415: 
416:  public:
417:   // SECTION registration
418:   // NOTE: The name of this method doesn't matter.
419:   static void registerNatives() {
420:     javaClassStatic()->registerNatives({
````
- EN: Implements callable logic such as `concatMatches`, `size`, `transformBuffer`, `registerNatives`.
- CN: 实现可调用逻辑，例如 `concatMatches`, `size`, `transformBuffer`, `registerNatives`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 421-451
````cpp
421:       makeNativeMethod("nativeVoidMethod", DocTests::nativeVoidMethod),
422:       makeNativeMethod("staticNativeVoidMethod", DocTests::staticNativeVoidMethod),
423:       // END
424:       makeNativeMethod("addSomeNumbers", DocTests::addSomeNumbers),
425:       makeNativeMethod("fancyCat", DocTests::fancyCat),
426:       makeNativeMethod("getCString", DocTests::getCString),
427:       makeNativeMethod("primitiveArrays", DocTests::primitiveArrays),
428:       makeNativeMethod("convertReferences", DocTests::convertReferences),
429:       makeNativeMethod("castReferences", DocTests::castReferences),
430:       makeNativeMethod("runConstructor", DocTests::runConstructor),
431:       makeNativeMethod("callGetAndSetFields", DocTests::callGetAndSetFields),
432:       makeNativeMethod("showJObject", DocTests::showJObject),
433:       makeNativeMethod("catchAndThrow", DocTests::catchAndThrow),
434:       makeNativeMethod("scaleUp", DocTests::scaleUp),
435:       makeNativeMethod("concatMatches", DocTests::concatMatches),
436:       makeNativeMethod("transformBuffer", DocTests::transformBuffer),
437:     });
438:   }
439: };
440: 
441: 
442: } // Anonymous namespace
443: 
444: 
445: // SECTION registration
446: jint JNI_OnLoad(JavaVM* vm, void*) {
447:   return facebook::jni::initialize(vm, [] {
448:       DocTests::registerNatives();
449:   });
450: }
451: // END
````
- EN: Implements callable logic such as `JNI_OnLoad`.
- CN: 实现可调用逻辑，例如 `JNI_OnLoad`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Validation and test coverage / 校验与测试覆盖
- Symbol `JMyBaseClass` / 符号 `JMyBaseClass`
- Symbol `JMyDerivedClass` / 符号 `JMyDerivedClass`
- Symbol `is` / 符号 `is`
- Symbol `JNested` / 符号 `JNested`

## Dependencies / 依赖关系
- C/C++ includes: `cassert`, `cstring`, `type_traits`, `stdexcept`, `vector`, `fbjni/fbjni.h`, `fbjni/ByteBuffer.h`
- C/C++ 头文件: `cassert`, `cstring`, `type_traits`, `stdexcept`, `vector`, `fbjni/fbjni.h`, `fbjni/ByteBuffer.h`
