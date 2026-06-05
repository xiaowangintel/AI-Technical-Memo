# pytorch_jni_common.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/pytorch_android/src/main/cpp/pytorch_jni_common.cpp`
- Repository: `pytorch`
- Purpose (EN): Supports Android/mobile runtime integration in the PyTorch repository.
- 用途 (CN): 为 PyTorch 仓库中的 Android/移动运行时集成提供支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````cpp
 1: #include <cassert>
 2: #include <iostream>
 3: #include <memory>
 4: #include <string>
 5: 
 6: #include <c10/core/MemoryFormat.h>
 7: #include <c10/util/irange.h>
 8: 
 9: #include <fbjni/ByteBuffer.h>
10: #include <fbjni/fbjni.h>
11: 
12: #include "pytorch_jni_common.h"
13: #if defined(__ANDROID__)
14: #ifndef USE_PTHREADPOOL
15: #define USE_PTHREADPOOL
16: #endif /* USE_PTHREADPOOL */
17: #include <caffe2/utils/threadpool/pthreadpool-cpp.h>
18: #endif
19: 
20: namespace pytorch_jni {
21: 
22: c10::DeviceType deviceJniCodeToDeviceType(jint deviceJniCode) {
23:   if (deviceJniCode == kDeviceCPU) {
24:     return at::kCPU;
25:   } else if (deviceJniCode == kDeviceVulkan) {
26:     return at::kVulkan;
27:   }
28: 
29:   facebook::jni::throwNewJavaException(
30:       facebook::jni::gJavaLangIllegalArgumentException, "Unknown device");
31: }
32: 
33: bool Trace::is_initialized_ = false;
34: 
35: #if defined(TRACE_ENABLED) && defined(__ANDROID__)
36: Trace::fp_ATrace_beginSection Trace::ATrace_beginSection;
37: Trace::fp_ATrace_endSection Trace::ATrace_endSection;
38: #endif
39: 
40: void Trace::init() {
41: #if defined(TRACE_ENABLED) && defined(__ANDROID__)
42:   void* lib = dlopen("libandroid.so", RTLD_NOW || RTLD_LOCAL);
43:   if (lib != NULL) {
44:     Trace::ATrace_beginSection = reinterpret_cast<fp_ATrace_beginSection>(
45:         dlsym(lib, "ATrace_beginSection"));
46:     Trace::ATrace_endSection =
47:         reinterpret_cast<fp_ATrace_endSection>(dlsym(lib, "ATrace_endSection"));
48:   }
49: #endif
50: }
51: 
52: // NOTE: Codes must be kept in sync with DType.java.
53: // NOTE: Never serialize these, because they can change between releases.
54: constexpr static int kTensorDTypeUInt8 = 1;
55: constexpr static int kTensorDTypeInt8 = 2;
56: constexpr static int kTensorDTypeInt32 = 3;
57: constexpr static int kTensorDTypeFloat32 = 4;
58: constexpr static int kTensorDTypeInt64 = 5;
59: constexpr static int kTensorDTypeFloat64 = 6;
60: 
````
- EN: Pulls in native headers such as `cassert`, `iostream`, `memory`, `string`.
- CN: 引入原生头文件，例如 `cassert`, `iostream`, `memory`, `string`。
- EN: Implements callable logic such as `deviceJniCodeToDeviceType`, `Trace::init`.
- CN: 实现可调用逻辑，例如 `deviceJniCodeToDeviceType`, `Trace::init`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 61-120
````cpp
 61: constexpr static int kTensorMemoryFormatContiguous = 1;
 62: constexpr static int kTensorMemoryFormatChannelsLast = 2;
 63: constexpr static int kTensorMemoryFormatChannelsLast3d = 3;
 64: 
 65: template <typename K = jobject, typename V = jobject>
 66: struct JHashMap
 67:     : facebook::jni::JavaClass<JHashMap<K, V>, facebook::jni::JMap<K, V>> {
 68:   constexpr static auto kJavaDescriptor = "Ljava/util/HashMap;";
 69: 
 70:   using Super =
 71:       facebook::jni::JavaClass<JHashMap<K, V>, facebook::jni::JMap<K, V>>;
 72: 
 73:   static facebook::jni::local_ref<JHashMap<K, V>> create() {
 74:     return Super::newInstance();
 75:   }
 76: 
 77:   void put(
 78:       facebook::jni::alias_ref<facebook::jni::JObject::javaobject> key,
 79:       facebook::jni::alias_ref<facebook::jni::JObject::javaobject> value) {
 80:     static auto putMethod =
 81:         Super::javaClassStatic()
 82:             ->template getMethod<facebook::jni::alias_ref<
 83:                 facebook::jni::JObject::javaobject>(
 84:                 facebook::jni::alias_ref<facebook::jni::JObject::javaobject>,
 85:                 facebook::jni::alias_ref<facebook::jni::JObject::javaobject>)>(
 86:                 "put");
 87:     putMethod(Super::self(), key, value);
 88:   }
 89: };
 90: 
 91: static at::Tensor newAtTensor(
 92:     facebook::jni::alias_ref<facebook::jni::JBuffer> jbuffer,
 93:     facebook::jni::alias_ref<jlongArray> jshape,
 94:     jint jdtype,
 95:     jint jmemoryFormat) {
 96:   const auto rank = jshape->size();
 97:   const auto shapeArr = jshape->getRegion(0, rank);
 98:   std::vector<int64_t> shapeVec{};
 99:   shapeVec.reserve(rank);
100:   auto numel = 1;
101:   for (const auto i : c10::irange(rank)) {
102:     shapeVec.push_back(shapeArr[i]);
103:     numel *= shapeArr[i];
104:   }
105:   JNIEnv* jni = facebook::jni::Environment::current();
106:   caffe2::TypeMeta typeMeta{};
107:   int dataElementSizeBytes = 0;
108:   if (kTensorDTypeFloat32 == jdtype) {
109:     dataElementSizeBytes = 4;
110:     typeMeta = caffe2::TypeMeta::Make<float>();
111:   } else if (kTensorDTypeInt32 == jdtype) {
112:     dataElementSizeBytes = 4;
113:     typeMeta = caffe2::TypeMeta::Make<int32_t>();
114:   } else if (kTensorDTypeInt8 == jdtype) {
115:     dataElementSizeBytes = 1;
116:     typeMeta = caffe2::TypeMeta::Make<int8_t>();
117:   } else if (kTensorDTypeUInt8 == jdtype) {
118:     dataElementSizeBytes = 1;
119:     typeMeta = caffe2::TypeMeta::Make<uint8_t>();
120:   } else if (kTensorDTypeFloat64 == jdtype) {
````
- EN: Declares or extends types including `JHashMap`.
- CN: 声明或扩展类型，包括 `JHashMap`。
- EN: Implements callable logic such as `create`, `put`, `newAtTensor`.
- CN: 实现可调用逻辑，例如 `create`, `put`, `newAtTensor`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````cpp
121:     dataElementSizeBytes = 8;
122:     typeMeta = caffe2::TypeMeta::Make<double>();
123:   } else if (kTensorDTypeInt64 == jdtype) {
124:     dataElementSizeBytes = 8;
125:     typeMeta = caffe2::TypeMeta::Make<int64_t>();
126:   } else {
127:     facebook::jni::throwNewJavaException(
128:         facebook::jni::gJavaLangIllegalArgumentException,
129:         "Unknown Tensor jdtype %d",
130:         jdtype);
131:   }
132:   const auto dataCapacity = jni->GetDirectBufferCapacity(jbuffer.get());
133:   if (dataCapacity != numel) {
134:     facebook::jni::throwNewJavaException(
135:         facebook::jni::gJavaLangIllegalArgumentException,
136:         "Tensor dimensions(elements number:%d, element byte size:%d, total "
137:         "bytes:%d) inconsistent with buffer capacity(%d)",
138:         numel,
139:         dataElementSizeBytes,
140:         numel * dataElementSizeBytes,
141:         dataCapacity);
142:   }
143: 
144:   if (jmemoryFormat == kTensorMemoryFormatChannelsLast) {
145:     auto sizes = torch::IntArrayRef(shapeVec);
146:     return torch::from_blob(
147:         jni->GetDirectBufferAddress(jbuffer.get()),
148:         sizes,
149:         torch::IntArrayRef(c10::get_channels_last_strides_2d(sizes)),
150:         at::TensorOptions(typeMeta).memory_format(
151:             at::MemoryFormat::ChannelsLast));
152:   } else if (jmemoryFormat == kTensorMemoryFormatChannelsLast3d) {
153:     auto sizes = torch::IntArrayRef(shapeVec);
154:     return torch::from_blob(
155:         jni->GetDirectBufferAddress(jbuffer.get()),
156:         sizes,
157:         torch::IntArrayRef(c10::get_channels_last_strides_3d(sizes)),
158:         at::TensorOptions(typeMeta).memory_format(
159:             at::MemoryFormat::ChannelsLast3d));
160:   }
161:   return torch::from_blob(
162:       jni->GetDirectBufferAddress(jbuffer.get()),
163:       torch::IntArrayRef(shapeVec),
164:       at::TensorOptions(typeMeta));
165: }
166: 
167: class TensorHybrid : public facebook::jni::HybridClass<TensorHybrid> {
168:  public:
169:   constexpr static const char* kJavaDescriptor = "Lorg/pytorch/Tensor;";
170: 
171:   explicit TensorHybrid(at::Tensor tensor) : tensor_(tensor) {}
172: 
173:   static facebook::jni::local_ref<TensorHybrid::jhybriddata> initHybrid(
174:       facebook::jni::alias_ref<TensorHybrid::javaobject> jTensorThis) {
175:     static auto cls = TensorHybrid::javaClassStatic();
176:     static const auto jMethodDTypeCode = cls->getMethod<jint()>("dtypeJniCode");
177:     static const auto jMethodMemoryFormatCode =
178:         cls->getMethod<jint()>("memoryFormatJniCode");
179:     static const auto jFieldShape = cls->getField<jlongArray>("shape");
180:     static const auto jMethodGetDataBuffer = cls->getMethod<
````
- EN: Declares or extends types including `TensorHybrid`.
- CN: 声明或扩展类型，包括 `TensorHybrid`。
- EN: Implements callable logic such as `TensorHybrid`, `initHybrid`.
- CN: 实现可调用逻辑，例如 `TensorHybrid`, `initHybrid`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````cpp
181:         facebook::jni::local_ref<facebook::jni::JBuffer::javaobject>()>(
182:         "getRawDataBuffer");
183: 
184:     at::Tensor tensor = newAtTensor(
185:         jMethodGetDataBuffer(jTensorThis),
186:         jTensorThis->getFieldValue(jFieldShape),
187:         jMethodDTypeCode(jTensorThis),
188:         jMethodMemoryFormatCode(jTensorThis));
189:     return makeCxxInstance(std::move(tensor));
190:   }
191: 
192:   static facebook::jni::local_ref<TensorHybrid::javaobject>
193:   newJTensorFromAtTensor(const at::Tensor& input_tensor) {
194:     // Java wrapper currently only supports contiguous tensors.
195: 
196:     int jmemoryFormat = 0;
197:     at::Tensor tensor{};
198:     if (input_tensor.is_contiguous(at::MemoryFormat::ChannelsLast)) {
199:       tensor = input_tensor;
200:       jmemoryFormat = kTensorMemoryFormatChannelsLast;
201:     } else if (input_tensor.is_contiguous(at::MemoryFormat::ChannelsLast3d)) {
202:       tensor = input_tensor;
203:       jmemoryFormat = kTensorMemoryFormatChannelsLast3d;
204:     } else {
205:       tensor = input_tensor.contiguous();
206:       jmemoryFormat = kTensorMemoryFormatContiguous;
207:     }
208: 
209:     const auto scalarType = tensor.scalar_type();
210:     int jdtype = 0;
211:     if (at::kFloat == scalarType) {
212:       jdtype = kTensorDTypeFloat32;
213:     } else if (at::kInt == scalarType) {
214:       jdtype = kTensorDTypeInt32;
215:     } else if (at::kByte == scalarType) {
216:       jdtype = kTensorDTypeUInt8;
217:     } else if (at::kChar == scalarType) {
218:       jdtype = kTensorDTypeInt8;
219:     } else if (at::kLong == scalarType) {
220:       jdtype = kTensorDTypeInt64;
221:     } else if (at::kDouble == scalarType) {
222:       jdtype = kTensorDTypeFloat64;
223:     } else {
224:       facebook::jni::throwNewJavaException(
225:           facebook::jni::gJavaLangIllegalArgumentException,
226:           "at::Tensor scalar type %s is not supported on java side",
227:           c10::toString(scalarType));
228:     }
229: 
230:     const auto& tensorShape = tensor.sizes();
231:     std::vector<jlong> tensorShapeVec;
232:     for (const auto& s : tensorShape) {
233:       tensorShapeVec.push_back(s);
234:     }
235:     facebook::jni::local_ref<jlongArray> jTensorShape =
236:         facebook::jni::make_long_array(tensorShapeVec.size());
237:     jTensorShape->setRegion(0, tensorShapeVec.size(), tensorShapeVec.data());
238: 
239:     static auto cls = TensorHybrid::javaClassStatic();
240:     facebook::jni::local_ref<facebook::jni::JByteBuffer> jTensorBuffer =
````
- EN: Implements callable logic such as `newJTensorFromAtTensor`.
- CN: 实现可调用逻辑，例如 `newJTensorFromAtTensor`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````cpp
241:         facebook::jni::JByteBuffer::wrapBytes(
242:             (uint8_t*)tensor.data_ptr(), tensor.nbytes());
243:     jTensorBuffer->order(facebook::jni::JByteOrder::nativeOrder());
244: 
245:     static const auto jMethodNewTensor =
246:         cls->getStaticMethod<facebook::jni::local_ref<TensorHybrid::javaobject>(
247:             facebook::jni::alias_ref<facebook::jni::JByteBuffer>,
248:             facebook::jni::alias_ref<jlongArray>,
249:             jint,
250:             jint,
251:             facebook::jni::alias_ref<jhybriddata>)>("nativeNewTensor");
252:     return jMethodNewTensor(
253:         cls,
254:         jTensorBuffer,
255:         jTensorShape,
256:         jdtype,
257:         jmemoryFormat,
258:         makeCxxInstance(tensor));
259:   }
260: 
261:   static at::Tensor newAtTensorFromJTensor(
262:       facebook::jni::alias_ref<TensorHybrid::javaobject> jtensor) {
263:     static auto cls = TensorHybrid::javaClassStatic();
264:     static const auto dtypeMethod = cls->getMethod<jint()>("dtypeJniCode");
265:     jint jdtype = dtypeMethod(jtensor);
266: 
267:     static const auto memoryFormatMethod =
268:         cls->getMethod<jint()>("memoryFormatJniCode");
269:     jint jmemoryFormat = memoryFormatMethod(jtensor);
270: 
271:     static const auto shapeField = cls->getField<jlongArray>("shape");
272:     auto jshape = jtensor->getFieldValue(shapeField);
273: 
274:     static auto dataBufferMethod = cls->getMethod<
275:         facebook::jni::local_ref<facebook::jni::JBuffer::javaobject>()>(
276:         "getRawDataBuffer");
277:     facebook::jni::local_ref<facebook::jni::JBuffer> jbuffer =
278:         dataBufferMethod(jtensor);
279:     return newAtTensor(jbuffer, jshape, jdtype, jmemoryFormat);
280:   }
281: 
282:   at::Tensor tensor() const {
283:     return tensor_;
284:   }
285: 
286:  private:
287:   friend HybridBase;
288:   at::Tensor tensor_;
289: };
290: 
291: facebook::jni::local_ref<JIValue> JIValue::newJIValueFromStringDict(
292:     c10::Dict<c10::IValue, c10::IValue> dict) {
293:   static auto jMethodDictStringKey =
294:       JIValue::javaClassStatic()
295:           ->getStaticMethod<facebook::jni::local_ref<JIValue>(
296:               facebook::jni::alias_ref<facebook::jni::JMap<
297:                   facebook::jni::alias_ref<facebook::jni::JString::javaobject>,
298:                   facebook::jni::alias_ref<JIValue::javaobject>>>)>(
299:               "dictStringKeyFrom");
300: 
````
- EN: Implements callable logic such as `newAtTensorFromJTensor`, `tensor`, `JIValue::newJIValueFromStringDict`.
- CN: 实现可调用逻辑，例如 `newAtTensorFromJTensor`, `tensor`, `JIValue::newJIValueFromStringDict`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````cpp
301:   auto jmap = JHashMap<
302:       facebook::jni::alias_ref<facebook::jni::JString::javaobject>,
303:       facebook::jni::alias_ref<JIValue::javaobject>>::create();
304:   for (auto& pair : dict) {
305:     jmap->put(
306:         facebook::jni::make_jstring(pair.key().toStringRef()),
307:         JIValue::newJIValueFromAtIValue(pair.value()));
308:   }
309:   return jMethodDictStringKey(JIValue::javaClassStatic(), jmap);
310: }
311: 
312: facebook::jni::local_ref<JIValue> JIValue::newJIValueFromIntDict(
313:     c10::Dict<c10::IValue, c10::IValue> dict) {
314:   static auto jMethodDictLongKey =
315:       JIValue::javaClassStatic()
316:           ->getStaticMethod<facebook::jni::local_ref<JIValue>(
317:               facebook::jni::alias_ref<facebook::jni::JMap<
318:                   facebook::jni::alias_ref<facebook::jni::JLong::javaobject>,
319:                   facebook::jni::alias_ref<JIValue::javaobject>>>)>(
320:               "dictLongKeyFrom");
321:   auto jmap = JHashMap<
322:       facebook::jni::alias_ref<facebook::jni::JLong::javaobject>,
323:       facebook::jni::alias_ref<JIValue::javaobject>>::create();
324:   for (auto& pair : dict) {
325:     jmap->put(
326:         facebook::jni::JLong::valueOf(pair.key().toInt()),
327:         JIValue::newJIValueFromAtIValue(pair.value()));
328:   }
329:   return jMethodDictLongKey(JIValue::javaClassStatic(), jmap);
330: }
331: 
332: facebook::jni::local_ref<JIValue> JIValue::newJIValueFromAtIValue(
333:     const at::IValue& ivalue,
334:     DictCallback stringDictCallback,
335:     DictCallback intDictCallback) {
336:   Trace _s{"jni::JIValue::newJIValueFromAtIValue"};
337:   if (ivalue.isNone()) {
338:     static auto jMethodOptionalNull =
339:         JIValue::javaClassStatic()
340:             ->getStaticMethod<facebook::jni::local_ref<JIValue>()>(
341:                 "optionalNull");
342:     return jMethodOptionalNull(JIValue::javaClassStatic());
343:   } else if (ivalue.isTensor()) {
344:     static auto jMethodTensor =
345:         JIValue::javaClassStatic()
346:             ->getStaticMethod<facebook::jni::local_ref<JIValue>(
347:                 facebook::jni::local_ref<TensorHybrid::javaobject>)>("from");
348:     const auto& tensor = ivalue.toTensor();
349:     return jMethodTensor(
350:         JIValue::javaClassStatic(),
351:         TensorHybrid::newJTensorFromAtTensor(tensor));
352:   } else if (ivalue.isBool()) {
353:     static auto jMethodBool =
354:         JIValue::javaClassStatic()
355:             ->getStaticMethod<facebook::jni::local_ref<JIValue>(jboolean)>(
356:                 "from");
357:     return jMethodBool(JIValue::javaClassStatic(), ivalue.toBool());
358:   } else if (ivalue.isInt()) {
359:     static auto jMethodInt =
360:         JIValue::javaClassStatic()
````
- EN: Implements callable logic such as `JIValue::newJIValueFromIntDict`, `JIValue::newJIValueFromAtIValue`.
- CN: 实现可调用逻辑，例如 `JIValue::newJIValueFromIntDict`, `JIValue::newJIValueFromAtIValue`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-420
````cpp
361:             ->getStaticMethod<facebook::jni::local_ref<JIValue>(jlong)>("from");
362:     return jMethodInt(JIValue::javaClassStatic(), ivalue.toInt());
363:   } else if (ivalue.isDouble()) {
364:     static auto jMethodDouble =
365:         JIValue::javaClassStatic()
366:             ->getStaticMethod<facebook::jni::local_ref<JIValue>(jdouble)>(
367:                 "from");
368:     return jMethodDouble(JIValue::javaClassStatic(), ivalue.toDouble());
369:   } else if (ivalue.isString()) {
370:     static auto jMethodString =
371:         JIValue::javaClassStatic()
372:             ->getStaticMethod<facebook::jni::local_ref<JIValue>(
373:                 facebook::jni::alias_ref<facebook::jni::JString::javaobject>)>(
374:                 "from");
375:     return jMethodString(
376:         JIValue::javaClassStatic(),
377:         facebook::jni::make_jstring(ivalue.toStringRef()));
378:   } else if (ivalue.isTuple()) {
379:     auto elementsVec = ivalue.toTupleRef().elements();
380:     static auto jMethodTupleArr =
381:         JIValue::javaClassStatic()
382:             ->getStaticMethod<facebook::jni::local_ref<JIValue>(
383:                 facebook::jni::alias_ref<facebook::jni::JArrayClass<
384:                     JIValue::javaobject>::javaobject>)>("tupleFrom");
385:     auto jElementsArray =
386:         facebook::jni::JArrayClass<JIValue::javaobject>::newArray(
387:             elementsVec.size());
388:     auto index = 0;
389:     for (const auto& e : elementsVec) {
390:       (*jElementsArray)[index++] = JIValue::newJIValueFromAtIValue(e);
391:     }
392:     return jMethodTupleArr(JIValue::javaClassStatic(), jElementsArray);
393:   } else if (ivalue.isBoolList()) {
394:     auto list = ivalue.toBoolList();
395:     static auto jMethodBoolListArr =
396:         JIValue::javaClassStatic()
397:             ->getStaticMethod<facebook::jni::local_ref<JIValue>(
398:                 facebook::jni::alias_ref<jbooleanArray>)>("listFrom");
399:     size_t n = list.size();
400:     auto jArray = facebook::jni::make_boolean_array(n);
401:     auto jArrayPinned = jArray->pin();
402:     auto index = 0;
403:     for (const auto& e : list) {
404:       jArrayPinned[index++] = e;
405:     }
406:     return jMethodBoolListArr(JIValue::javaClassStatic(), jArray);
407:   } else if (ivalue.isIntList()) {
408:     auto list = ivalue.toIntList();
409:     static auto jMethodLongListArr =
410:         JIValue::javaClassStatic()
411:             ->getStaticMethod<facebook::jni::local_ref<JIValue>(
412:                 facebook::jni::alias_ref<jlongArray>)>("listFrom");
413:     size_t n = list.size();
414:     auto jArray = facebook::jni::make_long_array(n);
415:     auto jArrayPinned = jArray->pin();
416:     auto index = 0;
417:     for (const auto& e : list) {
418:       jArrayPinned[index++] = e;
419:     }
420:     return jMethodLongListArr(JIValue::javaClassStatic(), jArray);
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 421-480
````cpp
421:   } else if (ivalue.isDoubleList()) {
422:     auto list = ivalue.toDoubleList();
423:     static auto jMethoDoubleListArr =
424:         JIValue::javaClassStatic()
425:             ->getStaticMethod<facebook::jni::local_ref<JIValue>(
426:                 facebook::jni::alias_ref<jdoubleArray>)>("listFrom");
427:     size_t n = list.size();
428:     auto jArray = facebook::jni::make_double_array(n);
429:     auto jArrayPinned = jArray->pin();
430:     auto index = 0;
431:     for (const auto& e : list) {
432:       jArrayPinned[index++] = e;
433:     }
434:     return jMethoDoubleListArr(JIValue::javaClassStatic(), jArray);
435:   } else if (ivalue.isTensorList()) {
436:     auto list = ivalue.toTensorList();
437:     static auto jMethodTensorListArr =
438:         JIValue::javaClassStatic()
439:             ->getStaticMethod<facebook::jni::local_ref<JIValue>(
440:                 facebook::jni::alias_ref<facebook::jni::JArrayClass<
441:                     TensorHybrid::javaobject>::javaobject>)>("listFrom");
442:     auto jArray =
443:         facebook::jni::JArrayClass<TensorHybrid::javaobject>::newArray(
444:             list.size());
445:     auto index = 0;
446:     for (const auto& e : list) {
447:       (*jArray)[index++] = TensorHybrid::newJTensorFromAtTensor(e);
448:     }
449:     return jMethodTensorListArr(JIValue::javaClassStatic(), jArray);
450:   } else if (ivalue.isList()) {
451:     auto list = ivalue.toList();
452:     static auto jMethodListArr =
453:         JIValue::javaClassStatic()
454:             ->getStaticMethod<facebook::jni::local_ref<JIValue>(
455:                 facebook::jni::alias_ref<facebook::jni::JArrayClass<
456:                     JIValue::javaobject>::javaobject>)>("listFrom");
457:     auto jArray =
458:         facebook::jni::JArrayClass<JIValue::javaobject>::newArray(list.size());
459:     auto index = 0;
460:     for (const auto& e : list) {
461:       (*jArray)[index++] = JIValue::newJIValueFromAtIValue(e);
462:     }
463:     return jMethodListArr(JIValue::javaClassStatic(), jArray);
464:   } else if (ivalue.isGenericDict()) {
465:     auto dict = ivalue.toGenericDict();
466:     const auto keyType = dict.keyType();
467: 
468:     if (!keyType) {
469:       facebook::jni::throwNewJavaException(
470:           facebook::jni::gJavaLangIllegalArgumentException,
471:           "Unknown IValue-Dict key type");
472:     }
473: 
474:     if (*keyType == *c10::StringType::get()) {
475:       return stringDictCallback(std::move(dict));
476:     } else if (*keyType == *c10::IntType::get()) {
477:       return intDictCallback(std::move(dict));
478:     }
479: 
480:     facebook::jni::throwNewJavaException(
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 481-540
````cpp
481:         facebook::jni::gJavaLangIllegalArgumentException,
482:         "Unsupported IValue-Dict key type: %s",
483:         keyType->str().c_str());
484:   }
485: 
486:   facebook::jni::throwNewJavaException(
487:       facebook::jni::gJavaLangIllegalArgumentException,
488:       "Unsupported IValue type %s",
489:       ivalue.tagKind().c_str());
490: }
491: 
492: at::IValue JIValue::JIValueToAtIValue(
493:     facebook::jni::alias_ref<JIValue> jivalue) {
494:   Trace _s{"jni::JIValue::JIValueToAtIValue"};
495:   static const auto typeCodeField =
496:       JIValue::javaClassStatic()->getField<jint>("mTypeCode");
497:   const auto typeCode = jivalue->getFieldValue(typeCodeField);
498:   if (JIValue::kTypeCodeNull == typeCode) {
499:     return at::IValue{};
500:   } else if (JIValue::kTypeCodeTensor == typeCode) {
501:     static const auto jMethodGetTensor =
502:         JIValue::javaClassStatic()
503:             ->getMethod<facebook::jni::alias_ref<TensorHybrid::javaobject>()>(
504:                 "toTensor");
505:     return TensorHybrid::newAtTensorFromJTensor(jMethodGetTensor(jivalue));
506:   } else if (JIValue::kTypeCodeBool == typeCode) {
507:     static const auto jMethodGetBool =
508:         JIValue::javaClassStatic()->getMethod<jboolean()>("toBool");
509:     // explicit cast to bool as jboolean is defined as uint8_t, IValue ctor
510:     // for int will be called for jboolean
511:     bool b = jMethodGetBool(jivalue);
512:     return at::IValue{b};
513:   } else if (JIValue::kTypeCodeLong == typeCode) {
514:     static const auto jMethodGetLong =
515:         JIValue::javaClassStatic()->getMethod<jlong()>("toLong");
516:     return at::IValue{(int64_t)jMethodGetLong(jivalue)};
517:   } else if (JIValue::kTypeCodeDouble == typeCode) {
518:     static const auto jMethodGetDouble =
519:         JIValue::javaClassStatic()->getMethod<jdouble()>("toDouble");
520:     return at::IValue{jMethodGetDouble(jivalue)};
521:   } else if (JIValue::kTypeCodeString == typeCode) {
522:     static const auto jMethodGetString =
523:         JIValue::javaClassStatic()->getMethod<jstring()>("toStr");
524:     return at::IValue{jMethodGetString(jivalue)->toStdString()};
525:   } else if (JIValue::kTypeCodeTuple == typeCode) {
526:     static const auto jMethodGetTuple =
527:         JIValue::javaClassStatic()
528:             ->getMethod<
529:                 facebook::jni::JArrayClass<JIValue::javaobject>::javaobject()>(
530:                 "toTuple");
531:     auto jarray = jMethodGetTuple(jivalue);
532:     size_t n = jarray->size();
533: 
534:     std::vector<at::IValue> elements;
535:     elements.reserve(n);
536:     for (const auto i : c10::irange(n)) {
537:       auto jivalue_element = jarray->getElement(i);
538:       auto element = JIValue::JIValueToAtIValue(jivalue_element);
539:       elements.push_back(std::move(element));
540:     }
````
- EN: Implements callable logic such as `JIValue::JIValueToAtIValue`.
- CN: 实现可调用逻辑，例如 `JIValue::JIValueToAtIValue`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 541-600
````cpp
541:     return c10::ivalue::Tuple::create(std::move(elements));
542:   } else if (JIValue::kTypeCodeBoolList == typeCode) {
543:     static const auto jMethodGetBoolList =
544:         JIValue::javaClassStatic()->getMethod<jbooleanArray()>("toBoolList");
545:     auto jArray = jMethodGetBoolList(jivalue);
546:     auto jArrayPinned = jArray->pin();
547:     size_t n = jArrayPinned.size();
548:     c10::List<bool> list{};
549:     list.reserve(n);
550:     for (const auto i : c10::irange(n)) {
551:       list.push_back(jArrayPinned[i]);
552:     }
553:     return at::IValue{std::move(list)};
554:   } else if (JIValue::kTypeCodeLongList == typeCode) {
555:     static const auto jMethodGetLongList =
556:         JIValue::javaClassStatic()->getMethod<jlongArray()>("toLongList");
557:     auto jArray = jMethodGetLongList(jivalue);
558:     auto jArrayPinned = jArray->pin();
559:     size_t n = jArrayPinned.size();
560:     c10::List<int64_t> list{};
561:     list.reserve(n);
562:     for (const auto i : c10::irange(n)) {
563:       list.push_back(jArrayPinned[i]);
564:     }
565:     return at::IValue{std::move(list)};
566:   } else if (JIValue::kTypeCodeDoubleList == typeCode) {
567:     static const auto jMethodGetDoubleList =
568:         JIValue::javaClassStatic()->getMethod<jdoubleArray()>("toDoubleList");
569:     auto jArray = jMethodGetDoubleList(jivalue);
570:     auto jArrayPinned = jArray->pin();
571:     size_t n = jArrayPinned.size();
572:     c10::List<double> list{};
573:     list.reserve(n);
574:     for (const auto i : c10::irange(n)) {
575:       list.push_back(jArrayPinned[i]);
576:     }
577:     return at::IValue{std::move(list)};
578:   } else if (JIValue::kTypeCodeTensorList == typeCode) {
579:     static const auto jMethodGetTensorList =
580:         JIValue::javaClassStatic()
581:             ->getMethod<facebook::jni::JArrayClass<
582:                 TensorHybrid::javaobject>::javaobject()>("toTensorList");
583:     auto jArray = jMethodGetTensorList(jivalue);
584:     size_t n = jArray->size();
585:     c10::List<at::Tensor> list{};
586:     list.reserve(n);
587:     for (const auto i : c10::irange(n)) {
588:       list.push_back(
589:           TensorHybrid::newAtTensorFromJTensor(jArray->getElement(i)));
590:     }
591:     return at::IValue{std::move(list)};
592:   } else if (JIValue::kTypeCodeList == typeCode) {
593:     static const auto jMethodGetList =
594:         JIValue::javaClassStatic()
595:             ->getMethod<
596:                 facebook::jni::JArrayClass<JIValue::javaobject>::javaobject()>(
597:                 "toList");
598:     auto jarray = jMethodGetList(jivalue);
599:     size_t n = jarray->size();
600:     if (n == 0) {
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 601-660
````cpp
601:       return at::IValue{c10::impl::GenericList(c10::TensorType::get())};
602:     }
603: 
604:     auto jivalue_first_element = jarray->getElement(0);
605:     auto first_element = JIValue::JIValueToAtIValue(jivalue_first_element);
606:     c10::impl::GenericList list{c10::unshapedType(first_element.type())};
607:     list.reserve(n);
608:     list.push_back(first_element);
609:     for (const auto i : c10::irange(1, n)) {
610:       auto jivalue_element = jarray->getElement(i);
611:       auto element = JIValue::JIValueToAtIValue(jivalue_element);
612:       list.push_back(element);
613:     }
614:     return at::IValue{list};
615:   } else if (JIValue::kTypeCodeDictStringKey == typeCode) {
616:     static const auto jMethodGetDictStringKey =
617:         JIValue::javaClassStatic()
618:             ->getMethod<facebook::jni::JMap<jstring, JIValue::javaobject>::
619:                             javaobject()>("toDictStringKey");
620:     auto jmap = jMethodGetDictStringKey(jivalue);
621:     auto it = jmap->begin();
622:     if (it == jmap->end()) {
623:       return at::IValue{c10::impl::GenericDict(
624:           c10::StringType::get(), c10::TensorType::get())};
625:     }
626: 
627:     auto firstEntryValue = JIValue::JIValueToAtIValue(it->second);
628:     c10::impl::GenericDict dict{
629:         c10::StringType::get(), c10::unshapedType(firstEntryValue.type())};
630:     dict.insert(it->first->toStdString(), firstEntryValue);
631:     it++;
632:     for (; it != jmap->end(); it++) {
633:       dict.insert(
634:           it->first->toStdString(), JIValue::JIValueToAtIValue(it->second));
635:     }
636:     return at::IValue{dict};
637:   } else if (JIValue::kTypeCodeDictLongKey == typeCode) {
638:     static const auto jMethodGetDictLongKey =
639:         JIValue::javaClassStatic()
640:             ->getMethod<facebook::jni::JMap<
641:                 facebook::jni::JLong::javaobject,
642:                 JIValue::javaobject>::javaobject()>("toDictLongKey");
643:     auto jmap = jMethodGetDictLongKey(jivalue);
644:     auto it = jmap->begin();
645:     if (it == jmap->end()) {
646:       return at::IValue{
647:           c10::impl::GenericDict(c10::IntType::get(), c10::TensorType::get())};
648:     }
649: 
650:     auto firstEntryValue = JIValue::JIValueToAtIValue(it->second);
651:     c10::impl::GenericDict dict{
652:         c10::IntType::get(), c10::unshapedType(firstEntryValue.type())};
653:     dict.insert((int64_t)it->first->longValue(), firstEntryValue);
654:     it++;
655:     for (; it != jmap->end(); it++) {
656:       dict.insert(
657:           (int64_t)it->first->longValue(),
658:           JIValue::JIValueToAtIValue(it->second));
659:     }
660:     return at::IValue{dict};
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 661-697
````cpp
661:   }
662: 
663:   facebook::jni::throwNewJavaException(
664:       facebook::jni::gJavaLangIllegalArgumentException,
665:       "Unknown IValue typeCode %d",
666:       typeCode);
667: }
668: 
669: #if defined(__ANDROID__)
670: class PyTorchAndroidJni : public facebook::jni::JavaClass<PyTorchAndroidJni> {
671:  public:
672:   constexpr static auto kJavaDescriptor = "Lorg/pytorch/PyTorchAndroid;";
673: 
674:   static void registerNatives() {
675:     javaClassStatic()->registerNatives({
676:         makeNativeMethod(
677:             "nativeSetNumThreads", PyTorchAndroidJni::setNumThreads),
678:     });
679:   }
680: 
681:   static void setNumThreads(facebook::jni::alias_ref<jclass>, jint numThreads) {
682:     caffe2::pthreadpool()->set_thread_count(numThreads);
683:   }
684: };
685: #endif
686: 
687: void common_registerNatives() {
688:   static const int once = []() {
689: #if defined(__ANDROID__)
690:     pytorch_jni::PyTorchAndroidJni::registerNatives();
691: #endif
692:     return 0;
693:   }();
694:   ((void)once);
695: }
696: 
697: } // namespace pytorch_jni
````
- EN: Declares or extends types including `PyTorchAndroidJni`.
- CN: 声明或扩展类型，包括 `PyTorchAndroidJni`。
- EN: Implements callable logic such as `registerNatives`, `setNumThreads`, `common_registerNatives`.
- CN: 实现可调用逻辑，例如 `registerNatives`, `setNumThreads`, `common_registerNatives`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- Symbol `JHashMap` / 符号 `JHashMap`
- Symbol `TensorHybrid` / 符号 `TensorHybrid`
- Symbol `PyTorchAndroidJni` / 符号 `PyTorchAndroidJni`
- Symbol `deviceJniCodeToDeviceType` / 符号 `deviceJniCodeToDeviceType`

## Dependencies / 依赖关系
- C/C++ includes: `cassert`, `iostream`, `memory`, `string`, `c10/core/MemoryFormat.h`, `c10/util/irange.h`, `fbjni/ByteBuffer.h`, `fbjni/fbjni.h`, `pytorch_jni_common.h`, `caffe2/utils/threadpool/pthreadpool-cpp.h`
- C/C++ 头文件: `cassert`, `iostream`, `memory`, `string`, `c10/core/MemoryFormat.h`, `c10/util/irange.h`, `fbjni/ByteBuffer.h`, `fbjni/fbjni.h`, `pytorch_jni_common.h`, `caffe2/utils/threadpool/pthreadpool-cpp.h`
