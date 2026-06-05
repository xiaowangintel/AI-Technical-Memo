# Meta.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/Meta.h`
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
17: /** @file meta.h
18:  *
19:  * Provides wrappers for meta data such as methods and fields.
20:  */
21: 
22: #pragma once
23: 
24: #include <type_traits>
25: #include <string>
26: 
27: #include <jni.h>
28: 
29: #include <fbjni/detail/FbjniApi.h>
30: #include <fbjni/detail/SimpleFixedString.h>
31: #include "References-forward.h"
32: 
33: #ifdef __ANDROID__
34: # include <android/log.h>
35: # define XLOG_TAG "fb-jni"
36: # define XLOGV(...) __android_log_print(ANDROID_LOG_VERBOSE, XLOG_TAG, __VA_ARGS__)
37: # define XLOGD(...) __android_log_print(ANDROID_LOG_DEBUG, XLOG_TAG, __VA_ARGS__)
38: # define XLOGI(...) __android_log_print(ANDROID_LOG_INFO, XLOG_TAG, __VA_ARGS__)
39: # define XLOGW(...) __android_log_print(ANDROID_LOG_WARN, XLOG_TAG, __VA_ARGS__)
40: # define XLOGE(...) __android_log_print(ANDROID_LOG_ERROR, XLOG_TAG, __VA_ARGS__)
41: # define XLOGWTF(...) __android_log_print(ANDROID_LOG_FATAL, XLOG_TAG, __VA_ARGS__)
42: #endif
43: 
44: namespace facebook {
45: namespace jni {
46: 
47: // This will get the reflected Java Method from the method_id, get it's invoke
48: // method, and call the method via that. This shouldn't ever be needed, but
49: // Android 6.0 crashes when calling a method on a java.lang.Proxy via jni.
50: template <typename... Args>
51: local_ref<jobject> slowCall(jmethodID method_id, alias_ref<jobject> self, Args... args);
52: 
53: class JObject;
54: 
55: 
56: /// Wrapper of a jmethodID. Provides a common base for JMethod specializations
57: class JMethodBase {
58:  public:
59:   /// Verify that the method is valid
60:   explicit operator bool() const noexcept;
````
- EN: Pulls in native headers such as `type_traits`, `string`, `jni.h`, `fbjni/detail/FbjniApi.h`.
- CN: 引入原生头文件，例如 `type_traits`, `string`, `jni.h`, `fbjni/detail/FbjniApi.h`。
- EN: Declares or extends types including `JObject`, `JMethodBase`.
- CN: 声明或扩展类型，包括 `JObject`, `JMethodBase`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````cpp
 61: 
 62:   /// Access the wrapped id
 63:   jmethodID getId() const noexcept;
 64: 
 65:  protected:
 66:   /// Create a wrapper of a method id
 67:   explicit JMethodBase(jmethodID method_id = nullptr) noexcept;
 68: 
 69:  private:
 70:   jmethodID method_id_;
 71: };
 72: 
 73: 
 74: /// Representation of a jmethodID
 75: template<typename F>
 76: class JMethod;
 77: 
 78: /// @cond INTERNAL
 79: #pragma push_macro("DEFINE_PRIMITIVE_METHOD_CLASS")
 80: 
 81: #undef DEFINE_PRIMITIVE_METHOD_CLASS
 82: 
 83: // Defining JMethod specializations based on return value
 84: #define DEFINE_PRIMITIVE_METHOD_CLASS(TYPE)                                      \
 85: template<typename... Args>                                                       \
 86: class JMethod<TYPE(Args...)> : public JMethodBase {                              \
 87:  public:                                                                         \
 88:   static_assert(std::is_void<TYPE>::value || IsJniPrimitive<TYPE>(),             \
 89:       "TYPE must be primitive or void");                                         \
 90:                                                                                  \
 91:   using JMethodBase::JMethodBase;                                                \
 92:   JMethod() noexcept {};                                                         \
 93:   JMethod(const JMethod& other) noexcept = default;                              \
 94:                                                                                  \
 95:   TYPE operator()(alias_ref<jobject> self, Args... args) const;                  \
 96:                                                                                  \
 97:   friend class JClass;                                                           \
 98: }
 99: 
100: DEFINE_PRIMITIVE_METHOD_CLASS(void);
101: DEFINE_PRIMITIVE_METHOD_CLASS(jboolean);
102: DEFINE_PRIMITIVE_METHOD_CLASS(jbyte);
103: DEFINE_PRIMITIVE_METHOD_CLASS(jchar);
104: DEFINE_PRIMITIVE_METHOD_CLASS(jshort);
105: DEFINE_PRIMITIVE_METHOD_CLASS(jint);
106: DEFINE_PRIMITIVE_METHOD_CLASS(jlong);
107: DEFINE_PRIMITIVE_METHOD_CLASS(jfloat);
108: DEFINE_PRIMITIVE_METHOD_CLASS(jdouble);
109: 
110: #pragma pop_macro("DEFINE_PRIMITIVE_METHOD_CLASS")
111: /// @endcond
112: 
113: 
114: /// Convenience type representing constructors
115: /// These should only be used with JClass::getConstructor and JClass::newObject.
116: template<typename F>
117: struct JConstructor : private JMethod<F> {
118:   using JMethod<F>::JMethod;
119:  private:
120:   JConstructor(const JMethod<F>& other) : JMethod<F>(other.getId()) {}
````
- EN: Declares or extends types including `JMethod`, `JClass`, `JConstructor`.
- CN: 声明或扩展类型，包括 `JMethod`, `JClass`, `JConstructor`。
- EN: Implements callable logic such as `JConstructor`.
- CN: 实现可调用逻辑，例如 `JConstructor`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````cpp
121:   friend class JClass;
122: };
123: 
124: /// Representation of a jStaticMethodID
125: template<typename F>
126: class JStaticMethod;
127: 
128: /// @cond INTERNAL
129: #pragma push_macro("DEFINE_PRIMITIVE_STATIC_METHOD_CLASS")
130: 
131: #undef DEFINE_PRIMITIVE_STATIC_METHOD_CLASS
132: 
133: // Defining JStaticMethod specializations based on return value
134: #define DEFINE_PRIMITIVE_STATIC_METHOD_CLASS(TYPE)                          \
135: template<typename... Args>                                                  \
136: class JStaticMethod<TYPE(Args...)> : public JMethodBase {                   \
137:   static_assert(std::is_void<TYPE>::value || IsJniPrimitive<TYPE>(),        \
138:       "T must be a JNI primitive or void");                                 \
139:                                                                             \
140:  public:                                                                    \
141:   using JMethodBase::JMethodBase;                                           \
142:   JStaticMethod() noexcept {};                                              \
143:   JStaticMethod(const JStaticMethod& other) noexcept = default;             \
144:                                                                             \
145:   TYPE operator()(alias_ref<jclass> cls, Args... args) const;               \
146:                                                                             \
147:   friend class JClass;                                                      \
148: }
149: 
150: DEFINE_PRIMITIVE_STATIC_METHOD_CLASS(void);
151: DEFINE_PRIMITIVE_STATIC_METHOD_CLASS(jboolean);
152: DEFINE_PRIMITIVE_STATIC_METHOD_CLASS(jbyte);
153: DEFINE_PRIMITIVE_STATIC_METHOD_CLASS(jchar);
154: DEFINE_PRIMITIVE_STATIC_METHOD_CLASS(jshort);
155: DEFINE_PRIMITIVE_STATIC_METHOD_CLASS(jint);
156: DEFINE_PRIMITIVE_STATIC_METHOD_CLASS(jlong);
157: DEFINE_PRIMITIVE_STATIC_METHOD_CLASS(jfloat);
158: DEFINE_PRIMITIVE_STATIC_METHOD_CLASS(jdouble);
159: 
160: #pragma pop_macro("DEFINE_PRIMITIVE_STATIC_METHOD_CLASS")
161: /// @endcond
162: 
163: 
164: /// Representation of a jNonvirtualMethodID
165: template<typename F>
166: class JNonvirtualMethod;
167: 
168: /// @cond INTERNAL
169: #pragma push_macro("DEFINE_PRIMITIVE_NON_VIRTUAL_METHOD_CLASS")
170: 
171: #undef DEFINE_PRIMITIVE_NON_VIRTUAL_METHOD_CLASS
172: 
173: // Defining JNonvirtualMethod specializations based on return value
174: #define DEFINE_PRIMITIVE_NON_VIRTUAL_METHOD_CLASS(TYPE)                     \
175: template<typename... Args>                                                  \
176: class JNonvirtualMethod<TYPE(Args...)> : public JMethodBase {               \
177:   static_assert(std::is_void<TYPE>::value || IsJniPrimitive<TYPE>(),        \
178:       "T must be a JNI primitive or void");                                 \
179:                                                                             \
180:  public:                                                                    \
````
- EN: Declares or extends types including `JClass`, `JStaticMethod`, `JNonvirtualMethod`.
- CN: 声明或扩展类型，包括 `JClass`, `JStaticMethod`, `JNonvirtualMethod`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````cpp
181:   using JMethodBase::JMethodBase;                                           \
182:   JNonvirtualMethod() noexcept {};                                          \
183:   JNonvirtualMethod(const JNonvirtualMethod& other) noexcept = default;     \
184:                                                                             \
185:   TYPE operator()(alias_ref<jobject> self, alias_ref<jclass> cls, Args... args) const; \
186:                                                                             \
187:   friend class JClass;                                                      \
188: }
189: 
190: DEFINE_PRIMITIVE_NON_VIRTUAL_METHOD_CLASS(void);
191: DEFINE_PRIMITIVE_NON_VIRTUAL_METHOD_CLASS(jboolean);
192: DEFINE_PRIMITIVE_NON_VIRTUAL_METHOD_CLASS(jbyte);
193: DEFINE_PRIMITIVE_NON_VIRTUAL_METHOD_CLASS(jchar);
194: DEFINE_PRIMITIVE_NON_VIRTUAL_METHOD_CLASS(jshort);
195: DEFINE_PRIMITIVE_NON_VIRTUAL_METHOD_CLASS(jint);
196: DEFINE_PRIMITIVE_NON_VIRTUAL_METHOD_CLASS(jlong);
197: DEFINE_PRIMITIVE_NON_VIRTUAL_METHOD_CLASS(jfloat);
198: DEFINE_PRIMITIVE_NON_VIRTUAL_METHOD_CLASS(jdouble);
199: 
200: #pragma pop_macro("DEFINE_PRIMITIVE_NON_VIRTUAL_METHOD_CLASS")
201: /// @endcond
202: 
203: 
204: /**
205:  * JField represents typed fields and simplifies their access. Note that object types return
206:  * raw pointers which generally should promptly get a wrap_local treatment.
207:  */
208: template<typename T>
209: class JField {
210:   static_assert(IsJniScalar<T>(), "T must be a JNI scalar");
211: 
212:  public:
213:   /// Wraps an existing field id
214:   explicit JField(jfieldID field = nullptr) noexcept;
215: 
216:   /// Verify that the id is valid
217:   explicit operator bool() const noexcept;
218: 
219:   /// Access the wrapped id
220:   jfieldID getId() const noexcept;
221: 
222:  private:
223:   jfieldID field_id_;
224: 
225:   /// Get field value
226:   /// @pre object != nullptr
227:   T get(jobject object) const noexcept;
228: 
229:   /// Set field value
230:   /// @pre object != nullptr
231:   void set(jobject object, T value) noexcept;
232: 
233:   friend class JObject;
234: };
235: 
236: 
237: /**
238:  * JStaticField represents typed fields and simplifies their access. Note that object types
239:  * return raw pointers which generally should promptly get a wrap_local treatment.
240:  */
````
- EN: Declares or extends types including `JClass`, `JField`, `JObject`.
- CN: 声明或扩展类型，包括 `JClass`, `JField`, `JObject`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````cpp
241: template<typename T>
242: class JStaticField {
243:   static_assert(IsJniScalar<T>(), "T must be a JNI scalar");
244: 
245:  public:
246:   /// Wraps an existing field id
247:   explicit JStaticField(jfieldID field = nullptr) noexcept;
248: 
249:   /// Verify that the id is valid
250:   explicit operator bool() const noexcept;
251: 
252:   /// Access the wrapped id
253:   jfieldID getId() const noexcept;
254: 
255:  private:
256:   jfieldID field_id_;
257: 
258:   /// Get field value
259:   /// @pre object != nullptr
260:   T get(jclass jcls) const noexcept;
261: 
262:   /// Set field value
263:   /// @pre object != nullptr
264:   void set(jclass jcls, T value) noexcept;
265: 
266:   friend class JClass;
267:   friend class JObject;
268: };
269: 
270: 
271: // jtype_traits ////////////////////////////////////////////////////////////////////////////////////
272: 
273: template<typename T>
274: struct jtype_traits {
275: private:
276:   using Repr = ReprType<T>;
277:   static constexpr auto /* detail::SimpleFixedString<_> */ descriptor() {
278:     constexpr auto len = Repr::kJavaDescriptor
279:       ? detail::constexpr_strlen(Repr::kJavaDescriptor)
280:       : Repr::get_instantiated_java_descriptor().size();
281:     if (Repr::kJavaDescriptor) {
282:       return detail::SimpleFixedString<len>(Repr::kJavaDescriptor, len);
283:     } else {
284:       return detail::SimpleFixedString<len>(Repr::get_instantiated_java_descriptor());
285:     }
286:   }
287:   static constexpr auto /* detail::SimpleFixedString<_> */ base_name() {
288:     constexpr auto len = Repr::kJavaDescriptor ? detail::constexpr_strlen(Repr::kJavaDescriptor) - 2 : Repr::get_instantiated_base_name().size();
289:     if (Repr::kJavaDescriptor) {
290:       detail::SimpleFixedString<len + 2> result(Repr::kJavaDescriptor, len + 2);
291:       return detail::SimpleFixedString<len>(result.substr(1, result.size() - 2));
292:     }
293:     return detail::SimpleFixedString<len>(Repr::get_instantiated_base_name());
294:   }
295:  public:
296:   using descriptorType = decltype(jtype_traits<T>::descriptor());
297:   using basenameType = decltype(jtype_traits<T>::base_name());
298: 
299:   // The jni type signature (described at
300:   // http://docs.oracle.com/javase/1.5.0/docs/guide/jni/spec/types.html).
````
- EN: Declares or extends types including `JStaticField`, `JClass`, `JObject`, `jtype_traits`.
- CN: 声明或扩展类型，包括 `JStaticField`, `JClass`, `JObject`, `jtype_traits`。
- EN: Implements callable logic such as `descriptor`, `base_name`.
- CN: 实现可调用逻辑，例如 `descriptor`, `base_name`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````cpp
301:   static constexpr descriptorType /* detail::SimpleFixedString<_> */ kDescriptor = descriptor();
302: 
303:   // The signature used for class lookups. See
304:   // http://docs.oracle.com/javase/6/docs/api/java/lang/Class.html#getName().
305:   static constexpr basenameType /* detail::SimpleFixedString<_> */ kBaseName = base_name();
306: };
307: 
308: template <typename T>
309: constexpr typename jtype_traits<T>::descriptorType jtype_traits<T>::kDescriptor;
310: template <typename T>
311: constexpr typename jtype_traits<T>::basenameType jtype_traits<T>::kBaseName;
312: 
313: static_assert(
314:   std::is_same<jint, int>::value,
315:   "jint must be int.  On Windows, try using Android's jni.h.");
316: 
317: #pragma push_macro("DEFINE_FIELD_AND_ARRAY_TRAIT")
318: #undef DEFINE_FIELD_AND_ARRAY_TRAIT
319: 
320: // NOTE: When updating this definition, see also DEFINE_CONSTANTS_FOR_FIELD_AND_ARRAY_TRAIT in Meta.cpp.
321: #define DEFINE_FIELD_AND_ARRAY_TRAIT(TYPE, DSC)                         \
322:   template<>                                                            \
323: struct FBJNI_API jtype_traits<TYPE> {                                             \
324:   static constexpr decltype(detail::makeSimpleFixedString(#DSC)) kDescriptor = detail::makeSimpleFixedString(#DSC); \
325:   static constexpr decltype(kDescriptor) kBaseName = kDescriptor;       \
326:   using array_type = TYPE ## Array;                                     \
327: };                                                                      \
328:                                                                         \
329: template<>                                                              \
330: struct FBJNI_API jtype_traits<TYPE ## Array> {                                    \
331:   static constexpr decltype(detail::makeSimpleFixedString("[" #DSC)) kDescriptor = detail::makeSimpleFixedString("[" #DSC); \
332:   static constexpr decltype(jtype_traits<TYPE ## Array>::kDescriptor) kBaseName = kDescriptor; \
333:   using entry_type = TYPE;                                              \
334: };
335: 
336: 
337: // There is no voidArray, handle that without the macro.
338: template<>
339: struct FBJNI_API jtype_traits<void> {
340:   static constexpr detail::SimpleFixedString<1> kDescriptor = detail::makeSimpleFixedString("V");
341: };
342: 
343: DEFINE_FIELD_AND_ARRAY_TRAIT(jboolean, Z)
344: DEFINE_FIELD_AND_ARRAY_TRAIT(jbyte,    B)
345: DEFINE_FIELD_AND_ARRAY_TRAIT(jchar,    C)
346: DEFINE_FIELD_AND_ARRAY_TRAIT(jshort,   S)
347: DEFINE_FIELD_AND_ARRAY_TRAIT(jint,     I)
348: DEFINE_FIELD_AND_ARRAY_TRAIT(jlong,    J)
349: DEFINE_FIELD_AND_ARRAY_TRAIT(jfloat,   F)
350: DEFINE_FIELD_AND_ARRAY_TRAIT(jdouble,  D)
351: 
352: #pragma pop_macro("DEFINE_FIELD_AND_ARRAY_TRAIT")
353: 
354: 
355: template <typename T>
356: struct jmethod_traits_from_cxx;
357: 
358: }}
359: 
360: #include "Meta-inl.h"
````
- EN: Pulls in native headers such as `Meta-inl.h`.
- CN: 引入原生头文件，例如 `Meta-inl.h`。
- EN: Declares or extends types including `lookups`, `FBJNI_API`, `jmethod_traits_from_cxx`.
- CN: 声明或扩展类型，包括 `lookups`, `FBJNI_API`, `jmethod_traits_from_cxx`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 361-377
````cpp
361: 
362: namespace facebook {
363: namespace jni {
364: /// Template magic to provide @ref jmethod_traits
365: template<typename R, typename... Args>
366: struct jmethod_traits<R(Args...)> {
367:   static constexpr decltype(internal::JMethodDescriptor<R, Args...>()) /* detail::SimpleFixedString */ kDescriptor = internal::JMethodDescriptor<R, Args...>();
368:   static constexpr decltype(internal::JMethodDescriptor<void, Args...>()) /* detail::SimpleFixedString */ kConstructorDescriptor = internal::JMethodDescriptor<void, Args...>();
369: };
370: 
371: template<typename R, typename...Args>
372: /*static*/ constexpr decltype(internal::JMethodDescriptor<R, Args...>()) /* detail::SimpleFixedString */ jmethod_traits<R(Args...)>::kDescriptor;
373: 
374: template<typename R, typename...Args>
375: /*static*/ constexpr decltype(internal::JMethodDescriptor<void, Args...>()) /* detail::SimpleFixedString */ jmethod_traits<R(Args...)>::kConstructorDescriptor;
376: 
377: }}
````
- EN: Declares or extends types including `jmethod_traits`.
- CN: 声明或扩展类型，包括 `jmethod_traits`。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `JObject` / 符号 `JObject`
- Symbol `JMethodBase` / 符号 `JMethodBase`
- Symbol `JMethod` / 符号 `JMethod`
- Symbol `JClass` / 符号 `JClass`

## Dependencies / 依赖关系
- C/C++ includes: `type_traits`, `string`, `jni.h`, `fbjni/detail/FbjniApi.h`, `fbjni/detail/SimpleFixedString.h`, `References-forward.h`, `Meta-inl.h`
- C/C++ 头文件: `type_traits`, `string`, `jni.h`, `fbjni/detail/FbjniApi.h`, `fbjni/detail/SimpleFixedString.h`, `References-forward.h`, `Meta-inl.h`
