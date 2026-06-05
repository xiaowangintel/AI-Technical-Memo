# byte_buffer_tests.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/test/jni/byte_buffer_tests.cpp`
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
17: #include <vector>
18: 
19: #include <fbjni/fbjni.h>
20: #include <fbjni/ByteBuffer.h>
21: 
22: #include "expect.h"
23: 
24: using namespace facebook::jni;
25: 
26: namespace {
27: 
28: std::vector<uint8_t> vec{1, 0, 0, 0};
29: 
30: size_t ByteBufferCapacity(alias_ref<JByteBuffer> buffer) {
31:   static auto meth = JByteBuffer::javaClassStatic()->getMethod<int()>("capacity");
32:   return meth(buffer);
33: }
34: 
35: jboolean testDirectByteBuffer(JNIEnv*, jclass) {
36:   std::vector<uint8_t> vec{5, 4, 3, 2, 1, 0};
37:   auto nbb = JByteBuffer::wrapBytes(vec.data(), vec.size());
38: 
39:   EXPECT(ByteBufferCapacity(nbb) == vec.size());
40:   EXPECT(nbb->isDirect());
````
- EN: Pulls in native headers such as `vector`, `fbjni/fbjni.h`, `fbjni/ByteBuffer.h`, `expect.h`.
- CN: 引入原生头文件，例如 `vector`, `fbjni/fbjni.h`, `fbjni/ByteBuffer.h`, `expect.h`。
- EN: Implements callable logic such as `ByteBufferCapacity`, `testDirectByteBuffer`.
- CN: 实现可调用逻辑，例如 `ByteBufferCapacity`, `testDirectByteBuffer`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 41-80
````cpp
41:   EXPECT(nbb->getDirectSize() == vec.size());
42: 
43:   auto bytes = nbb->getDirectBytes();
44: 
45:   for (size_t i = 0; i < vec.size(); i++) {
46:     EXPECT(bytes[i] == vec[i]);
47:   }
48:   return JNI_TRUE;
49: }
50: 
51: jboolean testEmptyDirectByteBuffer(JNIEnv*, jclass) {
52:   uint8_t data;
53:   auto nbb = JByteBuffer::wrapBytes(&data, 0);
54: 
55:   EXPECT(ByteBufferCapacity(nbb) == 0);
56:   EXPECT(nbb->isDirect());
57:   EXPECT(nbb->getDirectSize() == 0);
58: 
59:   return JNI_TRUE;
60: }
61: 
62: jboolean testRewindBuffer(alias_ref<jobject> self) {
63:   std::vector<uint8_t> vec{0, 0, 0, 0, 0, 0, 0, 0};
64:   auto nbb = JByteBuffer::wrapBytes(vec.data(), vec.size());
65: 
66:   auto cls = self->getClass();
67:   auto writeBytes =
68:     cls->getStaticMethod<void(JByteBuffer, jbyte, jbyte, jbyte, jbyte)>("writeBytes");
69: 
70:   writeBytes(cls, *nbb, 0, 1, 2, 3);
71:   nbb->rewind();
72:   writeBytes(cls, *nbb, 4, 5, 6, 7);
73: 
74:   EXPECT(vec[0] == 4);
75:   EXPECT(vec[1] == 5);
76:   EXPECT(vec[2] == 6);
77:   EXPECT(vec[3] == 7);
78:   EXPECT(vec[4] == 0);
79:   EXPECT(vec[5] == 0);
80:   EXPECT(vec[6] == 0);
````
- EN: Implements callable logic such as `testEmptyDirectByteBuffer`, `testRewindBuffer`.
- CN: 实现可调用逻辑，例如 `testEmptyDirectByteBuffer`, `testRewindBuffer`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````cpp
 81:   EXPECT(vec[7] == 0);
 82: 
 83:   return JNI_TRUE;
 84: }
 85: 
 86: local_ref<JByteBuffer> nativeAllocateDirect(alias_ref<jobject> self, int size) {
 87:   return JByteBuffer::allocateDirect(size);
 88: }
 89: 
 90: jboolean testFloatBuffer(alias_ref<jobject> self, alias_ref<facebook::jni::JBuffer> buffer) {
 91:   EXPECT(buffer->isDirect());
 92:   EXPECT(buffer->getDirectCapacity() == 5);
 93:   float* raw = (float*)buffer->getDirectAddress();
 94:   EXPECT(raw);
 95: 
 96:   EXPECT(raw[0] == 1);
 97:   EXPECT(raw[1] == 2);
 98:   EXPECT(raw[2] == 2.5);
 99:   EXPECT(raw[3] == 2.75);
100:   EXPECT(raw[4] == 3);
101: 
102:   return JNI_TRUE;
103: }
104: 
105: local_ref<JByteBuffer> nativeByteBufferOrder(alias_ref<jobject> self) {
106:   auto nbb = JByteBuffer::wrapBytes(vec.data(), vec.size());
107:   return nbb->order(JByteOrder::nativeOrder());
108: }
109: 
110: }
111: 
112: void RegisterByteBufferTests() {
113:   registerNatives("com/facebook/jni/ByteBufferTests", {
114:     makeNativeMethod("nativeTestDirectByteBuffer", testDirectByteBuffer),
115:     makeNativeMethod("nativeTestEmptyDirectByteBuffer", testEmptyDirectByteBuffer),
116:     makeNativeMethod("nativeTestRewindBuffer", testRewindBuffer),
117:     makeNativeMethod("nativeAllocateDirect", nativeAllocateDirect),
118:     makeNativeMethod("nativeTestFloatBuffer", testFloatBuffer),
119:     makeNativeMethod("nativeByteBufferOrder", nativeByteBufferOrder),
120:   });
````
- EN: Implements callable logic such as `nativeAllocateDirect`, `testFloatBuffer`, `nativeByteBufferOrder`, `RegisterByteBufferTests`.
- CN: 实现可调用逻辑，例如 `nativeAllocateDirect`, `testFloatBuffer`, `nativeByteBufferOrder`, `RegisterByteBufferTests`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-121
````cpp
121: }
````
- EN: This range contributes implementation details for the file goal: Implements JNI bridge abstractions used by Android-facing native code.
- CN: 该范围为文件目标提供实现细节：实现供 Android 原生代码使用的 JNI 桥接抽象。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Validation and test coverage / 校验与测试覆盖
- Symbol `ByteBufferCapacity` / 符号 `ByteBufferCapacity`
- Symbol `testDirectByteBuffer` / 符号 `testDirectByteBuffer`
- Symbol `testEmptyDirectByteBuffer` / 符号 `testEmptyDirectByteBuffer`
- Symbol `testRewindBuffer` / 符号 `testRewindBuffer`

## Dependencies / 依赖关系
- C/C++ includes: `vector`, `fbjni/fbjni.h`, `fbjni/ByteBuffer.h`, `expect.h`
- C/C++ 头文件: `vector`, `fbjni/fbjni.h`, `fbjni/ByteBuffer.h`, `expect.h`
