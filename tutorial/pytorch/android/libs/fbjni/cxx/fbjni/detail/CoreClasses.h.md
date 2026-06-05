# CoreClasses.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/CoreClasses.h`
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
19: /** @file CoreClasses.h
20:  *
21:  * In CoreClasses.h wrappers for the core classes (jobject, jclass, and jstring) is defined
22:  * to provide access to corresponding JNI functions + some conveniance.
23:  */
24: 
25: #include "References-forward.h"
26: #include "Meta-forward.h"
27: #include "TypeTraits.h"
28: 
29: #include <memory>
30: 
31: #include <jni.h>
32: 
33: #include <fbjni/detail/SimpleFixedString.h>
34: 
35: namespace facebook {
36: namespace jni {
37: 
38: class JClass;
39: class JObject;
40: 
41: namespace detail {
42: 
43: /// Lookup a class by name.  This should only be used internally.
44: jclass findClass(JNIEnv* env, const char* name);
45: 
46: }
47: 
48: /// Lookup a class by name. Note this functions returns an alias_ref that
49: /// points to a leaked global reference.  This is appropriate for classes
50: /// that are never unloaded (which is any class in an Android app and most
51: /// Java programs).
52: ///
53: /// The most common use case for this is storing the result
54: /// in a "static auto" variable, or a static global.
55: ///
56: /// @return Returns a leaked global reference to the class
57: alias_ref<JClass> findClassStatic(const char* name);
58: 
59: /// Lookup a class by name. Note this functions returns a local reference,
60: /// which means that it must not be stored in a static variable.
````
- EN: Pulls in native headers such as `References-forward.h`, `Meta-forward.h`, `TypeTraits.h`, `memory`.
- CN: 引入原生头文件，例如 `References-forward.h`, `Meta-forward.h`, `TypeTraits.h`, `memory`。
- EN: Declares or extends types including `JClass`, `JObject`, `by`, `in`.
- CN: 声明或扩展类型，包括 `JClass`, `JObject`, `by`, `in`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 61-120
````cpp
 61: ///
 62: /// The most common use case for this is one-time initialization
 63: /// (like caching method ids).
 64: ///
 65: /// @return Returns a global reference to the class
 66: local_ref<JClass> findClassLocal(const char* name);
 67: 
 68: /// Check to see if two references refer to the same object. Comparison with nullptr
 69: /// returns true if and only if compared to another nullptr. A weak reference that
 70: /// refers to a reclaimed object count as nullptr.
 71: bool isSameObject(alias_ref<JObject> lhs, alias_ref<JObject> rhs) noexcept;
 72: 
 73: // Together, these classes allow convenient use of any class with the fbjni
 74: // helpers.  To use:
 75: //
 76: // struct MyClass : public JavaClass<MyClass> {
 77: //   constexpr static auto kJavaDescriptor = "Lcom/example/package/MyClass;";
 78: // };
 79: //
 80: // Then, an alias_ref<MyClass> will be backed by an instance of MyClass.
 81: // JavaClass provides a convenient way to add functionality to these
 82: // smart references.
 83: //
 84: // For example:
 85: //
 86: // struct MyClass : public JavaClass<MyClass> {
 87: //   constexpr static auto kJavaDescriptor = "Lcom/example/package/MyClass;";
 88: //
 89: //   void foo() {
 90: //     static const auto method = javaClassStatic()->getMethod<void()>("foo");
 91: //     method(self());
 92: //   }
 93: //
 94: //   static local_ref<javaobject> create(int i) {
 95: //     return newInstance(i);
 96: //   }
 97: // };
 98: //
 99: // auto obj = MyClass::create(10);
100: // obj->foo();
101: //
102: // While users of a JavaClass-type can lookup methods and fields through the
103: // underlying JClass, those calls can only be checked at runtime. It is recommended
104: // that the JavaClass-type instead explicitly expose it's methods as in the example
105: // above.
106: 
107: namespace detail {
108: template<typename JC, typename... Args>
109: static local_ref<JC> newInstance(Args... args);
110: }
111: 
112: class MonitorLock;
113: 
114: class JObject : detail::JObjectBase {
115: public:
116:   static constexpr auto kJavaDescriptor = "Ljava/lang/Object;";
117: 
118:   static constexpr detail::SimpleFixedString<0> get_instantiated_java_descriptor() { return ""; }
119:   static constexpr detail::SimpleFixedString<0> get_instantiated_base_name() { return ""; }
120: 
````
- EN: Declares or extends types including `local_ref`, `with`, `MyClass`, `MonitorLock`.
- CN: 声明或扩展类型，包括 `local_ref`, `with`, `MyClass`, `MonitorLock`。
- EN: Implements callable logic such as `foo`, `create`, `get_instantiated_java_descriptor`, `get_instantiated_base_name`.
- CN: 实现可调用逻辑，例如 `foo`, `create`, `get_instantiated_java_descriptor`, `get_instantiated_base_name`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````cpp
121:   /// Get a @ref local_ref of the object's class
122:   local_ref<JClass> getClass() const noexcept;
123: 
124:   /// Checks if the object is an instance of a class
125:   bool isInstanceOf(alias_ref<JClass> cls) const noexcept;
126: 
127:   /// Get the primitive value of a field
128:   template<typename T>
129:   T getFieldValue(JField<T> field) const noexcept;
130: 
131:   /// Get and wrap the value of a field in a @ref local_ref
132:   template<typename T>
133:   local_ref<T*> getFieldValue(JField<T*> field) const noexcept;
134: 
135:   /// Set the value of field. Any Java type is accepted.
136:   template<typename T>
137:   void setFieldValue(JField<T> field, T value) noexcept;
138:   template<typename T,
139:            typename = typename std::enable_if<IsPlainJniReference<T>(), T>::type>
140:   void setFieldValue(JField<T> field, alias_ref<T> value) noexcept;
141: 
142:   /// Convenience method to create a std::string representing the object
143:   std::string toString() const;
144: 
145:   // Take this object's monitor lock
146:   MonitorLock lock() const noexcept;
147: 
148:   typedef _jobject _javaobject;
149:   typedef _javaobject* javaobject;
150: 
151: protected:
152:   jobject self() const noexcept;
153: private:
154:   friend void swap(JObject& a, JObject& b) noexcept;
155:   template<typename>
156:   friend struct detail::ReprAccess;
157:   template<typename, typename, typename>
158:   friend class JavaClass;
159: };
160: 
161: namespace detail {
162: template <typename, typename Base, typename JType>
163: struct JTypeFor {
164:   static_assert(
165:       std::is_base_of<
166:         std::remove_pointer<jobject>::type,
167:         typename std::remove_pointer<JType>::type
168:       >::value, "");
169:   using _javaobject = typename std::remove_pointer<JType>::type;
170:   using javaobject = JType;
171: };
172: 
173: template <typename T, typename Base>
174: struct JTypeFor<T, Base, void> {
175:   // JNI pattern for jobject assignable pointer
176:   struct _javaobject :  Base::_javaobject {
177:     // This allows us to map back to the defining type (in ReprType, for
178:     // example).
179:     typedef T JniRefRepr;
180:   };
````
- EN: Declares or extends types including `local_ref`, `bool`, `detail`, `JavaClass`.
- CN: 声明或扩展类型，包括 `local_ref`, `bool`, `detail`, `JavaClass`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 181-240
````cpp
181:   using javaobject = _javaobject*;
182: };
183: }
184: 
185: // JavaClass provides a method to inform fbjni about user-defined Java types.
186: // Given a class:
187: // struct Foo : JavaClass<Foo> {
188: //   static constexpr auto kJavaDescriptor = "Lcom/example/package/Foo;";
189: // };
190: // fbjni can determine the java type/method signatures for Foo::javaobject and
191: // smart refs (like alias_ref<Foo>) will hold an instance of Foo
192: // and provide access to it through the -> and * operators.
193: //
194: // The "Base" template argument can be used to specify the JavaClass superclass
195: // of this type (for instance, JString's Base is JObject).
196: //
197: // The "JType" template argument is used to provide a jni type (like jstring,
198: // jthrowable) to be used as javaobject. This should only be necessary for
199: // built-in jni types and not user-defined ones.
200: template <typename T, typename Base = JObject, typename JType = void>
201: class JavaClass : public Base {
202:   using JObjType = typename detail::JTypeFor<T, Base, JType>;
203: public:
204:   using _javaobject = typename JObjType::_javaobject;
205:   using javaobject = typename JObjType::javaobject;
206: 
207:   using JavaBase = JavaClass;
208: 
209:   static alias_ref<JClass> javaClassStatic();
210:   static local_ref<JClass> javaClassLocal();
211: protected:
212:   /// Allocates a new object and invokes the specified constructor
213:   /// Like JClass's getConstructor, this function can only check at runtime if
214:   /// the class actually has a constructor that accepts the corresponding types.
215:   /// While a JavaClass-type can expose this function directly, it is recommended
216:   /// to instead to use this to explicitly only expose those constructors that
217:   /// the Java class actually has (i.e. with static create() functions).
218:   template<typename... Args>
219:   static local_ref<T> newInstance(Args... args) {
220:     return detail::newInstance<T>(args...);
221:   }
222: 
223:   javaobject self() const noexcept;
224: };
225: 
226: /// Wrapper to provide functionality to jclass references
227: class JClass : public JavaClass<JClass, JObject, jclass> {
228:  public:
229:   /// Java type descriptor
230:   static constexpr const char* kJavaDescriptor = "Ljava/lang/Class;";
231: 
232:   /// Get a @local_ref to the super class of this class
233:   local_ref<JClass> getSuperclass() const noexcept;
234: 
235:   /// Register native methods for the class.  Usage looks like this:
236:   ///
237:   /// classRef->registerNatives({
238:   ///     makeNativeMethod("nativeMethodWithAutomaticDescriptor",
239:   ///                      methodWithAutomaticDescriptor),
240:   ///     makeNativeMethod("nativeMethodWithExplicitDescriptor",
````
- EN: Declares or extends types including `Foo`, `JavaClass`, `actually`, `JClass`.
- CN: 声明或扩展类型，包括 `Foo`, `JavaClass`, `actually`, `JClass`。
- EN: Implements callable logic such as `has`.
- CN: 实现可调用逻辑，例如 `has`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````cpp
241:   ///                      "(Lcom/facebook/example/MyClass;)V",
242:   ///                      methodWithExplicitDescriptor),
243:   ///     makeCriticalNativeMethod_DO_NOT_USE_OR_YOU_WILL_BE_FIRED("criticalNativeMethodWithAutomaticDescriptor",
244:   ///                              criticalNativeMethodWithAutomaticDescriptor),
245:   ///     makeCriticalNativeMethod_DO_NOT_USE_OR_YOU_WILL_BE_FIRED("criticalNativeMethodWithExplicitDescriptor",
246:   ///                              "(IIF)Z",
247:   ///                              criticalNativeMethodWithExplicitDescriptor),
248:   ///  });
249:   ///
250:   /// By default, C++ exceptions raised will be converted to Java exceptions.
251:   /// To avoid this and get the "standard" JNI behavior of a crash when a C++
252:   /// exception is crashing out of the JNI method, declare the method noexcept.
253:   /// This does NOT apply to critical native methods, where exceptions causes
254:   /// a crash.
255:   void registerNatives(std::initializer_list<JNINativeMethod> methods);
256: 
257:   /// Check to see if the class is assignable from another class
258:   /// @pre cls != nullptr
259:   bool isAssignableFrom(alias_ref<JClass> cls) const noexcept;
260: 
261:   /// Convenience method to lookup the constructor with descriptor as specified by the
262:   /// type arguments
263:   template<typename F>
264:   JConstructor<F> getConstructor() const;
265: 
266:   /// Convenience method to lookup the constructor with specified descriptor
267:   template<typename F>
268:   JConstructor<F> getConstructor(const char* descriptor) const;
269: 
270:   /// Look up the method with given name and descriptor as specified with the type arguments
271:   template<typename F>
272:   JMethod<F> getMethod(const char* name) const;
273: 
274:   /// Look up the method with given name and descriptor
275:   template<typename F>
276:   JMethod<F> getMethod(const char* name, const char* descriptor) const;
277: 
278:   /// Lookup the field with the given name and deduced descriptor
279:   template<typename T>
280:   JField<PrimitiveOrJniType<T>> getField(const char* name) const;
281: 
282:   /// Lookup the field with the given name and descriptor
283:   template<typename T>
284:   JField<PrimitiveOrJniType<T>> getField(const char* name, const char* descriptor) const;
285: 
286:   /// Lookup the static field with the given name and deduced descriptor
287:   template<typename T>
288:   JStaticField<PrimitiveOrJniType<T>> getStaticField(const char* name) const;
289: 
290:   /// Lookup the static field with the given name and descriptor
291:   template<typename T>
292:   JStaticField<PrimitiveOrJniType<T>> getStaticField(
293:       const char* name,
294:       const char* descriptor) const;
295: 
296:   /// Get the primitive value of a static field
297:   template<typename T>
298:   T getStaticFieldValue(JStaticField<T> field) const noexcept;
299: 
300:   /// Get and wrap the value of a field in a @ref local_ref
````
- EN: Declares or extends types including `is`.
- CN: 声明或扩展类型，包括 `is`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 301-360
````cpp
301:   template<typename T>
302:   local_ref<T*> getStaticFieldValue(JStaticField<T*> field) noexcept;
303: 
304:   /// Set the value of field. Any Java type is accepted.
305:   template<typename T>
306:   void setStaticFieldValue(JStaticField<T> field, T value) noexcept;
307:   template<typename T,
308:            typename = typename std::enable_if<IsPlainJniReference<T>(), T>::type>
309:   void setStaticFieldValue(JStaticField<T> field, alias_ref<T> value) noexcept;
310: 
311:   /// Allocates a new object and invokes the specified constructor
312:   template<typename R, typename... Args>
313:   local_ref<R> newObject(JConstructor<R(Args...)> constructor, Args... args) const;
314: 
315:   /// Look up the static method with given name and descriptor as specified with the type arguments
316:   template<typename F>
317:   JStaticMethod<F> getStaticMethod(const char* name) const;
318: 
319:   /// Look up the static method with given name and descriptor
320:   template<typename F>
321:   JStaticMethod<F> getStaticMethod(const char* name, const char* descriptor) const;
322: 
323:   /// Look up the non virtual method with given name and descriptor as specified with the
324:   /// type arguments
325:   template<typename F>
326:   JNonvirtualMethod<F> getNonvirtualMethod(const char* name) const;
327: 
328:   /// Look up the non virtual method with given name and descriptor
329:   template<typename F>
330:   JNonvirtualMethod<F> getNonvirtualMethod(const char* name, const char* descriptor) const;
331: 
332: private:
333:   jclass self() const noexcept;
334: };
335: 
336: // Convenience method to register methods on a class without holding
337: // onto the class object.
338: void registerNatives(const char* name, std::initializer_list<JNINativeMethod> methods);
339: 
340: /// Wrapper to provide functionality to jstring references
341: class JString : public JavaClass<JString, JObject, jstring> {
342:  public:
343:   /// Java type descriptor
344:   static constexpr const char* kJavaDescriptor = "Ljava/lang/String;";
345: 
346:   /// Convenience method to convert a jstring object to a std::string
347:   std::string toStdString() const;
348: 
349:   /// Convenience method to convert a jstring object to a std::u16string
350:   std::u16string toU16String() const;
351: };
352: 
353: /// Convenience functions to convert a const char*, std::string, or std::u16string
354: /// into a @ref local_ref to a jstring.
355: local_ref<JString> make_jstring(const char* utf8);
356: local_ref<JString> make_jstring(const std::string& utf8);
357: local_ref<JString> make_jstring(const std::u16string& utf16);
358: 
359: namespace detail {
360: template<typename Target>
````
- EN: Declares or extends types including `without`, `object`, `JString`.
- CN: 声明或扩展类型，包括 `without`, `object`, `JString`。

### Lines 361-420
````cpp
361: class ElementProxy {
362:  private:
363:   Target* target_;
364:   size_t idx_;
365: 
366:  public:
367:   using T = typename Target::javaentry;
368:   ElementProxy(Target* target, size_t idx);
369: 
370:   ElementProxy(const ElementProxy&) noexcept = default;
371: 
372:   ElementProxy& operator=(const T& o);
373: 
374:   ElementProxy& operator=(alias_ref<T>& o);
375: 
376:   ElementProxy& operator=(alias_ref<T>&& o);
377: 
378:   ElementProxy& operator=(const ElementProxy& o);
379: 
380:   operator const local_ref<T> () const;
381: 
382:   operator local_ref<T> ();
383: };
384: }
385: 
386: namespace detail {
387: class JArray : public JavaClass<JArray, JObject, jarray> {
388:  public:
389:   // This cannot be used in a scope that derives a descriptor (like in a method
390:   // signature). Use a more derived type instead (like JArrayInt or
391:   // JArrayClass<T>).
392:   static constexpr const char* kJavaDescriptor = nullptr;
393:   size_t size() const noexcept;
394: };
395: 
396: // This is used so that the JArrayClass<T> javaobject extends jni's
397: // jobjectArray. This class should not be used directly. A general Object[]
398: // should use JArrayClass<jobject>.
399: class JTypeArray : public JavaClass<JTypeArray, JArray, jobjectArray> {
400:   // This cannot be used in a scope that derives a descriptor (like in a method
401:   // signature).
402:   static constexpr const char* kJavaDescriptor = nullptr;
403: };
404: }
405: 
406: template<typename T>
407: class JArrayClass : public JavaClass<JArrayClass<T>, detail::JTypeArray> {
408:  public:
409:   static_assert(
410:       IsPlainJniReference<JniType<T>>(),
411:       "Element type must be a JNI reference or JavaClass type.");
412:   // javaentry is the jni type of an entry in the array (i.e. JObject).
413:   using javaentry = T;
414:   // javaobject is the jni type of the array.
415:   using javaobject = typename JavaClass<JArrayClass<T>, detail::JTypeArray>::javaobject;
416:   static constexpr const char* kJavaDescriptor = nullptr;
417:   static constexpr auto /* detail::SimpleFixedString<_> */ get_instantiated_java_descriptor() {
418:     return "[" + jtype_traits<T>::kDescriptor;
419:   }
420: 
````
- EN: Declares or extends types including `ElementProxy`, `JArray`, `should`, `JTypeArray`.
- CN: 声明或扩展类型，包括 `ElementProxy`, `JArray`, `should`, `JTypeArray`。
- EN: Implements callable logic such as `get_instantiated_java_descriptor`.
- CN: 实现可调用逻辑，例如 `get_instantiated_java_descriptor`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 421-480
````cpp
421:   static constexpr auto /* detail::SimpleFixedString<_> */ get_instantiated_base_name() {
422:     return get_instantiated_java_descriptor();
423:   }
424: 
425:   /// Allocate a new array from Java heap, for passing as a JNI parameter or return value.
426:   /// NOTE: if using as a return value, you want to call release() instead of get() on the
427:   /// smart pointer.
428:   static local_ref<javaobject> newArray(size_t count);
429: 
430:   /// Assign an object to the array.
431:   /// Typically you will use the shorthand (*ref)[idx]=value;
432:   void setElement(size_t idx, T value);
433: 
434:   /// Read an object from the array.
435:   /// Typically you will use the shorthand
436:   ///   T value = (*ref)[idx];
437:   /// If you use auto, you'll get an ElementProxy, which may need to be cast.
438:   local_ref<T> getElement(size_t idx);
439: 
440:   /// EXPERIMENTAL SUBSCRIPT SUPPORT
441:   /// This implementation of [] returns a proxy object which then has a bunch of specializations
442:   /// (adopt_local free function, operator= and casting overloads on the ElementProxy) that can
443:   /// make code look like it is dealing with a T rather than an obvious proxy. In particular, the
444:   /// proxy in this iteration does not read a value and therefore does not create a LocalRef
445:   /// until one of these other operators is used. There are certainly holes that you may find
446:   /// by using idioms that haven't been tried yet. Consider yourself warned. On the other hand,
447:   /// it does make for some idiomatic assignment code; see TestBuildStringArray in fbjni_tests
448:   /// for some examples.
449:   detail::ElementProxy<JArrayClass> operator[](size_t idx);
450: };
451: 
452: template <typename T>
453: using jtypeArray = typename JArrayClass<T>::javaobject;
454: 
455: template<typename T>
456: local_ref<typename JArrayClass<T>::javaobject> adopt_local_array(jobjectArray ref);
457: 
458: template<typename Target>
459: local_ref<typename Target::javaentry> adopt_local(detail::ElementProxy<Target> elementProxy) {
460:   return static_cast<local_ref<typename Target::javaentry>>(elementProxy);
461: }
462: 
463: template <typename T, typename PinAlloc>
464: class PinnedPrimitiveArray;
465: 
466: template <typename T> class PinnedArrayAlloc;
467: template <typename T> class PinnedRegionAlloc;
468: template <typename T> class PinnedCriticalAlloc;
469: 
470: /// Wrapper to provide functionality to jarray references.
471: /// This is an empty holder by itself. Construct a PinnedPrimitiveArray to actually interact with
472: /// the elements of the array.
473: template <typename JArrayType>
474: class JPrimitiveArray :
475:     public JavaClass<JPrimitiveArray<JArrayType>, detail::JArray, JArrayType> {
476:   static_assert(is_jni_primitive_array<JArrayType>(), "");
477:  public:
478:   static constexpr const char* kJavaDescriptor = nullptr;
479:   static constexpr auto /* detail::SimpleFixedString<_> */ get_instantiated_java_descriptor() {
480:     return jtype_traits<JArrayType>::kDescriptor;
````
- EN: Declares or extends types including `PinnedPrimitiveArray`, `PinnedArrayAlloc`, `PinnedRegionAlloc`, `PinnedCriticalAlloc`.
- CN: 声明或扩展类型，包括 `PinnedPrimitiveArray`, `PinnedArrayAlloc`, `PinnedRegionAlloc`, `PinnedCriticalAlloc`。
- EN: Implements callable logic such as `get_instantiated_base_name`, `adopt_local`, `get_instantiated_java_descriptor`.
- CN: 实现可调用逻辑，例如 `get_instantiated_base_name`, `adopt_local`, `get_instantiated_java_descriptor`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 481-540
````cpp
481:   }
482:   static constexpr auto /* detail::SimpleFixedString<_> */ get_instantiated_base_name() {
483:     return JPrimitiveArray::get_instantiated_java_descriptor();
484:   }
485: 
486:   using T = typename jtype_traits<JArrayType>::entry_type;
487: 
488:   static local_ref<JArrayType> newArray(size_t count);
489: 
490:   void getRegion(jsize start, jsize length, T* buf);
491:   std::unique_ptr<T[]> getRegion(jsize start, jsize length);
492:   void setRegion(jsize start, jsize length, const T* buf);
493: 
494:   /// Returns a view of the underlying array. This will either be a "pinned"
495:   /// version of the array (in which case changes to one immediately affect the
496:   /// other) or a copy of the array (in which cases changes to the view will take
497:   /// affect when destroyed or on calls to release()/commit()).
498:   PinnedPrimitiveArray<T, PinnedArrayAlloc<T>> pin();
499: 
500:   /// Returns a view of part of the underlying array. A pinned region is always
501:   /// backed by a copy of the region.
502:   PinnedPrimitiveArray<T, PinnedRegionAlloc<T>> pinRegion(jsize start, jsize length);
503: 
504:   /// Returns a view of the underlying array like pin(). However, while the pin
505:   /// is held, the code is considered within a "critical region". In a critical
506:   /// region, native code must not call JNI functions or make any calls that may
507:   /// block on other Java threads. These restrictions make it more likely that
508:   /// the view will be "pinned" rather than copied (for example, the VM may
509:   /// suspend garbage collection within a critical region).
510:   PinnedPrimitiveArray<T, PinnedCriticalAlloc<T>> pinCritical();
511: 
512: private:
513:   friend class PinnedArrayAlloc<T>;
514:   T* getElements(jboolean* isCopy);
515:   void releaseElements(T* elements, jint mode);
516: };
517: 
518: local_ref<jbooleanArray> make_boolean_array(jsize size);
519: local_ref<jbyteArray> make_byte_array(jsize size);
520: local_ref<jcharArray> make_char_array(jsize size);
521: local_ref<jshortArray> make_short_array(jsize size);
522: local_ref<jintArray> make_int_array(jsize size);
523: local_ref<jlongArray> make_long_array(jsize size);
524: local_ref<jfloatArray> make_float_array(jsize size);
525: local_ref<jdoubleArray> make_double_array(jsize size);
526: 
527: using JArrayBoolean = JPrimitiveArray<jbooleanArray>;
528: using JArrayByte = JPrimitiveArray<jbyteArray>;
529: using JArrayChar = JPrimitiveArray<jcharArray>;
530: using JArrayShort = JPrimitiveArray<jshortArray>;
531: using JArrayInt = JPrimitiveArray<jintArray>;
532: using JArrayLong = JPrimitiveArray<jlongArray>;
533: using JArrayFloat = JPrimitiveArray<jfloatArray>;
534: using JArrayDouble = JPrimitiveArray<jdoubleArray>;
535: 
536: /// RAII class for pinned primitive arrays
537: /// This currently only supports read/write access to existing java arrays. You can't create a
538: /// primitive array this way yet. This class also pins the entire array into memory during the
539: /// lifetime of the PinnedPrimitiveArray. If you need to unpin the array manually, call the
540: /// release() or abort() functions. During a long-running block of code, you
````
- EN: Declares or extends types including `PinnedArrayAlloc`, `for`, `also`.
- CN: 声明或扩展类型，包括 `PinnedArrayAlloc`, `for`, `also`。
- EN: Implements callable logic such as `get_instantiated_base_name`.
- CN: 实现可调用逻辑，例如 `get_instantiated_base_name`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 541-600
````cpp
541: /// should unpin the array as soon as you're done with it, to avoid holding up
542: /// the Java garbage collector.
543: template <typename T, typename PinAlloc>
544: class PinnedPrimitiveArray {
545:   public:
546:    static_assert(is_jni_primitive<T>::value,
547:        "PinnedPrimitiveArray requires primitive jni type.");
548: 
549:    using ArrayType = typename jtype_traits<T>::array_type;
550: 
551:    PinnedPrimitiveArray(PinnedPrimitiveArray&&);
552:    PinnedPrimitiveArray(const PinnedPrimitiveArray&) = delete;
553:    ~PinnedPrimitiveArray() noexcept;
554: 
555:    PinnedPrimitiveArray& operator=(PinnedPrimitiveArray&&);
556:    PinnedPrimitiveArray& operator=(const PinnedPrimitiveArray&) = delete;
557: 
558:    T* get();
559:    void release();
560:    /// Unpins the array. If the array is a copy, pending changes are discarded.
561:    void abort();
562:    /// If the array is a copy, copies pending changes to the underlying java array.
563:    void commit();
564: 
565:    bool isCopy() const noexcept;
566: 
567:    const T& operator[](size_t index) const;
568:    T& operator[](size_t index);
569:    size_t size() const noexcept;
570: 
571:   private:
572:    alias_ref<ArrayType> array_;
573:    size_t start_;
574:    T* elements_;
575:    jboolean isCopy_;
576:    size_t size_;
577: 
578:    void allocate(alias_ref<ArrayType>, jint start, jint length);
579:    void releaseImpl(jint mode);
580:    void clear() noexcept;
581: 
582:    PinnedPrimitiveArray(alias_ref<ArrayType>, jint start, jint length);
583: 
584:    friend class JPrimitiveArray<typename jtype_traits<T>::array_type>;
585: };
586: 
587: struct JStackTraceElement : JavaClass<JStackTraceElement> {
588:   static auto constexpr kJavaDescriptor = "Ljava/lang/StackTraceElement;";
589: 
590:   static local_ref<javaobject> create(const std::string& declaringClass, const std::string& methodName, const std::string& file, int line);
591: 
592:   std::string getClassName() const;
593:   std::string getMethodName() const;
594:   std::string getFileName() const;
595:   int getLineNumber() const;
596: };
597: 
598: /// Wrapper to provide functionality to jthrowable references
599: class JThrowable : public JavaClass<JThrowable, JObject, jthrowable> {
600:  public:
````
- EN: Declares or extends types including `PinnedPrimitiveArray`, `JPrimitiveArray`, `JStackTraceElement`, `JThrowable`.
- CN: 声明或扩展类型，包括 `PinnedPrimitiveArray`, `JPrimitiveArray`, `JStackTraceElement`, `JThrowable`。

### Lines 601-637
````cpp
601:   static constexpr const char* kJavaDescriptor = "Ljava/lang/Throwable;";
602: 
603:   using JStackTrace = JArrayClass<JStackTraceElement::javaobject>;
604: 
605:   local_ref<JThrowable> initCause(alias_ref<JThrowable> cause);
606:   local_ref<JStackTrace> getStackTrace();
607:   void setStackTrace(alias_ref<JArrayClass<JStackTraceElement::javaobject>>);
608: };
609: 
610: #pragma push_macro("PlainJniRefMap")
611: #undef PlainJniRefMap
612: #define PlainJniRefMap(rtype, jtype) \
613: namespace detail { \
614: template<> \
615: struct RefReprType<jtype> { \
616:   using type = rtype; \
617: }; \
618: }
619: 
620: PlainJniRefMap(JArrayBoolean, jbooleanArray);
621: PlainJniRefMap(JArrayByte, jbyteArray);
622: PlainJniRefMap(JArrayChar, jcharArray);
623: PlainJniRefMap(JArrayShort, jshortArray);
624: PlainJniRefMap(JArrayInt, jintArray);
625: PlainJniRefMap(JArrayLong, jlongArray);
626: PlainJniRefMap(JArrayFloat, jfloatArray);
627: PlainJniRefMap(JArrayDouble, jdoubleArray);
628: PlainJniRefMap(JObject, jobject);
629: PlainJniRefMap(JClass, jclass);
630: PlainJniRefMap(JString, jstring);
631: PlainJniRefMap(JThrowable, jthrowable);
632: 
633: #pragma pop_macro("PlainJniRefMap")
634: 
635: }}
636: 
637: #include "CoreClasses-inl.h"
````
- EN: Pulls in native headers such as `CoreClasses-inl.h`.
- CN: 引入原生头文件，例如 `CoreClasses-inl.h`。
- EN: Declares or extends types including `RefReprType`.
- CN: 声明或扩展类型，包括 `RefReprType`。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `JClass` / 符号 `JClass`
- Symbol `JObject` / 符号 `JObject`
- Symbol `by` / 符号 `by`
- Symbol `in` / 符号 `in`

## Dependencies / 依赖关系
- C/C++ includes: `References-forward.h`, `Meta-forward.h`, `TypeTraits.h`, `memory`, `jni.h`, `fbjni/detail/SimpleFixedString.h`, `CoreClasses-inl.h`
- C/C++ 头文件: `References-forward.h`, `Meta-forward.h`, `TypeTraits.h`, `memory`, `jni.h`, `fbjni/detail/SimpleFixedString.h`, `CoreClasses-inl.h`
