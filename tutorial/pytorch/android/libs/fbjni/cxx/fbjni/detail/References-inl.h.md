# References-inl.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/References-inl.h`
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
19: #include <new>
20: #include "CoreClasses.h"
21: 
22: namespace facebook {
23: namespace jni {
24: 
25: template<typename T>
26: inline enable_if_t<IsPlainJniReference<T>(), T> getPlainJniReference(T ref) {
27:   return ref;
28: }
29: 
30: template<typename T>
31: inline JniType<T> getPlainJniReference(alias_ref<T> ref) {
32:   return ref.get();
33: }
34: 
35: template<typename T, typename A>
36: inline JniType<T> getPlainJniReference(const base_owned_ref<T, A>& ref) {
37:   return ref.get();
38: }
39: 
40: 
41: namespace detail {
42: template <typename Repr>
43: struct ReprAccess {
44:   using javaobject = JniType<Repr>;
45:   static void set(Repr& repr, javaobject obj) noexcept {
46:     repr.JObjectBase::set(obj);
47:   }
48:   static javaobject get(const Repr& repr) {
49:     return static_cast<javaobject>(repr.JObjectBase::get());
50:   }
51: };
52: 
53: namespace {
54: template <typename Repr>
55: void StaticAssertValidRepr() noexcept {
56:   static_assert(std::is_base_of<JObject, Repr>::value,
57:       "A smart ref representation must be derived from JObject.");
58:   static_assert(IsPlainJniReference<JniType<Repr>>(), "T must be a JNI reference");
59:   static_assert(sizeof(Repr) == sizeof(JObjectBase), "");
60:   static_assert(alignof(Repr) == alignof(JObjectBase), "");
````
- EN: Pulls in native headers such as `new`, `CoreClasses.h`.
- CN: 引入原生头文件，例如 `new`, `CoreClasses.h`。
- EN: Declares or extends types including `ReprAccess`.
- CN: 声明或扩展类型，包括 `ReprAccess`。
- EN: Implements callable logic such as `getPlainJniReference`, `get`.
- CN: 实现可调用逻辑，例如 `getPlainJniReference`, `get`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````cpp
 61: }
 62: }
 63: 
 64: template <typename Repr>
 65: ReprStorage<Repr>::ReprStorage(JniType<Repr> obj) noexcept {
 66:   StaticAssertValidRepr<Repr>();
 67:   set(obj);
 68: }
 69: 
 70: template <typename Repr>
 71: void ReprStorage<Repr>::set(JniType<Repr> obj) noexcept {
 72:   new (&storage_) Repr;
 73:   ReprAccess<Repr>::set(get(), obj);
 74: }
 75: 
 76: template <typename Repr>
 77: Repr& ReprStorage<Repr>::get() noexcept {
 78:   return *reinterpret_cast<Repr*>(&storage_);
 79: }
 80: 
 81: template <typename Repr>
 82: const Repr& ReprStorage<Repr>::get() const noexcept {
 83:   return *reinterpret_cast<const Repr*>(&storage_);
 84: }
 85: 
 86: template <typename Repr>
 87: JniType<Repr> ReprStorage<Repr>::jobj() const noexcept {
 88:   return ReprAccess<Repr>::get(get());
 89: }
 90: 
 91: template <typename Repr>
 92: void ReprStorage<Repr>::swap(ReprStorage& other) noexcept {
 93:   StaticAssertValidRepr<Repr>();
 94:   using std::swap;
 95:   swap(get(), other.get());
 96: }
 97: 
 98: inline void JObjectBase::set(jobject reference) noexcept {
 99:   this_ = reference;
100: }
101: 
102: inline jobject JObjectBase::get() const noexcept {
103:   return this_;
104: }
105: 
106: template<typename T, typename Alloc>
107: enable_if_t<IsNonWeakReference<T>(), plain_jni_reference_t<T>> make_ref(const T& reference) {
108:   auto old_reference = getPlainJniReference(reference);
109:   if (!old_reference) {
110:     return nullptr;
111:   }
112: 
113:   auto ref = Alloc{}.newReference(old_reference);
114:   if (!ref) {
115:     // Note that we end up here if we pass a weak ref that refers to a collected object.
116:     // Thus, it's hard to come up with a reason why this function should be used with
117:     // weak references.
118:     throw std::bad_alloc{};
119:   }
120: 
````
- EN: Implements callable logic such as `make_ref`.
- CN: 实现可调用逻辑，例如 `make_ref`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````cpp
121:   return static_cast<plain_jni_reference_t<T>>(ref);
122: }
123: 
124: } // namespace detail
125: 
126: template<typename T>
127: inline local_ref<T> adopt_local(T ref) noexcept {
128:   static_assert(IsPlainJniReference<T>(), "T must be a plain jni reference");
129:   return local_ref<T>{ref};
130: }
131: 
132: template<typename T>
133: inline global_ref<T> adopt_global(T ref) noexcept {
134:   static_assert(IsPlainJniReference<T>(), "T must be a plain jni reference");
135:   return global_ref<T>{ref};
136: }
137: 
138: template<typename T>
139: inline weak_ref<T> adopt_weak_global(T ref) noexcept {
140:   static_assert(IsPlainJniReference<T>(), "T must be a plain jni reference");
141:   return weak_ref<T>{ref};
142: }
143: 
144: 
145: template<typename T>
146: inline enable_if_t<IsPlainJniReference<T>(), alias_ref<T>> wrap_alias(T ref) noexcept {
147:   return alias_ref<T>(ref);
148: }
149: 
150: 
151: template<typename T>
152: enable_if_t<IsPlainJniReference<T>(), alias_ref<T>> wrap_alias(T ref) noexcept;
153: 
154: 
155: template<typename T>
156: enable_if_t<IsNonWeakReference<T>(), local_ref<plain_jni_reference_t<T>>>
157: make_local(const T& ref) {
158:   return adopt_local(detail::make_ref<T, LocalReferenceAllocator>(ref));
159: }
160: 
161: template<typename T>
162: enable_if_t<IsNonWeakReference<T>(), global_ref<plain_jni_reference_t<T>>>
163: make_global(const T& ref) {
164:   return adopt_global(detail::make_ref<T, GlobalReferenceAllocator>(ref));
165: }
166: 
167: template<typename T>
168: enable_if_t<IsNonWeakReference<T>(), weak_ref<plain_jni_reference_t<T>>>
169: make_weak(const T& ref) {
170:   return adopt_weak_global(detail::make_ref<T, WeakGlobalReferenceAllocator>(ref));
171: }
172: 
173: template<typename T1, typename T2>
174: inline enable_if_t<IsNonWeakReference<T1>() && IsNonWeakReference<T2>(), bool>
175: operator==(const T1& a, const T2& b) {
176:   return isSameObject(getPlainJniReference(a), getPlainJniReference(b));
177: }
178: 
179: template<typename T1, typename T2>
180: inline enable_if_t<IsNonWeakReference<T1>() && IsNonWeakReference<T2>(), bool>
````
- EN: Implements callable logic such as `make_local`, `make_global`, `make_weak`.
- CN: 实现可调用逻辑，例如 `make_local`, `make_global`, `make_weak`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````cpp
181: operator!=(const T1& a, const T2& b) {
182:   return !(a == b);
183: }
184: 
185: template<typename T1>
186: inline enable_if_t<IsNonWeakReference<T1>(), bool>
187: operator==(const T1& a, std::nullptr_t) {
188:   return getPlainJniReference(a) == nullptr;
189: }
190: 
191: template<typename T1>
192: inline enable_if_t<IsNonWeakReference<T1>(), bool>
193: operator==(std::nullptr_t, const T1& a) {
194:   return nullptr == getPlainJniReference(a);
195: }
196: 
197: template<typename T1>
198: inline enable_if_t<IsNonWeakReference<T1>(), bool>
199: operator!=(const T1& a, std::nullptr_t) {
200:   return !(a == nullptr);
201: }
202: 
203: template<typename T1>
204: inline enable_if_t<IsNonWeakReference<T1>(), bool>
205: operator!=(std::nullptr_t, const T1& a) {
206:   return !(nullptr == getPlainJniReference(a));
207: }
208: 
209: // base_owned_ref ///////////////////////////////////////////////////////////////////////
210: 
211: template<typename T, typename Alloc>
212: inline base_owned_ref<T, Alloc>::base_owned_ref() noexcept
213:   : base_owned_ref(nullptr)
214: {}
215: 
216: template<typename T, typename Alloc>
217: inline base_owned_ref<T, Alloc>::base_owned_ref(std::nullptr_t t) noexcept
218:   : base_owned_ref(static_cast<javaobject>(nullptr))
219: {
220:   (void)t;
221: }
222: 
223: template<typename T, typename Alloc>
224: inline base_owned_ref<T, Alloc>::base_owned_ref(const base_owned_ref& other)
225:   : storage_{static_cast<javaobject>(Alloc{}.newReference(other.get()))}
226: {}
227: 
228: template<typename T, typename Alloc>
229: template<typename U>
230: inline base_owned_ref<T, Alloc>::base_owned_ref(const base_owned_ref<U, Alloc>& other)
231:   : storage_{static_cast<javaobject>(Alloc{}.newReference(other.get()))}
232: {
233:   static_assert(std::is_convertible<JniType<U>, javaobject>::value, "");
234: }
235: 
236: template<typename T, typename Alloc>
237: inline facebook::jni::base_owned_ref<T, Alloc>::base_owned_ref(
238:     javaobject reference) noexcept
239:   : storage_(reference) {
240:   assert(Alloc{}.verifyReference(reference));
````
- EN: Implements callable logic such as `base_owned_ref`.
- CN: 实现可调用逻辑，例如 `base_owned_ref`。
- EN: Includes validation, assertions, or defensive checks.
- CN: 包含校验、断言或防御性检查。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````cpp
241:   internal::dbglog("New wrapped ref=%p this=%p", get(), this);
242: }
243: 
244: template<typename T, typename Alloc>
245: inline base_owned_ref<T, Alloc>::base_owned_ref(
246:     base_owned_ref<T, Alloc>&& other) noexcept
247:   : storage_(other.get()) {
248:   internal::dbglog("New move from ref=%p other=%p", other.get(), &other);
249:   internal::dbglog("New move to ref=%p this=%p", get(), this);
250:   // JObject is a simple type and does not support move semantics so we explicitly
251:   // clear other
252:   other.set(nullptr);
253: }
254: 
255: template<typename T, typename Alloc>
256: template<typename U>
257: base_owned_ref<T, Alloc>::base_owned_ref(base_owned_ref<U, Alloc>&& other) noexcept
258:   : storage_(other.get()) {
259:   internal::dbglog("New move from ref=%p other=%p", other.get(), &other);
260:   internal::dbglog("New move to ref=%p this=%p", get(), this);
261:   // JObject is a simple type and does not support move semantics so we explicitly
262:   // clear other
263:   other.set(nullptr);
264: }
265: 
266: template<typename T, typename Alloc>
267: inline base_owned_ref<T, Alloc>::~base_owned_ref() noexcept {
268:   reset();
269:   internal::dbglog("Ref destruct ref=%p this=%p", get(), this);
270: }
271: 
272: template<typename T, typename Alloc>
273: inline auto base_owned_ref<T, Alloc>::release() noexcept -> javaobject {
274:   auto value = get();
275:   internal::dbglog("Ref release ref=%p this=%p", value, this);
276:   set(nullptr);
277:   return value;
278: }
279: 
280: template<typename T, typename Alloc>
281: inline void base_owned_ref<T,Alloc>::reset() noexcept {
282:   reset(nullptr);
283: }
284: 
285: template<typename T, typename Alloc>
286: inline void base_owned_ref<T,Alloc>::reset(javaobject reference) noexcept {
287:   if (get()) {
288:     assert(Alloc{}.verifyReference(reference));
289:     Alloc{}.deleteReference(get());
290:   }
291:   set(reference);
292: }
293: 
294: template<typename T, typename Alloc>
295: inline auto base_owned_ref<T, Alloc>::get() const noexcept -> javaobject {
296:   return storage_.jobj();
297: }
298: 
299: template<typename T, typename Alloc>
300: inline void base_owned_ref<T, Alloc>::set(javaobject ref) noexcept {
````
- EN: Implements callable logic such as `base_owned_ref`.
- CN: 实现可调用逻辑，例如 `base_owned_ref`。
- EN: Includes validation, assertions, or defensive checks.
- CN: 包含校验、断言或防御性检查。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````cpp
301:   storage_.set(ref);
302: }
303: 
304: 
305: // weak_ref ///////////////////////////////////////////////////////////////////////
306: 
307: template<typename T>
308: inline weak_ref<T>& weak_ref<T>::operator=(
309:     const weak_ref& other) {
310:   auto otherCopy = other;
311:   swap(*this, otherCopy);
312:   return *this;
313: }
314: 
315: template<typename T>
316: inline weak_ref<T>& weak_ref<T>::operator=(
317:     weak_ref<T>&& other) noexcept {
318:   internal::dbglog("Op= move ref=%p this=%p oref=%p other=%p",
319:       get(), this, other.get(), &other);
320:   reset(other.release());
321:   return *this;
322: }
323: 
324: template<typename T>
325: local_ref<T> weak_ref<T>::lockLocal() const {
326:   return adopt_local(
327:       static_cast<javaobject>(LocalReferenceAllocator{}.newReference(get())));
328: }
329: 
330: template<typename T>
331: global_ref<T> weak_ref<T>::lockGlobal() const {
332:   return adopt_global(
333:       static_cast<javaobject>(GlobalReferenceAllocator{}.newReference(get())));
334: }
335: 
336: template<typename T>
337: inline void swap(
338:     weak_ref<T>& a,
339:     weak_ref<T>& b) noexcept {
340:   internal::dbglog("Ref swap a.ref=%p a=%p b.ref=%p b=%p",
341:       a.get(), &a, b.get(), &b);
342:   a.storage_.swap(b.storage_);
343: }
344: 
345: 
346: // basic_strong_ref ////////////////////////////////////////////////////////////////////////////
347: 
348: template<typename T, typename Alloc>
349: inline basic_strong_ref<T, Alloc>& basic_strong_ref<T, Alloc>::operator=(
350:     const basic_strong_ref& other) {
351:   auto otherCopy = other;
352:   swap(*this, otherCopy);
353:   return *this;
354: }
355: 
356: template<typename T, typename Alloc>
357: inline basic_strong_ref<T, Alloc>& basic_strong_ref<T, Alloc>::operator=(
358:     basic_strong_ref<T, Alloc>&& other) noexcept {
359:   internal::dbglog("Op= move ref=%p this=%p oref=%p other=%p",
360:       get(), this, other.get(), &other);
````
- EN: Implements callable logic such as `lockLocal`, `lockGlobal`.
- CN: 实现可调用逻辑，例如 `lockLocal`, `lockGlobal`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-420
````cpp
361:   reset(other.release());
362:   return *this;
363: }
364: 
365: template<typename T, typename Alloc>
366: inline alias_ref<T> basic_strong_ref<T, Alloc>::releaseAlias() noexcept {
367:   return wrap_alias(release());
368: }
369: 
370: template<typename T, typename Alloc>
371: inline basic_strong_ref<T, Alloc>::operator bool() const noexcept {
372:   return get() != nullptr;
373: }
374: 
375: template<typename T, typename Alloc>
376: inline auto basic_strong_ref<T, Alloc>::operator->() noexcept -> Repr* {
377:   return &storage_.get();
378: }
379: 
380: template<typename T, typename Alloc>
381: inline auto basic_strong_ref<T, Alloc>::operator->() const noexcept -> const Repr* {
382:   return &storage_.get();
383: }
384: 
385: template<typename T, typename Alloc>
386: inline auto basic_strong_ref<T, Alloc>::operator*() noexcept -> Repr& {
387:   return storage_.get();
388: }
389: 
390: template<typename T, typename Alloc>
391: inline auto basic_strong_ref<T, Alloc>::operator*() const noexcept -> const Repr& {
392:   return storage_.get();
393: }
394: 
395: template<typename T, typename Alloc>
396: inline void swap(
397:     basic_strong_ref<T, Alloc>& a,
398:     basic_strong_ref<T, Alloc>& b) noexcept {
399:   internal::dbglog("Ref swap a.ref=%p a=%p b.ref=%p b=%p",
400:       a.get(), &a, b.get(), &b);
401:   using std::swap;
402:   a.storage_.swap(b.storage_);
403: }
404: 
405: 
406: // alias_ref //////////////////////////////////////////////////////////////////////////////
407: 
408: template<typename T>
409: inline alias_ref<T>::alias_ref() noexcept
410:   : storage_{nullptr}
411: {}
412: 
413: template<typename T>
414: inline alias_ref<T>::alias_ref(std::nullptr_t) noexcept
415:   : storage_{nullptr}
416: {}
417: 
418: template<typename T>
419: inline alias_ref<T>::alias_ref(const alias_ref& other) noexcept
420:   : storage_{other.get()}
````
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 421-480
````cpp
421: {}
422: 
423: template<typename T>
424: inline alias_ref<T>::alias_ref(javaobject ref) noexcept
425:   : storage_(ref) {
426:   assert(
427:       LocalReferenceAllocator{}.verifyReference(ref) ||
428:       GlobalReferenceAllocator{}.verifyReference(ref));
429: }
430: 
431: template<typename T>
432: template<typename TOther, typename /* for SFINAE */>
433: inline alias_ref<T>::alias_ref(alias_ref<TOther> other) noexcept
434:   : storage_{other.get()}
435: {}
436: 
437: template<typename T>
438: template<typename TOther, typename AOther, typename /* for SFINAE */>
439: inline alias_ref<T>::alias_ref(const basic_strong_ref<TOther, AOther>& other) noexcept
440:   : storage_{other.get()}
441: {}
442: 
443: template<typename T>
444: inline alias_ref<T>& alias_ref<T>::operator=(alias_ref other) noexcept {
445:   swap(*this, other);
446:   return *this;
447: }
448: 
449: template<typename T>
450: inline alias_ref<T>::operator bool() const noexcept {
451:   return get() != nullptr;
452: }
453: 
454: template<typename T>
455: inline auto facebook::jni::alias_ref<T>::get() const noexcept -> javaobject {
456:   return storage_.jobj();
457: }
458: 
459: template<typename T>
460: inline auto alias_ref<T>::operator->() noexcept -> Repr* {
461:   return &(**this);
462: }
463: 
464: template<typename T>
465: inline auto alias_ref<T>::operator->() const noexcept -> const Repr* {
466:   return &(**this);
467: }
468: 
469: template<typename T>
470: inline auto alias_ref<T>::operator*() noexcept -> Repr& {
471:   return storage_.get();
472: }
473: 
474: template<typename T>
475: inline auto alias_ref<T>::operator*() const noexcept -> const Repr& {
476:   return storage_.get();
477: }
478: 
479: template<typename T>
480: inline void alias_ref<T>::set(javaobject ref) noexcept {
````
- EN: Implements callable logic such as `alias_ref`.
- CN: 实现可调用逻辑，例如 `alias_ref`。
- EN: Includes validation, assertions, or defensive checks.
- CN: 包含校验、断言或防御性检查。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 481-540
````cpp
481:   storage_.set(ref);
482: }
483: 
484: template<typename T>
485: inline void swap(alias_ref<T>& a, alias_ref<T>& b) noexcept {
486:   a.storage_.swap(b.storage_);
487: }
488: 
489: // Could reduce code duplication by using a pointer-to-function
490: // template argument.  I'm not sure whether that would make the code
491: // more maintainable (DRY), or less (too clever/confusing.).
492: template<typename T, typename U>
493: enable_if_t<IsPlainJniReference<JniType<T>>(), local_ref<T>>
494: static_ref_cast(const local_ref<U>& ref) noexcept
495: {
496:   JniType<T> p = static_cast<JniType<T>>(ref.get());
497:   return make_local(p);
498: }
499: 
500: template<typename T, typename U>
501: enable_if_t<IsPlainJniReference<JniType<T>>(), global_ref<T>>
502: static_ref_cast(const global_ref<U>& ref) noexcept
503: {
504:   JniType<T> p = static_cast<JniType<T>>(ref.get());
505:   return make_global(p);
506: }
507: 
508: template<typename T, typename U>
509: enable_if_t<IsPlainJniReference<JniType<T>>(), alias_ref<T>>
510: static_ref_cast(const alias_ref<U>& ref) noexcept
511: {
512:   JniType<T> p = static_cast<JniType<T>>(ref.get());
513:   return wrap_alias(p);
514: }
515: 
516: template<typename T, typename RefType>
517: auto dynamic_ref_cast(const RefType& ref) ->
518: enable_if_t<IsPlainJniReference<JniType<T>>(), decltype(static_ref_cast<T>(ref))>
519: {
520:   if (!ref) {
521:     return decltype(static_ref_cast<T>(ref))();
522:   }
523: 
524:   static alias_ref<jclass> target_class = findClassStatic(jtype_traits<T>::kBaseName.c_str());
525:   if (!target_class) {
526:     throwNewJavaException("java/lang/ClassCastException",
527:                           "Could not find class %s.",
528:                           jtype_traits<T>::kBaseName.c_str());
529: 
530:   }
531: 
532:   local_ref<jclass> source_class = ref->getClass();
533: 
534:   if (!target_class->isAssignableFrom(source_class)) {
535:     throwNewJavaException("java/lang/ClassCastException",
536:                           "Tried to cast from %s to %s.",
537:                           source_class->toString().c_str(),
538:                           jtype_traits<T>::kBaseName.c_str());
539:   }
540: 
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 541-544
````cpp
541:   return static_ref_cast<T>(ref);
542: }
543: 
544: }}
````
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `ReprAccess` / 符号 `ReprAccess`
- Symbol `getPlainJniReference` / 符号 `getPlainJniReference`
- Symbol `get` / 符号 `get`
- Symbol `make_ref` / 符号 `make_ref`

## Dependencies / 依赖关系
- C/C++ includes: `new`, `CoreClasses.h`
- C/C++ 头文件: `new`, `CoreClasses.h`
