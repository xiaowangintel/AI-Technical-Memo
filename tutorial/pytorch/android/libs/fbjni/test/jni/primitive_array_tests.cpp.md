# primitive_array_tests.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/test/jni/primitive_array_tests.cpp`
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
17: #include <cmath>
18: #include <vector>
19: 
20: #include <fbjni/fbjni.h>
21: 
22: #include "expect.h"
23: 
24: using namespace facebook::jni;
25: 
26: local_ref<jbooleanArray> testMakeBoolArray(alias_ref<jclass>, jint size) {
27:   return make_boolean_array(size);
28: }
29: 
30: local_ref<jbyteArray> testMakeByteArray(alias_ref<jclass>, jint size) {
31:   return make_byte_array(size);
32: }
33: 
34: local_ref<jcharArray> testMakeCharArray(alias_ref<jclass>, jint size) {
35:   return make_char_array(size);
36: }
37: 
38: local_ref<jshortArray> testMakeShortArray(alias_ref<jclass>, jint size) {
39:   return make_short_array(size);
40: }
41: 
42: local_ref<jintArray> testMakeIntArray(alias_ref<jclass>, jint size) {
43:   return make_int_array(size);
44: }
45: 
46: local_ref<jlongArray> testMakeLongArray(alias_ref<jclass>, jint size) {
47:   return make_long_array(size);
48: }
49: 
50: local_ref<jfloatArray> testMakeFloatArray(alias_ref<jclass>, jint size) {
51:   return make_float_array(size);
52: }
53: 
54: local_ref<jdoubleArray> testMakeDoubleArray(alias_ref<jclass>, jint size) {
55:   return make_double_array(size);
56: }
57: 
58: jboolean testGetSetBooleanArray(alias_ref<jclass>, alias_ref<jbooleanArray> array) {
59:   EXPECT(array);
60: 
````
- EN: Pulls in native headers such as `cmath`, `vector`, `fbjni/fbjni.h`, `expect.h`.
- CN: 引入原生头文件，例如 `cmath`, `vector`, `fbjni/fbjni.h`, `expect.h`。
- EN: Implements callable logic such as `testMakeBoolArray`, `testMakeByteArray`, `testMakeCharArray`, `testMakeShortArray`.
- CN: 实现可调用逻辑，例如 `testMakeBoolArray`, `testMakeByteArray`, `testMakeCharArray`, `testMakeShortArray`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 61-120
````cpp
 61:   auto n = array->size();
 62:   EXPECT(n == 2);
 63: 
 64:   auto vec = std::vector<jboolean>(n);
 65:   array->getRegion(0, n, vec.data());
 66:   auto smartbuf = array->getRegion(0, n);
 67: 
 68:   EXPECT(!vec[0] && vec[1]);
 69:   EXPECT(!smartbuf[0] && smartbuf[1]);
 70: 
 71:   for (auto i = 0u; i < n; ++i) {
 72:     smartbuf[i] = !smartbuf[i];
 73:   }
 74: 
 75:   array->setRegion(0, n, smartbuf.get());
 76: 
 77:   return JNI_TRUE;
 78: }
 79: 
 80: jboolean testPinBooleanArray(alias_ref<jclass>, alias_ref<jbooleanArray> array) {
 81:   EXPECT(array);
 82: 
 83:   auto n = static_cast<jboolean>(array->size());
 84:   auto pinned = array->pin();
 85: 
 86:   EXPECT(!pinned[0] && pinned[1]);
 87: 
 88:   for (auto i = 0; i < n; ++i) {
 89:     pinned[i] = !pinned[i];
 90:   }
 91: 
 92:   return JNI_TRUE;
 93: }
 94: 
 95: constexpr double kEps = 1e-3;
 96: 
 97: template <typename JArrayType>
 98: jboolean testGetSetArray(alias_ref<jclass>, alias_ref<JArrayType> array) {
 99:   EXPECT(array);
100:   int n = array->size();
101:   auto vec = std::vector<typename jtype_traits<JArrayType>::entry_type>(n);
102:   array->getRegion(0, n, vec.data());
103:   auto smartbuf = array->getRegion(0, n);
104: 
105:   for (auto i = 0; i < n; ++i) {
106:     EXPECT(std::abs(static_cast<double>(vec[i] - i)) < kEps);
107:     EXPECT(std::abs(static_cast<double>(smartbuf[i] - i)) < kEps);
108:   }
109: 
110:   for (auto i = 0; i < n; ++i) {
111:     smartbuf[i] *= 2;
112:   }
113: 
114:   array->setRegion(0, n, smartbuf.get());
115: 
116:   return JNI_TRUE;
117: }
118: 
119: template <typename JArrayType>
120: jboolean testPinArray(alias_ref<jclass>, alias_ref<JArrayType> array) {
````
- EN: Implements callable logic such as `testPinBooleanArray`, `testGetSetArray`, `testPinArray`.
- CN: 实现可调用逻辑，例如 `testPinBooleanArray`, `testGetSetArray`, `testPinArray`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````cpp
121:   EXPECT(array);
122: 
123:   int n = array->size();
124:   auto pinned = array->pin();
125: 
126:   for (auto i = 0; i < n; ++i) {
127:     EXPECT(std::abs(static_cast<double>(pinned[i] - i)) < kEps);
128:   }
129: 
130:   for (auto i = 0; i < n; ++i) {
131:     pinned[i] *= 2;
132:   }
133: 
134:   return JNI_TRUE;
135: }
136: 
137: template <typename JArrayType>
138: jboolean testPinArrayRegion(alias_ref<jclass>, alias_ref<JArrayType> array) {
139:   EXPECT(array);
140:   EXPECT(array->size() > 5);
141: 
142:   int splits[] = {0, 1, 3, static_cast<int>(array->size())};
143:   for (int i = 0; i < 3; i++) {
144:     auto pinned = array->pinRegion(splits[i], splits[i + 1] - splits[i]);
145:     for (int j = 0; j < static_cast<int>(pinned.size()); j++) {
146:       EXPECT(std::abs(static_cast<double>(pinned[j] - j - splits[i])) < kEps);
147:       pinned[j] *= 2;
148:     }
149:   }
150:   return JNI_TRUE;
151: }
152: 
153: template <typename JArrayType>
154: jboolean testPinArrayCritical(alias_ref<jclass>, alias_ref<JArrayType> array) {
155:   EXPECT(array);
156: 
157:   int n = array->size();
158:   auto pinned = array->pinCritical();
159: 
160:   for (auto i = 0; i < n; ++i) {
161:     EXPECT(std::abs(static_cast<double>(pinned[i] - i)) < kEps);
162:   }
163: 
164:   for (auto i = 0; i < n; ++i) {
165:     pinned[i] *= 2;
166:   }
167: 
168:   return JNI_TRUE;
169: }
170: 
171: jboolean testIndexOutOfBoundsInRegions(alias_ref<jclass>) {
172:   constexpr auto N = 7;
173:   constexpr auto TOO_MUCH = 10;
174:   constexpr auto NEGATIVE = -1;
175: 
176:   auto array = make_int_array(N);
177: 
178:   try {
179:     auto buf = array->getRegion(TOO_MUCH, N);
180:     EXPECT(false);
````
- EN: Implements callable logic such as `testPinArrayRegion`, `testPinArrayCritical`, `testIndexOutOfBoundsInRegions`.
- CN: 实现可调用逻辑，例如 `testPinArrayRegion`, `testPinArrayCritical`, `testIndexOutOfBoundsInRegions`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````cpp
181:   } catch (JniException& ex) {
182:   }
183: 
184:   try {
185:     auto buf = array->getRegion(NEGATIVE, N);
186:     EXPECT(false);
187:   } catch (JniException& ex) {
188:   }
189: 
190:   try {
191:     auto vec = std::vector<jint>(TOO_MUCH);
192:     array->setRegion(0, vec.size(), vec.data());
193:     EXPECT(false);
194:   } catch (JniException& ex) {
195:   }
196: 
197:   try {
198:     auto vec = std::vector<jint>(1);
199:     array->setRegion(NEGATIVE, vec.size(), vec.data());
200:     EXPECT(false);
201:   } catch (JniException& ex) {
202:   }
203: 
204:   return JNI_TRUE;
205: }
206: 
207: jboolean TestBooleanArrayIndexing(alias_ref<jobject> self, alias_ref<jbooleanArray> input, jint idx) {
208:   auto array = input->pin();
209:   jboolean value = array[idx];
210:   return value;
211: }
212: 
213: jint TestIntegerArrayIndexing(alias_ref<jobject> self, alias_ref<jintArray> input, jint idx) {
214:   auto array = input->pin();
215:   jint value = array[idx];
216:   return value;
217: }
218: 
219: jsize TestIntegerArraySize(alias_ref<jobject> self, alias_ref<jintArray> input) {
220:   auto array = input->pin();
221:   jsize size = array.size();
222:   return size;
223: }
224: 
225: alias_ref<jintArray> TestIntegerArrayIncrement(alias_ref<jobject> self, alias_ref<jintArray> input) {
226:   auto array = input->pin();
227:   for (size_t ii = 0; ii < array.size(); ii++) {
228:     array[ii]++;
229:   }
230:   return input;
231: }
232: 
233: void TestIntegerArrayMoveAssignment(alias_ref<jobject> self, alias_ref<jintArray> input) {
234:   auto array = input->pin();
235:   array[0] = 0;
236:   array.release();
237: }
238: 
239: jboolean isPinnedArrayACopy(alias_ref<jobject>, alias_ref<jintArray> input) {
240:   return input->pin().isCopy();
````
- EN: Implements callable logic such as `TestBooleanArrayIndexing`, `TestIntegerArrayIndexing`, `TestIntegerArraySize`, `TestIntegerArrayIncrement`.
- CN: 实现可调用逻辑，例如 `TestBooleanArrayIndexing`, `TestIntegerArrayIndexing`, `TestIntegerArraySize`, `TestIntegerArrayIncrement`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````cpp
241: }
242: 
243: jboolean testCopiedPinnedArray(alias_ref<jobject>, alias_ref<jintArray> input) {
244:   EXPECT(input->size() > 0);
245:   input->pin()[0] = 100;
246:   auto pin = input->pin();
247:   EXPECT(pin.isCopy());
248:   EXPECT(pin[0] == input->pin()[0]);
249: 
250:   pin[0] = 200;
251:   EXPECT(input->pin()[0] == 100);
252: 
253:   pin.commit();
254:   EXPECT(input->pin()[0] == 200);
255:   pin[0] = 300;
256:   EXPECT(input->pin()[0] == 200);
257:   pin.abort();
258:   EXPECT(input->pin()[0] == 200);
259:   pin = input->pin();
260:   pin[0] = 400;
261:   pin.release();
262:   EXPECT(input->pin()[0] == 400);
263:   return JNI_TRUE;
264: }
265: 
266: jboolean testNonCopiedPinnedArray(alias_ref<jobject>, alias_ref<jintArray> input) {
267:   EXPECT(input->size() > 0);
268:   auto pin = input->pin();
269:   EXPECT(!pin.isCopy());
270:   EXPECT(pin[0] == input->pin()[0]);
271:   pin.commit();
272:   EXPECT(pin[0] == input->pin()[0]);
273:   pin[0] = 100;
274:   EXPECT(pin[0] == input->pin()[0]);
275:   input->pin()[0] = 200;
276:   EXPECT(pin[0] == input->pin()[0]);
277:   pin.abort();
278:   return JNI_TRUE;
279: }
280: 
281: 
282: void RegisterPrimitiveArrayTests() {
283:   registerNatives("com/facebook/jni/PrimitiveArrayTests", {
284:     makeNativeMethod("nativeTestMakeBooleanArray", testMakeBoolArray),
285:     makeNativeMethod("nativeTestMakeByteArray", testMakeByteArray),
286:     makeNativeMethod("nativeTestMakeCharArray", testMakeCharArray),
287:     makeNativeMethod("nativeTestMakeShortArray", testMakeShortArray),
288:     makeNativeMethod("nativeTestMakeIntArray", testMakeIntArray),
289:     makeNativeMethod("nativeTestMakeLongArray", testMakeLongArray),
290:     makeNativeMethod("nativeTestMakeFloatArray", testMakeFloatArray),
291:     makeNativeMethod("nativeTestMakeDoubleArray", testMakeDoubleArray),
292: 
293:     makeNativeMethod("nativeTestGetSetBooleanArray", testGetSetBooleanArray),
294:     makeNativeMethod("nativeTestGetSetByteArray", testGetSetArray<jbyteArray>),
295:     makeNativeMethod("nativeTestGetSetCharArray", testGetSetArray<jcharArray>),
296:     makeNativeMethod("nativeTestGetSetShortArray", testGetSetArray<jshortArray>),
297:     makeNativeMethod("nativeTestGetSetIntArray", testGetSetArray<jintArray>),
298:     makeNativeMethod("nativeTestGetSetLongArray", testGetSetArray<jlongArray>),
299:     makeNativeMethod("nativeTestGetSetFloatArray", testGetSetArray<jfloatArray>),
300:     makeNativeMethod("nativeTestGetSetDoubleArray", testGetSetArray<jdoubleArray>),
````
- EN: Implements callable logic such as `testCopiedPinnedArray`, `testNonCopiedPinnedArray`, `RegisterPrimitiveArrayTests`.
- CN: 实现可调用逻辑，例如 `testCopiedPinnedArray`, `testNonCopiedPinnedArray`, `RegisterPrimitiveArrayTests`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-346
````cpp
301: 
302:     makeNativeMethod("nativeTestPinBooleanArray", testPinBooleanArray),
303:     makeNativeMethod("nativeTestPinByteArray", testPinArray<jbyteArray>),
304:     makeNativeMethod("nativeTestPinCharArray", testPinArray<jcharArray>),
305:     makeNativeMethod("nativeTestPinShortArray", testPinArray<jshortArray>),
306:     makeNativeMethod("nativeTestPinIntArray", testPinArray<jintArray>),
307:     makeNativeMethod("nativeTestPinLongArray", testPinArray<jlongArray>),
308:     makeNativeMethod("nativeTestPinFloatArray", testPinArray<jfloatArray>),
309:     makeNativeMethod("nativeTestPinDoubleArray", testPinArray<jdoubleArray>),
310: 
311:     makeNativeMethod("nativeTestPinByteArrayRegion", testPinArrayRegion<jbyteArray>),
312:     makeNativeMethod("nativeTestPinCharArrayRegion", testPinArrayRegion<jcharArray>),
313:     makeNativeMethod("nativeTestPinShortArrayRegion", testPinArrayRegion<jshortArray>),
314:     makeNativeMethod("nativeTestPinIntArrayRegion", testPinArrayRegion<jintArray>),
315:     makeNativeMethod("nativeTestPinLongArrayRegion", testPinArrayRegion<jlongArray>),
316:     makeNativeMethod("nativeTestPinFloatArrayRegion", testPinArrayRegion<jfloatArray>),
317:     makeNativeMethod("nativeTestPinDoubleArrayRegion", testPinArrayRegion<jdoubleArray>),
318: 
319:     makeNativeMethod("nativeTestPinByteArrayCritical", testPinArrayCritical<jbyteArray>),
320:     makeNativeMethod("nativeTestPinCharArrayCritical", testPinArrayCritical<jcharArray>),
321:     makeNativeMethod("nativeTestPinShortArrayCritical", testPinArrayCritical<jshortArray>),
322:     makeNativeMethod("nativeTestPinIntArrayCritical", testPinArrayCritical<jintArray>),
323:     makeNativeMethod("nativeTestPinLongArrayCritical", testPinArrayCritical<jlongArray>),
324:     makeNativeMethod("nativeTestPinFloatArrayCritical", testPinArrayCritical<jfloatArray>),
325:     makeNativeMethod("nativeTestPinDoubleArrayCritical", testPinArrayCritical<jdoubleArray>),
326: 
327:     makeNativeMethod("nativeTestIndexOutOfBoundsInRegions", testIndexOutOfBoundsInRegions),
328: 
329:     makeNativeMethod("nativeTestBooleanArrayIndexing",
330:                      TestBooleanArrayIndexing),
331:     makeNativeMethod("nativeTestIntegerArrayIndexing",
332:                      TestIntegerArrayIndexing),
333:     makeNativeMethod("nativeTestIntegerArraySize",
334:                      TestIntegerArraySize),
335:     makeNativeMethod("nativeTestIntegerArrayIncrement",
336:                      TestIntegerArrayIncrement),
337:     makeNativeMethod("nativeTestIntegerArrayMoveAssignment",
338:                      TestIntegerArrayMoveAssignment),
339: 
340:     makeNativeMethod("nativeIsPinnedArrayACopy", isPinnedArrayACopy),
341:     makeNativeMethod("nativeTestCopiedPinnedArray",
342:                      testCopiedPinnedArray),
343:     makeNativeMethod("nativeTestNonCopiedPinnedArray",
344:                      testNonCopiedPinnedArray),
345:   });
346: }
````
- EN: This range contributes implementation details for the file goal: Implements JNI bridge abstractions used by Android-facing native code.
- CN: 该范围为文件目标提供实现细节：实现供 Android 原生代码使用的 JNI 桥接抽象。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Validation and test coverage / 校验与测试覆盖
- Symbol `testMakeBoolArray` / 符号 `testMakeBoolArray`
- Symbol `testMakeByteArray` / 符号 `testMakeByteArray`
- Symbol `testMakeCharArray` / 符号 `testMakeCharArray`
- Symbol `testMakeShortArray` / 符号 `testMakeShortArray`

## Dependencies / 依赖关系
- C/C++ includes: `cmath`, `vector`, `fbjni/fbjni.h`, `expect.h`
- C/C++ 头文件: `cmath`, `vector`, `fbjni/fbjni.h`, `expect.h`
