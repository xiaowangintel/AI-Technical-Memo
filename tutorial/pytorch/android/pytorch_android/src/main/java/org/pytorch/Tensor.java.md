# Tensor.java — Code Analysis / 代码分析

## Source / 来源
- File: `android/pytorch_android/src/main/java/org/pytorch/Tensor.java`
- Repository: `pytorch`
- Purpose (EN): Supports Android/mobile runtime integration in the PyTorch repository.
- 用途 (CN): 为 PyTorch 仓库中的 Android/移动运行时集成提供支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````java
 1: package org.pytorch;
 2: 
 3: import com.facebook.jni.HybridData;
 4: import com.facebook.jni.annotations.DoNotStrip;
 5: import java.nio.Buffer;
 6: import java.nio.ByteBuffer;
 7: import java.nio.ByteOrder;
 8: import java.nio.DoubleBuffer;
 9: import java.nio.FloatBuffer;
10: import java.nio.IntBuffer;
11: import java.nio.LongBuffer;
12: import java.util.Arrays;
13: import java.util.Locale;
14: 
15: /**
16:  * Representation of a Tensor. Behavior is similar to PyTorch's tensor objects.
17:  *
18:  * <p>Most tensors will be constructed as {@code Tensor.fromBlob(data, shape)}, where {@code data}
19:  * can be an array or a direct {@link Buffer} (of the proper subclass). Helper methods are provided
20:  * to allocate buffers properly.
21:  *
22:  * <p>To access Tensor data, see {@link #dtype()}, {@link #shape()}, and various {@code getDataAs*}
23:  * methods.
24:  *
25:  * <p>When constructing {@code Tensor} objects with {@code data} as an array, it is not specified
26:  * whether this data is copied or retained as a reference so it is recommended not to modify it
27:  * after constructing. {@code data} passed as a {@link Buffer} is not copied, so it can be modified
28:  * between {@link Module} calls to avoid reallocation. Data retrieved from {@code Tensor} objects
29:  * may be copied or may be a reference to the {@code Tensor}'s internal data buffer. {@code shape}
30:  * is always copied.
31:  */
32: public abstract class Tensor {
33:   private static final String ERROR_MSG_DATA_BUFFER_NOT_NULL = "Data buffer must be not null";
34:   private static final String ERROR_MSG_DATA_ARRAY_NOT_NULL = "Data array must be not null";
35:   private static final String ERROR_MSG_SHAPE_NOT_NULL = "Shape must be not null";
36:   private static final String ERROR_MSG_SHAPE_NON_NEGATIVE = "Shape elements must be non negative";
37:   private static final String ERROR_MSG_DATA_BUFFER_MUST_HAVE_NATIVE_BYTE_ORDER =
38:       "Data buffer must have native byte order (java.nio.ByteOrder#nativeOrder)";
39:   private static final String ERROR_MSG_DATA_BUFFER_MUST_BE_DIRECT =
40:       "Data buffer must be direct (java.nio.ByteBuffer#allocateDirect)";
41: 
42:   @DoNotStrip final long[] shape;
43:   final MemoryFormat memoryFormat;
44: 
45:   private static final int INT_SIZE_BYTES = 4;
46:   private static final int FLOAT_SIZE_BYTES = 4;
47:   private static final int LONG_SIZE_BYTES = 8;
48:   private static final int DOUBLE_SIZE_BYTES = 8;
49: 
50:   /**
51:    * Allocates a new direct {@link java.nio.ByteBuffer} with native byte order with specified
52:    * capacity that can be used in {@link Tensor#fromBlob(ByteBuffer, long[])}, {@link
53:    * Tensor#fromBlobUnsigned(ByteBuffer, long[])}.
54:    *
55:    * @param numElements capacity (number of elements) of result buffer.
56:    */
57:   public static ByteBuffer allocateByteBuffer(int numElements) {
58:     return ByteBuffer.allocateDirect(numElements).order(ByteOrder.nativeOrder());
59:   }
60: 
````
- EN: Handles module imports such as `com.facebook.jni.HybridData;`, `com.facebook.jni.annotations.DoNotStrip;`, `java.nio.Buffer;`, `java.nio.ByteBuffer;`.
- CN: 处理模块导入，例如 `com.facebook.jni.HybridData;`, `com.facebook.jni.annotations.DoNotStrip;`, `java.nio.Buffer;`, `java.nio.ByteBuffer;`。
- EN: Imports Java types such as `com.facebook.jni.HybridData`, `com.facebook.jni.annotations.DoNotStrip`, `java.nio.Buffer`, `java.nio.ByteBuffer`.
- CN: 导入 Java 类型，例如 `com.facebook.jni.HybridData`, `com.facebook.jni.annotations.DoNotStrip`, `java.nio.Buffer`, `java.nio.ByteBuffer`。
- EN: Declares or extends types including `Tensor`.
- CN: 声明或扩展类型，包括 `Tensor`。
- EN: Implements callable logic such as `order`, `allocateByteBuffer`.
- CN: 实现可调用逻辑，例如 `order`, `allocateByteBuffer`。

### Lines 61-120
````java
 61:   /**
 62:    * Allocates a new direct {@link java.nio.IntBuffer} with native byte order with specified
 63:    * capacity that can be used in {@link Tensor#fromBlob(IntBuffer, long[])}.
 64:    *
 65:    * @param numElements capacity (number of elements) of result buffer.
 66:    */
 67:   public static IntBuffer allocateIntBuffer(int numElements) {
 68:     return ByteBuffer.allocateDirect(numElements * INT_SIZE_BYTES)
 69:         .order(ByteOrder.nativeOrder())
 70:         .asIntBuffer();
 71:   }
 72: 
 73:   /**
 74:    * Allocates a new direct {@link java.nio.FloatBuffer} with native byte order with specified
 75:    * capacity that can be used in {@link Tensor#fromBlob(FloatBuffer, long[])}.
 76:    *
 77:    * @param numElements capacity (number of elements) of result buffer.
 78:    */
 79:   public static FloatBuffer allocateFloatBuffer(int numElements) {
 80:     return ByteBuffer.allocateDirect(numElements * FLOAT_SIZE_BYTES)
 81:         .order(ByteOrder.nativeOrder())
 82:         .asFloatBuffer();
 83:   }
 84: 
 85:   /**
 86:    * Allocates a new direct {@link java.nio.LongBuffer} with native byte order with specified
 87:    * capacity that can be used in {@link Tensor#fromBlob(LongBuffer, long[])}.
 88:    *
 89:    * @param numElements capacity (number of elements) of result buffer.
 90:    */
 91:   public static LongBuffer allocateLongBuffer(int numElements) {
 92:     return ByteBuffer.allocateDirect(numElements * LONG_SIZE_BYTES)
 93:         .order(ByteOrder.nativeOrder())
 94:         .asLongBuffer();
 95:   }
 96: 
 97:   /**
 98:    * Allocates a new direct {@link java.nio.DoubleBuffer} with native byte order with specified
 99:    * capacity that can be used in {@link Tensor#fromBlob(DoubleBuffer, long[])}.
100:    *
101:    * @param numElements capacity (number of elements) of result buffer.
102:    */
103:   public static DoubleBuffer allocateDoubleBuffer(int numElements) {
104:     return ByteBuffer.allocateDirect(numElements * DOUBLE_SIZE_BYTES)
105:         .order(ByteOrder.nativeOrder())
106:         .asDoubleBuffer();
107:   }
108: 
109:   /**
110:    * Creates a new Tensor instance with dtype torch.uint8 with specified shape and data as array of
111:    * bytes.
112:    *
113:    * @param data Tensor elements
114:    * @param shape Tensor shape
115:    */
116:   public static Tensor fromBlobUnsigned(byte[] data, long[] shape, MemoryFormat memoryFormat) {
117:     checkArgument(data != null, ERROR_MSG_DATA_ARRAY_NOT_NULL);
118:     checkArgument(shape != null, ERROR_MSG_SHAPE_NOT_NULL);
119:     checkShape(shape);
120:     checkShapeAndDataCapacityConsistency(data.length, shape);
````
- EN: Implements callable logic such as `allocateIntBuffer`, `allocateFloatBuffer`, `allocateLongBuffer`, `allocateDoubleBuffer`.
- CN: 实现可调用逻辑，例如 `allocateIntBuffer`, `allocateFloatBuffer`, `allocateLongBuffer`, `allocateDoubleBuffer`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````java
121:     final ByteBuffer byteBuffer = allocateByteBuffer((int) numel(shape));
122:     byteBuffer.put(data);
123:     return new Tensor_uint8(byteBuffer, shape, memoryFormat);
124:   }
125: 
126:   public static Tensor fromBlobUnsigned(byte[] data, long[] shape) {
127:     return fromBlobUnsigned(data, shape, MemoryFormat.CONTIGUOUS);
128:   }
129: 
130:   /**
131:    * Creates a new Tensor instance with dtype torch.int8 with specified shape and data as array of
132:    * bytes.
133:    *
134:    * @param data Tensor elements
135:    * @param shape Tensor shape
136:    */
137:   public static Tensor fromBlob(byte[] data, long[] shape, MemoryFormat memoryFormat) {
138:     checkArgument(data != null, ERROR_MSG_DATA_ARRAY_NOT_NULL);
139:     checkArgument(shape != null, ERROR_MSG_SHAPE_NOT_NULL);
140:     checkShape(shape);
141:     checkShapeAndDataCapacityConsistency(data.length, shape);
142:     final ByteBuffer byteBuffer = allocateByteBuffer((int) numel(shape));
143:     byteBuffer.put(data);
144:     return new Tensor_int8(byteBuffer, shape, memoryFormat);
145:   }
146: 
147:   public static Tensor fromBlob(byte[] data, long[] shape) {
148:     return fromBlob(data, shape, MemoryFormat.CONTIGUOUS);
149:   }
150: 
151:   /**
152:    * Creates a new Tensor instance with dtype torch.int32 with specified shape and data as array of
153:    * ints.
154:    *
155:    * @param data Tensor elements
156:    * @param shape Tensor shape
157:    */
158:   public static Tensor fromBlob(int[] data, long[] shape, MemoryFormat memoryFormat) {
159:     checkArgument(data != null, ERROR_MSG_DATA_ARRAY_NOT_NULL);
160:     checkArgument(shape != null, ERROR_MSG_SHAPE_NOT_NULL);
161:     checkShape(shape);
162:     checkShapeAndDataCapacityConsistency(data.length, shape);
163:     final IntBuffer intBuffer = allocateIntBuffer((int) numel(shape));
164:     intBuffer.put(data);
165:     return new Tensor_int32(intBuffer, shape, memoryFormat);
166:   }
167: 
168:   public static Tensor fromBlob(int[] data, long[] shape) {
169:     return fromBlob(data, shape, MemoryFormat.CONTIGUOUS);
170:   }
171: 
172:   /**
173:    * Creates a new Tensor instance with dtype torch.float32 with specified shape and data as array
174:    * of floats.
175:    *
176:    * @param data Tensor elements
177:    * @param shape Tensor shape
178:    */
179:   public static Tensor fromBlob(float[] data, long[] shape, MemoryFormat memoryFormat) {
180:     checkArgument(data != null, ERROR_MSG_DATA_ARRAY_NOT_NULL);
````
- EN: Implements callable logic such as `fromBlobUnsigned`, `fromBlob`.
- CN: 实现可调用逻辑，例如 `fromBlobUnsigned`, `fromBlob`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````java
181:     checkArgument(shape != null, ERROR_MSG_SHAPE_NOT_NULL);
182:     checkShape(shape);
183:     checkShapeAndDataCapacityConsistency(data.length, shape);
184:     final FloatBuffer floatBuffer = allocateFloatBuffer((int) numel(shape));
185:     floatBuffer.put(data);
186:     return new Tensor_float32(floatBuffer, shape, memoryFormat);
187:   }
188: 
189:   public static Tensor fromBlob(float[] data, long[] shape) {
190:     return fromBlob(data, shape, MemoryFormat.CONTIGUOUS);
191:   }
192: 
193:   /**
194:    * Creates a new Tensor instance with dtype torch.int64 with specified shape and data as array of
195:    * longs.
196:    *
197:    * @param data Tensor elements
198:    * @param shape Tensor shape
199:    */
200:   public static Tensor fromBlob(long[] data, long[] shape, MemoryFormat memoryFormat) {
201:     checkArgument(data != null, ERROR_MSG_DATA_ARRAY_NOT_NULL);
202:     checkArgument(shape != null, ERROR_MSG_SHAPE_NOT_NULL);
203:     checkShape(shape);
204:     checkShapeAndDataCapacityConsistency(data.length, shape);
205:     final LongBuffer longBuffer = allocateLongBuffer((int) numel(shape));
206:     longBuffer.put(data);
207:     return new Tensor_int64(longBuffer, shape, memoryFormat);
208:   }
209: 
210:   public static Tensor fromBlob(long[] data, long[] shape) {
211:     return fromBlob(data, shape, MemoryFormat.CONTIGUOUS);
212:   }
213: 
214:   /**
215:    * Creates a new Tensor instance with dtype torch.float64 with specified shape and data as array
216:    * of doubles.
217:    *
218:    * @param shape Tensor shape
219:    * @param data Tensor elements
220:    */
221:   public static Tensor fromBlob(double[] data, long[] shape, MemoryFormat memoryFormat) {
222:     checkArgument(data != null, ERROR_MSG_DATA_ARRAY_NOT_NULL);
223:     checkArgument(shape != null, ERROR_MSG_SHAPE_NOT_NULL);
224:     checkShape(shape);
225:     checkShapeAndDataCapacityConsistency(data.length, shape);
226:     final DoubleBuffer doubleBuffer = allocateDoubleBuffer((int) numel(shape));
227:     doubleBuffer.put(data);
228:     return new Tensor_float64(doubleBuffer, shape, memoryFormat);
229:   }
230: 
231:   public static Tensor fromBlob(double[] data, long[] shape) {
232:     return fromBlob(data, shape, MemoryFormat.CONTIGUOUS);
233:   }
234: 
235:   /**
236:    * Creates a new Tensor instance with dtype torch.uint8 with specified shape and data.
237:    *
238:    * @param data Direct buffer with native byte order that contains {@code Tensor.numel(shape)}
239:    *     elements. The buffer is used directly without copying, and changes to its content will
240:    *     change the tensor.
````
- EN: Implements callable logic such as `fromBlob`.
- CN: 实现可调用逻辑，例如 `fromBlob`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````java
241:    * @param shape Tensor shape
242:    */
243:   public static Tensor fromBlobUnsigned(ByteBuffer data, long[] shape, MemoryFormat memoryFormat) {
244:     checkArgument(data != null, ERROR_MSG_DATA_BUFFER_NOT_NULL);
245:     checkArgument(shape != null, ERROR_MSG_SHAPE_NOT_NULL);
246:     checkShape(shape);
247:     checkShapeAndDataCapacityConsistency(data.capacity(), shape);
248:     checkArgument(data.isDirect(), ERROR_MSG_DATA_BUFFER_MUST_BE_DIRECT);
249:     checkArgument(
250:         (data.order() == ByteOrder.nativeOrder()),
251:         ERROR_MSG_DATA_BUFFER_MUST_HAVE_NATIVE_BYTE_ORDER);
252:     return new Tensor_uint8(data, shape, memoryFormat);
253:   }
254: 
255:   public static Tensor fromBlobUnsigned(ByteBuffer data, long[] shape) {
256:     return fromBlobUnsigned(data, shape, MemoryFormat.CONTIGUOUS);
257:   }
258: 
259:   /**
260:    * Creates a new Tensor instance with dtype torch.int8 with specified shape and data.
261:    *
262:    * @param data Direct buffer with native byte order that contains {@code Tensor.numel(shape)}
263:    *     elements. The buffer is used directly without copying, and changes to its content will
264:    *     change the tensor.
265:    * @param shape Tensor shape
266:    */
267:   public static Tensor fromBlob(ByteBuffer data, long[] shape, MemoryFormat memoryFormat) {
268:     checkArgument(data != null, ERROR_MSG_DATA_BUFFER_NOT_NULL);
269:     checkArgument(shape != null, ERROR_MSG_SHAPE_NOT_NULL);
270:     checkShape(shape);
271:     checkShapeAndDataCapacityConsistency(data.capacity(), shape);
272:     checkArgument(data.isDirect(), ERROR_MSG_DATA_BUFFER_MUST_BE_DIRECT);
273:     checkArgument(
274:         (data.order() == ByteOrder.nativeOrder()),
275:         ERROR_MSG_DATA_BUFFER_MUST_HAVE_NATIVE_BYTE_ORDER);
276:     return new Tensor_int8(data, shape, memoryFormat);
277:   }
278: 
279:   public static Tensor fromBlob(ByteBuffer data, long[] shape) {
280:     return fromBlob(data, shape, MemoryFormat.CONTIGUOUS);
281:   }
282: 
283:   /**
284:    * Creates a new Tensor instance with dtype torch.int32 with specified shape and data.
285:    *
286:    * @param data Direct buffer with native byte order that contains {@code Tensor.numel(shape)}
287:    *     elements. The buffer is used directly without copying, and changes to its content will
288:    *     change the tensor.
289:    * @param shape Tensor shape
290:    */
291:   public static Tensor fromBlob(IntBuffer data, long[] shape, MemoryFormat memoryFormat) {
292:     checkArgument(data != null, ERROR_MSG_DATA_BUFFER_NOT_NULL);
293:     checkArgument(shape != null, ERROR_MSG_SHAPE_NOT_NULL);
294:     checkShape(shape);
295:     checkShapeAndDataCapacityConsistency(data.capacity(), shape);
296:     checkArgument(data.isDirect(), ERROR_MSG_DATA_BUFFER_MUST_BE_DIRECT);
297:     checkArgument(
298:         (data.order() == ByteOrder.nativeOrder()),
299:         ERROR_MSG_DATA_BUFFER_MUST_HAVE_NATIVE_BYTE_ORDER);
300:     return new Tensor_int32(data, shape, memoryFormat);
````
- EN: Implements callable logic such as `fromBlobUnsigned`, `fromBlob`.
- CN: 实现可调用逻辑，例如 `fromBlobUnsigned`, `fromBlob`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````java
301:   }
302: 
303:   public static Tensor fromBlob(IntBuffer data, long[] shape) {
304:     return fromBlob(data, shape, MemoryFormat.CONTIGUOUS);
305:   }
306: 
307:   /**
308:    * Creates a new Tensor instance with dtype torch.float32 with specified shape and data.
309:    *
310:    * @param data Direct buffer with native byte order that contains {@code Tensor.numel(shape)}
311:    *     elements. The buffer is used directly without copying, and changes to its content will
312:    *     change the tensor.
313:    * @param shape Tensor shape
314:    */
315:   public static Tensor fromBlob(FloatBuffer data, long[] shape, MemoryFormat memoryFormat) {
316:     checkArgument(data != null, ERROR_MSG_DATA_BUFFER_NOT_NULL);
317:     checkArgument(shape != null, ERROR_MSG_SHAPE_NOT_NULL);
318:     checkShape(shape);
319:     checkShapeAndDataCapacityConsistency(data.capacity(), shape);
320:     checkArgument(data.isDirect(), ERROR_MSG_DATA_BUFFER_MUST_BE_DIRECT);
321:     checkArgument(
322:         (data.order() == ByteOrder.nativeOrder()),
323:         ERROR_MSG_DATA_BUFFER_MUST_HAVE_NATIVE_BYTE_ORDER);
324:     return new Tensor_float32(data, shape, memoryFormat);
325:   }
326: 
327:   public static Tensor fromBlob(FloatBuffer data, long[] shape) {
328:     return fromBlob(data, shape, MemoryFormat.CONTIGUOUS);
329:   }
330: 
331:   /**
332:    * Creates a new Tensor instance with dtype torch.int64 with specified shape and data.
333:    *
334:    * @param data Direct buffer with native byte order that contains {@code Tensor.numel(shape)}
335:    *     elements. The buffer is used directly without copying, and changes to its content will
336:    *     change the tensor.
337:    * @param shape Tensor shape
338:    */
339:   public static Tensor fromBlob(LongBuffer data, long[] shape, MemoryFormat memoryFormat) {
340:     checkArgument(data != null, ERROR_MSG_DATA_BUFFER_NOT_NULL);
341:     checkArgument(shape != null, ERROR_MSG_SHAPE_NOT_NULL);
342:     checkShape(shape);
343:     checkShapeAndDataCapacityConsistency(data.capacity(), shape);
344:     checkArgument(data.isDirect(), ERROR_MSG_DATA_BUFFER_MUST_BE_DIRECT);
345:     checkArgument(
346:         (data.order() == ByteOrder.nativeOrder()),
347:         ERROR_MSG_DATA_BUFFER_MUST_HAVE_NATIVE_BYTE_ORDER);
348:     return new Tensor_int64(data, shape, memoryFormat);
349:   }
350: 
351:   public static Tensor fromBlob(LongBuffer data, long[] shape) {
352:     return fromBlob(data, shape, MemoryFormat.CONTIGUOUS);
353:   }
354: 
355:   /**
356:    * Creates a new Tensor instance with dtype torch.float64 with specified shape and data.
357:    *
358:    * @param data Direct buffer with native byte order that contains {@code Tensor.numel(shape)}
359:    *     elements. The buffer is used directly without copying, and changes to its content will
360:    *     change the tensor.
````
- EN: Implements callable logic such as `fromBlob`.
- CN: 实现可调用逻辑，例如 `fromBlob`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-420
````java
361:    * @param shape Tensor shape
362:    */
363:   public static Tensor fromBlob(DoubleBuffer data, long[] shape, MemoryFormat memoryFormat) {
364:     checkArgument(data != null, ERROR_MSG_DATA_BUFFER_NOT_NULL);
365:     checkArgument(shape != null, ERROR_MSG_SHAPE_NOT_NULL);
366:     checkShape(shape);
367:     checkShapeAndDataCapacityConsistency(data.capacity(), shape);
368:     checkArgument(data.isDirect(), ERROR_MSG_DATA_BUFFER_MUST_BE_DIRECT);
369:     checkArgument(
370:         (data.order() == ByteOrder.nativeOrder()),
371:         ERROR_MSG_DATA_BUFFER_MUST_HAVE_NATIVE_BYTE_ORDER);
372:     return new Tensor_float64(data, shape, memoryFormat);
373:   }
374: 
375:   public static Tensor fromBlob(DoubleBuffer data, long[] shape) {
376:     return fromBlob(data, shape, MemoryFormat.CONTIGUOUS);
377:   }
378: 
379:   @DoNotStrip private HybridData mHybridData;
380: 
381:   private Tensor(long[] shape, MemoryFormat memoryFormat) {
382:     checkShape(shape);
383:     this.shape = Arrays.copyOf(shape, shape.length);
384:     this.memoryFormat = memoryFormat;
385:   }
386: 
387:   /** Returns the number of elements in this tensor. */
388:   public long numel() {
389:     return numel(this.shape);
390:   }
391: 
392:   /** Calculates the number of elements in a tensor with the specified shape. */
393:   public static long numel(long[] shape) {
394:     checkShape(shape);
395:     int result = 1;
396:     for (long s : shape) {
397:       result *= s;
398:     }
399:     return result;
400:   }
401: 
402:   /** Returns the shape of this tensor. (The array is a fresh copy.) */
403:   public long[] shape() {
404:     return Arrays.copyOf(shape, shape.length);
405:   }
406: 
407:   /** Returns the memory format of this tensor. */
408:   public MemoryFormat memoryFormat() {
409:     return memoryFormat;
410:   }
411: 
412:   /** @return data type of this tensor. */
413:   public abstract DType dtype();
414: 
415:   // Called from native
416:   @DoNotStrip
417:   int dtypeJniCode() {
418:     return dtype().jniCode;
419:   }
420: 
````
- EN: Implements callable logic such as `fromBlob`, `numel`, `shape`, `memoryFormat`.
- CN: 实现可调用逻辑，例如 `fromBlob`, `numel`, `shape`, `memoryFormat`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 421-480
````java
421:   // Called from native
422:   @DoNotStrip
423:   int memoryFormatJniCode() {
424:     return memoryFormat.jniCode;
425:   }
426: 
427:   /**
428:    * @return a Java byte array that contains the tensor data. This may be a copy or reference.
429:    * @throws IllegalStateException if it is called for a non-int8 tensor.
430:    */
431:   public byte[] getDataAsByteArray() {
432:     throw new IllegalStateException(
433:         "Tensor of type " + getClass().getSimpleName() + " cannot return data as byte array.");
434:   }
435: 
436:   /**
437:    * @return a Java byte array that contains the tensor data. This may be a copy or reference.
438:    * @throws IllegalStateException if it is called for a non-uint8 tensor.
439:    */
440:   public byte[] getDataAsUnsignedByteArray() {
441:     throw new IllegalStateException(
442:         "Tensor of type " + getClass().getSimpleName() + " cannot return data as byte array.");
443:   }
444: 
445:   /**
446:    * @return a Java int array that contains the tensor data. This may be a copy or reference.
447:    * @throws IllegalStateException if it is called for a non-int32 tensor.
448:    */
449:   public int[] getDataAsIntArray() {
450:     throw new IllegalStateException(
451:         "Tensor of type " + getClass().getSimpleName() + " cannot return data as int array.");
452:   }
453: 
454:   /**
455:    * @return a Java float array that contains the tensor data. This may be a copy or reference.
456:    * @throws IllegalStateException if it is called for a non-float32 tensor.
457:    */
458:   public float[] getDataAsFloatArray() {
459:     throw new IllegalStateException(
460:         "Tensor of type " + getClass().getSimpleName() + " cannot return data as float array.");
461:   }
462: 
463:   /**
464:    * @return a Java long array that contains the tensor data. This may be a copy or reference.
465:    * @throws IllegalStateException if it is called for a non-int64 tensor.
466:    */
467:   public long[] getDataAsLongArray() {
468:     throw new IllegalStateException(
469:         "Tensor of type " + getClass().getSimpleName() + " cannot return data as long array.");
470:   }
471: 
472:   /**
473:    * @return a Java double array that contains the tensor data. This may be a copy or reference.
474:    * @throws IllegalStateException if it is called for a non-float64 tensor.
475:    */
476:   public double[] getDataAsDoubleArray() {
477:     throw new IllegalStateException(
478:         "Tensor of type " + getClass().getSimpleName() + " cannot return data as double array.");
479:   }
480: 
````
- EN: Implements callable logic such as `memoryFormatJniCode`, `getDataAsByteArray`, `getDataAsUnsignedByteArray`, `getDataAsIntArray`.
- CN: 实现可调用逻辑，例如 `memoryFormatJniCode`, `getDataAsByteArray`, `getDataAsUnsignedByteArray`, `getDataAsIntArray`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 481-540
````java
481:   @DoNotStrip
482:   Buffer getRawDataBuffer() {
483:     throw new IllegalStateException(
484:         "Tensor of type " + getClass().getSimpleName() + " cannot " + "return raw data buffer.");
485:   }
486: 
487:   static class Tensor_uint8 extends Tensor {
488:     private final ByteBuffer data;
489: 
490:     private Tensor_uint8(ByteBuffer data, long[] shape, MemoryFormat memoryFormat) {
491:       super(shape, memoryFormat);
492:       this.data = data;
493:     }
494: 
495:     @Override
496:     public DType dtype() {
497:       return DType.UINT8;
498:     }
499: 
500:     @Override
501:     Buffer getRawDataBuffer() {
502:       return data;
503:     }
504: 
505:     @Override
506:     public byte[] getDataAsUnsignedByteArray() {
507:       data.rewind();
508:       byte[] arr = new byte[data.remaining()];
509:       data.get(arr);
510:       return arr;
511:     }
512: 
513:     @Override
514:     public String toString() {
515:       return String.format("Tensor(%s, dtype=torch.uint8)", Arrays.toString(shape));
516:     }
517:   }
518: 
519:   static class Tensor_int8 extends Tensor {
520:     private final ByteBuffer data;
521: 
522:     private Tensor_int8(ByteBuffer data, long[] shape, MemoryFormat memoryFormat) {
523:       super(shape, memoryFormat);
524:       this.data = data;
525:     }
526: 
527:     @Override
528:     public DType dtype() {
529:       return DType.INT8;
530:     }
531: 
532:     @Override
533:     Buffer getRawDataBuffer() {
534:       return data;
535:     }
536: 
537:     @Override
538:     public byte[] getDataAsByteArray() {
539:       data.rewind();
540:       byte[] arr = new byte[data.remaining()];
````
- EN: Declares or extends types including `Tensor_uint8`, `Tensor_int8`.
- CN: 声明或扩展类型，包括 `Tensor_uint8`, `Tensor_int8`。
- EN: Implements callable logic such as `getRawDataBuffer`, `dtype`, `getDataAsUnsignedByteArray`, `toString`.
- CN: 实现可调用逻辑，例如 `getRawDataBuffer`, `dtype`, `getDataAsUnsignedByteArray`, `toString`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 541-600
````java
541:       data.get(arr);
542:       return arr;
543:     }
544: 
545:     @Override
546:     public String toString() {
547:       return String.format("Tensor(%s, dtype=torch.int8)", Arrays.toString(shape));
548:     }
549:   }
550: 
551:   static class Tensor_int32 extends Tensor {
552:     private final IntBuffer data;
553: 
554:     private Tensor_int32(IntBuffer data, long[] shape, MemoryFormat memoryFormat) {
555:       super(shape, memoryFormat);
556:       this.data = data;
557:     }
558: 
559:     @Override
560:     public DType dtype() {
561:       return DType.INT32;
562:     }
563: 
564:     @Override
565:     Buffer getRawDataBuffer() {
566:       return data;
567:     }
568: 
569:     @Override
570:     public int[] getDataAsIntArray() {
571:       data.rewind();
572:       int[] arr = new int[data.remaining()];
573:       data.get(arr);
574:       return arr;
575:     }
576: 
577:     @Override
578:     public String toString() {
579:       return String.format("Tensor(%s, dtype=torch.int32)", Arrays.toString(shape));
580:     }
581:   }
582: 
583:   static class Tensor_float32 extends Tensor {
584:     private final FloatBuffer data;
585: 
586:     Tensor_float32(FloatBuffer data, long[] shape, MemoryFormat memoryFormat) {
587:       super(shape, memoryFormat);
588:       this.data = data;
589:     }
590: 
591:     @Override
592:     public float[] getDataAsFloatArray() {
593:       data.rewind();
594:       float[] arr = new float[data.remaining()];
595:       data.get(arr);
596:       return arr;
597:     }
598: 
599:     @Override
600:     public DType dtype() {
````
- EN: Declares or extends types including `Tensor_int32`, `Tensor_float32`.
- CN: 声明或扩展类型，包括 `Tensor_int32`, `Tensor_float32`。
- EN: Implements callable logic such as `toString`, `dtype`, `getRawDataBuffer`, `getDataAsIntArray`.
- CN: 实现可调用逻辑，例如 `toString`, `dtype`, `getRawDataBuffer`, `getDataAsIntArray`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 601-660
````java
601:       return DType.FLOAT32;
602:     }
603: 
604:     @Override
605:     Buffer getRawDataBuffer() {
606:       return data;
607:     }
608: 
609:     @Override
610:     public String toString() {
611:       return String.format("Tensor(%s, dtype=torch.float32)", Arrays.toString(shape));
612:     }
613:   }
614: 
615:   static class Tensor_int64 extends Tensor {
616:     private final LongBuffer data;
617: 
618:     private Tensor_int64(LongBuffer data, long[] shape, MemoryFormat memoryFormat) {
619:       super(shape, memoryFormat);
620:       this.data = data;
621:     }
622: 
623:     @Override
624:     public DType dtype() {
625:       return DType.INT64;
626:     }
627: 
628:     @Override
629:     Buffer getRawDataBuffer() {
630:       return data;
631:     }
632: 
633:     @Override
634:     public long[] getDataAsLongArray() {
635:       data.rewind();
636:       long[] arr = new long[data.remaining()];
637:       data.get(arr);
638:       return arr;
639:     }
640: 
641:     @Override
642:     public String toString() {
643:       return String.format("Tensor(%s, dtype=torch.int64)", Arrays.toString(shape));
644:     }
645:   }
646: 
647:   static class Tensor_float64 extends Tensor {
648:     private final DoubleBuffer data;
649: 
650:     private Tensor_float64(DoubleBuffer data, long[] shape, MemoryFormat memoryFormat) {
651:       super(shape, memoryFormat);
652:       this.data = data;
653:     }
654: 
655:     @Override
656:     public DType dtype() {
657:       return DType.FLOAT64;
658:     }
659: 
660:     @Override
````
- EN: Declares or extends types including `Tensor_int64`, `Tensor_float64`.
- CN: 声明或扩展类型，包括 `Tensor_int64`, `Tensor_float64`。
- EN: Implements callable logic such as `getRawDataBuffer`, `toString`, `dtype`, `getDataAsLongArray`.
- CN: 实现可调用逻辑，例如 `getRawDataBuffer`, `toString`, `dtype`, `getDataAsLongArray`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 661-720
````java
661:     Buffer getRawDataBuffer() {
662:       return data;
663:     }
664: 
665:     @Override
666:     public double[] getDataAsDoubleArray() {
667:       data.rewind();
668:       double[] arr = new double[data.remaining()];
669:       data.get(arr);
670:       return arr;
671:     }
672: 
673:     @Override
674:     public String toString() {
675:       return String.format("Tensor(%s, dtype=torch.float64)", Arrays.toString(shape));
676:     }
677:   }
678: 
679:   // region checks
680:   private static void checkArgument(boolean expression, String errorMessage, Object... args) {
681:     if (!expression) {
682:       throw new IllegalArgumentException(String.format(Locale.US, errorMessage, args));
683:     }
684:   }
685: 
686:   private static void checkShape(long[] shape) {
687:     checkArgument(shape != null, ERROR_MSG_SHAPE_NOT_NULL);
688:     for (int i = 0; i < shape.length; i++) {
689:       checkArgument(shape[i] >= 0, ERROR_MSG_SHAPE_NON_NEGATIVE);
690:     }
691:   }
692: 
693:   private static void checkShapeAndDataCapacityConsistency(int dataCapacity, long[] shape) {
694:     final long numel = numel(shape);
695:     checkArgument(
696:         numel == dataCapacity,
697:         "Inconsistent data capacity:%d and shape number elements:%d shape:%s",
698:         dataCapacity,
699:         numel,
700:         Arrays.toString(shape));
701:   }
702:   // endregion checks
703: 
704:   // Called from native
705:   @DoNotStrip
706:   private static Tensor nativeNewTensor(
707:       ByteBuffer data, long[] shape, int dtype, int memoryFormatCode, HybridData hybridData) {
708:     Tensor tensor = null;
709: 
710:     MemoryFormat memoryFormat = MemoryFormat.CONTIGUOUS;
711:     if (MemoryFormat.CHANNELS_LAST.jniCode == memoryFormatCode) {
712:       memoryFormat = MemoryFormat.CHANNELS_LAST;
713:     } else if (MemoryFormat.CHANNELS_LAST_3D.jniCode == memoryFormatCode) {
714:       memoryFormat = MemoryFormat.CHANNELS_LAST_3D;
715:     }
716: 
717:     if (DType.FLOAT32.jniCode == dtype) {
718:       tensor = new Tensor_float32(data.asFloatBuffer(), shape, memoryFormat);
719:     } else if (DType.INT32.jniCode == dtype) {
720:       tensor = new Tensor_int32(data.asIntBuffer(), shape, memoryFormat);
````
- EN: Implements callable logic such as `getDataAsDoubleArray`, `toString`, `checkArgument`, `checkShape`.
- CN: 实现可调用逻辑，例如 `getDataAsDoubleArray`, `toString`, `checkArgument`, `checkShape`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 721-735
````java
721:     } else if (DType.INT64.jniCode == dtype) {
722:       tensor = new Tensor_int64(data.asLongBuffer(), shape, memoryFormat);
723:     } else if (DType.FLOAT64.jniCode == dtype) {
724:       tensor = new Tensor_float64(data.asDoubleBuffer(), shape, memoryFormat);
725:     } else if (DType.UINT8.jniCode == dtype) {
726:       tensor = new Tensor_uint8(data, shape, memoryFormat);
727:     } else if (DType.INT8.jniCode == dtype) {
728:       tensor = new Tensor_int8(data, shape, memoryFormat);
729:     } else {
730:       new IllegalArgumentException("Unknown Tensor dtype");
731:     }
732:     tensor.mHybridData = hybridData;
733:     return tensor;
734:   }
735: }
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- Symbol `Tensor` / 符号 `Tensor`
- Symbol `Tensor_uint8` / 符号 `Tensor_uint8`
- Symbol `Tensor_int8` / 符号 `Tensor_int8`
- Symbol `Tensor_int32` / 符号 `Tensor_int32`

## Dependencies / 依赖关系
- Java imports: `com.facebook.jni.HybridData`, `com.facebook.jni.annotations.DoNotStrip`, `java.nio.Buffer`, `java.nio.ByteBuffer`, `java.nio.ByteOrder`, `java.nio.DoubleBuffer`, `java.nio.FloatBuffer`, `java.nio.IntBuffer`, `java.nio.LongBuffer`, `java.util.Arrays`
- Java 导入: `com.facebook.jni.HybridData`, `com.facebook.jni.annotations.DoNotStrip`, `java.nio.Buffer`, `java.nio.ByteBuffer`, `java.nio.ByteOrder`, `java.nio.DoubleBuffer`, `java.nio.FloatBuffer`, `java.nio.IntBuffer`, `java.nio.LongBuffer`, `java.util.Arrays`
