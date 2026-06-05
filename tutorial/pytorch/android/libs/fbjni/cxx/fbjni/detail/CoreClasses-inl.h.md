# CoreClasses-inl.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/CoreClasses-inl.h`
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
17: #pragma once
18: 
19: #include <string.h>
20: #include <type_traits>
21: #include <stdlib.h>
22: 
23: #include "Common.h"
24: #include "Exceptions.h"
25: #include "Meta.h"
26: #include "MetaConvert.h"
27: 
28: namespace facebook {
29: namespace jni {
30: 
31: // jobject /////////////////////////////////////////////////////////////////////////////////////////
32: 
33: inline bool isSameObject(alias_ref<JObject> lhs, alias_ref<JObject> rhs) noexcept {
34:   return Environment::current()->IsSameObject(lhs.get(), rhs.get()) != JNI_FALSE;
35: }
36: 
37: inline local_ref<JClass> JObject::getClass() const noexcept {
38:   return adopt_local(Environment::current()->GetObjectClass(self()));
39: }
40: 
41: inline bool JObject::isInstanceOf(alias_ref<JClass> cls) const noexcept {
42:   return Environment::current()->IsInstanceOf(self(), cls.get()) != JNI_FALSE;
43: }
44: 
45: template<typename T>
46: inline T JObject::getFieldValue(JField<T> field) const noexcept {
47:   return field.get(self());
48: }
49: 
50: template<typename T>
51: inline local_ref<T*> JObject::getFieldValue(JField<T*> field) const noexcept {
52:   return adopt_local(field.get(self()));
53: }
54: 
55: template<typename T>
56: inline void JObject::setFieldValue(JField<T> field, T value) noexcept {
57:   field.set(self(), value);
58: }
59: 
60: template<typename T, typename>
````
- EN: Pulls in native headers such as `string.h`, `type_traits`, `stdlib.h`, `Common.h`.
- CN: 引入原生头文件，例如 `string.h`, `type_traits`, `stdlib.h`, `Common.h`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 61-120
````cpp
 61: inline void JObject::setFieldValue(JField<T> field, alias_ref<T> value) noexcept {
 62:   setFieldValue(field, value.get());
 63: }
 64: 
 65: inline std::string JObject::toString() const {
 66:   static const auto method = findClassLocal("java/lang/Object")->getMethod<jstring()>("toString");
 67: 
 68:   return method(self())->toStdString();
 69: }
 70: 
 71: 
 72: // Class is here instead of CoreClasses.h because we need
 73: // alias_ref to be complete.
 74: class MonitorLock {
 75:  public:
 76:   inline MonitorLock() noexcept;
 77:   inline MonitorLock(alias_ref<JObject> object) noexcept;
 78:   inline ~MonitorLock() noexcept;
 79: 
 80:   inline MonitorLock(MonitorLock&& other) noexcept;
 81:   inline MonitorLock& operator=(MonitorLock&& other) noexcept;
 82: 
 83:   inline MonitorLock(const MonitorLock&) = delete;
 84:   inline MonitorLock& operator=(const MonitorLock&) = delete;
 85: 
 86:  private:
 87:   inline void reset() noexcept;
 88:   alias_ref<JObject> owned_;
 89: };
 90: 
 91: MonitorLock::MonitorLock() noexcept : owned_(nullptr) {}
 92: 
 93: MonitorLock::MonitorLock(alias_ref<JObject> object) noexcept
 94:     : owned_(object) {
 95:   Environment::current()->MonitorEnter(object.get());
 96: }
 97: 
 98: void MonitorLock::reset() noexcept {
 99:   if (owned_) {
100:     Environment::current()->MonitorExit(owned_.get());
101:     if (Environment::current()->ExceptionCheck()) {
102:       abort(); // Lock mismatch
103:     }
104:     owned_ = nullptr;
105:   }
106: }
107: 
108: MonitorLock::~MonitorLock() noexcept {
109:   reset();
110: }
111: 
112: MonitorLock::MonitorLock(MonitorLock&& other) noexcept
113:     : owned_(other.owned_)
114: {
115:   other.owned_ = nullptr;
116: }
117: 
118: MonitorLock& MonitorLock::operator=(MonitorLock&& other) noexcept {
119:   reset();
120:   owned_ = other.owned_;
````
- EN: Declares or extends types including `MonitorLock`.
- CN: 声明或扩展类型，包括 `MonitorLock`。
- EN: Implements callable logic such as `JObject::toString`, `MonitorLock::MonitorLock`.
- CN: 实现可调用逻辑，例如 `JObject::toString`, `MonitorLock::MonitorLock`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````cpp
121:   other.owned_ = nullptr;
122:   return *this;
123: }
124: 
125: inline MonitorLock JObject::lock() const noexcept {
126:   return MonitorLock(this_);
127: }
128: 
129: inline jobject JObject::self() const noexcept {
130:   return this_;
131: }
132: 
133: inline void swap(JObject& a, JObject& b) noexcept {
134:   using std::swap;
135:   swap(a.this_, b.this_);
136: }
137: 
138: // JavaClass ///////////////////////////////////////////////////////////////////////////////////////
139: 
140: namespace detail {
141: template<typename JC, typename... Args>
142: static local_ref<JC> newInstance(Args... args) {
143:   static auto cls = JC::javaClassStatic();
144:   static const auto constructor = cls->template getConstructor<typename JC::javaobject(Args...)>();
145:   return cls->newObject(constructor, args...);
146: }
147: }
148: 
149: 
150: template <typename T, typename B, typename J>
151: auto JavaClass<T, B, J>::self() const noexcept -> javaobject {
152:   return static_cast<javaobject>(JObject::self());
153: }
154: 
155: // jclass //////////////////////////////////////////////////////////////////////////////////////////
156: 
157: 
158: inline local_ref<JClass> JClass::getSuperclass() const noexcept {
159:   return adopt_local(Environment::current()->GetSuperclass(self()));
160: }
161: 
162: inline void JClass::registerNatives(std::initializer_list<JNINativeMethod> methods) {
163:   const auto env = Environment::current();
164:   auto result = env->RegisterNatives(self(), methods.begin(), static_cast<int>(methods.size()));
165:   FACEBOOK_JNI_THROW_EXCEPTION_IF(result != JNI_OK);
166: }
167: 
168: inline bool JClass::isAssignableFrom(alias_ref<JClass> other) const noexcept {
169:   const auto env = Environment::current();
170:   // Ths method has behavior compatible with the
171:   // java.lang.Class#isAssignableFrom method.  The order of the
172:   // arguments to the JNI IsAssignableFrom C function is "opposite"
173:   // from what some might expect, which makes this code look a little
174:   // odd, but it is correct.
175:   const auto result = env->IsAssignableFrom(other.get(), self());
176:   return result;
177: }
178: 
179: template<typename F>
180: inline JConstructor<F> JClass::getConstructor() const {
````
- EN: Implements callable logic such as `newInstance`, `JClass::registerNatives`, `JClass::getConstructor`.
- CN: 实现可调用逻辑，例如 `newInstance`, `JClass::registerNatives`, `JClass::getConstructor`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````cpp
181:   return getConstructor<F>(jmethod_traits_from_cxx<F>::kConstructorDescriptor.c_str());
182: }
183: 
184: template<typename F>
185: inline JConstructor<F> JClass::getConstructor(const char* descriptor) const {
186:   constexpr auto constructor_method_name = "<init>";
187:   return getMethod<F>(constructor_method_name, descriptor);
188: }
189: 
190: template<typename F>
191: inline JMethod<F> JClass::getMethod(const char* name) const {
192:   return getMethod<F>(name, jmethod_traits_from_cxx<F>::kDescriptor.c_str());
193: }
194: 
195: template<typename F>
196: inline JMethod<F> JClass::getMethod(
197:     const char* name,
198:     const char* descriptor) const {
199:   const auto env = Environment::current();
200:   const auto method = env->GetMethodID(self(), name, descriptor);
201:   FACEBOOK_JNI_THROW_EXCEPTION_IF(!method);
202:   return JMethod<F>{method};
203: }
204: 
205: template<typename F>
206: inline JStaticMethod<F> JClass::getStaticMethod(const char* name) const {
207:   return getStaticMethod<F>(name, jmethod_traits_from_cxx<F>::kDescriptor.c_str());
208: }
209: 
210: template<typename F>
211: inline JStaticMethod<F> JClass::getStaticMethod(
212:     const char* name,
213:     const char* descriptor) const {
214:   const auto env = Environment::current();
215:   const auto method = env->GetStaticMethodID(self(), name, descriptor);
216:   FACEBOOK_JNI_THROW_EXCEPTION_IF(!method);
217:   return JStaticMethod<F>{method};
218: }
219: 
220: template<typename F>
221: inline JNonvirtualMethod<F> JClass::getNonvirtualMethod(const char* name) const {
222:   return getNonvirtualMethod<F>(name, jmethod_traits_from_cxx<F>::kDescriptor.c_str());
223: }
224: 
225: template<typename F>
226: inline JNonvirtualMethod<F> JClass::getNonvirtualMethod(
227:     const char* name,
228:     const char* descriptor) const {
229:   const auto env = Environment::current();
230:   const auto method = env->GetMethodID(self(), name, descriptor);
231:   FACEBOOK_JNI_THROW_EXCEPTION_IF(!method);
232:   return JNonvirtualMethod<F>{method};
233: }
234: 
235: template<typename T>
236: inline JField<PrimitiveOrJniType<T>>
237: JClass::getField(const char* name) const {
238:   return getField<T>(name, jtype_traits<T>::kDescriptor.c_str());
239: }
240: 
````
- EN: Implements callable logic such as `JClass::getConstructor`, `JClass::getMethod`, `JClass::getStaticMethod`, `JClass::getNonvirtualMethod`.
- CN: 实现可调用逻辑，例如 `JClass::getConstructor`, `JClass::getMethod`, `JClass::getStaticMethod`, `JClass::getNonvirtualMethod`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````cpp
241: template<typename T>
242: inline JField<PrimitiveOrJniType<T>> JClass::getField(
243:     const char* name,
244:     const char* descriptor) const {
245:   const auto env = Environment::current();
246:   auto field = env->GetFieldID(self(), name, descriptor);
247:   FACEBOOK_JNI_THROW_EXCEPTION_IF(!field);
248:   return JField<PrimitiveOrJniType<T>>{field};
249: }
250: 
251: template<typename T>
252: inline JStaticField<PrimitiveOrJniType<T>> JClass::getStaticField(
253:     const char* name) const {
254:   return getStaticField<T>(name, jtype_traits<T>::kDescriptor.c_str());
255: }
256: 
257: template<typename T>
258: inline JStaticField<PrimitiveOrJniType<T>> JClass::getStaticField(
259:     const char* name,
260:     const char* descriptor) const {
261:   const auto env = Environment::current();
262:   auto field = env->GetStaticFieldID(self(), name, descriptor);
263:   FACEBOOK_JNI_THROW_EXCEPTION_IF(!field);
264:   return JStaticField<PrimitiveOrJniType<T>>{field};
265: }
266: 
267: template<typename T>
268: inline T JClass::getStaticFieldValue(JStaticField<T> field) const noexcept {
269:   return field.get(self());
270: }
271: 
272: template<typename T>
273: inline local_ref<T*> JClass::getStaticFieldValue(JStaticField<T*> field) noexcept {
274:   return adopt_local(field.get(self()));
275: }
276: 
277: template<typename T>
278: inline void JClass::setStaticFieldValue(JStaticField<T> field, T value) noexcept {
279:   field.set(self(), value);
280: }
281: 
282: template<typename T, typename>
283: inline void JClass::setStaticFieldValue(JStaticField<T> field, alias_ref<T> value) noexcept {
284:   setStaticFieldValue(field, value.get());
285: }
286: 
287: template<typename R, typename... Args>
288: inline local_ref<R> JClass::newObject(
289:     JConstructor<R(Args...)> constructor,
290:     Args... args) const {
291:   const auto env = Environment::current();
292:   auto object = env->NewObject(self(), constructor.getId(),
293:       detail::callToJni(
294:         detail::Convert<typename std::decay<Args>::type>::toCall(args))...);
295:   FACEBOOK_JNI_THROW_EXCEPTION_IF(!object);
296:   return adopt_local(static_cast<R>(object));
297: }
298: 
299: inline jclass JClass::self() const noexcept {
300:   return static_cast<jclass>(JObject::self());
````
- EN: Implements callable logic such as `JClass::getField`, `JClass::getStaticField`, `JClass::newObject`.
- CN: 实现可调用逻辑，例如 `JClass::getField`, `JClass::getStaticField`, `JClass::newObject`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````cpp
301: }
302: 
303: inline void registerNatives(const char* name, std::initializer_list<JNINativeMethod> methods) {
304:   findClassLocal(name)->registerNatives(methods);
305: }
306: 
307: 
308: // jstring /////////////////////////////////////////////////////////////////////////////////////////
309: 
310: inline local_ref<JString> make_jstring(const std::string& utf8) {
311:   return make_jstring(utf8.c_str());
312: }
313: 
314: namespace detail {
315: // convert to std::string from jstring
316: template <>
317: struct Convert<std::string> {
318:   typedef jstring jniType;
319:   static std::string fromJni(jniType t) {
320:     return wrap_alias(t)->toStdString();
321:   }
322:   static jniType toJniRet(const std::string& t) {
323:     return make_jstring(t).release();
324:   }
325:   static local_ref<JString> toCall(const std::string& t) {
326:     return make_jstring(t);
327:   }
328: };
329: 
330: // convert return from const char*
331: template <>
332: struct Convert<const char*> {
333:   typedef jstring jniType;
334:   // no automatic synthesis of const char*.  (It can't be freed.)
335:   static jniType toJniRet(const char* t) {
336:     return make_jstring(t).release();
337:   }
338:   static local_ref<JString> toCall(const char* t) {
339:     return make_jstring(t);
340:   }
341: };
342: }
343: 
344: // jtypeArray //////////////////////////////////////////////////////////////////////////////////////
345: 
346: namespace detail {
347: inline size_t JArray::size() const noexcept {
348:   const auto env = Environment::current();
349:   return env->GetArrayLength(self());
350: }
351: }
352: 
353: namespace detail {
354: template<typename Target>
355: inline ElementProxy<Target>::ElementProxy(
356:     Target* target,
357:     size_t idx)
358:     : target_{target}, idx_{idx} {}
359: 
360: template<typename Target>
````
- EN: Declares or extends types including `Convert`.
- CN: 声明或扩展类型，包括 `Convert`。
- EN: Implements callable logic such as `registerNatives`, `make_jstring`, `fromJni`, `toJniRet`.
- CN: 实现可调用逻辑，例如 `registerNatives`, `make_jstring`, `fromJni`, `toJniRet`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-420
````cpp
361: inline ElementProxy<Target>& ElementProxy<Target>::operator=(const T& o) {
362:   target_->setElement(idx_, o);
363:   return *this;
364: }
365: 
366: template<typename Target>
367: inline ElementProxy<Target>& ElementProxy<Target>::operator=(alias_ref<typename Target::javaentry>& o) {
368:   target_->setElement(idx_, o.get());
369:   return *this;
370: }
371: 
372: template<typename Target>
373: inline ElementProxy<Target>& ElementProxy<Target>::operator=(alias_ref<typename Target::javaentry>&& o) {
374:   target_->setElement(idx_, o.get());
375:   return *this;
376: }
377: 
378: template<typename Target>
379: inline ElementProxy<Target>& ElementProxy<Target>::operator=(const ElementProxy<Target>& o) {
380:   auto src = o.target_->getElement(o.idx_);
381:   target_->setElement(idx_, src.get());
382:   return *this;
383: }
384: 
385: template<typename Target>
386: inline ElementProxy<Target>::ElementProxy::operator const local_ref<typename Target::javaentry> () const {
387:   return target_->getElement(idx_);
388: }
389: 
390: template<typename Target>
391: inline ElementProxy<Target>::ElementProxy::operator local_ref<typename Target::javaentry> () {
392:   return target_->getElement(idx_);
393: }
394: }
395: 
396: template<typename T>
397: auto JArrayClass<T>::newArray(size_t size) -> local_ref<javaobject> {
398:   static const auto elementClass = findClassStatic(jtype_traits<T>::kBaseName.c_str());
399:   const auto env = Environment::current();
400:   auto rawArray = env->NewObjectArray(size, elementClass.get(), nullptr);
401:   FACEBOOK_JNI_THROW_EXCEPTION_IF(!rawArray);
402:   return adopt_local(static_cast<javaobject>(rawArray));
403: }
404: 
405: template<typename T>
406: inline void JArrayClass<T>::setElement(size_t idx, T value) {
407:   const auto env = Environment::current();
408:   env->SetObjectArrayElement(this->self(), idx, detail::toPlainJniReference(value));
409: }
410: 
411: template<typename T>
412: inline local_ref<T> JArrayClass<T>::getElement(size_t idx) {
413:   const auto env = Environment::current();
414:   auto rawElement = env->GetObjectArrayElement(this->self(), idx);
415:   return adopt_local(static_cast<JniType<T>>(rawElement));
416: }
417: 
418: template<typename T>
419: inline detail::ElementProxy<JArrayClass<T>> JArrayClass<T>::operator[](size_t index) {
420:   return detail::ElementProxy<JArrayClass<T>>(this, index);
````
- EN: Implements callable logic such as `setElement`, `getElement`.
- CN: 实现可调用逻辑，例如 `setElement`, `getElement`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 421-480
````cpp
421: }
422: 
423: template<typename T>
424: local_ref<typename JArrayClass<T>::javaobject> adopt_local_array(jobjectArray ref) {
425:   return adopt_local(static_cast<typename JArrayClass<T>::javaobject>(ref));
426: }
427: 
428: // jarray /////////////////////////////////////////////////////////////////////////////////////////
429: 
430: template <typename JArrayType>
431: auto JPrimitiveArray<JArrayType>::getRegion(jsize start, jsize length)
432:     -> std::unique_ptr<T[]> {
433:   auto buf = std::unique_ptr<T[]>{new T[length]};
434:   getRegion(start, length, buf.get());
435:   return buf;
436: }
437: 
438: template <typename JArrayType>
439: auto JPrimitiveArray<JArrayType>::pin() -> PinnedPrimitiveArray<T, PinnedArrayAlloc<T>> {
440:   return PinnedPrimitiveArray<T, PinnedArrayAlloc<T>>{this->self(), 0, 0};
441: }
442: 
443: template <typename JArrayType>
444: auto JPrimitiveArray<JArrayType>::pinRegion(jsize start, jsize length)
445:     -> PinnedPrimitiveArray<T, PinnedRegionAlloc<T>> {
446:   return PinnedPrimitiveArray<T, PinnedRegionAlloc<T>>{this->self(), start, length};
447: }
448: 
449: template <typename JArrayType>
450: auto JPrimitiveArray<JArrayType>::pinCritical()
451:     -> PinnedPrimitiveArray<T, PinnedCriticalAlloc<T>> {
452:   return PinnedPrimitiveArray<T, PinnedCriticalAlloc<T>>{this->self(), 0, 0};
453: }
454: 
455: template <typename T>
456: class PinnedArrayAlloc {
457:  public:
458:   static void allocate(
459:       alias_ref<typename jtype_traits<T>::array_type> array,
460:       jsize start,
461:       jsize length,
462:       T** elements,
463:       size_t* size,
464:       jboolean* isCopy) {
465:     (void) start;
466:     (void) length;
467:     *elements = array->getElements(isCopy);
468:     *size = array->size();
469:   }
470:   static void release(
471:       alias_ref<typename jtype_traits<T>::array_type> array,
472:       T* elements,
473:       jint start,
474:       jint size,
475:       jint mode) {
476:     (void) start;
477:     (void) size;
478:     array->releaseElements(elements, mode);
479:   }
480: };
````
- EN: Declares or extends types including `PinnedArrayAlloc`.
- CN: 声明或扩展类型，包括 `PinnedArrayAlloc`。
- EN: Implements callable logic such as `adopt_local_array`, `allocate`, `release`.
- CN: 实现可调用逻辑，例如 `adopt_local_array`, `allocate`, `release`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 481-540
````cpp
481: 
482: template <typename T>
483: class PinnedCriticalAlloc {
484:  public:
485:   static void allocate(
486:       alias_ref<typename jtype_traits<T>::array_type> array,
487:       jsize start,
488:       jsize length,
489:       T** elements,
490:       size_t* size,
491:       jboolean* isCopy) {
492:     (void)start;
493:     (void)length;
494:     const auto env = Environment::current();
495:     *elements = static_cast<T*>(env->GetPrimitiveArrayCritical(array.get(), isCopy));
496:     FACEBOOK_JNI_THROW_EXCEPTION_IF(!elements);
497:     *size = array->size();
498:   }
499:   static void release(
500:       alias_ref<typename jtype_traits<T>::array_type> array,
501:       T* elements,
502:       jint start,
503:       jint size,
504:       jint mode) {
505:     (void)start;
506:     (void)size;
507:     const auto env = Environment::current();
508:     env->ReleasePrimitiveArrayCritical(array.get(), elements, mode);
509:   }
510: };
511: 
512: template <typename T>
513: class PinnedRegionAlloc {
514:  public:
515:   static void allocate(
516:       alias_ref<typename jtype_traits<T>::array_type> array,
517:       jsize start,
518:       jsize length,
519:       T** elements,
520:       size_t* size,
521:       jboolean* isCopy) {
522:     auto buf = array->getRegion(start, length);
523:     FACEBOOK_JNI_THROW_EXCEPTION_IF(!buf);
524:     *elements = buf.release();
525:     *size = length;
526:     *isCopy = true;
527:   }
528:   static void release(
529:       alias_ref<typename jtype_traits<T>::array_type> array,
530:       T* elements,
531:       jint start,
532:       jint size,
533:       jint mode) {
534:     std::unique_ptr<T[]> holder;
535:     if (mode == 0 || mode == JNI_ABORT) {
536:       holder.reset(elements);
537:     }
538:     if (mode == 0 || mode == JNI_COMMIT) {
539:       array->setRegion(start, size, elements);
540:     }
````
- EN: Declares or extends types including `PinnedCriticalAlloc`, `PinnedRegionAlloc`.
- CN: 声明或扩展类型，包括 `PinnedCriticalAlloc`, `PinnedRegionAlloc`。
- EN: Implements callable logic such as `allocate`, `release`.
- CN: 实现可调用逻辑，例如 `allocate`, `release`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 541-600
````cpp
541:   }
542: };
543: 
544: // PinnedPrimitiveArray ///////////////////////////////////////////////////////////////////////////
545: 
546: template<typename T, typename Alloc>
547: PinnedPrimitiveArray<T, Alloc>::PinnedPrimitiveArray(PinnedPrimitiveArray&& o) {
548:   *this = std::move(o);
549: }
550: 
551: template<typename T, typename Alloc>
552: PinnedPrimitiveArray<T, Alloc>&
553: PinnedPrimitiveArray<T, Alloc>::operator=(PinnedPrimitiveArray&& o) {
554:   if (array_) {
555:     release();
556:   }
557:   array_ = std::move(o.array_);
558:   elements_ = o.elements_;
559:   isCopy_ = o.isCopy_;
560:   size_ = o.size_;
561:   start_ = o.start_;
562:   o.clear();
563:   return *this;
564: }
565: 
566: template<typename T, typename Alloc>
567: T* PinnedPrimitiveArray<T, Alloc>::get() {
568:   return elements_;
569: }
570: 
571: template<typename T, typename Alloc>
572: inline void PinnedPrimitiveArray<T, Alloc>::release() {
573:   releaseImpl(0);
574:   clear();
575: }
576: 
577: template<typename T, typename Alloc>
578: inline void PinnedPrimitiveArray<T, Alloc>::commit() {
579:   releaseImpl(JNI_COMMIT);
580: }
581: 
582: template<typename T, typename Alloc>
583: inline void PinnedPrimitiveArray<T, Alloc>::abort() {
584:   releaseImpl(JNI_ABORT);
585:   clear();
586: }
587: 
588: template <typename T, typename Alloc>
589: inline void PinnedPrimitiveArray<T, Alloc>::releaseImpl(jint mode) {
590:   FACEBOOK_JNI_THROW_EXCEPTION_IF(array_.get() == nullptr);
591:   Alloc::release(array_, elements_, start_, size_, mode);
592: }
593: 
594: template<typename T, typename Alloc>
595: inline void PinnedPrimitiveArray<T, Alloc>::clear() noexcept {
596:   array_ = nullptr;
597:   elements_ = nullptr;
598:   isCopy_ = false;
599:   start_ = 0;
600:   size_ = 0;
````
- EN: Implements callable logic such as `PinnedPrimitiveArray`, `get`, `release`, `commit`.
- CN: 实现可调用逻辑，例如 `PinnedPrimitiveArray`, `get`, `release`, `commit`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 601-645
````cpp
601: }
602: 
603: template<typename T, typename Alloc>
604: inline T& PinnedPrimitiveArray<T, Alloc>::operator[](size_t index) {
605:   FACEBOOK_JNI_THROW_EXCEPTION_IF(elements_ == nullptr);
606:   return elements_[index];
607: }
608: 
609: template<typename T, typename Alloc>
610: inline bool PinnedPrimitiveArray<T, Alloc>::isCopy() const noexcept {
611:   return isCopy_ == JNI_TRUE;
612: }
613: 
614: template<typename T, typename Alloc>
615: inline size_t PinnedPrimitiveArray<T, Alloc>::size() const noexcept {
616:   return size_;
617: }
618: 
619: template<typename T, typename Alloc>
620: inline PinnedPrimitiveArray<T, Alloc>::~PinnedPrimitiveArray() noexcept {
621:   if (elements_) {
622:     release();
623:   }
624: }
625: 
626: template<typename T, typename Alloc>
627: inline PinnedPrimitiveArray<T, Alloc>::PinnedPrimitiveArray(alias_ref<typename jtype_traits<T>::array_type> array, jint start, jint length) {
628:   array_ = array;
629:   start_ = start;
630:   Alloc::allocate(array, start, length, &elements_, &size_, &isCopy_);
631: }
632: 
633: template<typename T, typename Base, typename JType>
634: inline alias_ref<JClass> JavaClass<T, Base, JType>::javaClassStatic() {
635:   static auto cls = findClassStatic(jtype_traits<typename T::javaobject>::kBaseName.c_str());
636:   return cls;
637: }
638: 
639: template<typename T, typename Base, typename JType>
640: inline local_ref<JClass> JavaClass<T, Base, JType>::javaClassLocal() {
641:   std::string className(jtype_traits<typename T::javaobject>::kBaseName.c_str());
642:   return findClassLocal(className.c_str());
643: }
644: 
645: }}
````
- EN: Implements callable logic such as `PinnedPrimitiveArray`, `javaClassStatic`, `javaClassLocal`.
- CN: 实现可调用逻辑，例如 `PinnedPrimitiveArray`, `javaClassStatic`, `javaClassLocal`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `MonitorLock` / 符号 `MonitorLock`
- Symbol `Convert` / 符号 `Convert`
- Symbol `PinnedArrayAlloc` / 符号 `PinnedArrayAlloc`
- Symbol `PinnedCriticalAlloc` / 符号 `PinnedCriticalAlloc`

## Dependencies / 依赖关系
- C/C++ includes: `string.h`, `type_traits`, `stdlib.h`, `Common.h`, `Exceptions.h`, `Meta.h`, `MetaConvert.h`
- C/C++ 头文件: `string.h`, `type_traits`, `stdlib.h`, `Common.h`, `Exceptions.h`, `Meta.h`, `MetaConvert.h`
