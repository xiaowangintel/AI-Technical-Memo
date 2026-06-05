# MetaConvert.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/MetaConvert.h`
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
19: #include <jni.h>
20: 
21: #include "Common.h"
22: #include "References.h"
23: 
24: namespace facebook {
25: namespace jni {
26: 
27: namespace detail {
28: 
29: // In order to avoid potentially filling the jni locals table,
30: // temporary objects (right now, this is just jstrings) need to be
31: // released. This is done by returning a holder which autoconverts to
32: // jstring.
33: template <typename T>
34: inline T callToJni(T&& t) {
35:   return t;
36: }
37: 
38: template <typename T>
39: inline JniType<T> callToJni(local_ref<T>&& sref) {
40:   return sref.get();
````
- EN: Pulls in native headers such as `jni.h`, `Common.h`, `References.h`.
- CN: 引入原生头文件，例如 `jni.h`, `Common.h`, `References.h`。
- EN: Implements callable logic such as `objects`, `callToJni`.
- CN: 实现可调用逻辑，例如 `objects`, `callToJni`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 41-80
````cpp
41: }
42: 
43: template<typename T>
44: enable_if_t<IsPlainJniReference<T>(), T> toPlainJniReference(T obj) {
45:   return obj;
46: }
47: 
48: template<typename T>
49: enable_if_t<IsJavaClassType<T>(), JniType<T>> toPlainJniReference(T repr) {
50:   return ReprAccess<T>::get(repr);
51: }
52: 
53: // Normally, pass through types unmolested.
54: template <typename T, typename Enabled = void>
55: struct Convert {
56:   typedef T jniType;
57:   static jniType fromJni(jniType t) {
58:     return t;
59:   }
60:   static jniType toJniRet(jniType t) {
61:     return t;
62:   }
63:   static jniType toCall(jniType t) {
64:     return t;
65:   }
66: };
67: 
68: // This is needed for return conversion
69: template <>
70: struct Convert<void> {
71:   typedef void jniType;
72: };
73: 
74: // jboolean is an unsigned char, not a bool. Allow it to work either way.
75: template<>
76: struct Convert<bool> {
77:   typedef jboolean jniType;
78:   static bool fromJni(jniType t) {
79:     return t;
80:   }
````
- EN: Declares or extends types including `Convert`.
- CN: 声明或扩展类型，包括 `Convert`。
- EN: Implements callable logic such as `toPlainJniReference`, `fromJni`, `toJniRet`, `toCall`.
- CN: 实现可调用逻辑，例如 `toPlainJniReference`, `fromJni`, `toJniRet`, `toCall`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````cpp
 81:   static jniType toJniRet(bool t) {
 82:     return t;
 83:   }
 84:   static jniType toCall(bool t) {
 85:     return t;
 86:   }
 87: };
 88: 
 89: // Sometimes (64-bit Android) jlong is "long long", but int64_t is "long".
 90: // Allow int64_t to work as jlong.
 91: template<typename T>
 92: struct Convert<T,
 93:     typename std::enable_if<
 94:       (std::is_same<T, long long>::value || std::is_same<T, int64_t>::value) && !std::is_same<T, jlong>::value
 95:     >::type> {
 96:   typedef jlong jniType;
 97:   static T fromJni(jniType t) {
 98:     return t;
 99:   }
100:   static jniType toJniRet(T t) {
101:     return t;
102:   }
103:   static jniType toCall(T t) {
104:     return t;
105:   }
106: };
107: 
108: // convert to alias_ref<T> from T
109: template <typename T>
110: struct Convert<alias_ref<T>> {
111:   typedef JniType<T> jniType;
112:   static alias_ref<jniType> fromJni(jniType t) {
113:     return wrap_alias(t);
114:   }
115:   static jniType toJniRet(alias_ref<jniType> t) {
116:     return t.get();
117:   }
118:   static jniType toCall(const alias_ref<jniType>& t) {
119:     return t.get();
120:   }
````
- EN: Declares or extends types including `Convert`.
- CN: 声明或扩展类型，包括 `Convert`。
- EN: Implements callable logic such as `toJniRet`, `toCall`, `fromJni`.
- CN: 实现可调用逻辑，例如 `toJniRet`, `toCall`, `fromJni`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-160
````cpp
121: };
122: 
123: // convert return from local_ref<T>
124: template <typename T>
125: struct Convert<local_ref<T>> {
126:   typedef JniType<T> jniType;
127:   // No automatic synthesis of local_ref
128:   static jniType toJniRet(local_ref<jniType> t) {
129:     return t.release();
130:   }
131:   static jniType toCall(const local_ref<jniType>& t) {
132:     return t.get();
133:   }
134: };
135: 
136: // convert return from global_ref<T>
137: template <typename T>
138: struct Convert<global_ref<T>> {
139:   typedef JniType<T> jniType;
140:   // No automatic synthesis of global_ref
141:   static jniType toJniRet(global_ref<jniType>&& t) {
142:     // If this gets called, ownership the global_ref was passed in here.  (It's
143:     // probably a copy of a persistent global_ref made when a function was
144:     // declared to return a global_ref, but it could moved out or otherwise not
145:     // referenced elsewhere.  Doesn't matter.)  Either way, the only safe way
146:     // to return it is to make a local_ref, release it, and return the
147:     // underlying local jobject.
148:     auto ret = make_local(t);
149:     return ret.release();
150:   }
151:   static jniType toJniRet(const global_ref<jniType>& t) {
152:     // If this gets called, the function was declared to return const&.  We
153:     // have a ref to a global_ref whose lifetime will exceed this call, so we
154:     // can just get the underlying jobject and return it to java without
155:     // needing to make a local_ref.
156:     return t.get();
157:   }
158:   static jniType toCall(const global_ref<jniType>& t) {
159:     return t.get();
160:   }
````
- EN: Declares or extends types including `Convert`.
- CN: 声明或扩展类型，包括 `Convert`。
- EN: Implements callable logic such as `toJniRet`, `toCall`.
- CN: 实现可调用逻辑，例如 `toJniRet`, `toCall`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 161-179
````cpp
161: };
162: 
163: template <typename T> struct jni_sig_from_cxx_t;
164: template <typename R, typename... Args>
165: struct jni_sig_from_cxx_t<R(Args...)> {
166:   using JniRet = typename Convert<typename std::decay<R>::type>::jniType;
167:   using JniSig = JniRet(typename Convert<typename std::decay<Args>::type>::jniType...);
168: };
169: 
170: template <typename T>
171: using jni_sig_from_cxx = typename jni_sig_from_cxx_t<T>::JniSig;
172: 
173: } // namespace detail
174: 
175: template <typename R, typename... Args>
176: struct jmethod_traits_from_cxx<R(Args...)> : jmethod_traits<detail::jni_sig_from_cxx<R(Args...)>> {
177: };
178: 
179: }}
````
- EN: Declares or extends types including `jni_sig_from_cxx_t`, `jmethod_traits_from_cxx`.
- CN: 声明或扩展类型，包括 `jni_sig_from_cxx_t`, `jmethod_traits_from_cxx`。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `Convert` / 符号 `Convert`
- Symbol `jni_sig_from_cxx_t` / 符号 `jni_sig_from_cxx_t`
- Symbol `jmethod_traits_from_cxx` / 符号 `jmethod_traits_from_cxx`
- Symbol `objects` / 符号 `objects`

## Dependencies / 依赖关系
- C/C++ includes: `jni.h`, `Common.h`, `References.h`
- C/C++ 头文件: `jni.h`, `Common.h`, `References.h`
