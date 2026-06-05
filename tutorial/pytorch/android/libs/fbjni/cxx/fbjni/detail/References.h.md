# References.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/References.h`
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
17: 
18: /** @file References.h
19:  *
20:  * Functionality similar to smart pointers, but for references into the VM. Four main reference
21:  * types are provided: local_ref, global_ref, weak_ref, and alias_ref. All are generic
22:  * templates that and refer to objects in the jobject hierarchy. The type of the referred objects
23:  * are specified using the template parameter. All reference types except alias_ref own their
24:  * underlying reference, just as a std smart pointer owns the underlying raw pointer. In the context
25:  * of std smart pointers, these references behave like unique_ptr, and have basically the same
26:  * interface. Thus, when the reference is destructed, the plain JNI reference, i.e. the underlying
27:  * JNI reference (like the parameters passed directly to JNI functions), is released. The alias
28:  * references provides no ownership and is a simple wrapper for plain JNI references.
29:  *
30:  * All but the weak references provides access to the underlying object using dereferencing, and a
31:  * get() method. It is also possible to convert these references to booleans to test for nullity.
32:  * To access the underlying object of a weak reference, the reference must either be released, or
33:  * the weak reference can be used to create a local or global reference.
34:  *
35:  * An owning reference is created either by moving the reference from an existing owned reference,
36:  * by copying an existing owned reference (which creates a new underlying reference), by using the
37:  * default constructor which initialize the reference to nullptr, or by using a helper function. The
38:  * helper function exist in two flavors: make_XXX or adopt_XXX.
39:  *
40:  * Adopting takes a plain JNI reference and wrap it in an owned reference. It takes ownership of the
41:  * plain JNI reference so be sure that no one else owns the reference when you adopt it, and make
42:  * sure that you know what kind of reference it is.
43:  *
44:  * New owned references can be created from existing plain JNI references, alias references, local
45:  * references, and global references (i.e. non-weak references) using the make_local, make_global,
46:  * and make_weak functions.
47:  *
48:  * Alias references can be implicitly initialized using global, local and plain JNI references using
49:  * the wrap_alias function. Here, we don't assume ownership of the passed-in reference, but rather
50:  * create a separate reference that we do own, leaving the passed-in reference to its fate.
51:  *
52:  * Similar rules apply for assignment. An owned reference can be copy or move assigned using a smart
53:  * reference of the same type. In the case of copy assignment a new reference is created. Alias
54:  * reference can also be assigned new values, but since they are simple wrappers of plain JNI
55:  * references there is no move semantics involved.
56:  *
57:  * Alias references are special in that they do not own the object and can therefore safely be
58:  * converted to and from its corresponding plain JNI reference. They are useful as parameters of
59:  * functions that do not affect the lifetime of a reference. Usage can be compared with using plain
60:  * JNI pointers as parameters where a function does not take ownership of the underlying object.
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````cpp
 61:  *
 62:  * The local, global, and alias references makes it possible to access methods in the underlying
 63:  * objects. A core set of classes are implemented in CoreClasses.h, and user defined wrappers are
 64:  * supported (see example below). The wrappers also supports inheritance so a wrapper can inherit
 65:  * from another wrapper to gain access to its functionality. As an example the jstring wrapper
 66:  * inherits from the jobject wrapper, so does the jclass wrapper. That means that you can for
 67:  * example call the toString() method using the jclass wrapper, or any other class that inherits
 68:  * from the jobject wrapper.
 69:  *
 70:  * Note that the wrappers are parameterized on the static type of your (jobject) pointer, thus if
 71:  * you have a jobject that refers to a Java String you will need to cast it to jstring to get the
 72:  * jstring wrapper. This also mean that if you make a down cast that is invalid there will be no one
 73:  * stopping you and the wrappers currently does not detect this which can cause crashes. Thus, cast
 74:  * wisely.
 75:  *
 76:  * @include WrapperSample.cpp
 77:  */
 78: 
 79: #pragma once
 80: 
 81: #include <cassert>
 82: #include <cstddef>
 83: #include <type_traits>
 84: 
 85: #include <jni.h>
 86: 
 87: #include "ReferenceAllocators.h"
 88: #include "TypeTraits.h"
 89: #include "References-forward.h"
 90: 
 91: namespace facebook {
 92: namespace jni {
 93: 
 94: /// Convenience function to wrap an existing local reference
 95: template<typename T>
 96: local_ref<T> adopt_local(T ref) noexcept;
 97: 
 98: /// Convenience function to wrap an existing global reference
 99: template<typename T>
100: global_ref<T> adopt_global(T ref) noexcept;
101: 
102: /// Convenience function to wrap an existing weak reference
103: template<typename T>
104: weak_ref<T> adopt_weak_global(T ref) noexcept;
105: 
106: 
107: /// Swaps two owning references of the same type
108: template<typename T>
109: void swap(weak_ref<T>& a, weak_ref<T>& b) noexcept;
110: 
111: /// Swaps two owning references of the same type
112: template<typename T, typename Alloc>
113: void swap(basic_strong_ref<T, Alloc>& a, basic_strong_ref<T, Alloc>& b) noexcept;
114: 
115: /**
116:  * Retrieve the plain reference from a plain reference.
117:  */
118: template<typename T>
119: enable_if_t<IsPlainJniReference<T>(), T> getPlainJniReference(T ref);
120: 
````
- EN: Pulls in native headers such as `cassert`, `cstddef`, `type_traits`, `jni.h`.
- CN: 引入原生头文件，例如 `cassert`, `cstddef`, `type_traits`, `jni.h`。
- EN: Declares or extends types including `that`.
- CN: 声明或扩展类型，包括 `that`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-180
````cpp
121: /**
122:  * Retrieve the plain reference from an alias reference.
123:  */
124: template<typename T>
125: JniType<T> getPlainJniReference(alias_ref<T> ref);
126: 
127: /**
128:  * Retrieve the plain JNI reference from any reference owned reference.
129:  */
130: template<typename T, typename Alloc>
131: JniType<T> getPlainJniReference(const base_owned_ref<T, Alloc>& ref);
132: 
133: class JObject;
134: class JClass;
135: 
136: namespace detail {
137: 
138: template<typename T>
139: constexpr bool IsJavaClassType() {
140:   return std::is_base_of<JObject, T>::value;
141: }
142: 
143: template <typename T, typename Enable = void>
144: struct HasJniRefRepr : std::false_type {};
145: 
146: template <typename T>
147: struct HasJniRefRepr<T, typename std::enable_if<!std::is_same<typename T::JniRefRepr, void>::value, void>::type> : std::true_type {
148:   using type = typename T::JniRefRepr;
149: };
150: 
151: template <typename T>
152: struct RefReprType<T*> {
153:   static_assert(HasJniRefRepr<T>::value, "Repr type missing JniRefRepr.");
154:   using type = typename HasJniRefRepr<T>::type;
155:   static_assert(IsJavaClassType<type>(),
156:       "Repr type missing JObject base.");
157:   static_assert(std::is_same<type, typename RefReprType<type>::type>::value,
158:       "RefReprType<T> not idempotent");
159: };
160: 
161: template <typename T>
162: struct RefReprType<T, typename std::enable_if<IsJavaClassType<T>(), void>::type> {
163:   using type = T;
164:   static_assert(IsJavaClassType<type>(),
165:       "Repr type missing JObject base.");
166:   static_assert(std::is_same<type, typename RefReprType<type>::type>::value,
167:       "RefReprType<T> not idempotent");
168: };
169: 
170: template <typename T>
171: struct JavaObjectType {
172:   using type = typename RefReprType<T>::type::javaobject;
173:   static_assert(IsPlainJniReference<type>(),
174:       "JavaObjectType<T> not a plain jni reference");
175:   static_assert(std::is_same<type, typename JavaObjectType<type>::type>::value,
176:       "JavaObjectType<T> not idempotent");
177: };
178: 
179: template <typename T>
180: struct JavaObjectType<T*> {
````
- EN: Declares or extends types including `JObject`, `JClass`, `HasJniRefRepr`, `RefReprType`.
- CN: 声明或扩展类型，包括 `JObject`, `JClass`, `HasJniRefRepr`, `RefReprType`。
- EN: Implements callable logic such as `IsJavaClassType`.
- CN: 实现可调用逻辑，例如 `IsJavaClassType`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````cpp
181:   using type = T*;
182:   static_assert(IsPlainJniReference<type>(),
183:       "JavaObjectType<T> not a plain jni reference");
184:   static_assert(std::is_same<type, typename JavaObjectType<type>::type>::value,
185:       "JavaObjectType<T> not idempotent");
186: };
187: 
188: template <typename T>
189: struct PrimitiveOrJavaObjectType<T, enable_if_t<IsJniPrimitive<T>(), void>> {
190:   using type = T;
191:   static_assert(IsJniPrimitive<type>(),
192:       "PrimitiveOrJavaObjectType<T> not a jni primitive");
193:   static_assert(std::is_same<type, typename PrimitiveOrJavaObjectType<type>::type>::value,
194:       "PrimitiveOrJavaObjectType<T> not idempotent");
195: };
196: 
197: template <typename T>
198: struct PrimitiveOrJavaObjectType<T, enable_if_t<IsPlainJniReference<T>(), void>> {
199:   using type = T;
200:   static_assert(IsPlainJniReference<type>(),
201:       "PrimitiveOrJavaObjectType<T> not a plain jni reference");
202:   static_assert(std::is_same<type, typename PrimitiveOrJavaObjectType<type>::type>::value,
203:       "PrimitiveOrJavaObjectType<T> not idempotent");
204: };
205: 
206: template <typename T>
207: struct PrimitiveOrJavaObjectType<T, enable_if_t<IsJavaClassType<T>(), void>> {
208:   using type = JniType<T>;
209:   static_assert(IsPlainJniReference<type>(),
210:       "PrimitiveOrJavaObjectType<T> not a plain jni reference");
211:   static_assert(std::is_same<type, typename PrimitiveOrJavaObjectType<type>::type>::value,
212:       "PrimitiveOrJavaObjectType<T> not idempotent");
213: };
214: 
215: template <typename Repr>
216: struct ReprStorage {
217:   explicit ReprStorage(JniType<Repr> obj) noexcept;
218: 
219:   void set(JniType<Repr> obj) noexcept;
220: 
221:   Repr& get() noexcept;
222:   const Repr& get() const noexcept;
223:   JniType<Repr> jobj() const noexcept;
224: 
225:   void swap(ReprStorage& other) noexcept;
226: 
227:   ReprStorage() = delete;
228:   ReprStorage(const ReprStorage&) = delete;
229:   ReprStorage(ReprStorage&&) = delete;
230:   ReprStorage& operator=(const ReprStorage&) = delete;
231:   ReprStorage& operator=(ReprStorage&&) = delete;
232: 
233:  private:
234:   using Storage = typename std::aligned_storage<sizeof(JObjectBase), alignof(JObjectBase)>::type;
235:   Storage storage_;
236: };
237: 
238: } // namespace detail
239: 
240: /**
````
- EN: Declares or extends types including `PrimitiveOrJavaObjectType`, `ReprStorage`.
- CN: 声明或扩展类型，包括 `PrimitiveOrJavaObjectType`, `ReprStorage`。

### Lines 241-300
````cpp
241:  * Create a new local reference from an existing reference
242:  *
243:  * @param ref a plain JNI, alias, or strong reference
244:  * @return an owned local reference (referring to null if the input does)
245:  * @throws std::bad_alloc if the JNI reference could not be created
246:  */
247: template<typename T>
248: enable_if_t<IsNonWeakReference<T>(), local_ref<plain_jni_reference_t<T>>>
249: make_local(const T& r);
250: 
251: /**
252:  * Create a new global reference from an existing reference
253:  *
254:  * @param ref a plain JNI, alias, or strong reference
255:  * @return an owned global reference (referring to null if the input does)
256:  * @throws std::bad_alloc if the JNI reference could not be created
257:  */
258: template<typename T>
259: enable_if_t<IsNonWeakReference<T>(), global_ref<plain_jni_reference_t<T>>>
260: make_global(const T& r);
261: 
262: /**
263:  * Create a new weak global reference from an existing reference
264:  *
265:  * @param ref a plain JNI, alias, or strong reference
266:  * @return an owned weak global reference (referring to null if the input does)
267:  * @throws std::bad_alloc if the returned reference is null
268:  */
269: template<typename T>
270: enable_if_t<IsNonWeakReference<T>(), weak_ref<plain_jni_reference_t<T>>>
271: make_weak(const T& r);
272: 
273: /**
274:  * Compare two references to see if they refer to the same object
275:  */
276: template<typename T1, typename T2>
277: enable_if_t<IsNonWeakReference<T1>() && IsNonWeakReference<T2>(), bool>
278: operator==(const T1& a, const T2& b);
279: 
280: /**
281:  * Compare two references to see if they don't refer to the same object
282:  */
283: template<typename T1, typename T2>
284: enable_if_t<IsNonWeakReference<T1>() && IsNonWeakReference<T2>(), bool>
285: operator!=(const T1& a, const T2& b);
286: 
287: /**
288:  * Compare references against nullptr
289:  */
290: template<typename T1>
291: enable_if_t<IsNonWeakReference<T1>(), bool>
292: operator==(const T1& a, std::nullptr_t);
293: 
294: template<typename T1>
295: enable_if_t<IsNonWeakReference<T1>(), bool>
296: operator==(std::nullptr_t, const T1& a);
297: 
298: template<typename T1>
299: enable_if_t<IsNonWeakReference<T1>(), bool>
300: operator!=(const T1& a, std::nullptr_t);
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````cpp
301: 
302: template<typename T1>
303: enable_if_t<IsNonWeakReference<T1>(), bool>
304: operator!=(std::nullptr_t, const T1& a);
305: 
306: 
307: template<typename T, typename Alloc>
308: class base_owned_ref {
309:  public:
310:   using javaobject = JniType<T>;
311: 
312:   /**
313:    * Release the ownership and set the reference to null. Thus no deleter is invoked.
314:    * @return Returns the reference
315:    */
316:   javaobject release() noexcept;
317: 
318:   /**
319:    * Reset the reference to refer to nullptr.
320:    */
321:   void reset() noexcept;
322: 
323:  protected:
324:   using Repr = ReprType<T>;
325:   detail::ReprStorage<Repr> storage_;
326: 
327:   javaobject get() const noexcept;
328:   void set(javaobject ref) noexcept;
329: 
330:   /*
331:    * Wrap an existing reference and transfers its ownership to the newly created unique reference.
332:    * NB! Does not create a new reference
333:    */
334:   explicit base_owned_ref(javaobject reference) noexcept;
335: 
336:   /// Create a null reference
337:   base_owned_ref() noexcept;
338: 
339:   /// Create a null reference
340:   explicit base_owned_ref(std::nullptr_t) noexcept;
341: 
342:   /// Copy constructor (note creates a new reference)
343:   base_owned_ref(const base_owned_ref& other);
344:   template<typename U>
345:   base_owned_ref(const base_owned_ref<U, Alloc>& other);
346: 
347:   /// Transfers ownership of an underlying reference from one unique reference to another
348:   base_owned_ref(base_owned_ref&& other) noexcept;
349:   template<typename U>
350:   base_owned_ref(base_owned_ref<U, Alloc>&& other) noexcept;
351: 
352:   /// The delete the underlying reference if applicable
353:   ~base_owned_ref() noexcept;
354: 
355: 
356:   /// Assignment operator (note creates a new reference)
357:   base_owned_ref& operator=(const base_owned_ref& other);
358: 
359:   /// Assignment by moving a reference thus not creating a new reference
360:   base_owned_ref& operator=(base_owned_ref&& rhs) noexcept;
````
- EN: Declares or extends types including `base_owned_ref`.
- CN: 声明或扩展类型，包括 `base_owned_ref`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-420
````cpp
361: 
362:   void reset(javaobject reference) noexcept;
363: 
364:   friend javaobject jni::getPlainJniReference<>(const base_owned_ref<T, Alloc>& ref);
365: 
366:   template<typename U, typename UAlloc>
367:   friend class base_owned_ref;
368: };
369: 
370: 
371: /**
372:  * A smart reference that owns its underlying JNI reference. The class provides basic
373:  * functionality to handle a reference but gives no access to it unless the reference is
374:  * released, thus no longer owned. The API is stolen with pride from unique_ptr and the
375:  * semantics should be basically the same. This class should not be used directly, instead use
376:  * @ref weak_ref
377:  */
378: template<typename T>
379: class weak_ref : public base_owned_ref<T, WeakGlobalReferenceAllocator> {
380:  public:
381:   using javaobject = JniType<T>;
382: 
383:   using Allocator = WeakGlobalReferenceAllocator;
384: 
385:   // This inherits non-default, non-copy, non-move ctors.
386:   using base_owned_ref<T, Allocator>::base_owned_ref;
387: 
388:   /// Create a null reference
389:   weak_ref() noexcept
390:     : base_owned_ref<T, Allocator>{} {}
391: 
392:   /// Create a null reference
393:   /* implicit */ weak_ref(std::nullptr_t) noexcept
394:     : base_owned_ref<T, Allocator>{nullptr} {}
395: 
396:   /// Copy constructor (note creates a new reference)
397:   weak_ref(const weak_ref& other)
398:     : base_owned_ref<T, Allocator>{other} {}
399: 
400:   // This needs to be explicit to change its visibility.
401:   template<typename U>
402:   weak_ref(const weak_ref<U>& other)
403:     : base_owned_ref<T, Allocator>{other} {}
404: 
405:   /// Transfers ownership of an underlying reference from one unique reference to another
406:   weak_ref(weak_ref&& other) noexcept
407:     : base_owned_ref<T, Allocator>{std::move(other)} {}
408: 
409:   // Move from ref to compatible type.
410:   template<typename U>
411:   weak_ref(weak_ref<U>&& other)
412:       : base_owned_ref<T, Allocator>{std::move(other)} {}
413: 
414:   /// Assignment operator (note creates a new reference)
415:   weak_ref& operator=(const weak_ref& other);
416: 
417:   /// Assignment by moving a reference thus not creating a new reference
418:   weak_ref& operator=(weak_ref&& rhs) noexcept;
419: 
420:   // Creates an owned local reference to the referred object or to null if the object is reclaimed
````
- EN: Declares or extends types including `base_owned_ref`, `provides`, `should`, `weak_ref`.
- CN: 声明或扩展类型，包括 `base_owned_ref`, `provides`, `should`, `weak_ref`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 421-480
````cpp
421:   local_ref<T> lockLocal() const;
422: 
423:   // Creates an owned global reference to the referred object or to null if the object is reclaimed
424:   global_ref<T> lockGlobal() const;
425: 
426:  private:
427:   // get/release/reset on weak_ref are not exposed to users.
428:   using base_owned_ref<T, Allocator>::get;
429:   using base_owned_ref<T, Allocator>::release;
430:   using base_owned_ref<T, Allocator>::reset;
431:   /*
432:    * Wrap an existing reference and transfers its ownership to the newly created unique reference.
433:    * NB! Does not create a new reference
434:    */
435:   explicit weak_ref(javaobject reference) noexcept
436:     : base_owned_ref<T, Allocator>{reference} {}
437: 
438:   template<typename T2> friend class weak_ref;
439:   friend weak_ref<javaobject> adopt_weak_global<javaobject>(javaobject ref) noexcept;
440:   friend void swap<T>(weak_ref& a, weak_ref& b) noexcept;
441: };
442: 
443: 
444: /**
445:  * A class representing owned strong references to Java objects. This class
446:  * should not be used directly, instead use @ref local_ref, or @ref global_ref.
447:  */
448: template<typename T, typename Alloc>
449: class basic_strong_ref : public base_owned_ref<T, Alloc> {
450:   using typename base_owned_ref<T, Alloc>::Repr;
451:  public:
452:   using javaobject = JniType<T>;
453: 
454:   using Allocator = Alloc;
455: 
456:   // This inherits non-default, non-copy, non-move ctors.
457:   using base_owned_ref<T, Alloc>::base_owned_ref;
458:   using base_owned_ref<T, Alloc>::release;
459:   using base_owned_ref<T, Alloc>::reset;
460: 
461:   /// Create a null reference
462:   basic_strong_ref() noexcept
463:     : base_owned_ref<T, Alloc>{} {}
464: 
465:   /// Create a null reference
466:   /* implicit */ basic_strong_ref(std::nullptr_t) noexcept
467:     : base_owned_ref<T, Alloc>{nullptr} {}
468: 
469:   /// Copy constructor (note creates a new reference)
470:   basic_strong_ref(const basic_strong_ref& other)
471:     : base_owned_ref<T, Alloc>{other} {}
472: 
473:   // This needs to be explicit to change its visibility.
474:   template<typename U>
475:   basic_strong_ref(const basic_strong_ref<U, Alloc>& other)
476:     : base_owned_ref<T, Alloc>{other} {}
477: 
478:   // Move from ref to compatible type.
479:   template<typename U>
480:   basic_strong_ref(basic_strong_ref<U, Alloc>&& other)
````
- EN: Declares or extends types including `weak_ref`, `representing`, `basic_strong_ref`.
- CN: 声明或扩展类型，包括 `weak_ref`, `representing`, `basic_strong_ref`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 481-540
````cpp
481:     : base_owned_ref<T, Alloc>{std::move(other)} {}
482: 
483:   /// Transfers ownership of an underlying reference from one unique reference to another
484:   basic_strong_ref(basic_strong_ref&& other) noexcept
485:     : base_owned_ref<T, Alloc>{std::move(other)} {}
486: 
487:   /// Assignment operator (note creates a new reference)
488:   basic_strong_ref& operator=(const basic_strong_ref& other);
489: 
490:   /// Assignment by moving a reference thus not creating a new reference
491:   basic_strong_ref& operator=(basic_strong_ref&& rhs) noexcept;
492: 
493:   /// Get the plain JNI reference
494:   using base_owned_ref<T, Allocator>::get;
495: 
496:   /// Release the ownership of the reference and return the wrapped reference in an alias
497:   alias_ref<T> releaseAlias() noexcept;
498: 
499:   /// Checks if the reference points to a non-null object
500:   explicit operator bool() const noexcept;
501: 
502:   /// Access the functionality provided by the object wrappers
503:   Repr* operator->() noexcept;
504: 
505:   /// Access the functionality provided by the object wrappers
506:   const Repr* operator->() const noexcept;
507: 
508:   /// Provide a reference to the underlying wrapper (be sure that it is non-null before invoking)
509:   Repr& operator*() noexcept;
510: 
511:   /// Provide a const reference to the underlying wrapper (be sure that it is non-null
512:   /// before invoking)
513:   const Repr& operator*() const noexcept;
514: 
515:  private:
516: 
517:   using base_owned_ref<T, Alloc>::storage_;
518: 
519:   /*
520:    * Wrap an existing reference and transfers its ownership to the newly created unique reference.
521:    * NB! Does not create a new reference
522:    */
523:   explicit basic_strong_ref(javaobject reference) noexcept
524:     : base_owned_ref<T, Alloc>{reference} {}
525: 
526: 
527:   friend local_ref<T> adopt_local<T>(T ref) noexcept;
528:   friend global_ref<T> adopt_global<T>(T ref) noexcept;
529:   friend void swap<T, Alloc>(basic_strong_ref& a, basic_strong_ref& b) noexcept;
530: };
531: 
532: 
533: template<typename T>
534: enable_if_t<IsPlainJniReference<T>(), alias_ref<T>> wrap_alias(T ref) noexcept;
535: 
536: /// Swaps to alias reference of the same type
537: template<typename T>
538: void swap(alias_ref<T>& a, alias_ref<T>& b) noexcept;
539: 
540: /**
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 541-600
````cpp
541:  * A non-owning variant of the smart references (a dumb
542:  * reference). Use this representation when you don't want to claim
543:  * ownership of the underlying reference (compare to using raw
544:  * pointers instead of smart pointers.)
545:  */
546: template<typename T>
547: class alias_ref {
548:   using Repr = ReprType<T>;
549: 
550:  public:
551:   using javaobject = JniType<T>;
552: 
553:   /// Create a null reference
554:   alias_ref() noexcept;
555: 
556:   /// Create a null reference
557:   /* implicit */ alias_ref(std::nullptr_t) noexcept;
558: 
559:   /// Copy constructor
560:   alias_ref(const alias_ref& other) noexcept;
561: 
562:   /// Wrap an existing plain JNI reference
563:   /* implicit */ alias_ref(javaobject ref) noexcept;
564: 
565:   /// Wrap an existing smart reference of any type convertible to T
566:   template<
567:     typename TOther,
568:     typename = enable_if_t<
569:       IsConvertible<JniType<TOther>, javaobject>(), T>
570:     >
571:   alias_ref(alias_ref<TOther> other) noexcept;
572: 
573:   /// Wrap an existing alias reference of a type convertible to T
574:   template<
575:     typename TOther,
576:     typename AOther,
577:     typename = enable_if_t<
578:       IsConvertible<JniType<TOther>, javaobject>(), T>
579:     >
580:   alias_ref(const basic_strong_ref<TOther, AOther>& other) noexcept;
581: 
582:   /// Assignment operator
583:   alias_ref& operator=(alias_ref other) noexcept;
584: 
585:   /// Checks if the reference points to a non-null object
586:   explicit operator bool() const noexcept;
587: 
588:   /// Converts back to a plain JNI reference
589:   javaobject get() const noexcept;
590: 
591:   /// Access the functionality provided by the object wrappers
592:   Repr* operator->() noexcept;
593: 
594:   /// Access the functionality provided by the object wrappers
595:   const Repr* operator->() const noexcept;
596: 
597:   /// Provide a guaranteed non-null reference (be sure that it is non-null before invoking)
598:   Repr& operator*() noexcept;
599: 
600:   /// Provide a guaranteed non-null reference (be sure that it is non-null before invoking)
````
- EN: Declares or extends types including `alias_ref`.
- CN: 声明或扩展类型，包括 `alias_ref`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 601-646
````cpp
601:   const Repr& operator*() const noexcept;
602: 
603:  private:
604:   void set(javaobject ref) noexcept;
605: 
606:   detail::ReprStorage<Repr> storage_;
607: 
608:   friend void swap<T>(alias_ref& a, alias_ref& b) noexcept;
609: };
610: 
611: 
612: /**
613:  * RAII object to create a local JNI frame, using PushLocalFrame/PopLocalFrame.
614:  *
615:  * This is useful when you have a call which is initiated from C++-land, and therefore
616:  * doesn't automatically get a local JNI frame managed for you by the JNI framework.
617:  */
618: class JniLocalScope {
619: public:
620:   JniLocalScope(JNIEnv* p_env, jint capacity);
621:   ~JniLocalScope();
622: 
623: private:
624:   JNIEnv* env_;
625:   bool hasFrame_;
626: };
627: 
628: template<typename T, typename U>
629: enable_if_t<IsPlainJniReference<JniType<T>>(), local_ref<T>>
630: static_ref_cast(const local_ref<U>& ref) noexcept;
631: 
632: template<typename T, typename U>
633: enable_if_t<IsPlainJniReference<JniType<T>>(), global_ref<T>>
634: static_ref_cast(const global_ref<U>& ref) noexcept;
635: 
636: template<typename T, typename U>
637: enable_if_t<IsPlainJniReference<JniType<T>>(), alias_ref<T>>
638: static_ref_cast(const alias_ref<U>& ref) noexcept;
639: 
640: template<typename T, typename RefType>
641: auto dynamic_ref_cast(const RefType& ref) ->
642: enable_if_t<IsPlainJniReference<JniType<T>>(), decltype(static_ref_cast<T>(ref))> ;
643: 
644: }}
645: 
646: #include "References-inl.h"
````
- EN: Pulls in native headers such as `References-inl.h`.
- CN: 引入原生头文件，例如 `References-inl.h`。
- EN: Declares or extends types including `JniLocalScope`.
- CN: 声明或扩展类型，包括 `JniLocalScope`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `that` / 符号 `that`
- Symbol `JObject` / 符号 `JObject`
- Symbol `JClass` / 符号 `JClass`
- Symbol `HasJniRefRepr` / 符号 `HasJniRefRepr`

## Dependencies / 依赖关系
- C/C++ includes: `cassert`, `cstddef`, `type_traits`, `jni.h`, `ReferenceAllocators.h`, `TypeTraits.h`, `References-forward.h`, `References-inl.h`
- C/C++ 头文件: `cassert`, `cstddef`, `type_traits`, `jni.h`, `ReferenceAllocators.h`, `TypeTraits.h`, `References-forward.h`, `References-inl.h`
