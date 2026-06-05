# PrimitiveArrayTests.java — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/test/PrimitiveArrayTests.java`
- Repository: `pytorch`
- Purpose (EN): Implements JNI bridge abstractions used by Android-facing native code.
- 用途 (CN): 实现供 Android 原生代码使用的 JNI 桥接抽象。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````java
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
17: package com.facebook.jni;
18: 
19: import static org.fest.assertions.api.Assertions.assertThat;
20: import static org.fest.assertions.api.Assertions.offset;
21: import static org.junit.Assume.assumeFalse;
22: import static org.junit.Assume.assumeTrue;
23: 
24: import org.junit.Ignore;
25: import org.junit.Test;
26: 
27: public class PrimitiveArrayTests extends BaseFBJniTests {
28: 
29:   private static final int MAGIC = 42;
30: 
31:   @Test
32:   public void testMakeArrays() {
33:     assertThat(nativeTestMakeBooleanArray(MAGIC).length).isEqualTo(MAGIC);
34:     assertThat(nativeTestMakeByteArray(MAGIC).length).isEqualTo(MAGIC);
35:     assertThat(nativeTestMakeCharArray(MAGIC).length).isEqualTo(MAGIC);
36:     assertThat(nativeTestMakeShortArray(MAGIC).length).isEqualTo(MAGIC);
37:     assertThat(nativeTestMakeIntArray(MAGIC).length).isEqualTo(MAGIC);
38:     assertThat(nativeTestMakeLongArray(MAGIC).length).isEqualTo(MAGIC);
39:     assertThat(nativeTestMakeFloatArray(MAGIC).length).isEqualTo(MAGIC);
40:     assertThat(nativeTestMakeDoubleArray(MAGIC).length).isEqualTo(MAGIC);
41:   }
42: 
43:   private static native boolean[] nativeTestMakeBooleanArray(int size);
44: 
45:   private static native byte[] nativeTestMakeByteArray(int size);
46: 
47:   private static native char[] nativeTestMakeCharArray(int size);
48: 
49:   private static native short[] nativeTestMakeShortArray(int size);
50: 
51:   private static native int[] nativeTestMakeIntArray(int size);
52: 
53:   private static native long[] nativeTestMakeLongArray(int size);
54: 
55:   private static native float[] nativeTestMakeFloatArray(int size);
56: 
57:   private static native double[] nativeTestMakeDoubleArray(int size);
58: 
59:   @Test
60:   public void testGetSetBooleanArray() {
````
- EN: Handles module imports such as `static org.fest.assertions.api.Assertions.assertThat;`, `static org.fest.assertions.api.Assertions.offset;`, `static org.junit.Assume.assumeFalse;`, `static org.junit.Assume.assumeTrue;`.
- CN: 处理模块导入，例如 `static org.fest.assertions.api.Assertions.assertThat;`, `static org.fest.assertions.api.Assertions.offset;`, `static org.junit.Assume.assumeFalse;`, `static org.junit.Assume.assumeTrue;`。
- EN: Imports Java types such as `org.junit.Ignore`, `org.junit.Test`.
- CN: 导入 Java 类型，例如 `org.junit.Ignore`, `org.junit.Test`。
- EN: Declares or extends types including `PrimitiveArrayTests`.
- CN: 声明或扩展类型，包括 `PrimitiveArrayTests`。
- EN: Implements callable logic such as `testMakeArrays`, `nativeTestMakeBooleanArray`, `nativeTestMakeByteArray`, `nativeTestMakeCharArray`.
- CN: 实现可调用逻辑，例如 `testMakeArrays`, `nativeTestMakeBooleanArray`, `nativeTestMakeByteArray`, `nativeTestMakeCharArray`。

### Lines 61-120
````java
 61:     boolean[] array = {false, true};
 62: 
 63:     assertThat(nativeTestGetSetBooleanArray(array)).isTrue();
 64:     assertThat(array).isEqualTo(new boolean[] {true, false});
 65:   }
 66: 
 67:   private static native boolean nativeTestGetSetBooleanArray(boolean[] array);
 68: 
 69:   @Test
 70:   public void testPinBooleanArray() {
 71:     boolean[] array = {false, true};
 72: 
 73:     assertThat(nativeTestPinBooleanArray(array)).isTrue();
 74:     assertThat(array).isEqualTo(new boolean[] {true, false});
 75:   }
 76: 
 77:   private static native boolean nativeTestPinBooleanArray(boolean[] array);
 78: 
 79:   @Test
 80:   public void testGetSetByteArray() {
 81:     byte[] array = new byte[MAGIC];
 82:     for (int i = 0; i < array.length; ++i) {
 83:       array[i] = (byte) i;
 84:     }
 85: 
 86:     assertThat(nativeTestGetSetByteArray(array)).isTrue();
 87: 
 88:     for (int i = 0; i < array.length; ++i) {
 89:       assertThat(array[i]).isEqualTo((byte) (2 * i));
 90:     }
 91:   }
 92: 
 93:   private static native boolean nativeTestGetSetByteArray(byte[] array);
 94: 
 95:   @Test
 96:   public void testGetSetCharArray() {
 97:     char[] array = new char[MAGIC];
 98:     for (int i = 0; i < array.length; ++i) {
 99:       array[i] = (char) i;
100:     }
101: 
102:     assertThat(nativeTestGetSetCharArray(array)).isTrue();
103: 
104:     for (int i = 0; i < array.length; ++i) {
105:       assertThat(array[i]).isEqualTo((char) (2 * i));
106:     }
107:   }
108: 
109:   private static native boolean nativeTestGetSetCharArray(char[] array);
110: 
111:   @Test
112:   public void testGetSetShortArray() {
113:     short[] array = new short[MAGIC];
114:     for (int i = 0; i < array.length; ++i) {
115:       array[i] = (short) i;
116:     }
117: 
118:     assertThat(nativeTestGetSetShortArray(array)).isTrue();
119: 
120:     for (int i = 0; i < array.length; ++i) {
````
- EN: Implements callable logic such as `nativeTestGetSetBooleanArray`, `testPinBooleanArray`, `nativeTestPinBooleanArray`, `testGetSetByteArray`.
- CN: 实现可调用逻辑，例如 `nativeTestGetSetBooleanArray`, `testPinBooleanArray`, `nativeTestPinBooleanArray`, `testGetSetByteArray`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-180
````java
121:       assertThat(array[i]).isEqualTo((short) (2 * i));
122:     }
123:   }
124: 
125:   private static native boolean nativeTestGetSetShortArray(short[] array);
126: 
127:   @Test
128:   public void testGetSetIntArray() {
129:     int[] intArray = new int[MAGIC];
130:     for (int i = 0; i < intArray.length; ++i) {
131:       intArray[i] = i;
132:     }
133: 
134:     assertThat(nativeTestGetSetIntArray(intArray)).isTrue();
135: 
136:     for (int i = 0; i < intArray.length; ++i) {
137:       assertThat(intArray[i]).isEqualTo(2 * i);
138:     }
139:   }
140: 
141:   private static native boolean nativeTestGetSetIntArray(int[] array);
142: 
143:   @Test
144:   public void testGetSetLongArray() {
145:     long[] longArray = new long[MAGIC];
146:     for (int i = 0; i < longArray.length; ++i) {
147:       longArray[i] = (long) i;
148:     }
149: 
150:     assertThat(nativeTestGetSetLongArray(longArray)).isTrue();
151: 
152:     for (int i = 0; i < longArray.length; ++i) {
153:       assertThat(longArray[i]).isEqualTo(2 * i);
154:     }
155:   }
156: 
157:   private static native boolean nativeTestGetSetLongArray(long[] array);
158: 
159:   @Test
160:   public void testGetSetFloatArray() {
161:     float[] floatArray = new float[MAGIC];
162:     for (int i = 0; i < floatArray.length; ++i) {
163:       floatArray[i] = i;
164:     }
165: 
166:     assertThat(nativeTestGetSetFloatArray(floatArray)).isTrue();
167: 
168:     for (int i = 0; i < floatArray.length; ++i) {
169:       assertThat(floatArray[i]).isEqualTo(2 * i, offset(1e-3f));
170:     }
171:   }
172: 
173:   private static native boolean nativeTestGetSetFloatArray(float[] array);
174: 
175:   @Test
176:   public void testGetSetDoubleArray() {
177:     double[] doubleArray = new double[MAGIC];
178:     for (int i = 0; i < doubleArray.length; ++i) {
179:       doubleArray[i] = i;
180:     }
````
- EN: Implements callable logic such as `nativeTestGetSetShortArray`, `testGetSetIntArray`, `nativeTestGetSetIntArray`, `testGetSetLongArray`.
- CN: 实现可调用逻辑，例如 `nativeTestGetSetShortArray`, `testGetSetIntArray`, `nativeTestGetSetIntArray`, `testGetSetLongArray`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 181-240
````java
181: 
182:     assertThat(nativeTestGetSetDoubleArray(doubleArray)).isTrue();
183: 
184:     for (int i = 0; i < doubleArray.length; ++i) {
185:       assertThat(doubleArray[i]).isEqualTo(2 * i, offset(1e-3));
186:     }
187:   }
188: 
189:   private static native boolean nativeTestGetSetDoubleArray(double[] array);
190: 
191:   @Test
192:   public void testPinByteArray() {
193:     byte[] array = new byte[MAGIC];
194:     for (int i = 0; i < array.length; ++i) {
195:       array[i] = (byte) i;
196:     }
197: 
198:     assertThat(nativeTestPinByteArray(array)).isTrue();
199: 
200:     for (int i = 0; i < array.length; ++i) {
201:       assertThat(array[i]).isEqualTo((byte) (2 * i));
202:     }
203:   }
204: 
205:   private static native boolean nativeTestPinByteArray(byte[] array);
206: 
207:   @Test
208:   public void testPinCharArray() {
209:     char[] array = new char[MAGIC];
210:     for (int i = 0; i < array.length; ++i) {
211:       array[i] = (char) i;
212:     }
213: 
214:     assertThat(nativeTestPinCharArray(array)).isTrue();
215: 
216:     for (int i = 0; i < array.length; ++i) {
217:       assertThat(array[i]).isEqualTo((char) (2 * i));
218:     }
219:   }
220: 
221:   private static native boolean nativeTestPinCharArray(char[] array);
222: 
223:   @Test
224:   public void testPinShortArray() {
225:     short[] array = new short[MAGIC];
226:     for (int i = 0; i < array.length; ++i) {
227:       array[i] = (short) i;
228:     }
229: 
230:     assertThat(nativeTestPinShortArray(array)).isTrue();
231: 
232:     for (int i = 0; i < array.length; ++i) {
233:       assertThat(array[i]).isEqualTo((short) (2 * i));
234:     }
235:   }
236: 
237:   private static native boolean nativeTestPinShortArray(short[] array);
238: 
239:   @Test
240:   public void testPinIntArray() {
````
- EN: Implements callable logic such as `nativeTestGetSetDoubleArray`, `testPinByteArray`, `nativeTestPinByteArray`, `testPinCharArray`.
- CN: 实现可调用逻辑，例如 `nativeTestGetSetDoubleArray`, `testPinByteArray`, `nativeTestPinByteArray`, `testPinCharArray`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 241-300
````java
241:     int[] intArray = new int[MAGIC];
242:     for (int i = 0; i < intArray.length; ++i) {
243:       intArray[i] = i;
244:     }
245: 
246:     assertThat(nativeTestPinIntArray(intArray)).isTrue();
247: 
248:     for (int i = 0; i < intArray.length; ++i) {
249:       assertThat(intArray[i]).isEqualTo(2 * i);
250:     }
251:   }
252: 
253:   private static native boolean nativeTestPinIntArray(int[] array);
254: 
255:   @Test
256:   public void testPinLongArray() {
257:     long[] longArray = new long[MAGIC];
258:     for (int i = 0; i < longArray.length; ++i) {
259:       longArray[i] = (long) i;
260:     }
261: 
262:     assertThat(nativeTestPinLongArray(longArray)).isTrue();
263: 
264:     for (int i = 0; i < longArray.length; ++i) {
265:       assertThat(longArray[i]).isEqualTo(2 * i);
266:     }
267:   }
268: 
269:   private static native boolean nativeTestPinLongArray(long[] array);
270: 
271:   @Test
272:   public void testPinFloatArray() {
273:     float[] floatArray = new float[MAGIC];
274:     for (int i = 0; i < floatArray.length; ++i) {
275:       floatArray[i] = (long) i;
276:     }
277: 
278:     assertThat(nativeTestPinFloatArray(floatArray)).isTrue();
279: 
280:     for (int i = 0; i < floatArray.length; ++i) {
281:       assertThat(floatArray[i]).isEqualTo(2 * i, offset(1e-3f));
282:     }
283:   }
284: 
285:   private static native boolean nativeTestPinFloatArray(float[] array);
286: 
287:   @Test
288:   public void testPinDoubleArray() {
289:     double[] doubleArray = new double[MAGIC];
290:     for (int i = 0; i < doubleArray.length; ++i) {
291:       doubleArray[i] = (double) i;
292:     }
293: 
294:     assertThat(nativeTestPinDoubleArray(doubleArray)).isTrue();
295: 
296:     for (int i = 0; i < doubleArray.length; ++i) {
297:       assertThat(doubleArray[i]).isEqualTo(2 * i, offset(1e-3));
298:     }
299:   }
300: 
````
- EN: Implements callable logic such as `nativeTestPinIntArray`, `testPinLongArray`, `nativeTestPinLongArray`, `testPinFloatArray`.
- CN: 实现可调用逻辑，例如 `nativeTestPinIntArray`, `testPinLongArray`, `nativeTestPinLongArray`, `testPinFloatArray`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 301-360
````java
301:   private static native boolean nativeTestPinDoubleArray(double[] array);
302: 
303:   @Test
304:   public void testPinByteArrayRegion() {
305:     byte[] array = new byte[MAGIC];
306:     for (int i = 0; i < array.length; ++i) {
307:       array[i] = (byte) i;
308:     }
309: 
310:     assertThat(nativeTestPinByteArrayRegion(array)).isTrue();
311: 
312:     for (int i = 0; i < array.length; ++i) {
313:       assertThat(array[i]).isEqualTo((byte) (2 * i));
314:     }
315:   }
316: 
317:   private static native boolean nativeTestPinByteArrayRegion(byte[] array);
318: 
319:   @Test
320:   public void testPinCharArrayRegion() {
321:     char[] array = new char[MAGIC];
322:     for (int i = 0; i < array.length; ++i) {
323:       array[i] = (char) i;
324:     }
325: 
326:     assertThat(nativeTestPinCharArrayRegion(array)).isTrue();
327: 
328:     for (int i = 0; i < array.length; ++i) {
329:       assertThat(array[i]).isEqualTo((char) (2 * i));
330:     }
331:   }
332: 
333:   private static native boolean nativeTestPinCharArrayRegion(char[] array);
334: 
335:   @Test
336:   public void testPinShortArrayRegion() {
337:     short[] array = new short[MAGIC];
338:     for (int i = 0; i < array.length; ++i) {
339:       array[i] = (short) i;
340:     }
341: 
342:     assertThat(nativeTestPinShortArrayRegion(array)).isTrue();
343: 
344:     for (int i = 0; i < array.length; ++i) {
345:       assertThat(array[i]).isEqualTo((short) (2 * i));
346:     }
347:   }
348: 
349:   private static native boolean nativeTestPinShortArrayRegion(short[] array);
350: 
351:   @Test
352:   public void testPinIntArrayRegion() {
353:     int[] intArray = new int[MAGIC];
354:     for (int i = 0; i < intArray.length; ++i) {
355:       intArray[i] = i;
356:     }
357: 
358:     assertThat(nativeTestPinIntArrayRegion(intArray)).isTrue();
359: 
360:     for (int i = 0; i < intArray.length; ++i) {
````
- EN: Implements callable logic such as `nativeTestPinDoubleArray`, `testPinByteArrayRegion`, `nativeTestPinByteArrayRegion`, `testPinCharArrayRegion`.
- CN: 实现可调用逻辑，例如 `nativeTestPinDoubleArray`, `testPinByteArrayRegion`, `nativeTestPinByteArrayRegion`, `testPinCharArrayRegion`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 361-420
````java
361:       assertThat(intArray[i]).isEqualTo(2 * i);
362:     }
363:   }
364: 
365:   private static native boolean nativeTestPinIntArrayRegion(int[] array);
366: 
367:   @Test
368:   public void testPinLongArrayRegion() {
369:     long[] longArray = new long[MAGIC];
370:     for (int i = 0; i < longArray.length; ++i) {
371:       longArray[i] = (long) i;
372:     }
373: 
374:     assertThat(nativeTestPinLongArrayRegion(longArray)).isTrue();
375: 
376:     for (int i = 0; i < longArray.length; ++i) {
377:       assertThat(longArray[i]).isEqualTo(2 * i);
378:     }
379:   }
380: 
381:   private static native boolean nativeTestPinLongArrayRegion(long[] array);
382: 
383:   @Test
384:   public void testPinFloatArrayRegion() {
385:     float[] floatArray = new float[MAGIC];
386:     for (int i = 0; i < floatArray.length; ++i) {
387:       floatArray[i] = (long) i;
388:     }
389: 
390:     assertThat(nativeTestPinFloatArrayRegion(floatArray)).isTrue();
391: 
392:     for (int i = 0; i < floatArray.length; ++i) {
393:       assertThat(floatArray[i]).isEqualTo(2 * i, offset(1e-3f));
394:     }
395:   }
396: 
397:   private static native boolean nativeTestPinFloatArrayRegion(float[] array);
398: 
399:   @Test
400:   public void testPinDoubleArrayRegion() {
401:     double[] doubleArray = new double[MAGIC];
402:     for (int i = 0; i < doubleArray.length; ++i) {
403:       doubleArray[i] = (double) i;
404:     }
405: 
406:     assertThat(nativeTestPinDoubleArrayRegion(doubleArray)).isTrue();
407: 
408:     for (int i = 0; i < doubleArray.length; ++i) {
409:       assertThat(doubleArray[i]).isEqualTo(2 * i, offset(1e-3));
410:     }
411:   }
412: 
413:   private static native boolean nativeTestPinDoubleArrayRegion(double[] array);
414: 
415:   @Test
416:   public void testPinByteArrayCritical() {
417:     byte[] array = new byte[MAGIC];
418:     for (int i = 0; i < array.length; ++i) {
419:       array[i] = (byte) i;
420:     }
````
- EN: Implements callable logic such as `nativeTestPinIntArrayRegion`, `testPinLongArrayRegion`, `nativeTestPinLongArrayRegion`, `testPinFloatArrayRegion`.
- CN: 实现可调用逻辑，例如 `nativeTestPinIntArrayRegion`, `testPinLongArrayRegion`, `nativeTestPinLongArrayRegion`, `testPinFloatArrayRegion`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 421-480
````java
421: 
422:     assertThat(nativeTestPinByteArrayCritical(array)).isTrue();
423: 
424:     for (int i = 0; i < array.length; ++i) {
425:       assertThat(array[i]).isEqualTo((byte) (2 * i));
426:     }
427:   }
428: 
429:   private static native boolean nativeTestPinByteArrayCritical(byte[] array);
430: 
431:   @Test
432:   public void testPinCharArrayCritical() {
433:     char[] array = new char[MAGIC];
434:     for (int i = 0; i < array.length; ++i) {
435:       array[i] = (char) i;
436:     }
437: 
438:     assertThat(nativeTestPinCharArrayCritical(array)).isTrue();
439: 
440:     for (int i = 0; i < array.length; ++i) {
441:       assertThat(array[i]).isEqualTo((char) (2 * i));
442:     }
443:   }
444: 
445:   private static native boolean nativeTestPinCharArrayCritical(char[] array);
446: 
447:   @Test
448:   public void testPinShortArrayCritical() {
449:     short[] array = new short[MAGIC];
450:     for (int i = 0; i < array.length; ++i) {
451:       array[i] = (short) i;
452:     }
453: 
454:     assertThat(nativeTestPinShortArrayCritical(array)).isTrue();
455: 
456:     for (int i = 0; i < array.length; ++i) {
457:       assertThat(array[i]).isEqualTo((short) (2 * i));
458:     }
459:   }
460: 
461:   private static native boolean nativeTestPinShortArrayCritical(short[] array);
462: 
463:   @Test
464:   public void testPinIntArrayCritical() {
465:     int[] intArray = new int[MAGIC];
466:     for (int i = 0; i < intArray.length; ++i) {
467:       intArray[i] = i;
468:     }
469: 
470:     assertThat(nativeTestPinIntArrayCritical(intArray)).isTrue();
471: 
472:     for (int i = 0; i < intArray.length; ++i) {
473:       assertThat(intArray[i]).isEqualTo(2 * i);
474:     }
475:   }
476: 
477:   private static native boolean nativeTestPinIntArrayCritical(int[] array);
478: 
479:   @Test
480:   public void testPinLongArrayCritical() {
````
- EN: Implements callable logic such as `nativeTestPinByteArrayCritical`, `testPinCharArrayCritical`, `nativeTestPinCharArrayCritical`, `testPinShortArrayCritical`.
- CN: 实现可调用逻辑，例如 `nativeTestPinByteArrayCritical`, `testPinCharArrayCritical`, `nativeTestPinCharArrayCritical`, `testPinShortArrayCritical`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 481-540
````java
481:     long[] longArray = new long[MAGIC];
482:     for (int i = 0; i < longArray.length; ++i) {
483:       longArray[i] = (long) i;
484:     }
485: 
486:     assertThat(nativeTestPinLongArrayCritical(longArray)).isTrue();
487: 
488:     for (int i = 0; i < longArray.length; ++i) {
489:       assertThat(longArray[i]).isEqualTo(2 * i);
490:     }
491:   }
492: 
493:   private static native boolean nativeTestPinLongArrayCritical(long[] array);
494: 
495:   @Test
496:   public void testPinFloatArrayCritical() {
497:     float[] floatArray = new float[MAGIC];
498:     for (int i = 0; i < floatArray.length; ++i) {
499:       floatArray[i] = (long) i;
500:     }
501: 
502:     assertThat(nativeTestPinFloatArrayCritical(floatArray)).isTrue();
503: 
504:     for (int i = 0; i < floatArray.length; ++i) {
505:       assertThat(floatArray[i]).isEqualTo(2 * i, offset(1e-3f));
506:     }
507:   }
508: 
509:   private static native boolean nativeTestPinFloatArrayCritical(float[] array);
510: 
511:   @Test
512:   public void testPinDoubleArrayCritical() {
513:     double[] doubleArray = new double[MAGIC];
514:     for (int i = 0; i < doubleArray.length; ++i) {
515:       doubleArray[i] = (double) i;
516:     }
517: 
518:     assertThat(nativeTestPinDoubleArrayCritical(doubleArray)).isTrue();
519: 
520:     for (int i = 0; i < doubleArray.length; ++i) {
521:       assertThat(doubleArray[i]).isEqualTo(2 * i, offset(1e-3));
522:     }
523:   }
524: 
525:   private static native boolean nativeTestPinDoubleArrayCritical(double[] array);
526: 
527:   @Test
528:   public void testIndexOutOfBoundsInRegions() {
529:     assertThat(nativeTestIndexOutOfBoundsInRegions()).isTrue();
530:   }
531: 
532:   private static native boolean nativeTestIndexOutOfBoundsInRegions();
533: 
534:   @Test
535:   public void testBooleanArrayIndexing() {
536:     boolean[] array = {true, true, false, true, false};
537:     for (int ii = 0; ii < 5; ii++) {
538:       assertThat(nativeTestBooleanArrayIndexing(array, ii)).isEqualTo(array[ii]);
539:     }
540:   }
````
- EN: Implements callable logic such as `nativeTestPinLongArrayCritical`, `testPinFloatArrayCritical`, `nativeTestPinFloatArrayCritical`, `testPinDoubleArrayCritical`.
- CN: 实现可调用逻辑，例如 `nativeTestPinLongArrayCritical`, `testPinFloatArrayCritical`, `nativeTestPinFloatArrayCritical`, `testPinDoubleArrayCritical`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 541-600
````java
541: 
542:   private native boolean nativeTestBooleanArrayIndexing(boolean[] array, int idx);
543: 
544:   @Test
545:   public void testIntegerArrayIndexing() {
546:     int[] array = {0, 1, 2, 3, 4};
547:     for (int ii = 0; ii < 5; ii++) {
548:       assertThat(nativeTestIntegerArrayIndexing(array, ii)).isEqualTo(array[ii]);
549:     }
550:   }
551: 
552:   private native int nativeTestIntegerArrayIndexing(int[] array, int idx);
553: 
554:   @Test
555:   public void testIntegerArraySize() {
556:     int[] array = {0, 1, 2, 3, 4};
557:     assertThat(nativeTestIntegerArraySize(array)).isEqualTo(array.length);
558:   }
559: 
560:   private native int nativeTestIntegerArraySize(int[] array);
561: 
562:   @Test
563:   public void testIntegerArrayIncrement() {
564:     int[] array = {0, 1, 2, 3, 4};
565:     array = nativeTestIntegerArrayIncrement(array);
566:     for (int ii = 0; ii < 5; ii++) {
567:       assertThat(array[ii]).isEqualTo(ii + 1);
568:     }
569:   }
570: 
571:   private native int[] nativeTestIntegerArrayIncrement(int[] array);
572: 
573:   @Test
574:   public void testIntegerArrayMoveAssignment() {
575:     int[] array = {0, 1, 2, 3, 4};
576:     nativeTestIntegerArrayMoveAssignment(array);
577:     assertThat(array[0]).isEqualTo(0);
578:   }
579: 
580:   private native void nativeTestIntegerArrayMoveAssignment(int[] array);
581: 
582:   // On ART, a large array will be placed in the large heap. Arrays here are
583:   // non-movable and so the vm pins them in-place. A small array will be a
584:   // movable object and thus the pinned array will be a copy.
585:   // On Dalvik, all pinned arrays are in-place.
586: 
587:   @Test
588:   @Ignore("Flakey Test. See t8845133")
589:   public void testCopiedPinnedArray() {
590:     int[] array = new int[100];
591:     assumeTrue(nativeIsPinnedArrayACopy(array));
592:     assertThat(nativeTestCopiedPinnedArray(array)).isTrue();
593:   }
594: 
595:   @Test
596:   public void testNonCopiedPinnedArray() {
597:     int[] array = new int[1000000];
598:     assumeFalse(nativeIsPinnedArrayACopy(array));
599:     assertThat(nativeTestNonCopiedPinnedArray(array)).isTrue();
600:   }
````
- EN: Implements callable logic such as `nativeTestBooleanArrayIndexing`, `testIntegerArrayIndexing`, `nativeTestIntegerArrayIndexing`, `testIntegerArraySize`.
- CN: 实现可调用逻辑，例如 `nativeTestBooleanArrayIndexing`, `testIntegerArrayIndexing`, `nativeTestIntegerArrayIndexing`, `testIntegerArraySize`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 601-607
````java
601: 
602:   private native boolean nativeIsPinnedArrayACopy(int[] array);
603: 
604:   private native boolean nativeTestCopiedPinnedArray(int[] array);
605: 
606:   private native boolean nativeTestNonCopiedPinnedArray(int[] array);
607: }
````
- EN: Implements callable logic such as `nativeIsPinnedArrayACopy`, `nativeTestCopiedPinnedArray`, `nativeTestNonCopiedPinnedArray`.
- CN: 实现可调用逻辑，例如 `nativeIsPinnedArrayACopy`, `nativeTestCopiedPinnedArray`, `nativeTestNonCopiedPinnedArray`。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Validation and test coverage / 校验与测试覆盖
- Symbol `PrimitiveArrayTests` / 符号 `PrimitiveArrayTests`
- Symbol `testMakeArrays` / 符号 `testMakeArrays`
- Symbol `nativeTestMakeBooleanArray` / 符号 `nativeTestMakeBooleanArray`
- Symbol `nativeTestMakeByteArray` / 符号 `nativeTestMakeByteArray`

## Dependencies / 依赖关系
- Java imports: `org.junit.Ignore`, `org.junit.Test`
- Java 导入: `org.junit.Ignore`, `org.junit.Test`
