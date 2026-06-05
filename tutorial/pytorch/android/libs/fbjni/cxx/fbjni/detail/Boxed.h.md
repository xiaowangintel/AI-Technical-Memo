# Boxed.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/Boxed.h`
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
19: #include "CoreClasses.h"
20: 
21: namespace facebook {
22: namespace jni {
23: 
24: namespace detail {
25: template <typename T, typename jprim>
26: struct JPrimitive : JavaClass<T> {
27:   using typename JavaClass<T>::javaobject;
28:   using JavaClass<T>::javaClassStatic;
29:   static local_ref<javaobject> valueOf(jprim val) {
30:     static const auto cls = javaClassStatic();
31:     static const auto method =
32:       cls->template getStaticMethod<javaobject(jprim)>("valueOf");
33:     return method(cls, val);
34:   }
35:   jprim value() const {
36:     static const auto method =
37:       javaClassStatic()->template getMethod<jprim()>(T::kValueMethod);
38:     return method(this->self());
39:   }
40: };
````
- EN: Pulls in native headers such as `CoreClasses.h`.
- CN: 引入原生头文件，例如 `CoreClasses.h`。
- EN: Declares or extends types including `JPrimitive`.
- CN: 声明或扩展类型，包括 `JPrimitive`。
- EN: Implements callable logic such as `valueOf`, `value`.
- CN: 实现可调用逻辑，例如 `valueOf`, `value`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````cpp
41: 
42: } // namespace detail
43: 
44: 
45: #define DEFINE_BOXED_PRIMITIVE(LITTLE, BIG)                          \
46:   struct J ## BIG : detail::JPrimitive<J ## BIG, j ## LITTLE> {      \
47:     static auto constexpr kJavaDescriptor = "Ljava/lang/" #BIG ";";  \
48:     static auto constexpr kValueMethod = #LITTLE "Value";            \
49:     j ## LITTLE LITTLE ## Value() const {                            \
50:       return value();                                                \
51:     }                                                                \
52:   };                                                                 \
53:   inline local_ref<J ## BIG::javaobject> autobox(j ## LITTLE val) {  \
54:     return J ## BIG::valueOf(val);                                   \
55:   }
56: 
57: DEFINE_BOXED_PRIMITIVE(boolean, Boolean)
58: DEFINE_BOXED_PRIMITIVE(byte, Byte)
59: DEFINE_BOXED_PRIMITIVE(char, Character)
60: DEFINE_BOXED_PRIMITIVE(short, Short)
61: DEFINE_BOXED_PRIMITIVE(int, Integer)
62: DEFINE_BOXED_PRIMITIVE(long, Long)
63: DEFINE_BOXED_PRIMITIVE(float, Float)
64: DEFINE_BOXED_PRIMITIVE(double, Double)
65: 
66: #undef DEFINE_BOXED_PRIMITIVE
67: 
68: template<typename T>
69: inline typename std::enable_if<
70:   (std::is_same<T, long long>::value || std::is_same<T, int64_t>::value) && !std::is_same<T, jlong>::value,
71:   local_ref<jobject>
72: >::type autobox(T val) {
73:   return JLong::valueOf(val);
74: }
75: 
76: struct JVoid : public jni::JavaClass<JVoid> {
77:   static auto constexpr kJavaDescriptor = "Ljava/lang/Void;";
78: };
79: 
80: inline local_ref<jobject> autobox(alias_ref<jobject> val) {
````
- EN: Declares or extends types including `J`, `JVoid`.
- CN: 声明或扩展类型，包括 `J`, `JVoid`。
- EN: Implements callable logic such as `Value`, `autobox`, `DEFINE_BOXED_PRIMITIVE`.
- CN: 实现可调用逻辑，例如 `Value`, `autobox`, `DEFINE_BOXED_PRIMITIVE`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-84
````cpp
81:   return make_local(val);
82: }
83: 
84: }}
````
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `JPrimitive` / 符号 `JPrimitive`
- Symbol `J` / 符号 `J`
- Symbol `JVoid` / 符号 `JVoid`
- Symbol `valueOf` / 符号 `valueOf`

## Dependencies / 依赖关系
- C/C++ includes: `CoreClasses.h`
- C/C++ 头文件: `CoreClasses.h`
