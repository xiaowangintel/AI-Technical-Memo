# ByteBuffer.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/ByteBuffer.cpp`
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
17: #include <fbjni/ByteBuffer.h>
18: 
19: #include <stdexcept>
20: 
21: namespace facebook {
22: namespace jni {
23: 
24: void JBuffer::rewind() const {
25:   static auto meth = javaClassStatic()->getMethod<alias_ref<JBuffer>()>("rewind");
26:   meth(self());
27: }
28: 
29: void* JBuffer::getDirectAddress() const {
30:   if (!self()) {
31:     throwNewJavaException("java/lang/NullPointerException", "java.lang.NullPointerException");
32:   }
33:   void* addr = Environment::current()->GetDirectBufferAddress(self());
34:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();
35:   if (!addr) {
36:     throw std::runtime_error(
37:         isDirect() ?
38:           "Attempt to get direct bytes of non-direct buffer." :
39:           "Error getting direct bytes of buffer.");
40:   }
````
- EN: Pulls in native headers such as `fbjni/ByteBuffer.h`, `stdexcept`.
- CN: 引入原生头文件，例如 `fbjni/ByteBuffer.h`, `stdexcept`。
- EN: Implements callable logic such as `JBuffer::rewind`, `JBuffer::getDirectAddress`.
- CN: 实现可调用逻辑，例如 `JBuffer::rewind`, `JBuffer::getDirectAddress`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````cpp
41:   return addr;
42: }
43: 
44: size_t JBuffer::getDirectCapacity() const {
45:   if (!self()) {
46:     throwNewJavaException("java/lang/NullPointerException", "java.lang.NullPointerException");
47:   }
48:   int size = Environment::current()->GetDirectBufferCapacity(self());
49:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();
50:   if (size < 0) {
51:     throw std::runtime_error(
52:         isDirect() ?
53:           "Attempt to get direct size of non-direct buffer." :
54:           "Error getting direct size of buffer.");
55:   }
56:   return static_cast<size_t>(size);
57: }
58: 
59: bool JBuffer::isDirect() const {
60:   static auto meth = javaClassStatic()->getMethod<jboolean()>("isDirect");
61:   return meth(self());
62: }
63: 
64: local_ref<JByteOrder> JByteOrder::nativeOrder() {
65:   static auto meth = JByteOrder::javaClassStatic()->getStaticMethod<
66:       local_ref<JByteOrder>()>
67:       ("nativeOrder");
68:   return meth(JByteOrder::javaClassStatic());
69: }
70: 
71: local_ref<JByteBuffer> JByteBuffer::wrapBytes(uint8_t* data, size_t size) {
72:   // env->NewDirectByteBuffer requires that size is positive. Android's
73:   // dalvik returns an invalid result and Android's art aborts if size == 0.
74:   // Workaround this by using a slow path through Java in that case.
75:   if (!size) {
76:     return allocateDirect(0);
77:   }
78:   auto res = adopt_local(static_cast<javaobject>(Environment::current()->NewDirectByteBuffer(data, size)));
79:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();
80:   if (!res) {
````
- EN: Implements callable logic such as `JBuffer::getDirectCapacity`, `JBuffer::isDirect`, `JByteOrder::nativeOrder`, `JByteBuffer::wrapBytes`.
- CN: 实现可调用逻辑，例如 `JBuffer::getDirectCapacity`, `JBuffer::isDirect`, `JByteOrder::nativeOrder`, `JByteBuffer::wrapBytes`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-99
````cpp
81:     throw std::runtime_error("Direct byte buffers are unsupported.");
82:   }
83:   return res;
84: }
85: 
86: local_ref<JByteBuffer> JByteBuffer::allocateDirect(jint size) {
87:   static auto cls = JByteBuffer::javaClassStatic();
88:   static auto meth = cls->getStaticMethod<JByteBuffer(int)>("allocateDirect");
89:   return meth(cls, size);
90: }
91: 
92: local_ref<JByteBuffer> JByteBuffer::order(alias_ref<JByteOrder> order) {
93:   static auto meth = JByteBuffer::javaClassStatic()->getMethod<
94:       local_ref<JByteBuffer>(alias_ref<JByteOrder>)>
95:       ("order");
96:   return meth(self(), order);
97: }
98: 
99: }}
````
- EN: Implements callable logic such as `JByteBuffer::allocateDirect`, `JByteBuffer::order`.
- CN: 实现可调用逻辑，例如 `JByteBuffer::allocateDirect`, `JByteBuffer::order`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `JBuffer::rewind` / 符号 `JBuffer::rewind`
- Symbol `JBuffer::getDirectAddress` / 符号 `JBuffer::getDirectAddress`
- Symbol `JBuffer::getDirectCapacity` / 符号 `JBuffer::getDirectCapacity`
- Symbol `JBuffer::isDirect` / 符号 `JBuffer::isDirect`

## Dependencies / 依赖关系
- C/C++ includes: `fbjni/ByteBuffer.h`, `stdexcept`
- C/C++ 头文件: `fbjni/ByteBuffer.h`, `stdexcept`
