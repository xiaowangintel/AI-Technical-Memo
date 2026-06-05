# Meta-inl.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/Meta-inl.h`
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
19: #include <jni.h>
20: 
21: #include <fbjni/detail/SimpleFixedString.h>
22: 
23: #include "Common.h"
24: #include "Exceptions.h"
25: #include "MetaConvert.h"
26: #include "References.h"
27: #include "Boxed.h"
28: 
29: namespace facebook {
30: namespace jni {
31: 
32: // JMethod /////////////////////////////////////////////////////////////////////////////////////////
33: 
34: inline JMethodBase::JMethodBase(jmethodID method_id) noexcept
35:   : method_id_{method_id}
36: {}
37: 
38: inline JMethodBase::operator bool() const noexcept {
39:   return method_id_ != nullptr;
40: }
41: 
42: inline jmethodID JMethodBase::getId() const noexcept {
43:   return method_id_;
44: }
45: 
46: namespace {
47: 
48: template <int idx, typename... Args>
49: struct ArgsArraySetter;
50: 
51: template <int idx, typename Arg, typename... Args>
52: struct ArgsArraySetter<idx, Arg, Args...> {
53:   static void set(alias_ref<JArrayClass<jobject>> array, Arg arg0, Args... args) {
54:     // TODO(xxxxxxxx): Use Convert<Args>... to do conversions like the fast path.
55:     (*array)[idx] = autobox(arg0);
56:     ArgsArraySetter<idx + 1, Args...>::set(array, args...);
57:   }
58: };
59: 
60: template <int idx>
````
- EN: Pulls in native headers such as `jni.h`, `fbjni/detail/SimpleFixedString.h`, `Common.h`, `Exceptions.h`.
- CN: 引入原生头文件，例如 `jni.h`, `fbjni/detail/SimpleFixedString.h`, `Common.h`, `Exceptions.h`。
- EN: Declares or extends types including `ArgsArraySetter`.
- CN: 声明或扩展类型，包括 `ArgsArraySetter`。
- EN: Implements callable logic such as `set`.
- CN: 实现可调用逻辑，例如 `set`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````cpp
 61: struct ArgsArraySetter<idx> {
 62:   static void set(alias_ref<JArrayClass<jobject>> array) {
 63:     (void)array;
 64:   }
 65: };
 66: 
 67: template <typename... Args>
 68: local_ref<JArrayClass<jobject>> makeArgsArray(Args... args) {
 69:   auto arr = JArrayClass<jobject>::newArray(sizeof...(args));
 70:   ArgsArraySetter<0, Args...>::set(arr, args...);
 71:   return arr;
 72: }
 73: 
 74: }
 75: 
 76: template<typename... Args>
 77: inline void JMethod<void(Args...)>::operator()(alias_ref<jobject> self, Args... args) const {
 78:   const auto env = Environment::current();
 79:   env->CallVoidMethod(
 80:         self.get(),
 81:         getId(),
 82:         detail::callToJni(detail::Convert<typename std::decay<Args>::type>::toCall(args))...);
 83:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();
 84: }
 85: 
 86: #pragma push_macro("DEFINE_PRIMITIVE_CALL")
 87: #undef DEFINE_PRIMITIVE_CALL
 88: #define DEFINE_PRIMITIVE_CALL(TYPE, METHOD)                                                    \
 89: template<typename... Args>                                                                     \
 90: inline TYPE JMethod<TYPE(Args...)>::operator()(alias_ref<jobject> self, Args... args) const {  \
 91:   const auto env = Environment::current();                                                     \
 92:   auto result = env->Call ## METHOD ## Method(                                                 \
 93:         self.get(),                                                                            \
 94:         getId(),                                                                               \
 95:         detail::callToJni(detail::Convert<typename std::decay<Args>::type>::toCall(args))...); \
 96:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();                                                      \
 97:   return result;                                                                               \
 98: }
 99: 
100: DEFINE_PRIMITIVE_CALL(jboolean, Boolean)
101: DEFINE_PRIMITIVE_CALL(jbyte, Byte)
102: DEFINE_PRIMITIVE_CALL(jchar, Char)
103: DEFINE_PRIMITIVE_CALL(jshort, Short)
104: DEFINE_PRIMITIVE_CALL(jint, Int)
105: DEFINE_PRIMITIVE_CALL(jlong, Long)
106: DEFINE_PRIMITIVE_CALL(jfloat, Float)
107: DEFINE_PRIMITIVE_CALL(jdouble, Double)
108: #pragma pop_macro("DEFINE_PRIMITIVE_CALL")
109: 
110: /// JMethod specialization for references that wraps the return value in a @ref local_ref
111: template<typename R, typename... Args>
112: class JMethod<R(Args...)> : public JMethodBase {
113:  public:
114:    // TODO: static_assert is jobject-derived or local_ref jobject
115:   using JniRet = typename detail::Convert<typename std::decay<R>::type>::jniType;
116:   static_assert(
117:       IsPlainJniReference<JniRet>() || detail::IsJavaClassType<JniRet>(),
118:       "Return type must be a JNI reference or JavaClass type.");
119:   using JMethodBase::JMethodBase;
120:   JMethod() noexcept {};
````
- EN: Declares or extends types including `ArgsArraySetter`, `JMethod`.
- CN: 声明或扩展类型，包括 `ArgsArraySetter`, `JMethod`。
- EN: Implements callable logic such as `set`, `makeArgsArray`, `void`, `push_macro`.
- CN: 实现可调用逻辑，例如 `set`, `makeArgsArray`, `void`, `push_macro`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````cpp
121:   JMethod(const JMethod& other) noexcept = default;
122: 
123:   /// Invoke a method and return a local reference wrapping the result
124:   local_ref<JniRet> operator()(alias_ref<jobject> self, Args... args) const;
125: 
126:   friend class JClass;
127: };
128: 
129: template<typename R, typename... Args>
130: inline auto JMethod<R(Args...)>::operator()(alias_ref<jobject> self, Args... args) const -> local_ref<JniRet> {
131:   const auto env = Environment::current();
132:   auto result = env->CallObjectMethod(
133:       self.get(),
134:       getId(),
135:       detail::callToJni(detail::Convert<typename std::decay<Args>::type>::toCall(args))...);
136:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();
137:   return adopt_local(static_cast<JniType<JniRet>>(result));
138: }
139: 
140: template<typename... Args>
141: inline void JStaticMethod<void(Args...)>::operator()(alias_ref<jclass> cls, Args... args) const {
142:   const auto env = Environment::current();
143:   env->CallStaticVoidMethod(
144:         cls.get(),
145:         getId(),
146:         detail::callToJni(detail::Convert<typename std::decay<Args>::type>::toCall(args))...);
147:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();
148: }
149: 
150: #pragma push_macro("DEFINE_PRIMITIVE_STATIC_CALL")
151: #undef DEFINE_PRIMITIVE_STATIC_CALL
152: #define DEFINE_PRIMITIVE_STATIC_CALL(TYPE, METHOD)                                             \
153: template<typename... Args>                                                                     \
154: inline TYPE JStaticMethod<TYPE(Args...)>::operator()(alias_ref<jclass> cls, Args... args) const { \
155:   const auto env = Environment::current();                                                         \
156:   auto result = env->CallStatic ## METHOD ## Method(                                           \
157:         cls.get(),                                                                             \
158:         getId(),                                                                               \
159:         detail::callToJni(detail::Convert<typename std::decay<Args>::type>::toCall(args))...); \
160:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();                                                      \
161:         return result;                                                                         \
162: }
163: 
164: DEFINE_PRIMITIVE_STATIC_CALL(jboolean, Boolean)
165: DEFINE_PRIMITIVE_STATIC_CALL(jbyte, Byte)
166: DEFINE_PRIMITIVE_STATIC_CALL(jchar, Char)
167: DEFINE_PRIMITIVE_STATIC_CALL(jshort, Short)
168: DEFINE_PRIMITIVE_STATIC_CALL(jint, Int)
169: DEFINE_PRIMITIVE_STATIC_CALL(jlong, Long)
170: DEFINE_PRIMITIVE_STATIC_CALL(jfloat, Float)
171: DEFINE_PRIMITIVE_STATIC_CALL(jdouble, Double)
172: #pragma pop_macro("DEFINE_PRIMITIVE_STATIC_CALL")
173: 
174: /// JStaticMethod specialization for references that wraps the return value in a @ref local_ref
175: template<typename R, typename... Args>
176: class JStaticMethod<R(Args...)> : public JMethodBase {
177: 
178:  public:
179:   using JniRet = typename detail::Convert<typename std::decay<R>::type>::jniType;
180:   static_assert(
````
- EN: Declares or extends types including `JClass`, `JStaticMethod`.
- CN: 声明或扩展类型，包括 `JClass`, `JStaticMethod`。
- EN: Implements callable logic such as `void`, `push_macro`.
- CN: 实现可调用逻辑，例如 `void`, `push_macro`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````cpp
181:       IsPlainJniReference<JniRet>() || detail::IsJavaClassType<JniRet>(),
182:       "Return type must be a JNI reference or JavaClass type.");
183:   using JMethodBase::JMethodBase;
184:   JStaticMethod() noexcept {};
185:   JStaticMethod(const JStaticMethod& other) noexcept = default;
186: 
187:   /// Invoke a method and return a local reference wrapping the result
188:   local_ref<JniRet> operator()(alias_ref<jclass> cls, Args... args) const {
189:     const auto env = Environment::current();
190:     auto result = env->CallStaticObjectMethod(
191:           cls.get(),
192:           getId(),
193:           detail::callToJni(detail::Convert<typename std::decay<Args>::type>::toCall(args))...);
194:     FACEBOOK_JNI_THROW_PENDING_EXCEPTION();
195:     return adopt_local(static_cast<JniType<JniRet>>(result));
196:   }
197: 
198:   friend class JClass;
199: };
200: 
201: template<typename... Args>
202: inline void
203: JNonvirtualMethod<void(Args...)>::operator()(alias_ref<jobject> self, alias_ref<jclass> cls, Args... args) const {
204:   const auto env = Environment::current();
205:   env->CallNonvirtualVoidMethod(
206:         self.get(),
207:         cls.get(),
208:         getId(),
209:         detail::callToJni(detail::Convert<typename std::decay<Args>::type>::toCall(args))...);
210:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();
211: }
212: 
213: #pragma push_macro("DEFINE_PRIMITIVE_NON_VIRTUAL_CALL")
214: #undef DEFINE_PRIMITIVE_NON_VIRTUAL_CALL
215: #define DEFINE_PRIMITIVE_NON_VIRTUAL_CALL(TYPE, METHOD)                                                      \
216: template<typename... Args>                                                                                   \
217: inline TYPE                                                                                                  \
218: JNonvirtualMethod<TYPE(Args...)>::operator()(alias_ref<jobject> self, alias_ref<jclass> cls, Args... args) const { \
219:   const auto env = Environment::current();                                                                   \
220:   auto result = env->CallNonvirtual ## METHOD ## Method(                                                     \
221:         self.get(),                                                                                          \
222:         cls.get(),                                                                                           \
223:         getId(),                                                                                             \
224:         detail::callToJni(detail::Convert<typename std::decay<Args>::type>::toCall(args))...);               \
225:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();                                                                    \
226:   return result;                                                                                             \
227: }
228: 
229: DEFINE_PRIMITIVE_NON_VIRTUAL_CALL(jboolean, Boolean)
230: DEFINE_PRIMITIVE_NON_VIRTUAL_CALL(jbyte, Byte)
231: DEFINE_PRIMITIVE_NON_VIRTUAL_CALL(jchar, Char)
232: DEFINE_PRIMITIVE_NON_VIRTUAL_CALL(jshort, Short)
233: DEFINE_PRIMITIVE_NON_VIRTUAL_CALL(jint, Int)
234: DEFINE_PRIMITIVE_NON_VIRTUAL_CALL(jlong, Long)
235: DEFINE_PRIMITIVE_NON_VIRTUAL_CALL(jfloat, Float)
236: DEFINE_PRIMITIVE_NON_VIRTUAL_CALL(jdouble, Double)
237: #pragma pop_macro("DEFINE_PRIMITIVE_NON_VIRTUAL_CALL")
238: 
239: /// JNonvirtualMethod specialization for references that wraps the return value in a @ref local_ref
240: template<typename R, typename... Args>
````
- EN: Declares or extends types including `JClass`.
- CN: 声明或扩展类型，包括 `JClass`。
- EN: Implements callable logic such as `operator`, `void`, `push_macro`.
- CN: 实现可调用逻辑，例如 `operator`, `void`, `push_macro`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````cpp
241: class JNonvirtualMethod<R(Args...)> : public JMethodBase {
242:  public:
243:   using JniRet = typename detail::Convert<typename std::decay<R>::type>::jniType;
244:   static_assert(
245:       IsPlainJniReference<JniRet>() || detail::IsJavaClassType<JniRet>(),
246:       "Return type must be a JNI reference or JavaClass type.");
247:   using JMethodBase::JMethodBase;
248:   JNonvirtualMethod() noexcept {};
249:   JNonvirtualMethod(const JNonvirtualMethod& other) noexcept = default;
250: 
251:   /// Invoke a method and return a local reference wrapping the result
252:   local_ref<JniRet> operator()(alias_ref<jobject> self, alias_ref<jclass> cls, Args... args) const {
253:     const auto env = Environment::current();
254:     auto result = env->CallNonvirtualObjectMethod(
255:           self.get(),
256:           cls.get(),
257:           getId(),
258:           detail::callToJni(detail::Convert<typename std::decay<Args>::type>::toCall(args))...);
259:     FACEBOOK_JNI_THROW_PENDING_EXCEPTION();
260:     return adopt_local(static_cast<JniType<JniRet>>(result));
261:   }
262: 
263:   friend class JClass;
264: };
265: 
266: template <typename... Args>
267: local_ref<jobject> slowCall(jmethodID method_id, alias_ref<jobject> self, Args... args) {
268:     static auto invoke = findClassStatic("java/lang/reflect/Method")
269:       ->getMethod<jobject(jobject, JArrayClass<jobject>::javaobject)>("invoke");
270:     // TODO(xxxxxxx): Provide fbjni interface to ToReflectedMethod.
271:     auto reflected = adopt_local(Environment::current()->ToReflectedMethod(self->getClass().get(), method_id, JNI_FALSE));
272:     FACEBOOK_JNI_THROW_PENDING_EXCEPTION();
273:     if (!reflected) throw std::runtime_error("Unable to get reflected java.lang.reflect.Method");
274:     auto argsArray = makeArgsArray(args...);
275:     // No need to check for exceptions since invoke is itself a JMethod that will do that for us.
276:     return invoke(reflected, self.get(), argsArray.get());
277: }
278: 
279: 
280: // JField<T> ///////////////////////////////////////////////////////////////////////////////////////
281: 
282: template<typename T>
283: inline JField<T>::JField(jfieldID field) noexcept
284:   : field_id_{field}
285: {}
286: 
287: template<typename T>
288: inline JField<T>::operator bool() const noexcept {
289:   return field_id_ != nullptr;
290: }
291: 
292: template<typename T>
293: inline jfieldID JField<T>::getId() const noexcept {
294:   return field_id_;
295: }
296: 
297: #pragma push_macro("DEFINE_FIELD_PRIMITIVE_GET_SET")
298: #undef DEFINE_FIELD_PRIMITIVE_GET_SET
299: #define DEFINE_FIELD_PRIMITIVE_GET_SET(TYPE, METHOD)                 \
300: template<>                                                           \
````
- EN: Declares or extends types including `JNonvirtualMethod`, `JClass`, `to`.
- CN: 声明或扩展类型，包括 `JNonvirtualMethod`, `JClass`, `to`。
- EN: Implements callable logic such as `operator`, `slowCall`.
- CN: 实现可调用逻辑，例如 `operator`, `slowCall`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````cpp
301: inline TYPE JField<TYPE>::get(jobject object) const noexcept {       \
302:   const auto env = Environment::current();                           \
303:   return env->Get ## METHOD ## Field(object, field_id_);             \
304: }                                                                    \
305:                                                                      \
306: template<>                                                           \
307: inline void JField<TYPE>::set(jobject object, TYPE value) noexcept { \
308:   const auto env = Environment::current();                           \
309:   env->Set ## METHOD ## Field(object, field_id_, value);             \
310: }
311: 
312: DEFINE_FIELD_PRIMITIVE_GET_SET(jboolean, Boolean)
313: DEFINE_FIELD_PRIMITIVE_GET_SET(jbyte, Byte)
314: DEFINE_FIELD_PRIMITIVE_GET_SET(jchar, Char)
315: DEFINE_FIELD_PRIMITIVE_GET_SET(jshort, Short)
316: DEFINE_FIELD_PRIMITIVE_GET_SET(jint, Int)
317: DEFINE_FIELD_PRIMITIVE_GET_SET(jlong, Long)
318: DEFINE_FIELD_PRIMITIVE_GET_SET(jfloat, Float)
319: DEFINE_FIELD_PRIMITIVE_GET_SET(jdouble, Double)
320: #pragma pop_macro("DEFINE_FIELD_PRIMITIVE_GET_SET")
321: 
322: template<typename T>
323: inline T JField<T>::get(jobject object) const noexcept {
324:   return static_cast<T>(Environment::current()->GetObjectField(object, field_id_));
325: }
326: 
327: template<typename T>
328: inline void JField<T>::set(jobject object, T value) noexcept {
329:   Environment::current()->SetObjectField(object, field_id_, static_cast<jobject>(value));
330: }
331: 
332: // JStaticField<T> /////////////////////////////////////////////////////////////////////////////////
333: 
334: template<typename T>
335: inline JStaticField<T>::JStaticField(jfieldID field) noexcept
336:   : field_id_{field}
337: {}
338: 
339: template<typename T>
340: inline JStaticField<T>::operator bool() const noexcept {
341:   return field_id_ != nullptr;
342: }
343: 
344: template<typename T>
345: inline jfieldID JStaticField<T>::getId() const noexcept {
346:   return field_id_;
347: }
348: 
349: #pragma push_macro("DEFINE_STATIC_FIELD_PRIMITIVE_GET_SET")
350: #undef DEFINE_STATIC_FIELD_PRIMITIVE_GET_SET
351: #define DEFINE_STATIC_FIELD_PRIMITIVE_GET_SET(TYPE, METHOD)                \
352: template<>                                                                 \
353: inline TYPE JStaticField<TYPE>::get(jclass jcls) const noexcept {          \
354:   const auto env = Environment::current();                                 \
355:   return env->GetStatic ## METHOD ## Field(jcls, field_id_);               \
356: }                                                                          \
357:                                                                            \
358: template<>                                                                 \
359: inline void JStaticField<TYPE>::set(jclass jcls, TYPE value) noexcept {    \
360:   const auto env = Environment::current();                                 \
````
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-412
````cpp
361:   env->SetStatic ## METHOD ## Field(jcls, field_id_, value);               \
362: }
363: 
364: DEFINE_STATIC_FIELD_PRIMITIVE_GET_SET(jboolean, Boolean)
365: DEFINE_STATIC_FIELD_PRIMITIVE_GET_SET(jbyte, Byte)
366: DEFINE_STATIC_FIELD_PRIMITIVE_GET_SET(jchar, Char)
367: DEFINE_STATIC_FIELD_PRIMITIVE_GET_SET(jshort, Short)
368: DEFINE_STATIC_FIELD_PRIMITIVE_GET_SET(jint, Int)
369: DEFINE_STATIC_FIELD_PRIMITIVE_GET_SET(jlong, Long)
370: DEFINE_STATIC_FIELD_PRIMITIVE_GET_SET(jfloat, Float)
371: DEFINE_STATIC_FIELD_PRIMITIVE_GET_SET(jdouble, Double)
372: #pragma pop_macro("DEFINE_STATIC_FIELD_PRIMITIVE_GET_SET")
373: 
374: template<typename T>
375: inline T JStaticField<T>::get(jclass jcls) const noexcept {
376:   const auto env = Environment::current();
377:   return static_cast<T>(env->GetStaticObjectField(jcls, field_id_));
378: }
379: 
380: template<typename T>
381: inline void JStaticField<T>::set(jclass jcls, T value) noexcept {
382:   Environment::current()->SetStaticObjectField(jcls, field_id_, value);
383: }
384: 
385: 
386: // jmethod_traits //////////////////////////////////////////////////////////////////////////////////
387: 
388: // TODO(T6608405) Adapt this to implement a register natives method that requires no descriptor
389: namespace internal {
390: 
391: template<typename Head>
392: constexpr auto /* detail::SimpleFixedString<_> */ JavaDescriptor() {
393:   return jtype_traits<Head>::kDescriptor;
394: }
395: 
396: template<typename Head, typename Elem, typename... Tail>
397: constexpr auto /* detail::SimpleFixedString<_> */ JavaDescriptor() {
398:   return JavaDescriptor<Head>() + JavaDescriptor<Elem, Tail...>();
399: }
400: 
401: template<typename R, typename Arg1, typename... Args>
402: constexpr auto /* detail::SimpleFixedString<_> */ JMethodDescriptor() {
403:   return "(" + JavaDescriptor<Arg1, Args...>() + ")" + JavaDescriptor<R>();
404: }
405: 
406: template<typename R>
407: constexpr auto /* detail::SimpleFixedString<_> */  JMethodDescriptor() {
408:   return "()" + JavaDescriptor<R>();
409: }
410: 
411: } // internal
412: }}
````
- EN: Implements callable logic such as `JavaDescriptor`, `JMethodDescriptor`.
- CN: 实现可调用逻辑，例如 `JavaDescriptor`, `JMethodDescriptor`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `ArgsArraySetter` / 符号 `ArgsArraySetter`
- Symbol `JMethod` / 符号 `JMethod`
- Symbol `JClass` / 符号 `JClass`
- Symbol `JStaticMethod` / 符号 `JStaticMethod`

## Dependencies / 依赖关系
- C/C++ includes: `jni.h`, `fbjni/detail/SimpleFixedString.h`, `Common.h`, `Exceptions.h`, `MetaConvert.h`, `References.h`, `Boxed.h`
- C/C++ 头文件: `jni.h`, `fbjni/detail/SimpleFixedString.h`, `Common.h`, `Exceptions.h`, `MetaConvert.h`, `References.h`, `Boxed.h`
