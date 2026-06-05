# Registration-inl.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/Registration-inl.h`
- Repository: `pytorch`
- Purpose (EN): Implements JNI bridge abstractions used by Android-facing native code.
- 用途 (CN): 实现供 Android 原生代码使用的 JNI 桥接抽象。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
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
19: #include "Exceptions.h"
20: #include "Hybrid.h"
21: 
22: namespace facebook {
23: namespace jni {
24: 
25: namespace detail {
26: 
27: #ifdef __i386__
28: // X86 ABI forces 16 byte stack allignment on calls. Unfortunately
29: // sometimes Dalvik chooses not to obey the ABI:
30: // - https://code.google.com/p/android/issues/detail?id=61012
31: // - https://android.googlesource.com/platform/ndk/+/81696d2%5E!/
32: // Therefore, we tell the compiler to re-align the stack on entry
33: // to our JNI functions.
34: #define JNI_ENTRY_POINT __attribute__((force_align_arg_pointer))
35: #else
36: #define JNI_ENTRY_POINT
37: #endif
38: 
39: template <typename R>
40: struct CreateDefault {
````
- EN: Pulls in native headers such as `Exceptions.h`, `Hybrid.h`.
- CN: 引入原生头文件，例如 `Exceptions.h`, `Hybrid.h`。
- EN: Declares or extends types including `CreateDefault`.
- CN: 声明或扩展类型，包括 `CreateDefault`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````cpp
41:   static R create() {
42:     return R{};
43:   }
44: };
45: 
46: template <>
47: struct CreateDefault<void> {
48:   static void create() {}
49: };
50: 
51: template <typename R>
52: using Converter = Convert<typename std::decay<R>::type>;
53: 
54: template <typename F, typename R, typename C, typename... Args>
55: struct CallWithJniConversions {
56:   static typename Converter<R>::jniType call(JniType<C> obj, typename Converter<Args>::jniType... args, F func) {
57:     return Converter<R>::toJniRet(func(obj, Converter<Args>::fromJni(args)...));
58:   }
59: };
60: 
61: template <typename F, typename C, typename... Args>
62: struct CallWithJniConversions<F, void, C, Args...> {
63:   static void call(JniType<C> obj, typename Converter<Args>::jniType... args, F func) {
64:     func(obj, Converter<Args>::fromJni(args)...);
65:   }
66: };
67: 
68: // registration wrapper for legacy JNI-style functions
69: template<typename F, F func, typename C, typename R, typename... Args>
70: struct BareJniWrapper {
71:   JNI_ENTRY_POINT static R call(JNIEnv* env, jobject obj, Args... args) {
72:     detail::JniEnvCacher jec(env);
73:     try {
74:       return (*func)(env, static_cast<JniType<C>>(obj), args...);
75:     } catch (...) {
76:       translatePendingCppExceptionToJavaException();
77:       return CreateDefault<R>::create();
78:     }
79:   }
80: };
````
- EN: Declares or extends types including `CreateDefault`, `CallWithJniConversions`, `BareJniWrapper`.
- CN: 声明或扩展类型，包括 `CreateDefault`, `CallWithJniConversions`, `BareJniWrapper`。
- EN: Implements callable logic such as `create`, `call`.
- CN: 实现可调用逻辑，例如 `create`, `call`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````cpp
 81: 
 82: // registration wrappers for functions, with autoconversion of arguments.
 83: template<typename F, typename C, typename R, typename... Args>
 84: struct FunctionWrapper {
 85:   using jniRet = typename Converter<R>::jniType;
 86:   static jniRet call(JNIEnv* env, jobject obj, typename Converter<Args>::jniType... args, F funcPtr) {
 87:     detail::JniEnvCacher jec(env);
 88:     try {
 89:       return CallWithJniConversions<F, R, JniType<C>, Args...>::call(
 90:           static_cast<JniType<C>>(obj), args..., funcPtr);
 91:     } catch (...) {
 92:       translatePendingCppExceptionToJavaException();
 93:       return CreateDefault<jniRet>::create();
 94:     }
 95:   }
 96: };
 97: 
 98: // registration wrappers for functions, with autoconversion of arguments.
 99: // This is a separate class from FunctionWrapper because
100: // MethodWrapper::call does not want FunctionWrapper::call to be a
101: // JNI_ENTRY_POINT and thus not inlinable. However, we still want a
102: // JNI_ENTRY_POINT for top-level functions.
103: template<typename F, F func, typename C, typename R, typename... Args>
104: struct FunctionWrapperWithJniEntryPoint {
105:   using jniRet = typename FunctionWrapper<F, C, R, Args...>::jniRet;
106:   JNI_ENTRY_POINT static jniRet call(JNIEnv* env, jobject obj, typename Converter<Args>::jniType... args) {
107:     return FunctionWrapper<F, C, R, Args...>::call(env, obj, args..., func);
108:   }
109: };
110: 
111: // registration wrappers for non-static methods, with autoconvertion of arguments.
112: template<typename M, M method, typename C, typename R, typename... Args>
113: struct MethodWrapper {
114:   using jhybrid = typename C::jhybridobject;
115:   static R dispatch(alias_ref<jhybrid> ref, Args&&... args) {
116:     try {
117:       // This is usually a noop, but if the hybrid object is a
118:       // base class of other classes which register JNI methods,
119:       // this will get the right type for the registered method.
120:       auto cobj = static_cast<C*>(ref->cthis());
````
- EN: Declares or extends types including `FunctionWrapper`, `from`, `FunctionWrapperWithJniEntryPoint`, `MethodWrapper`.
- CN: 声明或扩展类型，包括 `FunctionWrapper`, `from`, `FunctionWrapperWithJniEntryPoint`, `MethodWrapper`。
- EN: Implements callable logic such as `call`, `dispatch`.
- CN: 实现可调用逻辑，例如 `call`, `dispatch`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-160
````cpp
121:       return (cobj->*method)(std::forward<Args>(args)...);
122:     } catch (const std::exception& ex) {
123:       C::mapException(ex);
124:       throw;
125:     }
126:   }
127: 
128:   JNI_ENTRY_POINT static typename Converter<R>::jniType call(
129:       JNIEnv* env, jobject obj, typename Converter<Args>::jniType... args) {
130:     return FunctionWrapper<R(*)(alias_ref<jhybrid>, Args&&...), jhybrid, R, Args...>::call(env, obj, args..., dispatch);
131:   }
132: };
133: 
134: template<typename F, F func, typename C, typename R, typename... Args>
135: constexpr inline void* exceptionWrapJNIMethod(R (*)(JNIEnv*, C, Args... args)) {
136:   // This intentionally erases the real type; JNI will do it anyway
137:   return (void*)(&(BareJniWrapper<F, func, C, R, Args...>::call));
138: }
139: 
140: template<typename F, F func, typename C, typename R, typename... Args>
141: constexpr inline void* exceptionWrapJNIMethod(R (*)(alias_ref<C>, Args... args)) {
142:   // This intentionally erases the real type; JNI will do it anyway
143:   return (void*)(&(FunctionWrapperWithJniEntryPoint<F, func, C, R, Args...>::call));
144: }
145: 
146: template<typename M, M method, typename C, typename R, typename... Args>
147: constexpr inline void* exceptionWrapJNIMethod(R (C::*method0)(Args... args)) {
148:   (void)method0;
149:   // This intentionally erases the real type; JNI will do it anyway
150:   return (void*)(&(MethodWrapper<M, method, C, R, Args...>::call));
151: }
152: 
153: template<typename R, typename C, typename... Args>
154: inline constexpr const auto& /* detail::SimpleFixedString<_> */ makeDescriptor(R (*)(JNIEnv*, C, Args... args)) {
155:   return jmethod_traits<R(Args...)>::kDescriptor;
156: }
157: 
158: template<typename R, typename C, typename... Args>
159: inline constexpr const auto& /* detail::SimpleFixedString<_> */ makeDescriptor(R (*)(alias_ref<C>, Args... args)) {
160:   return jmethod_traits_from_cxx<R(Args...)>::kDescriptor;
````
- EN: Implements callable logic such as `call`, `exceptionWrapJNIMethod`, `makeDescriptor`.
- CN: 实现可调用逻辑，例如 `call`, `exceptionWrapJNIMethod`, `makeDescriptor`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 161-189
````cpp
161: }
162: 
163: template<typename R, typename C, typename... Args>
164: inline constexpr const auto& /* detail::SimpleFixedString<_> */ makeDescriptor(R (C::*)(Args... args)) {
165:   return jmethod_traits_from_cxx<R(Args...)>::kDescriptor;
166: }
167: 
168: template<typename R, typename ...Args>
169: template<R(*func)(Args...)>
170: JNI_ENTRY_POINT R CriticalMethod<R(*)(Args...)>::call(alias_ref<jclass>, Args... args) noexcept {
171:   static_assert(
172:     IsJniPrimitive<R>() || std::is_void<R>(),
173:     "Critical Native Methods may only return primitive JNI types, or void.");
174:   static_assert(
175:     AreJniPrimitives<Args...>(),
176:     "Critical Native Methods may only use primitive JNI types as parameters");
177: 
178:   return func(std::forward<Args>(args)...);
179: }
180: 
181: template<typename R, typename ...Args>
182: template<R(*func)(Args...)>
183: inline constexpr auto CriticalMethod<R(*)(Args...)>::desc() {
184:   return makeDescriptor(call<func>);
185: }
186: 
187: }
188: 
189: }}
````
- EN: Implements callable logic such as `makeDescriptor`, `R`.
- CN: 实现可调用逻辑，例如 `makeDescriptor`, `R`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `CreateDefault` / 符号 `CreateDefault`
- Symbol `CallWithJniConversions` / 符号 `CallWithJniConversions`
- Symbol `BareJniWrapper` / 符号 `BareJniWrapper`
- Symbol `FunctionWrapper` / 符号 `FunctionWrapper`

## Dependencies / 依赖关系
- C/C++ includes: `Exceptions.h`, `Hybrid.h`
- C/C++ 头文件: `Exceptions.h`, `Hybrid.h`
