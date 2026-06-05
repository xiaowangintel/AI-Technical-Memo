# FBJniTests.java — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/test/FBJniTests.java`
- Repository: `pytorch`
- Purpose (EN): Implements JNI bridge abstractions used by Android-facing native code.
- 用途 (CN): 实现供 Android 原生代码使用的 JNI 桥接抽象。

## Line-by-Line Analysis / 逐行分析
### Lines 1-80
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
20: import static org.mockito.Mockito.verify;
21: 
22: import com.facebook.jni.annotations.DoNotStrip;
23: import java.io.IOException;
24: import java.util.ArrayList;
25: import java.util.concurrent.Callable;
26: import org.fest.assertions.api.Fail;
27: import org.junit.Test;
28: import org.junit.runner.RunWith;
29: import org.mockito.Mock;
30: import org.mockito.runners.MockitoJUnitRunner;
31: 
32: @RunWith(MockitoJUnitRunner.class)
33: public class FBJniTests extends BaseFBJniTests {
34:   class CustomException extends Throwable {
35:     int mGetMessageCalls = 0;
36: 
37:     @Override
38:     public String getMessage() {
39:       return "getMessages: " + (++mGetMessageCalls);
40:     }
41:   }
42: 
43:   public interface Callbacks {
44:     void voidFoo();
45: 
46:     boolean booleanFoo();
47: 
48:     byte byteFoo();
49: 
50:     char charFoo();
51: 
52:     short shortFoo();
53: 
54:     int intFoo();
55: 
56:     long longFoo();
57: 
58:     float floatFoo();
59: 
60:     double doubleFoo();
61: 
62:     Object objectFoo();
63: 
64:     String stringFoo();
65:   }
66: 
67:   public static class TestThing {
68:     int foo;
69:   }
70: 
71:   @Mock private static Callbacks mCallbacksMock;
72: 
73:   private int mIntFieldTest;
74:   private String mStringFieldTest;
75:   private TestThing mReferenceFieldTest;
76:   private static int sIntFieldTest;
77:   private static String sStringFieldTest;
78:   private static TestThing sReferenceFieldTest;
79: 
80:   @DoNotStrip // Resolved from fbjni_tests::TestFieldAccess
````
- EN: Handles module imports such as `static org.fest.assertions.api.Assertions.assertThat;`, `static org.mockito.Mockito.verify;`, `com.facebook.jni.annotations.DoNotStrip;`, `java.io.IOException;`.
- CN: 处理模块导入，例如 `static org.fest.assertions.api.Assertions.assertThat;`, `static org.mockito.Mockito.verify;`, `com.facebook.jni.annotations.DoNotStrip;`, `java.io.IOException;`。
- EN: Imports Java types such as `com.facebook.jni.annotations.DoNotStrip`, `java.io.IOException`, `java.util.ArrayList`, `java.util.concurrent.Callable`.
- CN: 导入 Java 类型，例如 `com.facebook.jni.annotations.DoNotStrip`, `java.io.IOException`, `java.util.ArrayList`, `java.util.concurrent.Callable`。
- EN: Declares or extends types including `FBJniTests`, `CustomException`, `Callbacks`, `TestThing`.
- CN: 声明或扩展类型，包括 `FBJniTests`, `CustomException`, `Callbacks`, `TestThing`。
- EN: Implements callable logic such as `getMessage`.
- CN: 实现可调用逻辑，例如 `getMessage`。

### Lines 81-160
````java
 81:   int bar(double d) {
 82:     return 42;
 83:   }
 84: 
 85:   // Test case for nonvirtual function
 86:   public boolean nonVirtualMethod(boolean s) {
 87:     return s;
 88:   }
 89: 
 90:   private static void verifyAllCallbacksCalled(Callbacks mock) {
 91:     verify(mock).voidFoo();
 92:     verify(mock).booleanFoo();
 93:     verify(mock).byteFoo();
 94:     verify(mock).charFoo();
 95:     verify(mock).shortFoo();
 96:     verify(mock).intFoo();
 97:     verify(mock).longFoo();
 98:     verify(mock).floatFoo();
 99:     verify(mock).doubleFoo();
100:     verify(mock).objectFoo();
101:     verify(mock).stringFoo();
102:   }
103: 
104:   // Instead of mocking, lets call non-static functions and verify them.
105:   public static void voidFooStatic() {
106:     mCallbacksMock.voidFoo();
107:   }
108: 
109:   public static boolean booleanFooStatic() {
110:     return mCallbacksMock.booleanFoo();
111:   }
112: 
113:   public static byte byteFooStatic() {
114:     return mCallbacksMock.byteFoo();
115:   }
116: 
117:   public static char charFooStatic(char c, int s) {
118:     return mCallbacksMock.charFoo();
119:   }
120: 
121:   public static short shortFooStatic(short s, short t) {
122:     return mCallbacksMock.shortFoo();
123:   }
124: 
125:   public static int intFooStatic(int s) {
126:     return mCallbacksMock.intFoo();
127:   }
128: 
129:   public static long longFooStatic() {
130:     return mCallbacksMock.longFoo();
131:   }
132: 
133:   public static float floatFooStatic() {
134:     return mCallbacksMock.floatFoo();
135:   }
136: 
137:   public static double doubleFooStatic() {
138:     return mCallbacksMock.doubleFoo();
139:   }
140: 
141:   public static Object objectFooStatic() {
142:     return mCallbacksMock.objectFoo();
143:   }
144: 
145:   public static String stringFooStatic() {
146:     return mCallbacksMock.stringFoo();
147:   }
148: 
149:   @Test
150:   public void resolveClass() throws ClassNotFoundException {
151:     assertThat(nativeTestClassResolution("java/lang/Object")).isTrue();
152:   }
153: 
154:   // Some versions of Android throw ClassNotFoundException while others throw NoClassDefFoundError.
155:   // Flatten that to always be ClassNotFoundException.
156:   private static void wrapClassLoadingErrors(Callable<?> code) throws Exception {
157:     try {
158:       code.call();
159:     } catch (NoClassDefFoundError ex) {
160:       throw new ClassNotFoundException("chained NoClassDefFoundError", ex);
````
- EN: Implements callable logic such as `nonVirtualMethod`, `verifyAllCallbacksCalled`, `voidFooStatic`, `booleanFooStatic`.
- CN: 实现可调用逻辑，例如 `nonVirtualMethod`, `verifyAllCallbacksCalled`, `voidFooStatic`, `booleanFooStatic`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 161-240
````java
161:     }
162:   }
163: 
164:   @Test(expected = ClassNotFoundException.class)
165:   public void failingToResolveClass() throws Exception {
166:     wrapClassLoadingErrors(
167:         new Callable<Boolean>() {
168:           @Override
169:           public Boolean call() throws Exception {
170:             return nativeTestClassResolution("ThisClassDoesNotExist");
171:           }
172:         });
173:   }
174: 
175:   private native boolean nativeTestClassResolution(String className) throws ClassNotFoundException;
176: 
177:   @Test
178:   public void lazyClassResolution() throws ClassNotFoundException {
179:     assertThat(nativeTestLazyClassResolution("java/lang/Object")).isTrue();
180:   }
181: 
182:   @Test(expected = ClassNotFoundException.class)
183:   public void failedLazyClassResolution() throws Exception {
184:     wrapClassLoadingErrors(
185:         new Callable<Boolean>() {
186:           @Override
187:           public Boolean call() throws Exception {
188:             return nativeTestLazyClassResolution("ThisClassDoesNotExist");
189:           }
190:         });
191:   }
192: 
193:   private native boolean nativeTestLazyClassResolution(String className)
194:       throws ClassNotFoundException;
195: 
196:   @Test
197:   public void instanceCreation() {
198:     assertThat(nativeCreateInstanceOf("java/lang/String"))
199:         .isInstanceOf(String.class)
200:         .isEqualTo("java/lang/String");
201:   }
202: 
203:   private native Object nativeCreateInstanceOf(String className);
204: 
205:   @Test
206:   public void typeDescriptors() {
207:     assertThat(nativeTestTypeDescriptors()).isTrue();
208:   }
209: 
210:   private native boolean nativeTestTypeDescriptors();
211: 
212:   @Test
213:   public void resolveVirtualMethod() throws ClassNotFoundException, NoSuchMethodException {
214:     assertThat(nativeTestVirtualMethodResolution_I("java/lang/Object", "hashCode")).isTrue();
215:   }
216: 
217:   @Test
218:   public void resolveVirtualMethodWithArray() throws ClassNotFoundException, NoSuchMethodException {
219:     assertThat(nativeTestVirtualMethodResolution_arrB("java/lang/String", "getBytes")).isTrue();
220:   }
221: 
222:   @Test
223:   public void resolveVirtualMethodWithObjectArray()
224:       throws ClassNotFoundException, NoSuchMethodException {
225:     assertThat(nativeTestVirtualMethodResolution_S_arrS("java/lang/String", "split")).isTrue();
226:   }
227: 
228:   @Test
229:   public void resolveVirtualMethodWithObjectArrayArray()
230:       throws ClassNotFoundException, NoSuchMethodException {
231:     assertThat(
232:             nativeTestVirtualMethodResolution_arrarrS(
233:                 "com/facebook/jni/FBJniTests", "returnMultidimensionalObjectArray"))
234:         .isTrue();
235:   }
236: 
237:   public static String[][] returnMultidimensionalObjectArray() {
238:     return null;
239:   }
240: 
````
- EN: Implements callable logic such as `failingToResolveClass`, `call`, `nativeTestClassResolution`, `lazyClassResolution`.
- CN: 实现可调用逻辑，例如 `failingToResolveClass`, `call`, `nativeTestClassResolution`, `lazyClassResolution`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-320
````java
241:   @Test
242:   public void resolveVirtualMethodWithPrimitiveArrayArray()
243:       throws ClassNotFoundException, NoSuchMethodException {
244:     assertThat(
245:             nativeTestVirtualMethodResolution_arrarrI(
246:                 "com/facebook/jni/FBJniTests", "returnMultidimensionalPrimitiveArray"))
247:         .isTrue();
248:   }
249: 
250:   public static int[][] returnMultidimensionalPrimitiveArray() {
251:     return null;
252:   }
253: 
254:   @Test(expected = NoSuchMethodError.class)
255:   public void failingToResolveVirtualMethod() throws ClassNotFoundException, NoSuchMethodError {
256:     nativeTestVirtualMethodResolution_I("java/lang/Object", "ThisMethodDoesNotExist");
257:   }
258: 
259:   private native boolean nativeTestVirtualMethodResolution_I(String className, String methodName)
260:       throws ClassNotFoundException, NoSuchMethodError;
261: 
262:   private native boolean nativeTestVirtualMethodResolution_arrB(String className, String methodName)
263:       throws ClassNotFoundException, NoSuchMethodError;
264: 
265:   private native boolean nativeTestVirtualMethodResolution_S_arrS(
266:       String className, String methodName) throws ClassNotFoundException, NoSuchMethodError;
267: 
268:   private native boolean nativeTestVirtualMethodResolution_arrarrS(
269:       String className, String methodName) throws ClassNotFoundException, NoSuchMethodError;
270: 
271:   private native boolean nativeTestVirtualMethodResolution_arrarrI(
272:       String className, String methodName) throws ClassNotFoundException, NoSuchMethodError;
273: 
274:   @Test
275:   public void lazyMethodResolution() throws ClassNotFoundException, NoSuchMethodError {
276:     assertThat(nativeTestLazyVirtualMethodResolution_I("java/lang/Object", "hashCode")).isTrue();
277:   }
278: 
279:   @Test(expected = NoSuchMethodError.class)
280:   public void failedLazyMethodResolution() throws ClassNotFoundException, NoSuchMethodError {
281:     nativeTestLazyVirtualMethodResolution_I("java/lang/Object", "ThisMethodDoesNotExist");
282:   }
283: 
284:   private native boolean nativeTestLazyVirtualMethodResolution_I(
285:       String className, String methodName);
286: 
287:   @Test
288:   public void callbacksUsingJMethod() {
289:     nativeTestJMethodCallbacks(mCallbacksMock);
290:     verifyAllCallbacksCalled(mCallbacksMock);
291:   }
292: 
293:   private native void nativeTestJMethodCallbacks(Callbacks callbacks);
294: 
295:   @Test
296:   public void callbacksUsingJStaticMethod() {
297:     nativeTestJStaticMethodCallbacks();
298:     verifyAllCallbacksCalled(mCallbacksMock);
299:   }
300: 
301:   private native void nativeTestJStaticMethodCallbacks();
302: 
303:   @Test
304:   public void isAssignableFrom() {
305:     assertThat(nativeTestIsAssignableFrom(String.class, String.class)).isTrue();
306:     assertThat(nativeTestIsAssignableFrom(String.class, Object.class)).isFalse();
307:     assertThat(nativeTestIsAssignableFrom(Object.class, String.class)).isTrue();
308:     assertThat(nativeTestIsAssignableFrom(ArrayList.class, Iterable.class)).isFalse();
309:     assertThat(nativeTestIsAssignableFrom(Iterable.class, ArrayList.class)).isTrue();
310:   }
311: 
312:   private native boolean nativeTestIsAssignableFrom(Class cls1, Class cls2);
313: 
314:   @Test
315:   public void isInstanceOf() {
316:     assertThat(nativeTestIsInstanceOf("", String.class)).isTrue();
317:     assertThat(nativeTestIsInstanceOf("", Object.class)).isTrue();
318:     assertThat(nativeTestIsInstanceOf(new Object(), String.class)).isFalse();
319:     assertThat(nativeTestIsInstanceOf(new ArrayList(), Iterable.class)).isTrue();
320:     assertThat(nativeTestIsInstanceOf(null, Iterable.class)).isTrue();
````
- EN: Implements callable logic such as `resolveVirtualMethodWithPrimitiveArrayArray`, `returnMultidimensionalPrimitiveArray`, `failingToResolveVirtualMethod`, `nativeTestVirtualMethodResolution_I`.
- CN: 实现可调用逻辑，例如 `resolveVirtualMethodWithPrimitiveArrayArray`, `returnMultidimensionalPrimitiveArray`, `failingToResolveVirtualMethod`, `nativeTestVirtualMethodResolution_I`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 321-400
````java
321:   }
322: 
323:   private native boolean nativeTestIsInstanceOf(Object object, Class cls);
324: 
325:   @Test
326:   public void isSameObject() {
327:     Object anObject = new Object();
328:     Object anotherObject = new Object();
329:     assertThat(nativeTestIsSameObject(anObject, anObject)).isTrue();
330:     assertThat(nativeTestIsSameObject(anObject, anotherObject)).isFalse();
331:     assertThat(nativeTestIsSameObject(null, anObject)).isFalse();
332:     assertThat(nativeTestIsSameObject(anObject, null)).isFalse();
333:     assertThat(nativeTestIsSameObject(null, null)).isTrue();
334:   }
335: 
336:   private native boolean nativeTestIsSameObject(Object a, Object b);
337: 
338:   @Test
339:   public void testGetSuperClass() {
340:     Class testClass = String.class;
341:     Class superClass = Object.class;
342:     Class notSuperClass = Integer.class;
343: 
344:     assertThat(nativeTestGetSuperclass(testClass, superClass)).isTrue();
345:     assertThat(nativeTestGetSuperclass(testClass, notSuperClass)).isFalse();
346:   }
347: 
348:   private native boolean nativeTestGetSuperclass(Class testClass, Class superOfTest);
349: 
350:   @Test
351:   public void testWeakRefs() {
352:     assertThat(nativeTestWeakRefs()).isTrue();
353:   }
354: 
355:   private native boolean nativeTestWeakRefs();
356: 
357:   @Test
358:   public void testAliasRefs() {
359:     assertThat(nativeTestAlias()).isTrue();
360:   }
361: 
362:   private native boolean nativeTestAlias();
363: 
364:   @Test
365:   public void testAliasRefConversions() {
366:     assertThat(nativeTestAliasRefConversions()).isTrue();
367:   }
368: 
369:   private native boolean nativeTestAliasRefConversions();
370: 
371:   @Test
372:   public void testNullJString() {
373:     assertThat(nativeTestNullJString()).isTrue();
374:   }
375: 
376:   private native boolean nativeTestNullJString();
377: 
378:   @Test
379:   public void testSwap() {
380:     assertThat(nativeTestSwap(new Object())).isTrue();
381:   }
382: 
383:   private native boolean nativeTestSwap(Object other);
384: 
385:   @Test
386:   public void testEqualOperator() {
387:     assertThat(nativeTestEqualOperator(new Object())).isTrue();
388:   }
389: 
390:   private native boolean nativeTestEqualOperator(Object other);
391: 
392:   @Test
393:   public void testRelaseAlias() {
394:     assertThat(nativeTestReleaseAlias()).isTrue();
395:   }
396: 
397:   private native boolean nativeTestReleaseAlias();
398: 
399:   @Test
400:   public void testLockingWeakReferences() {
````
- EN: Implements callable logic such as `nativeTestIsInstanceOf`, `isSameObject`, `nativeTestIsSameObject`, `testGetSuperClass`.
- CN: 实现可调用逻辑，例如 `nativeTestIsInstanceOf`, `isSameObject`, `nativeTestIsSameObject`, `testGetSuperClass`。

### Lines 401-480
````java
401:     assertThat(nativeTestLockingWeakReferences()).isTrue();
402:   }
403: 
404:   private native boolean nativeTestLockingWeakReferences();
405: 
406:   @Test
407:   public void testCreatingReferences() {
408:     assertThat(nativeTestCreatingReferences()).isTrue();
409:   }
410: 
411:   private native boolean nativeTestCreatingReferences();
412: 
413:   @Test
414:   public void testAssignmentAndCopyConstructors() {
415:     assertThat(nativeTestAssignmentAndCopyConstructors()).isTrue();
416:   }
417: 
418:   private native boolean nativeTestAssignmentAndCopyConstructors();
419: 
420:   @Test
421:   public void testAssignmentAndCopyCrossTypes() {
422:     assertThat(nativeTestAssignmentAndCopyCrossTypes()).isTrue();
423:   }
424: 
425:   private native boolean nativeTestAssignmentAndCopyCrossTypes();
426: 
427:   @Test
428:   public void testNullReferences() {
429:     assertThat(nativeTestNullReferences()).isTrue();
430:   }
431: 
432:   private native boolean nativeTestNullReferences();
433: 
434:   @Test
435:   public void testAutoAliasRefReturningVoid() {
436:     nativeTestAutoAliasRefReturningVoid();
437:   }
438: 
439:   private native void nativeTestAutoAliasRefReturningVoid();
440: 
441:   @Test
442:   public void testFieldAccess() {
443:     mIntFieldTest = 17;
444:     assertThat(nativeTestFieldAccess("mIntFieldTest", mIntFieldTest, 42)).isTrue();
445:     assertThat(mIntFieldTest).isEqualTo(42);
446:   }
447: 
448:   private native boolean nativeTestFieldAccess(String name, int oldVal, int newVal);
449: 
450:   @Test
451:   public void testStringFieldAccess() {
452:     mStringFieldTest = "initial";
453:     assertThat(nativeTestStringFieldAccess("mStringFieldTest", mStringFieldTest, "final")).isTrue();
454:     assertThat(mStringFieldTest).isEqualTo("final");
455:   }
456: 
457:   private native boolean nativeTestStringFieldAccess(String name, String oldVal, String newVal);
458: 
459:   @Test
460:   public void testReferenceFieldAccess() {
461:     for (boolean useWrapper : new boolean[] {false, true}) {
462:       mReferenceFieldTest = new TestThing();
463:       TestThing newthing = new TestThing();
464: 
465:       assertThat(
466:               nativeTestReferenceFieldAccess(
467:                   "mReferenceFieldTest", mReferenceFieldTest, newthing, useWrapper))
468:           .isTrue();
469:       assertThat(mReferenceFieldTest).isEqualTo(newthing);
470:     }
471:   }
472: 
473:   private native boolean nativeTestReferenceFieldAccess(
474:       String name, Object oldVal, Object newVal, boolean useWrapper);
475: 
476:   @Test
477:   public void testStaticFieldAccess() {
478:     sIntFieldTest = 17;
479:     assertThat(nativeTestStaticFieldAccess("sIntFieldTest", sIntFieldTest, 42)).isTrue();
480:     assertThat(sIntFieldTest).isEqualTo(42);
````
- EN: Implements callable logic such as `nativeTestLockingWeakReferences`, `testCreatingReferences`, `nativeTestCreatingReferences`, `testAssignmentAndCopyConstructors`.
- CN: 实现可调用逻辑，例如 `nativeTestLockingWeakReferences`, `testCreatingReferences`, `nativeTestCreatingReferences`, `testAssignmentAndCopyConstructors`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 481-560
````java
481:   }
482: 
483:   private native boolean nativeTestStaticFieldAccess(String name, int oldVal, int newVal);
484: 
485:   @Test
486:   public void testStaticStringFieldAccess() {
487:     sStringFieldTest = "initial";
488:     assertThat(nativeTestStaticStringFieldAccess("sStringFieldTest", sStringFieldTest, "final"))
489:         .isTrue();
490:     assertThat(sStringFieldTest).isEqualTo("final");
491:   }
492: 
493:   private native boolean nativeTestStaticStringFieldAccess(String name, String oVal, String nVal);
494: 
495:   @Test
496:   public void testStaticReferenceFieldAccess() {
497:     for (boolean useWrapper : new boolean[] {false, true}) {
498:       sReferenceFieldTest = new TestThing();
499:       TestThing newthing = new TestThing();
500: 
501:       assertThat(
502:               nativeTestStaticReferenceFieldAccess(
503:                   "sReferenceFieldTest", sReferenceFieldTest, newthing, useWrapper))
504:           .isTrue();
505:       assertThat(sReferenceFieldTest).isEqualTo(newthing);
506:     }
507:   }
508: 
509:   private native boolean nativeTestStaticReferenceFieldAccess(
510:       String name, Object oldVal, Object newVal, boolean useWrapper);
511: 
512:   @Test
513:   public void testNonVirtualMethod() {
514:     assertThat(nativeTestNonVirtualMethod(true)).isTrue();
515:   }
516: 
517:   private native boolean nativeTestNonVirtualMethod(boolean s);
518: 
519:   @Test
520:   public void testArrayCreation() {
521:     String[] expectedStrings = {"one", "two", "three"};
522:     String[] joinedStrings =
523:         nativeTestArrayCreation(expectedStrings[0], expectedStrings[1], expectedStrings[2]);
524:     assertThat(joinedStrings).isEqualTo(expectedStrings);
525:   }
526: 
527:   private native String[] nativeTestArrayCreation(String s0, String s1, String s2);
528: 
529:   @Test
530:   public void testMultidimensionalObjectArray() {
531:     String[] strings = {"one", "two", "three"};
532:     String[][] expectedStrings = {{"one", "two"}, {"three"}};
533:     String[][] joinedStrings =
534:         nativeTestMultidimensionalObjectArray(strings[0], strings[1], strings[2]);
535:     assertThat(joinedStrings).isEqualTo(expectedStrings);
536:   }
537: 
538:   private native String[][] nativeTestMultidimensionalObjectArray(String s0, String s1, String s2);
539: 
540:   @Test
541:   public void testMultidimensionalPrimitiveArray() {
542:     int[] nums = {1, 2, 3};
543:     int[][] expectedNums = {{1, 2}, {3}};
544:     int[][] gotNums = nativeTestMultidimensionalPrimitiveArray(nums[0], nums[1], nums[2]);
545:     assertThat(gotNums).isEqualTo(expectedNums);
546:   }
547: 
548:   private native int[][] nativeTestMultidimensionalPrimitiveArray(int i0, int i1, int i2);
549: 
550:   private String[] mCapturedStringArray = null;
551: 
552:   @DoNotStrip
553:   String captureStringArray(String[] input) {
554:     mCapturedStringArray = input;
555:     return "Stub";
556:   }
557: 
558:   @Test
559:   public void testBuildStringArray() throws Exception {
560:     String[] input = {"Four", "score", "and", "seven", "beers", "ago"};
````
- EN: Implements callable logic such as `nativeTestStaticFieldAccess`, `testStaticStringFieldAccess`, `nativeTestStaticStringFieldAccess`, `testStaticReferenceFieldAccess`.
- CN: 实现可调用逻辑，例如 `nativeTestStaticFieldAccess`, `testStaticStringFieldAccess`, `nativeTestStaticStringFieldAccess`, `testStaticReferenceFieldAccess`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 561-640
````java
561:     nativeTestBuildStringArray(input);
562:     assertThat(mCapturedStringArray).isEqualTo(input);
563:   }
564: 
565:   private native String nativeTestBuildStringArray(String... input);
566: 
567:   public Object methodResolutionWithCxxTypes(String t, long val) {
568:     if (!"test".equals(t) || val != 3) throw new RuntimeException();
569:     return null;
570:   }
571: 
572:   public void methodResolutionWithCxxTypesVoid(String t, long val) {
573:     if (!"test".equals(t) || val != 3) throw new RuntimeException();
574:   }
575: 
576:   public int methodResolutionWithCxxTypesInt(String t, long val) {
577:     if (!"test".equals(t) || val != 3) throw new RuntimeException();
578:     return 0;
579:   }
580: 
581:   public static Object methodResolutionWithCxxTypesStatic(String t, long val) {
582:     if (!"test".equals(t) || val != 3) throw new RuntimeException();
583:     return null;
584:   }
585: 
586:   public static void methodResolutionWithCxxTypesVoidStatic(String t, long val) {
587:     if (!"test".equals(t) || val != 3) throw new RuntimeException();
588:   }
589: 
590:   public static int methodResolutionWithCxxTypesIntStatic(String t, long val) {
591:     if (!"test".equals(t) || val != 3) throw new RuntimeException();
592:     return 0;
593:   }
594: 
595:   @Test
596:   public void testMethodResolutionWithCxxTypes() {
597:     testMethodResolutionWithCxxTypesNative("methodResolutionWithCxxTypes", "test", 3);
598:   }
599: 
600:   private native void testMethodResolutionWithCxxTypesNative(
601:       String callbackName, String str, long val);
602: 
603:   @Test(expected = CustomException.class)
604:   public void testHandleJavaCustomException() {
605:     testHandleJavaCustomExceptionNative();
606:   }
607: 
608:   private native void testHandleJavaCustomExceptionNative();
609: 
610:   @Test
611:   public void testHandleNullExceptionMessage() {
612:     testHandleNullExceptionMessageNative();
613:   }
614: 
615:   private native void testHandleNullExceptionMessageNative();
616: 
617:   @Test
618:   public void testHandleNestedException() {
619:     try {
620:       nativeTestHandleNestedException();
621:     } catch (Throwable e) {
622:       assertThat(e).isInstanceOf(ArrayIndexOutOfBoundsException.class);
623:       e = e.getCause();
624:       assertThat(e).isInstanceOf(RuntimeException.class);
625:       e = e.getCause();
626:       assertThat(e).isInstanceOf(CustomException.class).hasNoCause();
627:     }
628:   }
629: 
630:   private native void nativeTestHandleNestedException();
631: 
632:   @Test(expected = CppException.class)
633:   public void testHandleNoRttiException() {
634:     nativeTestHandleNoRttiException();
635:   }
636: 
637:   private native void nativeTestHandleNoRttiException();
638: 
639:   @Test
640:   public void testCopyConstructor() {
````
- EN: Implements callable logic such as `nativeTestBuildStringArray`, `methodResolutionWithCxxTypes`, `methodResolutionWithCxxTypesVoid`, `methodResolutionWithCxxTypesInt`.
- CN: 实现可调用逻辑，例如 `nativeTestBuildStringArray`, `methodResolutionWithCxxTypes`, `methodResolutionWithCxxTypesVoid`, `methodResolutionWithCxxTypesInt`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 641-720
````java
641:     assertThat(nativeTestCopyConstructor())
642:         .isEqualTo("com.facebook.jni.FBJniTests$CustomException: getMessages: 1");
643:   }
644: 
645:   private native String nativeTestCopyConstructor();
646: 
647:   @Test
648:   public void testMoveConstructorWithEmptyWhat() {
649:     assertThat(nativeTestMoveConstructorWithEmptyWhat())
650:         .isEqualTo("com.facebook.jni.FBJniTests$CustomException: getMessages: 1");
651:   }
652: 
653:   private native String nativeTestMoveConstructorWithEmptyWhat();
654: 
655:   @Test
656:   public void testMoveConstructorWithPopulatedWhat() {
657:     assertThat(nativeTestMoveConstructorWithPopulatedWhat())
658:         .isEqualTo("com.facebook.jni.FBJniTests$CustomException: getMessages: 1");
659:   }
660: 
661:   private native String nativeTestMoveConstructorWithPopulatedWhat();
662: 
663:   @DoNotStrip // Used in native code.
664:   protected void customExceptionThrower() throws CustomException {
665:     throw new CustomException();
666:   }
667: 
668:   @DoNotStrip // Used in native code.
669:   protected void nullMessageThrower() throws NullPointerException {
670:     // just like Preconditions.checkNotNull() does
671:     throw new NullPointerException();
672:   }
673: 
674:   @Test
675:   public void testHandleCppRuntimeError() {
676:     String message = "Sample runtime error.";
677:     thrown.expect(RuntimeException.class);
678:     thrown.expectMessage(message);
679:     nativeTestHandleCppRuntimeError(message);
680:   }
681: 
682:   private native void nativeTestHandleCppRuntimeError(String message);
683: 
684:   @Test(expected = IOException.class)
685:   public void testHandleCppIOBaseFailure() {
686:     nativeTestHandleCppIOBaseFailure();
687:   }
688: 
689:   private native void nativeTestHandleCppIOBaseFailure();
690: 
691:   @Test(expected = CppSystemErrorException.class)
692:   public void testHandleCppSystemError() {
693:     nativeTestHandleCppSystemError();
694:   }
695: 
696:   private native void nativeTestHandleCppSystemError();
697: 
698:   @Test(expected = RuntimeException.class)
699:   public void testInterDsoExceptionHandlingA() {
700:     nativeTestInterDsoExceptionHandlingA();
701:   }
702: 
703:   private native void nativeTestInterDsoExceptionHandlingA();
704: 
705:   @Test
706:   public void testInterDsoExceptionHandlingB() {
707:     assertThat(nativeTestInterDsoExceptionHandlingB()).isTrue();
708:   }
709: 
710:   private native boolean nativeTestInterDsoExceptionHandlingB();
711: 
712:   @Test
713:   public void testHandleNonStdExceptionThrow() {
714:     try {
715:       nativeTestHandleNonStdExceptionThrow();
716:       Fail.failBecauseExceptionWasNotThrown(UnknownCppException.class);
717:     } catch (UnknownCppException ex) {
718:       if (System.getProperty("os.name").startsWith("Windows")) {
719:         // Unknown exception types not supported on Windows.
720:         assertThat(ex.getMessage()).isEqualTo("Unknown");
````
- EN: Implements callable logic such as `nativeTestCopyConstructor`, `testMoveConstructorWithEmptyWhat`, `nativeTestMoveConstructorWithEmptyWhat`, `testMoveConstructorWithPopulatedWhat`.
- CN: 实现可调用逻辑，例如 `nativeTestCopyConstructor`, `testMoveConstructorWithEmptyWhat`, `nativeTestMoveConstructorWithEmptyWhat`, `testMoveConstructorWithPopulatedWhat`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 721-800
````java
721:         return;
722:       }
723:       // the actual string is implementation-defined and mangled, but in practice,
724:       // it has the name of the C++ type in it somewhere.
725:       assertThat(ex.getMessage()).startsWith("Unknown: ").contains("NonStdException");
726:     }
727:   }
728: 
729:   private native void nativeTestHandleNonStdExceptionThrow();
730: 
731:   @Test(expected = UnknownCppException.class)
732:   public void testHandleCppCharPointerThrow() {
733:     nativeTestHandleCppCharPointerThrow();
734:   }
735: 
736:   private native void nativeTestHandleCppCharPointerThrow();
737: 
738:   @Test(expected = IllegalArgumentException.class)
739:   public void testThrowJavaExceptionByName() {
740:     nativeTestThrowJavaExceptionByName();
741:   }
742: 
743:   private native void nativeTestThrowJavaExceptionByName();
744: 
745:   @Test(expected = UnknownCppException.class)
746:   public void testHandleCppIntThrow() {
747:     nativeTestHandleCppIntThrow();
748:   }
749: 
750:   private native void nativeTestHandleCppIntThrow();
751: 
752:   @Test
753:   public void testJThread() {
754:     assertThat(nativeTestJThread()).isEqualTo(1);
755:   }
756: 
757:   private native int nativeTestJThread();
758: 
759:   @Test
760:   public void testThreadScopeGuard() {
761:     assertThat(nativeTestThreadScopeGuard(17)).isEqualTo(42);
762:   }
763: 
764:   private native int nativeTestThreadScopeGuard(double input);
765: 
766:   @Test
767:   public void testNestedThreadScopeGuard() {
768:     assertThat(nativeTestNestedThreadScopeGuard(17)).isEqualTo(42);
769:   }
770: 
771:   private native int nativeTestNestedThreadScopeGuard(double input);
772: 
773:   @Test
774:   public void testClassLoadInWorker() {
775:     assertThat(nativeTestClassLoadInWorker()).isEqualTo(1);
776:   }
777: 
778:   private native int nativeTestClassLoadInWorker();
779: 
780:   @Test
781:   public void testClassLoadWorkerFastPath() {
782:     assertThat(nativeTestClassLoadWorkerFastPath()).isEqualTo(3);
783:   }
784: 
785:   private native int nativeTestClassLoadWorkerFastPath();
786: 
787:   @Test
788:   public void testToString() {
789:     assertThat(nativeTestToString()).isTrue();
790:   }
791: 
792:   private native boolean nativeTestToString();
793: 
794:   // Casting alias_ref
795: 
796:   @Test
797:   public void testCorrectStaticCastAliasRef() {
798:     // Static cast can't fail at run time.  If the object isn't actually
799:     // of that type, we just get undefined behaviour, which we can't
800:     // check for.  So we only do a positive test.
````
- EN: Implements callable logic such as `nativeTestHandleNonStdExceptionThrow`, `testHandleCppCharPointerThrow`, `nativeTestHandleCppCharPointerThrow`, `testThrowJavaExceptionByName`.
- CN: 实现可调用逻辑，例如 `nativeTestHandleNonStdExceptionThrow`, `testHandleCppCharPointerThrow`, `nativeTestHandleCppCharPointerThrow`, `testThrowJavaExceptionByName`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 801-880
````java
801:     assertThat(nativeStaticCastAliasRefToString("hello")).isTrue();
802:   }
803: 
804:   @Test
805:   public void testNullStaticCastAliasRef() {
806:     assertThat(nativeStaticCastAliasRefToString(null)).isTrue();
807:   }
808: 
809:   private native boolean nativeStaticCastAliasRefToString(Object a);
810: 
811:   @Test
812:   public void testDynamicCastAliasRefToSame() {
813:     assertThat(nativeDynamicCastAliasRefToThrowable(new Throwable())).isTrue();
814:   }
815: 
816:   public void testDynamicCastAliasRefToBase() {
817:     assertThat(nativeDynamicCastAliasRefToThrowable(new Exception())).isTrue();
818:   }
819: 
820:   @Test(expected = ClassCastException.class)
821:   public void testDynamicCastAliasRefToDerived() {
822:     nativeDynamicCastAliasRefToThrowable(new Object());
823:   }
824: 
825:   @Test(expected = ClassCastException.class)
826:   public void testDynamicCastAliasRefToUnrelated() {
827:     nativeDynamicCastAliasRefToThrowable(new Integer(23));
828:   }
829: 
830:   @Test
831:   public void testNullDynamicCastAliasRef() {
832:     assertThat(nativeDynamicCastAliasRefToThrowable(null)).isTrue();
833:   }
834: 
835:   private native boolean nativeDynamicCastAliasRefToThrowable(Object a);
836: 
837:   // Casting local_ref
838: 
839:   @Test
840:   public void testCorrectStaticCastLocalRef() {
841:     // Static cast can't fail at run time.  If the object isn't actually
842:     // of that type, we just get undefined behaviour, which we can't
843:     // check for.  So we only do a positive test.
844:     assertThat(nativeStaticCastLocalRefToString("hello")).isTrue();
845:   }
846: 
847:   @Test
848:   public void testNullStaticCastLocalRef() {
849:     assertThat(nativeStaticCastLocalRefToString(null)).isTrue();
850:   }
851: 
852:   private native boolean nativeStaticCastLocalRefToString(Object a);
853: 
854:   @Test
855:   public void testCorrectDynamicCastLocalRef() {
856:     assertThat(nativeDynamicCastLocalRefToString("hello")).isTrue();
857:   }
858: 
859:   @Test(expected = ClassCastException.class)
860:   public void testIncorrectDynamicCastLocalRef() {
861:     nativeDynamicCastLocalRefToString(new Integer(23));
862:   }
863: 
864:   @Test
865:   public void testNullDynamicCastLocalRef() {
866:     assertThat(nativeDynamicCastLocalRefToString(null)).isTrue();
867:   }
868: 
869:   private native boolean nativeDynamicCastLocalRefToString(Object a);
870: 
871:   // Casting global_ref
872: 
873:   @Test
874:   public void testCorrectStaticCastGlobalRef() {
875:     // Static cast can't fail at run time.  If the object isn't actually
876:     // of that type, we just get undefined behaviour, which we can't
877:     // check for.  So we only do a positive test.
878:     assertThat(nativeStaticCastGlobalRefToString("hello")).isTrue();
879:   }
880: 
````
- EN: Implements callable logic such as `testNullStaticCastAliasRef`, `nativeStaticCastAliasRefToString`, `testDynamicCastAliasRefToSame`, `testDynamicCastAliasRefToBase`.
- CN: 实现可调用逻辑，例如 `testNullStaticCastAliasRef`, `nativeStaticCastAliasRefToString`, `testDynamicCastAliasRefToSame`, `testDynamicCastAliasRefToBase`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 881-911
````java
881:   @Test
882:   public void testNullStaticCastGlobalRef() {
883:     assertThat(nativeStaticCastGlobalRefToString(null)).isTrue();
884:   }
885: 
886:   private native boolean nativeStaticCastGlobalRefToString(Object a);
887: 
888:   @Test
889:   public void testCorrectDynamicCastGlobalRef() {
890:     assertThat(nativeDynamicCastGlobalRefToString("hello")).isTrue();
891:   }
892: 
893:   @Test(expected = ClassCastException.class)
894:   public void testIncorrectDynamicCastGlobalRef() {
895:     nativeDynamicCastGlobalRefToString(new Integer(23));
896:   }
897: 
898:   @Test
899:   public void testNullDynamicCastGlobalRef() {
900:     assertThat(nativeDynamicCastGlobalRefToString(null)).isTrue();
901:   }
902: 
903:   private native boolean nativeDynamicCastGlobalRefToString(Object a);
904: 
905:   @Test
906:   public void testCriticalNativeMethodBindsAndCanBeInvoked() {
907:     assertThat(nativeCriticalNativeMethodBindsAndCanBeInvoked(12, 3.45f)).isTrue();
908:   }
909: 
910:   private static native boolean nativeCriticalNativeMethodBindsAndCanBeInvoked(int a, float b);
911: }
````
- EN: Implements callable logic such as `testNullStaticCastGlobalRef`, `nativeStaticCastGlobalRefToString`, `testCorrectDynamicCastGlobalRef`, `testIncorrectDynamicCastGlobalRef`.
- CN: 实现可调用逻辑，例如 `testNullStaticCastGlobalRef`, `nativeStaticCastGlobalRefToString`, `testCorrectDynamicCastGlobalRef`, `testIncorrectDynamicCastGlobalRef`。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Validation and test coverage / 校验与测试覆盖
- Symbol `FBJniTests` / 符号 `FBJniTests`
- Symbol `CustomException` / 符号 `CustomException`
- Symbol `Callbacks` / 符号 `Callbacks`
- Symbol `TestThing` / 符号 `TestThing`

## Dependencies / 依赖关系
- Java imports: `com.facebook.jni.annotations.DoNotStrip`, `java.io.IOException`, `java.util.ArrayList`, `java.util.concurrent.Callable`, `org.fest.assertions.api.Fail`, `org.junit.Test`, `org.junit.runner.RunWith`, `org.mockito.Mock`, `org.mockito.runners.MockitoJUnitRunner`
- Java 导入: `com.facebook.jni.annotations.DoNotStrip`, `java.io.IOException`, `java.util.ArrayList`, `java.util.concurrent.Callable`, `org.fest.assertions.api.Fail`, `org.junit.Test`, `org.junit.runner.RunWith`, `org.mockito.Mock`, `org.mockito.runners.MockitoJUnitRunner`
