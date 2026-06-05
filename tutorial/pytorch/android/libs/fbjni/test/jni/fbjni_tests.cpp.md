# fbjni_tests.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/test/jni/fbjni_tests.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements JNI bridge abstractions used by Android-facing native code.
- 用途 (CN): 实现供 Android 原生代码使用的 JNI 桥接抽象。

## Line-by-Line Analysis / 逐行分析
### Lines 1-80
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
17: #include <ios>
18: #include <stdexcept>
19: #include <system_error>
20: #include <thread>
21: #include <chrono>
22: 
23: #include <fbjni/fbjni.h>
24: #include <fbjni/JThread.h>
25: 
26: #include "expect.h"
27: #include "no_rtti.h"
28: 
29: #include "inter_dso_exception_test_2/Test.h"
30: 
31: #define EXPECT_SAME(A, B, C) EXPECT((A) == (B) && (B) == (C) && (C) == (A))
32: 
33: // A lot of the functions here are just confirming that compilation works.
34: #pragma GCC diagnostic ignored "-Wunused-function"
35: 
36: using namespace facebook::jni;
37: 
38: namespace {
39: 
40: struct Callbacks : public facebook::jni::JavaClass<Callbacks> {
41:   constexpr static auto kJavaDescriptor = "Lcom/facebook/jni/FBJniTests$Callbacks;";
42: };
43: 
44: struct TestThing : public JavaClass<TestThing> {
45:   constexpr static auto kJavaDescriptor = "Lcom/facebook/jni/FBJniTests$TestThing;";
46: };
47: 
48: // Yes, sloppy and does not handle conversions correctly but does it's job here
49: static std::string ToString(JNIEnv* env, jstring java_string) {
50:   auto chars = env->GetStringUTFChars(java_string, nullptr);
51:   if (chars == nullptr) {
52:     throw std::runtime_error{"Couldn't get UTF chars"};
53:   }
54: 
55:   std::string string{chars};
56:   env->ReleaseStringUTFChars(java_string, chars);
57: 
58:   return string;
59: }
60: 
61: jboolean TestClassResolution(JNIEnv* env, jobject self, jstring class_name) {
62:   auto resolved_class = findClassLocal(ToString(env, class_name).c_str());
63:   return resolved_class.get() != nullptr? JNI_TRUE : JNI_FALSE;
64: }
65: 
66: jboolean TestLazyClassResolution(JNIEnv* env, jobject self, jstring class_name) {
67:   auto resolved_class = alias_ref<jclass>{};
68:   resolved_class = findClassLocal(ToString(env, class_name).c_str());
69:   return resolved_class.get() != nullptr? JNI_TRUE : JNI_FALSE;
70: }
71: 
72: jobject TestCreateInstanceOf(JNIEnv* env, jobject self, jstring class_name) {
73:   auto clazz = findClassLocal(ToString(env, class_name).c_str());
74:   auto constructor = clazz->getConstructor<jobject(jstring)>();
75:   return clazz->newObject(constructor, class_name).release();
76: }
77: 
78: jboolean TestTypeDescriptors(JNIEnv* env, jobject self) {
79: #define FIXED_STRING_EXPECT_EQ(actual, expected)                        \
80:   static_assert((actual) == (expected), "descriptor mismatch")
````
- EN: Pulls in native headers such as `ios`, `stdexcept`, `system_error`, `thread`.
- CN: 引入原生头文件，例如 `ios`, `stdexcept`, `system_error`, `thread`。
- EN: Declares or extends types including `Callbacks`, `TestThing`.
- CN: 声明或扩展类型，包括 `Callbacks`, `TestThing`。
- EN: Implements callable logic such as `ToString`, `TestClassResolution`, `TestLazyClassResolution`, `TestCreateInstanceOf`.
- CN: 实现可调用逻辑，例如 `ToString`, `TestClassResolution`, `TestLazyClassResolution`, `TestCreateInstanceOf`。
- EN: Includes validation, assertions, or defensive checks.
- CN: 包含校验、断言或防御性检查。

### Lines 81-160
````cpp
 81: 
 82:   FIXED_STRING_EXPECT_EQ(jtype_traits<jboolean>::kDescriptor, "Z");
 83:   FIXED_STRING_EXPECT_EQ(jtype_traits<jbyte>::kDescriptor, "B");
 84:   FIXED_STRING_EXPECT_EQ(jtype_traits<jchar>::kDescriptor, "C");
 85:   FIXED_STRING_EXPECT_EQ(jtype_traits<jdouble>::kDescriptor, "D");
 86:   FIXED_STRING_EXPECT_EQ(jtype_traits<jfloat>::kDescriptor, "F");
 87:   FIXED_STRING_EXPECT_EQ(jtype_traits<jint>::kDescriptor, "I");
 88:   FIXED_STRING_EXPECT_EQ(jtype_traits<jlong>::kDescriptor, "J");
 89:   FIXED_STRING_EXPECT_EQ(jtype_traits<jshort>::kDescriptor, "S");
 90: 
 91:   FIXED_STRING_EXPECT_EQ(jtype_traits<jstring>::kDescriptor, "Ljava/lang/String;");
 92:   FIXED_STRING_EXPECT_EQ(jtype_traits<jobject>::kDescriptor, "Ljava/lang/Object;");
 93: 
 94:   FIXED_STRING_EXPECT_EQ(jtype_traits<jintArray>::kDescriptor, "[I");
 95:   FIXED_STRING_EXPECT_EQ(jtype_traits<jtypeArray<jstring>>::kDescriptor, "[Ljava/lang/String;");
 96:   FIXED_STRING_EXPECT_EQ(jtype_traits<jtypeArray<jtypeArray<jstring>>>::kDescriptor
 97:      , "[[Ljava/lang/String;");
 98:   FIXED_STRING_EXPECT_EQ(jtype_traits<jtypeArray<jintArray>>::kDescriptor, "[[I");
 99: 
100:   // base_name() is meaningless for primitive types.
101:   FIXED_STRING_EXPECT_EQ(jtype_traits<jstring>::kBaseName, "java/lang/String");
102:   FIXED_STRING_EXPECT_EQ(jtype_traits<jobject>::kBaseName, "java/lang/Object");
103: 
104:   FIXED_STRING_EXPECT_EQ(jtype_traits<jintArray>::kBaseName, "[I");
105:   FIXED_STRING_EXPECT_EQ(jtype_traits<jtypeArray<jstring>>::kBaseName, "[Ljava/lang/String;");
106:   FIXED_STRING_EXPECT_EQ(jtype_traits<jtypeArray<jtypeArray<jstring>>>::kBaseName, "[[Ljava/lang/String;");
107:   FIXED_STRING_EXPECT_EQ(jtype_traits<jtypeArray<jintArray>>::kBaseName, "[[I");
108: 
109:   return JNI_TRUE;
110: }
111: 
112: jboolean TestVirtualMethodResolution_I(
113:     JNIEnv* env,
114:     jobject self,
115:     jstring class_name,
116:     jstring method_name)
117: {
118:   auto resolved_class = findClassLocal(ToString(env, class_name).c_str());
119:   auto resolved_method =
120:     resolved_class->getMethod<jint()>(ToString(env, method_name).c_str());
121:   return static_cast<bool>(resolved_method);
122: }
123: 
124: jboolean TestVirtualMethodResolution_arrB(
125:     JNIEnv* env,
126:     jobject self,
127:     jstring class_name,
128:     jstring method_name)
129: {
130:   auto resolved_class = findClassLocal(ToString(env, class_name).c_str());
131:   auto resolved_method =
132:     resolved_class->getMethod<jbyteArray()>(ToString(env, method_name).c_str());
133:   return static_cast<bool>(resolved_method);
134: }
135: 
136: jboolean TestVirtualMethodResolution_S_arrS(
137:     JNIEnv* env,
138:     jobject self,
139:     jstring class_name,
140:     jstring method_name)
141: {
142:   auto resolved_class = findClassLocal(ToString(env, class_name).c_str());
143:   auto resolved_method =
144:     resolved_class->getMethod<jtypeArray<jstring>(jstring)>(ToString(env, method_name).c_str());
145:   return static_cast<bool>(resolved_method);
146: }
147: 
148: jboolean TestVirtualMethodResolution_arrarrS(
149:     JNIEnv* env,
150:     jobject self,
151:     jstring class_name,
152:     jstring method_name)
153: {
154:   auto resolved_class = findClassLocal(ToString(env, class_name).c_str());
155:   auto resolved_method =
156:     resolved_class->getStaticMethod<jtypeArray<jtypeArray<jstring>>()>(ToString(env, method_name).c_str());
157:   return static_cast<bool>(resolved_method);
158: }
159: 
160: jboolean TestVirtualMethodResolution_arrarrI(
````
- EN: Implements callable logic such as `TestVirtualMethodResolution_I`, `TestVirtualMethodResolution_arrB`, `TestVirtualMethodResolution_S_arrS`, `TestVirtualMethodResolution_arrarrS`.
- CN: 实现可调用逻辑，例如 `TestVirtualMethodResolution_I`, `TestVirtualMethodResolution_arrB`, `TestVirtualMethodResolution_S_arrS`, `TestVirtualMethodResolution_arrarrS`。
- EN: Includes validation, assertions, or defensive checks.
- CN: 包含校验、断言或防御性检查。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 161-240
````cpp
161:     JNIEnv* env,
162:     jobject self,
163:     jstring class_name,
164:     jstring method_name)
165: {
166:   auto resolved_class = findClassLocal(ToString(env, class_name).c_str());
167:   auto resolved_method =
168:     resolved_class->getStaticMethod<jtypeArray<jintArray>()>(ToString(env, method_name).c_str());
169:   return static_cast<bool>(resolved_method);
170: }
171: 
172: jboolean TestLazyVirtualMethodResolution_I(
173:     JNIEnv* env,
174:     jobject self,
175:     jstring class_name,
176:     jstring method_name)
177: {
178:   auto resolved_class = findClassLocal(ToString(env, class_name).c_str());
179:   auto resolved_method = JMethod<jint()>{};
180:   resolved_method = resolved_class->getMethod<jint()>(ToString(env, method_name).c_str());
181:   return static_cast<bool>(resolved_method);
182: }
183: 
184: void TestJMethodCallbacks(JNIEnv* env, jobject self, Callbacks::javaobject callbacks) {
185:   static const auto callbacks_class = Callbacks::javaClassStatic();
186: 
187:   static const auto void_foo = callbacks_class->getMethod<void()>("voidFoo");
188:   void_foo(callbacks);
189: 
190:   static const auto boolean_foo = callbacks_class->getMethod<jboolean()>("booleanFoo");
191:   boolean_foo(callbacks);
192: 
193:   static const auto byte_foo = callbacks_class->getMethod<jbyte()>("byteFoo");
194:   byte_foo(callbacks);
195: 
196:   static const auto char_foo = callbacks_class->getMethod<jchar()>("charFoo");
197:   char_foo(callbacks);
198: 
199:   static const auto short_foo = callbacks_class->getMethod<jshort()>("shortFoo");
200:   short_foo(callbacks);
201: 
202:   static const auto int_foo = callbacks_class->getMethod<jint()>("intFoo");
203:   int_foo(callbacks);
204: 
205:   static const auto long_foo = callbacks_class->getMethod<jlong()>("longFoo");
206:   long_foo(callbacks);
207: 
208:   static const auto float_foo = callbacks_class->getMethod<jfloat()>("floatFoo");
209:   float_foo(callbacks);
210: 
211:   static const auto double_foo = callbacks_class->getMethod<jdouble()>("doubleFoo");
212:   double_foo(callbacks);
213: 
214:   static const auto object_foo = callbacks_class->getMethod<jobject()>("objectFoo");
215:   object_foo(callbacks);
216: 
217:   static const auto string_foo = callbacks_class->getMethod<jstring()>("stringFoo");
218:   string_foo(callbacks);
219: }
220: 
221: // Try to test the static functions
222: void TestJStaticMethodCallbacks(JNIEnv* env, jobject self) {
223:   // static auto callbacks_class = findClassStatic(callbacks_class_name);
224:   auto cls = findClassLocal("com/facebook/jni/FBJniTests");
225:   jclass jcls = env->FindClass("com/facebook/jni/FBJniTests");
226: 
227:   static const auto void_foo_static = cls->getStaticMethod<void()>("voidFooStatic");
228:   void_foo_static(jcls);
229: 
230:   static const auto boolean_foo_static = cls->getStaticMethod<jboolean()>("booleanFooStatic");
231:   boolean_foo_static(jcls);
232: 
233:   static const auto byte_foo_static = cls->getStaticMethod<jbyte()>("byteFooStatic");
234:   byte_foo_static(jcls);
235: 
236:   static const auto char_foo_static = cls->getStaticMethod<jchar(jchar, jint)>("charFooStatic");
237:   char_foo_static(jcls, 'c', 5);
238: 
239:   static const auto short_foo_static = cls->getStaticMethod<jshort(jshort, jshort)>("shortFooStatic");
240:   short_foo_static(jcls, 17, 42);
````
- EN: Implements callable logic such as `TestLazyVirtualMethodResolution_I`, `TestJMethodCallbacks`, `TestJStaticMethodCallbacks`.
- CN: 实现可调用逻辑，例如 `TestLazyVirtualMethodResolution_I`, `TestJMethodCallbacks`, `TestJStaticMethodCallbacks`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-320
````cpp
241: 
242:   static const auto int_foo_static = cls->getStaticMethod<jint(jint)>("intFooStatic");
243:   int_foo_static(jcls, 5);
244: 
245:   static const auto long_foo_static = cls->getStaticMethod<jlong()>("longFooStatic");
246:   long_foo_static(jcls);
247: 
248:   static const auto float_foo_static = cls->getStaticMethod<jfloat()>("floatFooStatic");
249:   float_foo_static(jcls);
250: 
251:   static const auto double_foo_static = cls->getStaticMethod<jdouble()>("doubleFooStatic");
252:   double_foo_static(jcls);
253: 
254:   static const auto object_foo_static = cls->getStaticMethod<jobject()>("objectFooStatic");
255:   object_foo_static(jcls);
256: 
257:   static const auto string_foo_static = cls->getStaticMethod<jstring()>("stringFooStatic");
258:   string_foo_static(jcls);
259: }
260: 
261: jboolean TestIsAssignableFrom(JNIEnv* env, jobject self, jclass cls1, jclass cls2) {
262:   return adopt_local(cls1)->isAssignableFrom(cls2);
263: }
264: 
265: jboolean TestIsInstanceOf(JNIEnv* env, jobject self, jobject test_object, jclass cls) {
266:   auto clsref = adopt_local(test_object);
267:   return clsref->isInstanceOf(cls);
268: }
269: 
270: jboolean TestIsSameObject(JNIEnv* env, jobject self, jobject a, jobject b) {
271:   return isSameObject(a, b);
272: }
273: 
274: jboolean TestGetSuperclass(JNIEnv* env, jobject self, jclass test_class, jclass super_class) {
275:   return isSameObject(adopt_local(test_class)->getSuperclass().get(), super_class);
276: }
277: 
278: jboolean StaticCastAliasRefToString(JNIEnv *, jobject , jobject string_as_object) {
279:   alias_ref<jobject> string_as_object_alias_ref { string_as_object };
280:   alias_ref<jstring> string_alias_ref = static_ref_cast<jstring>(string_as_object_alias_ref);
281:   return isSameObject(string_alias_ref.get(), string_as_object_alias_ref.get());
282: }
283: 
284: jboolean DynamicCastAliasRefToThrowable(JNIEnv *, jobject , jobject might_actually_be_throwable) {
285:   alias_ref<jobject> might_actually_be_throwable_alias_ref { might_actually_be_throwable };
286:   // If the next line fails, it will throw an exception.
287:   alias_ref<jthrowable> throwable_alias_ref = dynamic_ref_cast<jthrowable>(might_actually_be_throwable_alias_ref);
288:   return isSameObject(throwable_alias_ref.get(), might_actually_be_throwable_alias_ref.get());
289: }
290: 
291: jboolean StaticCastLocalRefToString(JNIEnv *, jobject , jobject string_as_object) {
292:   local_ref<jobject> string_as_object_local_ref = adopt_local(string_as_object);
293:   local_ref<jstring> string_local_ref = static_ref_cast<jstring>(string_as_object_local_ref);
294:   return isSameObject(string_local_ref.get(), string_as_object_local_ref.get());
295: }
296: 
297: jboolean DynamicCastLocalRefToString(JNIEnv *, jobject , jobject might_actually_be_string) {
298:   local_ref<jobject> might_actually_be_string_local_ref = adopt_local(might_actually_be_string);
299:   // If the next line fails, it will throw an exception.
300:   local_ref<jstring> string_local_ref = dynamic_ref_cast<jstring>(might_actually_be_string_local_ref);
301:   return isSameObject(string_local_ref.get(), might_actually_be_string_local_ref.get());
302: }
303: 
304: jboolean StaticCastGlobalRefToString(JNIEnv *, jobject , jobject string_as_object) {
305:   global_ref<jobject> string_as_object_global_ref = make_global(string_as_object);
306:   global_ref<jstring> string_global_ref = static_ref_cast<jstring>(string_as_object_global_ref);
307:   return isSameObject(string_global_ref.get(), string_as_object_global_ref.get());
308: }
309: 
310: jboolean DynamicCastGlobalRefToString(JNIEnv *, jobject , jobject might_actually_be_string) {
311:   global_ref<jobject> might_actually_be_string_global_ref = make_global(might_actually_be_string);
312:   // If the next line fails, it will throw an exception.
313:   global_ref<jstring> string_global_ref = dynamic_ref_cast<jstring>(might_actually_be_string_global_ref);
314:   return isSameObject(string_global_ref.get(), might_actually_be_string_global_ref.get());
315: }
316: 
317: template<typename... Args>
318: static void Use(Args&&... args) {}
319: 
320: jboolean TestWeakRefs(JNIEnv*, jobject self) {
````
- EN: Implements callable logic such as `TestIsAssignableFrom`, `TestIsInstanceOf`, `TestIsSameObject`, `TestGetSuperclass`.
- CN: 实现可调用逻辑，例如 `TestIsAssignableFrom`, `TestIsInstanceOf`, `TestIsSameObject`, `TestGetSuperclass`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 321-400
````cpp
321:   using facebook::jni::internal::g_reference_stats;
322: 
323:   g_reference_stats.reset();
324:   {
325:     // Wrapping existing local that should be deleted (locals = 1)
326:     auto local = adopt_local(self);
327:     // Make new local (locals = 2)
328:     auto local2 = make_local(local);
329:     // Make weak (weaks = 1)
330:     auto weak = make_weak(local);
331:     // Make global (globals = 1)
332:     auto global = weak.lockGlobal();
333:     // No new refs
334:     auto banana = std::move(weak);
335:     // No new refs
336:     auto& binini = banana;
337:     // Create a global of the local (keeping the local) (globals = 2)
338:     auto dubglob = make_global(local);
339:     // Create a weak (weaks = 2)
340:     auto dupweak = make_weak(local);
341:     // No new refs
342:     swap(local, local2);
343: 
344:     Use(binini);
345:   }
346: 
347:   FBJNI_LOGE("locals: %d", g_reference_stats.locals_deleted.load());
348:   FBJNI_LOGE("globals: %d", g_reference_stats.globals_deleted.load());
349:   FBJNI_LOGE("weaks: %d", g_reference_stats.weaks_deleted.load());
350: 
351:   return (g_reference_stats.locals_deleted == 2 &&
352:           g_reference_stats.globals_deleted == 2 &&
353:           g_reference_stats.weaks_deleted == 2)? JNI_TRUE : JNI_FALSE;
354: }
355: 
356: jboolean TestAlias(JNIEnv* env, jobject self) {
357:   auto ref = alias_ref<jobject>{self};
358:   return ref->isInstanceOf(findClassLocal("java/lang/Object"));
359: }
360: 
361: jboolean testAliasRefConversions(JNIEnv*, jobject self) {
362:   auto aLocalString = make_jstring("foo");
363:   alias_ref<jstring> aString = aLocalString;
364:   alias_ref<jobject> anObject = aLocalString;
365:   anObject = (jstring) nullptr;
366:   anObject = aString;
367:   // aString = anObject; // Shouldn't compile
368: 
369:   return isSameObject(aString, anObject)? JNI_TRUE : JNI_FALSE;
370: }
371: 
372: void TestAutoAliasRefReturningVoid(facebook::jni::alias_ref<jobject> self) {
373:   // If this compiles, it succeeds.
374: }
375: 
376: jboolean testNullJString(JNIEnv*, jobject) {
377:   auto aNullJString = make_jstring(nullptr);
378:   EXPECT(aNullJString.get() == (jstring) nullptr);
379:   return JNI_TRUE;
380: }
381: 
382: jboolean testSwap(JNIEnv*, jobject self, jobject other) {
383:   auto selfAlias = wrap_alias(self);
384:   auto otherAlias = wrap_alias(other);
385: 
386:   swap(selfAlias, otherAlias);
387:   EXPECT(self == otherAlias);
388:   EXPECT(other == selfAlias);
389:   EXPECT(self != selfAlias);
390:   EXPECT(other != otherAlias);
391: 
392:   auto selfLocal = make_local(self);
393:   auto otherLocal = make_local(other);
394:   swap(selfLocal, otherLocal);
395:   EXPECT(self == otherLocal);
396:   EXPECT(other == selfLocal);
397:   EXPECT(self != selfLocal);
398:   EXPECT(other != otherLocal);
399: 
400:   auto selfGlobal = make_global(self);
````
- EN: Implements callable logic such as `TestAlias`, `testAliasRefConversions`, `TestAutoAliasRefReturningVoid`, `testNullJString`.
- CN: 实现可调用逻辑，例如 `TestAlias`, `testAliasRefConversions`, `TestAutoAliasRefReturningVoid`, `testNullJString`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 401-480
````cpp
401:   auto otherGlobal = make_global(other);
402:   swap(selfGlobal, otherGlobal);
403:   EXPECT(self == otherGlobal);
404:   EXPECT(other == selfGlobal);
405:   EXPECT(self != selfGlobal);
406:   EXPECT(other != otherGlobal);
407: 
408:   auto selfWeak = make_weak(self);
409:   auto otherWeak = make_weak(other);
410:   swap(selfWeak, otherWeak);
411:   auto selfLockedWeak = selfWeak.lockLocal();
412:   auto otherLockedWeak = otherWeak.lockLocal();
413:   EXPECT(self == otherLockedWeak);
414:   EXPECT(other == selfLockedWeak);
415:   EXPECT(self != selfLockedWeak);
416:   EXPECT(other != otherLockedWeak);
417: 
418:   return JNI_TRUE;
419: }
420: 
421: jboolean testEqualOperator(JNIEnv*, jobject self, jobject other) {
422:   auto selfAlias = wrap_alias(self);
423:   auto otherAlias = wrap_alias(other);
424:   auto selfLocal = adopt_local(self);
425:   auto otherLocal = adopt_local(other);
426:   auto selfGlobal = make_global(self);
427:   auto otherGlobal = make_global(other);
428:   auto selfWeak = make_weak(self);
429:   auto otherWeak = make_weak(other);
430:   auto selfLockedWeak = selfWeak.lockLocal();
431:   auto otherLockedWeak = otherWeak.lockLocal();
432: 
433:   EXPECT(self == selfAlias);
434:   EXPECT(selfAlias == selfLocal);
435:   EXPECT(selfLocal == selfGlobal);
436:   EXPECT(selfGlobal == selfLockedWeak);
437:   EXPECT(self != other);
438:   EXPECT(self != otherAlias);
439:   EXPECT(self != otherLocal);
440:   EXPECT(self != otherGlobal);
441:   EXPECT(self != otherLockedWeak);
442:   EXPECT(selfAlias != nullptr);
443:   EXPECT(!(selfAlias == nullptr));
444:   EXPECT(nullptr != selfLocal);
445:   EXPECT(!(nullptr == selfGlobal));
446: 
447:   return JNI_TRUE;
448: }
449: 
450: jboolean testReleaseAlias(JNIEnv*, jobject self) {
451:   auto local = adopt_local(self);
452:   auto alias = local.releaseAlias();
453: 
454:   EXPECT(typeid(alias) == typeid(alias_ref<jobject>));
455:   EXPECT(isSameObject(self, alias.get()));
456: 
457:   return JNI_TRUE;
458: }
459: 
460: jboolean testLockingWeakReferences(JNIEnv*, jobject self) {
461:   auto weak = make_weak(self);
462:   auto local = weak.lockLocal();
463:   auto global = weak.lockGlobal();
464: 
465:   EXPECT(typeid(local) == typeid(local_ref<jobject>));
466:   EXPECT(typeid(global) == typeid(global_ref<jobject>));
467:   EXPECT(self == local);
468:   EXPECT(self == global);
469: 
470:   return JNI_TRUE;
471: }
472: 
473: jboolean TestFieldAccess(alias_ref<jobject> self, const std::string& field_name,
474:                          jint oldval, jint newval) {
475:   auto cls = self->getClass();
476:   auto fld = cls->getField<jint>(field_name.c_str());
477:   auto method = cls->getMethod<jint(jdouble)>("bar");
478: 
479:   if (method(self.get(), 17) != 42) {
480:     return JNI_FALSE;
````
- EN: Implements callable logic such as `testEqualOperator`, `testReleaseAlias`, `testLockingWeakReferences`, `TestFieldAccess`.
- CN: 实现可调用逻辑，例如 `testEqualOperator`, `testReleaseAlias`, `testLockingWeakReferences`, `TestFieldAccess`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 481-560
````cpp
481:   }
482: 
483:   if (method(self, 17) != 42) {
484:     return JNI_FALSE;
485:   }
486: 
487:   if (self->getFieldValue(fld) != oldval) {
488:     return JNI_FALSE;
489:   }
490: 
491:   self->setFieldValue(fld, newval);
492: 
493:   return JNI_TRUE;
494: }
495: 
496: jboolean TestStringFieldAccess(
497:     JNIEnv* env,
498:     jobject self,
499:     jstring field_name,
500:     jstring oldval,
501:     jstring newval) {
502:   auto me = adopt_local(self);
503:   auto cls = me->getClass();
504:   auto fld = cls->getField<jstring>(ToString(env, field_name).c_str());
505:   auto oldvalStr = adopt_local(oldval)->toStdString();
506: 
507:   auto curvalRef = me->getFieldValue(fld);
508:   if (curvalRef->toStdString() != oldvalStr) {
509:     return JNI_FALSE;
510:   }
511: 
512:   const alias_ref<jobject> cme = me;
513:   if (cme->getFieldValue(fld)->toStdString() != oldvalStr) {
514:     return JNI_FALSE;
515:   }
516: 
517:   me->setFieldValue(fld, newval);
518: 
519:   return JNI_TRUE;
520: }
521: 
522: jboolean TestReferenceFieldAccess(
523:     alias_ref<jobject> self,
524:     std::string const& field_name,
525:     jobject oldval,
526:     jobject newval,
527:     jboolean useWrapper) {
528:   auto cls = self->getClass();
529:   auto rawfld = cls->getField<jobject>(field_name.c_str(), TestThing::kJavaDescriptor);
530: 
531:   if (self->getFieldValue(rawfld) != oldval) {
532:     return JNI_FALSE;
533:   }
534: 
535:   alias_ref<jobject> const cself = self;
536:   if (cself->getFieldValue(rawfld) != oldval) {
537:     return JNI_FALSE;
538:   }
539: 
540:   if (useWrapper) {
541:     auto newvalRef = adopt_local(static_cast<TestThing::javaobject>(newval));
542:     auto fld = cls->getField<TestThing::javaobject>(field_name.c_str());
543:     self->setFieldValue<TestThing::javaobject>(fld, newvalRef);
544:   } else {
545:     self->setFieldValue(rawfld, newval);
546:   }
547: 
548:   return JNI_TRUE;
549: }
550: 
551: jboolean TestStaticFieldAccess(
552:     JNIEnv* env,
553:     jobject self,
554:     jstring field_name,
555:     jint oldval,
556:     jint newval) {
557:   auto me = adopt_local(self);
558:   auto cls = me->getClass();
559:   auto fld = cls->getStaticField<jint>(ToString(env, field_name).c_str());
560: 
````
- EN: Implements callable logic such as `TestStringFieldAccess`, `TestReferenceFieldAccess`, `TestStaticFieldAccess`.
- CN: 实现可调用逻辑，例如 `TestStringFieldAccess`, `TestReferenceFieldAccess`, `TestStaticFieldAccess`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 561-640
````cpp
561:   if (cls->getStaticFieldValue(fld) != oldval) {
562:     return JNI_FALSE;
563:   }
564:   cls->setStaticFieldValue(fld, newval);
565:   return JNI_TRUE;
566: }
567: 
568: jboolean TestStaticStringFieldAccess(
569:     JNIEnv* env,
570:     jobject self,
571:     jstring field_name,
572:     jstring oldval,
573:     jstring newval) {
574:   auto me = adopt_local(self);
575:   auto cls = me->getClass();
576:   auto fld = cls->getStaticField<jstring>(ToString(env, field_name).c_str());
577: 
578:   auto curvalRef = cls->getStaticFieldValue(fld);
579:   if (curvalRef->toStdString() != adopt_local(oldval)->toStdString()) {
580:     return JNI_FALSE;
581:   }
582:   cls->setStaticFieldValue(fld, newval);
583:   return JNI_TRUE;
584: }
585: 
586: jboolean TestStaticReferenceFieldAccess(
587:     alias_ref<jobject> self,
588:     std::string const& field_name,
589:     jobject oldval,
590:     jobject newval,
591:     jboolean useWrapper) {
592:   auto cls = self->getClass();
593:   auto rawfld = cls->getStaticField<jobject>(field_name.c_str(), TestThing::kJavaDescriptor);
594: 
595:   auto curvalRef = cls->getStaticFieldValue(rawfld);
596:   if (curvalRef != oldval) {
597:     return JNI_FALSE;
598:   }
599: 
600:   if (useWrapper) {
601:     auto newvalRef = adopt_local(static_cast<TestThing::javaobject>(newval));
602:     auto fld = cls->getStaticField<TestThing::javaobject>(field_name.c_str());
603:     cls->setStaticFieldValue<TestThing::javaobject>(fld, newvalRef);
604:   } else {
605:     cls->setStaticFieldValue(rawfld, newval);
606:   }
607: 
608:   return JNI_TRUE;
609: }
610: 
611: jboolean TestNonVirtualMethod(JNIEnv* env, jobject self, jboolean s) {
612:   auto me = adopt_local(self);
613:   if (!me) {
614:     return JNI_FALSE;
615:   }
616: 
617:   auto cls = me->getClass();
618:   if (!cls) {
619:     return JNI_FALSE;
620:   }
621:   auto method = cls->getNonvirtualMethod<jboolean(jboolean)>("nonVirtualMethod");
622: 
623:   jclass jcls = env->FindClass("com/facebook/jni/FBJniTests");
624: 
625:   return method(self, jcls, s);
626: }
627: 
628: jtypeArray<jstring>
629: TestArrayCreation(JNIEnv* env, jobject self, jstring s0, jstring s1, jstring s2) {
630:   auto array = JArrayClass<jstring>::newArray(3);
631:   array->setElement(0, s0);
632:   array->setElement(1, s1);
633:   array->setElement(2, s2);
634:   return static_cast<jtypeArray<jstring>>(array.release());
635: }
636: 
637: jtypeArray<jtypeArray<jstring>>
638: TestMultidimensionalObjectArray(JNIEnv* env, jobject self, jstring s0, jstring s1, jstring s2) {
639:   auto array = JArrayClass<jtypeArray<jstring>>::newArray(2);
640:   auto row = JArrayClass<jstring>::newArray(2);
````
- EN: Implements callable logic such as `TestStaticStringFieldAccess`, `TestStaticReferenceFieldAccess`, `TestNonVirtualMethod`, `TestArrayCreation`.
- CN: 实现可调用逻辑，例如 `TestStaticStringFieldAccess`, `TestStaticReferenceFieldAccess`, `TestNonVirtualMethod`, `TestArrayCreation`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 641-720
````cpp
641:   row->setElement(0, s0);
642:   row->setElement(1, s1);
643:   (*array)[0] = row;
644:   row = JArrayClass<jstring>::newArray(1);
645:   row->setElement(0, s2);
646:   (*array)[1] = row;
647:   return array.release();
648: }
649: 
650: jtypeArray<jintArray>
651: TestMultidimensionalPrimitiveArray(JNIEnv* env, jobject self, jint i0, jint i1, jint i2) {
652:   auto array = JArrayClass<jintArray>::newArray(2);
653:   auto row = JArrayInt::newArray(2);
654:   row->setRegion(0, 1, &i0);
655:   row->setRegion(1, 1, &i1);
656:   (*array)[0] = row;
657:   row = JArrayInt::newArray(1);
658:   row->setRegion(0, 1, &i2);
659:   (*array)[1] = row;
660:   return array.release();
661: }
662: 
663: jstring TestBuildStringArray(JNIEnv* env, jobject self, jtypeArray<jstring> input) {
664:   auto me = adopt_local(self);
665:   auto cls = me->getClass();
666:   auto method = cls->getMethod<jstring(jtypeArray<jstring>)>("captureStringArray");
667: 
668:   auto niceInput = adopt_local_array<jstring>(input);
669:   auto length = niceInput->size();
670:   auto inputCopy = JArrayClass<jstring>::newArray(length);
671:   for (size_t idx = 0; idx < length; idx++) {
672:     switch (idx % 3) {
673:     case 0: {
674:       // Verify that assignment from a T works.
675:       jstring value = (jstring)env->GetObjectArrayElement(input, idx);
676:       (*inputCopy)[idx] = value; // Assignment from actual type.
677:       env->DeleteLocalRef(value);
678:       break;
679:     }
680:     case 1: {
681:       // Verify that direct assignment from an ElementProxy works.
682:       (*inputCopy)[idx] = (*niceInput)[idx];
683:       break;
684:     }
685:     case 2:
686:     default: {
687:       // Verify that assignment from a smart reference works.
688:       auto smartRef = adopt_local((*niceInput)[idx]);
689:       (*inputCopy)[idx] = smartRef;
690:       break;
691:     }
692:     }
693:   }
694: 
695:   return method(self, inputCopy.get()).release();
696: }
697: 
698: template <typename F, typename... Args>
699: void tryResolveMethodWithCxxTypes(std::string sig, alias_ref<jobject> me, std::string methodName, Args... args) {
700:   auto cls = me->getClass();
701:   auto method = cls->getMethod<F>(methodName.c_str());
702:   if (!method) throw std::runtime_error("method lookup failed with signature=" + sig);
703:   try {
704:     method(me, args...);
705:   } catch (std::exception&) {
706:     throw std::runtime_error("calling method failed with signature=" + sig);
707:   }
708: 
709:   auto nonVirtualMethod = cls->getNonvirtualMethod<F>(methodName.c_str());
710:   if (!nonVirtualMethod) throw std::runtime_error("method lookup failed with signature=" + sig);
711:   try {
712:     nonVirtualMethod(me, cls.get(), args...);
713:   } catch (std::exception&) {
714:     throw std::runtime_error("calling method failed with signature=" + sig);
715:   }
716: 
717:   auto staticMethod = cls->getStaticMethod<F>((methodName + "Static").c_str());
718:   if (!staticMethod) throw std::runtime_error("static method lookup failed with signature=" + sig);
719:   try {
720:     staticMethod(cls, args...);
````
- EN: Implements callable logic such as `TestMultidimensionalPrimitiveArray`, `TestBuildStringArray`, `tryResolveMethodWithCxxTypes`.
- CN: 实现可调用逻辑，例如 `TestMultidimensionalPrimitiveArray`, `TestBuildStringArray`, `tryResolveMethodWithCxxTypes`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 721-800
````cpp
721:   } catch (std::exception&) {
722:     throw std::runtime_error("calling static method failed with signature=" + sig);
723:   }
724: }
725: 
726: // Simple utility to give us a good error message.
727: #define runTest(sig, ...) \
728:   tryResolveMethodWithCxxTypes<sig>(#sig, self, method, __VA_ARGS__);
729: 
730: void TestMethodResolutionWithCxxTypes(alias_ref<jobject> self, alias_ref<jstring> jmethod, alias_ref<jstring> str, jlong v) {
731:   auto method = jmethod->toStdString();
732:   runTest(jobject(jstring, jlong), str.get(), v);
733:   runTest(local_ref<jobject>(jstring, jlong), str.get(), v);
734: 
735:   runTest(jobject(local_ref<jstring>, jlong), make_local(str), v);
736:   runTest(jobject(alias_ref<jstring>, jlong), str, v);
737: 
738:   runTest(jobject(alias_ref<jstring>, int64_t), str, (int64_t)v);
739:   runTest(jobject(alias_ref<jstring>, long long), str, (long long)v);
740: 
741:   runTest(jobject(const char*, int64_t), str->toStdString().c_str(), (int64_t)v);
742: 
743:   method = jmethod->toStdString() + "Void";
744:   runTest(void(jstring, int64_t), str.get(), v);
745: 
746:   method = jmethod->toStdString() + "Int";
747:   runTest(jint(jstring, int64_t), str.get(), v);
748: }
749: 
750: #undef runTest
751: 
752: void TestHandleJavaCustomException(JNIEnv* env, jobject self) {
753:   auto me = adopt_local(self);
754:   auto cls = me->getClass();
755:   auto method = cls->getMethod<void()>("customExceptionThrower");
756: 
757:   method(self);
758: }
759: 
760: void TestHandleNullExceptionMessage(JNIEnv* env, jobject self) {
761:   auto me = adopt_local(self);
762:   auto cls = me->getClass();
763:   auto method = cls->getMethod<void()>("nullMessageThrower");
764: 
765:   try {
766:     method(self);
767:   } catch (const std::exception& ex) {
768:     ex.what();
769:   }
770: }
771: 
772: void TestHandleNestedException(JNIEnv* env, jobject self) {
773:   auto me = adopt_local(self);
774:   auto cls = me->getClass();
775:   auto method = cls->getMethod<void()>("customExceptionThrower");
776: 
777:   try {
778:     try {
779:       method(self);
780:     } catch (...) {
781:       std::throw_with_nested(std::runtime_error("middle"));
782:     }
783:   } catch (...) {
784:     std::throw_with_nested(std::out_of_range("outer"));
785:   }
786: }
787: 
788: void TestHandleNoRttiException(JNIEnv* env, jobject self) {
789:   nortti::throwException();
790: }
791: 
792: jstring TestCopyConstructor(JNIEnv* env, jobject self) {
793:   auto me = adopt_local(self);
794:   auto cls = me->getClass();
795:   auto method = cls->getMethod<void()>("customExceptionThrower");
796: 
797:   try {
798:     method(self);
799:     return env->NewStringUTF("method did not throw");
800:   } catch (JniException ex) { // No & -- we're intentionally invoking the copy constructor.
````
- EN: Implements callable logic such as `TestMethodResolutionWithCxxTypes`, `TestHandleJavaCustomException`, `TestHandleNullExceptionMessage`, `TestHandleNestedException`.
- CN: 实现可调用逻辑，例如 `TestMethodResolutionWithCxxTypes`, `TestHandleJavaCustomException`, `TestHandleNullExceptionMessage`, `TestHandleNestedException`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 801-880
````cpp
801:     return env->NewStringUTF(ex.what());
802:   }
803: }
804: 
805: jstring TestMoveConstructorWithEmptyWhat(JNIEnv* env, jobject self) {
806:   auto me = adopt_local(self);
807:   auto cls = me->getClass();
808:   auto method = cls->getMethod<void()>("customExceptionThrower");
809: 
810:   try {
811:     method(self);
812:     return env->NewStringUTF("method did not throw");
813:   } catch (JniException& ex) {
814:     auto replacement = JniException(std::move(ex));
815:     return env->NewStringUTF(replacement.what());
816:   }
817: }
818: 
819: jstring TestMoveConstructorWithPopulatedWhat(JNIEnv* env, jobject self) {
820:   auto me = adopt_local(self);
821:   auto cls = me->getClass();
822:   auto method = cls->getMethod<void()>("customExceptionThrower");
823: 
824:   try {
825:     method(self);
826:     return env->NewStringUTF("method did not throw");
827:   } catch (JniException& ex) {
828:     ex.what();
829:     auto replacement = JniException(std::move(ex));
830:     return env->NewStringUTF(replacement.what());
831:   }
832: }
833: 
834: void TestHandleCppRuntimeError(JNIEnv* env, jobject self, jstring message) {
835:   throw std::runtime_error(ToString(env, message));
836: }
837: 
838: void TestHandleCppIOBaseFailure(JNIEnv* env, jobject self) {
839:   throw std::ios_base::failure("A C++ IO base failure.");
840: }
841: 
842: void TestHandleCppSystemError(JNIEnv* env, jobject self) {
843:   // Throw a sample std::system_error
844:   throw std::system_error(EFAULT, std::system_category());
845: }
846: 
847: void TestInterDsoExceptionHandlingA(JNIEnv* env, jobject self) {
848:   inter_dso_exception_test_2a();
849: }
850: 
851: jboolean TestInterDsoExceptionHandlingB(JNIEnv* env, jobject self) {
852:   return inter_dso_exception_test_2b();
853: }
854: 
855: struct NonStdException /* has no base class */ {};
856: 
857: void TestHandleNonStdException(JNIEnv* env, jobject self) {
858:   throw NonStdException();
859: }
860: 
861: void TestHandleCppIntThrow(JNIEnv* env, jobject self) {
862:   throw 42;
863: }
864: 
865: void TestHandleCppCharPointerThrow(JNIEnv* env, jobject self) {
866:   throw "Some random message";
867: }
868: 
869: void TestThrowJavaExceptionByName(JNIEnv* env, jobject self) {
870:   throwNewJavaException("java/lang/IllegalArgumentException", "bad news: %s", "it didn't work");
871: }
872: 
873: jint TestJThread(JNIEnv* env, jobject self) {
874:   jint i = -1;
875:   auto thread = JThread::create([&] {
876:       i = 0;
877:       std::this_thread::sleep_for(std::chrono::milliseconds(20));
878:       i = 1;
879:     });
880:   thread->start();
````
- EN: Declares or extends types including `NonStdException`.
- CN: 声明或扩展类型，包括 `NonStdException`。
- EN: Implements callable logic such as `TestMoveConstructorWithEmptyWhat`, `TestMoveConstructorWithPopulatedWhat`, `TestHandleCppRuntimeError`, `TestHandleCppIOBaseFailure`.
- CN: 实现可调用逻辑，例如 `TestMoveConstructorWithEmptyWhat`, `TestMoveConstructorWithPopulatedWhat`, `TestHandleCppRuntimeError`, `TestHandleCppIOBaseFailure`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 881-960
````cpp
881:   thread->join();
882:   return i;
883: }
884: 
885: // Global for simpleWorker tests. Relies on thread sync points (constructor, join) for "locking".
886: jint gWorkerValue;
887: 
888: void simpleWorker(jobject grefSelf, jdouble input) {
889:   auto attachGuard = ThreadScope(); // This tests the move constructor.
890:   auto self = adopt_global(grefSelf);
891:   // Claw up from the object to avoid classloader issues.
892:   auto barMethod = self->getClass()->getMethod<jint(jdouble)>("bar");
893:   gWorkerValue = barMethod(self.get(), input);
894: }
895: 
896: void nestedSimpleWorker(jobject grefSelf, jdouble input) {
897:   ThreadScope attachGuard; // More efficient version of guard; no move constructor required.
898:   simpleWorker(grefSelf, input);
899: }
900: 
901: jint TestThreadScopeGuard(JNIEnv* env, jobject self, jdouble input) {
902:   // Turn self into a global reference before passing it to a working thread.
903:   auto grefSelf = make_global(adopt_local(self));
904:   auto childThread = std::thread(simpleWorker, grefSelf.release(), input);
905:   childThread.join();
906:   return gWorkerValue;
907: }
908: 
909: jint TestNestedThreadScopeGuard(JNIEnv* env, jobject self, jdouble input) {
910:   // Turn self into a global reference before passing it to a working thread.
911:   auto grefSelf = make_global(adopt_local(self));
912:   auto childThread = std::thread(nestedSimpleWorker, grefSelf.release(), input);
913:   childThread.join();
914:   return gWorkerValue;
915: }
916: 
917: void classLoadWorker() {
918:   gWorkerValue = 0;
919:   try {
920:     // This should fail because we aren't attached.
921:     Callbacks::javaClassLocal();
922:     gWorkerValue = -1;
923:     return;
924:   } catch (std::exception& e) {
925:     // ignored
926:   }
927:   try {
928:     ThreadScope::WithClassLoader([&] {
929:       // This should now succeed.
930:       Callbacks::javaClassLocal();
931:       gWorkerValue = 1;
932:     });
933:   } catch (std::exception& e) {
934:     gWorkerValue = -2;
935:     // Catch this and log it so that we get a test failure instead of a crash.
936:     FBJNI_LOGE("%s", e.what());
937:   }
938: }
939: 
940: jint TestClassLoadInWorker(JNIEnv* env, jobject self) {
941:   std::thread t(classLoadWorker);
942:   t.join();
943:   return gWorkerValue;
944: }
945: 
946: jint TestClassLoadWorkerFastPath(JNIEnv* env, jobject self) {
947:   jint i = 0;
948:   ThreadScope::WithClassLoader([&] {
949:     // Execute on the fast path
950:     Callbacks::javaClassLocal();
951:     i += 1;
952:   });
953: 
954:   std::thread t([&] {
955:     ThreadScope::WithClassLoader([&] {
956:       // Execute on the slow path
957:       Callbacks::javaClassLocal();
958:       i += 1;
959:     });
960:   });
````
- EN: Implements callable logic such as `simpleWorker`, `nestedSimpleWorker`, `TestThreadScopeGuard`, `TestNestedThreadScopeGuard`.
- CN: 实现可调用逻辑，例如 `simpleWorker`, `nestedSimpleWorker`, `TestThreadScopeGuard`, `TestNestedThreadScopeGuard`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 961-1040
````cpp
 961:   t.join();
 962: 
 963:   std::thread t2([&] {
 964:     ThreadScope scope;
 965:     ThreadScope::WithClassLoader([&] {
 966:       // Execute on the slow path even though thread is already attached.
 967:       Callbacks::javaClassLocal();
 968:       i += 1;
 969:     });
 970:   });
 971:   t2.join();
 972: 
 973:   return i;
 974: }
 975: 
 976: void testNewObject(JNIEnv*, jobject self) {
 977:   // This is a compilation only test, verifies that all the types work out.
 978:   auto cls = findClassLocal("java/lang/String");
 979:   auto ctr = cls->getConstructor<jstring()>();
 980:   local_ref<jstring> obj = cls->newObject(ctr);
 981:   auto str = obj->toStdString();
 982:   Use(str);
 983: }
 984: 
 985: template<typename T>
 986: static jboolean copyAndVerify(T& orig) {
 987:   T copy{orig};
 988:   EXPECT(orig == copy);
 989: 
 990:   return JNI_TRUE;
 991: }
 992: 
 993: template<typename T>
 994: static jboolean assignAndVerify(T& orig) {
 995:   T copy{};
 996:   copy = orig;
 997:   EXPECT(orig == copy);
 998: 
 999:   return JNI_TRUE;
1000: }
1001: 
1002: jboolean testNullReferences(JNIEnv*, jobject) {
1003:   jobject nullobject = nullptr;
1004: 
1005:   auto local = local_ref<jobject>{};
1006:   EXPECT(!local);
1007: 
1008:   auto localWrap = adopt_local(nullobject);
1009:   EXPECT(!localWrap);
1010: 
1011:   auto localMake = make_local(local);
1012:   EXPECT(!localMake);
1013:   EXPECT_SAME(local, localWrap, localMake);
1014: 
1015:   auto global = global_ref<jobject>{};
1016:   EXPECT(!global);
1017: 
1018:   auto globalWrap = adopt_global(nullobject);
1019:   EXPECT(!globalWrap);
1020: 
1021:   auto globalMake = make_global(global);
1022:   EXPECT(!globalMake);
1023:   EXPECT_SAME(global, globalWrap, globalMake);
1024: 
1025:   weak_ref<jobject> weak_global = weak_ref<jobject>{};
1026:   EXPECT(!weak_global.lockLocal());
1027: 
1028:   weak_ref<jobject> weak_globalWrap = adopt_weak_global(nullobject);
1029:   EXPECT(!weak_globalWrap.lockLocal());
1030:   EXPECT(weak_global.lockLocal() == weak_globalWrap.lockGlobal());
1031:   EXPECT(!make_local(adopt_local(nullobject)));
1032:   EXPECT(!make_global(nullobject));
1033: 
1034:   return JNI_TRUE;
1035: }
1036: 
1037: jboolean testCreatingReferences(JNIEnv*, jobject self) {
1038:   auto a = wrap_alias(self);
1039:   auto l = adopt_local(self);
1040:   auto g = make_global(l);
````
- EN: Implements callable logic such as `testNewObject`, `copyAndVerify`, `assignAndVerify`, `testNullReferences`.
- CN: 实现可调用逻辑，例如 `testNewObject`, `copyAndVerify`, `assignAndVerify`, `testNullReferences`。
- EN: Includes validation, assertions, or defensive checks.
- CN: 包含校验、断言或防御性检查。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1041-1120
````cpp
1041:   auto w = make_weak(l);
1042: 
1043:   EXPECT(a == l && a == g && a == w.lockLocal());
1044: 
1045:   auto lp = make_local(self);
1046:   auto la = make_local(a);
1047:   auto ll = make_local(l);
1048:   auto lg = make_local(g);
1049: 
1050:   EXPECT(a == lp && a == la && a == ll && a == lg);
1051: 
1052:   auto gp = make_global(self);
1053:   auto ga = make_global(a);
1054:   auto gl = make_global(l);
1055:   auto gg = make_global(g);
1056: 
1057:   EXPECT(a == gp && a == ga && a == gl && a == gg);
1058: 
1059:   return JNI_TRUE;
1060: }
1061: 
1062: jboolean testAssignmentAndCopyConstructors(JNIEnv*, jobject self) {
1063:   using facebook::jni::internal::g_reference_stats;
1064: 
1065:   g_reference_stats.reset();
1066:   {
1067:     // Wrapping existing local that should be deleted (locals = 1)
1068:     auto local = adopt_local(self);
1069:     // Copy constructor (locals = 2)
1070:     EXPECT(copyAndVerify(local));
1071: 
1072:     // Assignment (locals = 3)
1073:     EXPECT(assignAndVerify(local));
1074: 
1075:     // Creating a new global (globals = 1)
1076:     auto global = make_global(local);
1077:     // Copy constructor (globals = 2)
1078:     EXPECT(copyAndVerify(global));
1079: 
1080:     // Assignment (globals = 3)
1081:     EXPECT(assignAndVerify(global));
1082: 
1083:     // Creating a new weak (weaks = 1)
1084:     auto weak = make_weak(local);
1085:     // Copy constructor (weaks = 2, globals = 5)
1086:     weak_ref<jobject> weakCopy{weak};
1087:     EXPECT(weak.lockGlobal() == weakCopy.lockGlobal());
1088: 
1089:     // Assignment (weaks = 3, globals = 7)
1090:     weakCopy = weak;
1091:     EXPECT(weak.lockGlobal() == weakCopy.lockGlobal());
1092: 
1093:     auto alias = alias_ref<jobject>{local};
1094:     alias_ref<jobject> aliasCopy{alias};
1095:     EXPECT(alias == aliasCopy);
1096: 
1097:     aliasCopy = alias;
1098:     EXPECT(alias == aliasCopy);
1099: 
1100:     alias = self;
1101:     alias = global;
1102:     // alias = weak; // Should not compile
1103:   }
1104: 
1105:   FBJNI_LOGE("locals: %d", g_reference_stats.locals_deleted.load());
1106:   FBJNI_LOGE("globals: %d", g_reference_stats.globals_deleted.load());
1107:   FBJNI_LOGE("weaks: %d", g_reference_stats.weaks_deleted.load());
1108: 
1109:   EXPECT(g_reference_stats.locals_deleted == 3 &&
1110:         g_reference_stats.globals_deleted == 7 &&
1111:         g_reference_stats.weaks_deleted == 3);
1112: 
1113:   return JNI_TRUE;
1114: }
1115: 
1116: template<template <typename> class RefType, typename T>
1117: static jboolean copyAndVerifyCross(RefType<T>& orig) {
1118:   RefType<ReprType<T>> reprCopy{orig};
1119:   RefType<JniType<T>> jniCopy{orig};
1120:   EXPECT(orig == reprCopy);
````
- EN: Declares or extends types including `RefType`.
- CN: 声明或扩展类型，包括 `RefType`。
- EN: Implements callable logic such as `testAssignmentAndCopyConstructors`, `copyAndVerifyCross`.
- CN: 实现可调用逻辑，例如 `testAssignmentAndCopyConstructors`, `copyAndVerifyCross`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1121-1200
````cpp
1121:   EXPECT(orig == jniCopy);
1122:   return JNI_TRUE;
1123: }
1124: 
1125: template<template <typename> class RefType, typename T>
1126: static jboolean assignAndVerifyCross(RefType<T>& orig) {
1127:   RefType<ReprType<T>> reprCopy{};
1128:   reprCopy = orig;
1129:   RefType<JniType<T>> jniCopy{};
1130:   jniCopy = orig;
1131:   EXPECT(orig == reprCopy);
1132:   EXPECT(orig == jniCopy);
1133:   return JNI_TRUE;
1134: }
1135: 
1136: template<template <typename> class RefType, typename T>
1137: static jboolean verifyMakeCross(RefType<T>& orig) {
1138:   RefType<ReprType<T>> copy{orig};
1139:   {
1140:     local_ref<T> local = make_local(copy);
1141:     global_ref<T> global = make_global(copy);
1142:     weak_ref<T> weak = make_weak(copy);
1143:   }
1144: 
1145:   {
1146:     local_ref<ReprType<T>> local = make_local(copy);
1147:     global_ref<ReprType<T>> global = make_global(copy);
1148:     weak_ref<ReprType<T>> weak = make_weak(copy);
1149:   }
1150: 
1151:   {
1152:     local_ref<T> local = make_local(orig);
1153:     global_ref<T> global = make_global(orig);
1154:     weak_ref<T> weak = make_weak(orig);
1155:   }
1156: 
1157:   {
1158:     local_ref<ReprType<T>> local = make_local(orig);
1159:     global_ref<ReprType<T>> global = make_global(orig);
1160:     weak_ref<ReprType<T>> weak = make_weak(orig);
1161:   }
1162: 
1163:   return JNI_TRUE;
1164: }
1165: 
1166: 
1167: jboolean testAssignmentAndCopyCrossTypes(JNIEnv*, jobject self) {
1168:   using facebook::jni::internal::g_reference_stats;
1169: 
1170:   size_t locals = 0, globals = 0, weaks = 0;
1171:   g_reference_stats.reset();
1172: #define VERIFY_REFERENCE_STATS() do {                                   \
1173:     bool referenceStatsMatch = g_reference_stats.locals_deleted == locals && \
1174:       g_reference_stats.globals_deleted == globals &&                   \
1175:       g_reference_stats.weaks_deleted == weaks;                         \
1176:     if (!referenceStatsMatch) {                                         \
1177:       FBJNI_LOGE("locals: %d, expected: %zd", g_reference_stats.locals_deleted.load(), locals); \
1178:       FBJNI_LOGE("globals: %d, expected: %zd", g_reference_stats.globals_deleted.load(), globals); \
1179:       FBJNI_LOGE("weaks: %d, expected: %zd", g_reference_stats.weaks_deleted.load(), weaks); \
1180:     }                                                                   \
1181:     EXPECT(referenceStatsMatch);                                        \
1182:   } while (0)
1183: 
1184:   {
1185:     VERIFY_REFERENCE_STATS();
1186: 
1187:     auto local = adopt_local(self);
1188:     VERIFY_REFERENCE_STATS();
1189: 
1190:     EXPECT(copyAndVerifyCross<local_ref>(local));
1191:     locals += 2;
1192:     VERIFY_REFERENCE_STATS();
1193: 
1194:     EXPECT(assignAndVerifyCross<local_ref>(local));
1195:     locals += 2;
1196:     VERIFY_REFERENCE_STATS();
1197: 
1198:     EXPECT(verifyMakeCross<local_ref>(local));
1199:     locals += 1;
1200:     locals += 4;
````
- EN: Declares or extends types including `RefType`.
- CN: 声明或扩展类型，包括 `RefType`。
- EN: Implements callable logic such as `assignAndVerifyCross`, `verifyMakeCross`, `testAssignmentAndCopyCrossTypes`.
- CN: 实现可调用逻辑，例如 `assignAndVerifyCross`, `verifyMakeCross`, `testAssignmentAndCopyCrossTypes`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1201-1280
````cpp
1201:     globals += 4;
1202:     weaks += 4;
1203:     VERIFY_REFERENCE_STATS();
1204: 
1205:     auto global = make_global(local);
1206:     VERIFY_REFERENCE_STATS();
1207: 
1208:     EXPECT(copyAndVerifyCross<global_ref>(global));
1209:     globals += 2;
1210:     VERIFY_REFERENCE_STATS();
1211: 
1212:     EXPECT(assignAndVerifyCross<global_ref>(global));
1213:     globals += 2;
1214:     VERIFY_REFERENCE_STATS();
1215: 
1216:     auto weak = make_weak(local);
1217:     VERIFY_REFERENCE_STATS();
1218: 
1219:     weak_ref<JObject> weakCopy{weak};
1220:     VERIFY_REFERENCE_STATS();
1221: 
1222:     EXPECT(weak.lockGlobal() == weakCopy.lockGlobal());
1223:     globals += 2;
1224:     VERIFY_REFERENCE_STATS();
1225: 
1226:     weakCopy = weak;
1227:     weaks += 1;
1228:     EXPECT(weak.lockGlobal() == weakCopy.lockGlobal());
1229:     globals += 2;
1230:     VERIFY_REFERENCE_STATS();
1231: 
1232:     auto alias = alias_ref<jobject>{local};
1233:     alias_ref<JObject>{local};
1234: 
1235:     alias_ref<JObject> aliasCopy{alias};
1236:     EXPECT(alias == aliasCopy);
1237: 
1238:     aliasCopy = alias;
1239:     alias = aliasCopy;
1240:     EXPECT(alias == aliasCopy);
1241: 
1242:     alias = self;
1243:     alias = global;
1244:     // alias = weak; // Should not compile
1245: 
1246:     weaks += 1;   // `weakCopy` going out of scope
1247:     weaks += 1;   // `weak` going out of scope
1248:     globals += 1; // `global` going out of scope
1249:     locals += 1;  // `local` going out of scope
1250:   }
1251: 
1252:   VERIFY_REFERENCE_STATS();
1253: 
1254:   return JNI_TRUE;
1255: 
1256: }
1257: 
1258: jboolean testToString(JNIEnv* env, jobject self) {
1259:   auto dateClass = findClassLocal("java/util/Date");
1260:   auto dateConstructor = dateClass->getConstructor<jobject()>();
1261:   auto date = dateClass->newObject(dateConstructor);
1262: 
1263:   auto objectClass = findClassLocal("java/lang/Object");
1264:   auto objectConstructor = objectClass->getConstructor<jobject()>();
1265:   auto object = objectClass->newObject(objectConstructor);
1266: 
1267:   // First call the date implementation of toString
1268:   auto dateString = date->toString();
1269:   // And ensure that we don't use Date's toString method when calling
1270:   // toString on the object. If this doesn't crash we are fine.
1271:   auto objectString = object->toString();
1272: 
1273:   return JNI_TRUE;
1274: }
1275: 
1276: jboolean testCriticalNativeMethodBindsAndCanBeInvoked(jint a, jfloat b) {
1277:   return JNI_TRUE;
1278: }
1279: 
1280: // These implicit nullptr tests aren't called, the test is that it
````
- EN: Implements callable logic such as `testToString`, `testCriticalNativeMethodBindsAndCanBeInvoked`.
- CN: 实现可调用逻辑，例如 `testToString`, `testCriticalNativeMethodBindsAndCanBeInvoked`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1281-1360
````cpp
1281: // compiles.
1282: alias_ref<JObject> returnNullAliasRef() {
1283:   return nullptr;
1284: }
1285: 
1286: local_ref<JObject> returnNullLocalRef() {
1287:   return nullptr;
1288: }
1289: 
1290: global_ref<JObject> returnNullGlobalRef() {
1291:   return nullptr;
1292: }
1293: 
1294: void takesAliasRef(alias_ref<JObject>) {}
1295: void takesLocalRef(local_ref<JObject>) {}
1296: void takesGlobalRef(global_ref<JObject>) {}
1297: 
1298: void callWithNullRefs() {
1299:   takesAliasRef(nullptr);
1300:   takesLocalRef(nullptr);
1301:   takesGlobalRef(nullptr);
1302: }
1303: 
1304: struct SomeJavaFoo;
1305: struct OtherJavaFoo;
1306: 
1307: struct SomeJavaFoo : JavaClass<SomeJavaFoo> {
1308:   // Ensure that smart references can be used in declarations using forward-declared types.
1309:   alias_ref<OtherJavaFoo> call(alias_ref<SomeJavaFoo>);
1310:   static alias_ref<OtherJavaFoo> funcWithForwardDeclaredRefs(local_ref<OtherJavaFoo> foo);
1311: };
1312: 
1313: 
1314: using sjf = SomeJavaFoo::javaobject;
1315: 
1316: static_assert(IsNonWeakReference<local_ref<jobject>>(), "");
1317: static_assert(IsNonWeakReference<local_ref<sjf>>(), "");
1318: static_assert(IsNonWeakReference<global_ref<jobject>>(), "");
1319: static_assert(IsNonWeakReference<jobject>(), "");
1320: static_assert(IsNonWeakReference<sjf>(), "");
1321: static_assert(IsNonWeakReference<jintArray>(), "");
1322: static_assert(IsNonWeakReference<jbooleanArray>(), "");
1323: 
1324: static_assert(!IsNonWeakReference<weak_ref<jobject>>(), "");
1325: static_assert(!IsNonWeakReference<weak_ref<sjf>>(), "");
1326: static_assert(!IsNonWeakReference<std::string>(), "");
1327: static_assert(!IsNonWeakReference<jint*>(), "");
1328: static_assert(!IsNonWeakReference<void>(), "");
1329: static_assert(!IsNonWeakReference<int>(), "");
1330: 
1331: 
1332: static_assert(IsAnyReference<local_ref<jobject>>(), "");
1333: static_assert(IsAnyReference<local_ref<sjf>>(), "");
1334: static_assert(IsAnyReference<global_ref<jobject>>(), "");
1335: static_assert(IsAnyReference<jobject>(), "");
1336: static_assert(IsAnyReference<sjf>(), "");
1337: static_assert(IsAnyReference<jintArray>(), "");
1338: static_assert(IsAnyReference<jbooleanArray>(), "");
1339: static_assert(IsAnyReference<weak_ref<jobject>>(), "");
1340: static_assert(IsAnyReference<weak_ref<sjf>>(), "");
1341: 
1342: static_assert(!IsAnyReference<std::string>(), "");
1343: static_assert(!IsAnyReference<jint*>(), "");
1344: static_assert(!IsAnyReference<void>(), "");
1345: static_assert(!IsAnyReference<int>(), "");
1346: 
1347: 
1348: static_assert(IsPlainJniReference<jobject>(), "");
1349: static_assert(IsPlainJniReference<sjf>(), "");
1350: static_assert(IsPlainJniReference<jintArray>(), "");
1351: static_assert(IsPlainJniReference<jbooleanArray>(), "");
1352: 
1353: static_assert(!IsPlainJniReference<local_ref<jobject>>(), "");
1354: static_assert(!IsPlainJniReference<local_ref<sjf>>(), "");
1355: static_assert(!IsPlainJniReference<global_ref<jobject>>(), "");
1356: static_assert(!IsPlainJniReference<weak_ref<jobject>>(), "");
1357: static_assert(!IsPlainJniReference<weak_ref<sjf>>(), "");
1358: static_assert(!IsPlainJniReference<std::string>(), "");
1359: static_assert(!IsPlainJniReference<jint*>(), "");
1360: static_assert(!IsPlainJniReference<void>(), "");
````
- EN: Declares or extends types including `SomeJavaFoo`, `OtherJavaFoo`.
- CN: 声明或扩展类型，包括 `SomeJavaFoo`, `OtherJavaFoo`。
- EN: Implements callable logic such as `returnNullAliasRef`, `returnNullLocalRef`, `returnNullGlobalRef`, `takesAliasRef`.
- CN: 实现可调用逻辑，例如 `returnNullAliasRef`, `returnNullLocalRef`, `returnNullGlobalRef`, `takesAliasRef`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1361-1440
````cpp
1361: static_assert(!IsPlainJniReference<int>(), "");
1362: 
1363: 
1364: static_assert(IsJniPrimitive<int>(), "");
1365: static_assert(IsJniPrimitive<jint>(), "");
1366: static_assert(IsJniPrimitive<jboolean>(), "");
1367: 
1368: static_assert(!IsJniPrimitive<jobject>(), "");
1369: static_assert(!IsJniPrimitive<sjf>(), "");
1370: static_assert(!IsJniPrimitive<jintArray>(), "");
1371: static_assert(!IsJniPrimitive<jbooleanArray>(), "");
1372: static_assert(!IsJniPrimitive<local_ref<jobject>>(), "");
1373: static_assert(!IsJniPrimitive<local_ref<sjf>>(), "");
1374: static_assert(!IsJniPrimitive<global_ref<jobject>>(), "");
1375: static_assert(!IsJniPrimitive<weak_ref<jobject>>(), "");
1376: static_assert(!IsJniPrimitive<weak_ref<sjf>>(), "");
1377: static_assert(!IsJniPrimitive<std::string>(), "");
1378: static_assert(!IsJniPrimitive<jint*>(), "");
1379: static_assert(!IsJniPrimitive<void>(), "");
1380: 
1381: 
1382: static_assert(IsJniPrimitiveArray<jintArray>(), "");
1383: static_assert(IsJniPrimitiveArray<jbooleanArray>(), "");
1384: 
1385: static_assert(!IsJniPrimitiveArray<int>(), "");
1386: static_assert(!IsJniPrimitiveArray<jint>(), "");
1387: static_assert(!IsJniPrimitiveArray<jboolean>(), "");
1388: static_assert(!IsJniPrimitiveArray<jobject>(), "");
1389: static_assert(!IsJniPrimitiveArray<sjf>(), "");
1390: static_assert(!IsJniPrimitiveArray<local_ref<jobject>>(), "");
1391: static_assert(!IsJniPrimitiveArray<local_ref<sjf>>(), "");
1392: static_assert(!IsJniPrimitiveArray<global_ref<jobject>>(), "");
1393: static_assert(!IsJniPrimitiveArray<weak_ref<jobject>>(), "");
1394: static_assert(!IsJniPrimitiveArray<weak_ref<sjf>>(), "");
1395: static_assert(!IsJniPrimitiveArray<std::string>(), "");
1396: static_assert(!IsJniPrimitiveArray<jint*>(), "");
1397: static_assert(!IsJniPrimitiveArray<void>(), "");
1398: 
1399: 
1400: static_assert(IsJniScalar<jintArray>(), "");
1401: static_assert(IsJniScalar<jbooleanArray>(), "");
1402: static_assert(IsJniScalar<int>(), "");
1403: static_assert(IsJniScalar<jint>(), "");
1404: static_assert(IsJniScalar<jboolean>(), "");
1405: static_assert(IsJniScalar<jobject>(), "");
1406: static_assert(IsJniScalar<sjf>(), "");
1407: 
1408: static_assert(!IsJniScalar<local_ref<jobject>>(), "");
1409: static_assert(!IsJniScalar<local_ref<sjf>>(), "");
1410: static_assert(!IsJniScalar<global_ref<jobject>>(), "");
1411: static_assert(!IsJniScalar<weak_ref<jobject>>(), "");
1412: static_assert(!IsJniScalar<weak_ref<sjf>>(), "");
1413: static_assert(!IsJniScalar<std::string>(), "");
1414: static_assert(!IsJniScalar<jint*>(), "");
1415: static_assert(!IsJniScalar<void>(), "");
1416: 
1417: 
1418: static_assert(IsJniType<jintArray>(), "");
1419: static_assert(IsJniType<jbooleanArray>(), "");
1420: static_assert(IsJniType<int>(), "");
1421: static_assert(IsJniType<jint>(), "");
1422: static_assert(IsJniType<jboolean>(), "");
1423: static_assert(IsJniType<jobject>(), "");
1424: static_assert(IsJniType<sjf>(), "");
1425: static_assert(IsJniType<void>(), "");
1426: 
1427: static_assert(!IsJniType<local_ref<jobject>>(), "");
1428: static_assert(!IsJniType<local_ref<sjf>>(), "");
1429: static_assert(!IsJniType<global_ref<jobject>>(), "");
1430: static_assert(!IsJniType<weak_ref<jobject>>(), "");
1431: static_assert(!IsJniType<weak_ref<sjf>>(), "");
1432: static_assert(!IsJniType<std::string>(), "");
1433: static_assert(!IsJniType<jint*>(), "");
1434: 
1435: constexpr const char* jaccess_class_name = "com/facebook/jni/FBJniTests";
1436: 
1437: struct _jfakeClass : _jobject {};
1438: using jFakeClass = _jfakeClass*;
1439: 
1440: // This gives a better error message than doing the static_assert inline
````
- EN: Declares or extends types including `_jfakeClass`.
- CN: 声明或扩展类型，包括 `_jfakeClass`。

### Lines 1441-1520
````cpp
1441: // (because it will contain the two resolved types).
1442: template <typename T, typename F>
1443: void StaticAssertSame() {
1444:   static_assert(std::is_same<T, F>::value, "");
1445: };
1446: 
1447: } // namespace
1448: 
1449: void RegisterFbjniTests() {
1450:   StaticAssertSame<JniType<jFakeClass>, jFakeClass>();
1451: 
1452:   StaticAssertSame<PrimitiveOrJniType<JObject>, jobject>();
1453:   StaticAssertSame<PrimitiveOrJniType<JClass>, jclass>();
1454:   StaticAssertSame<PrimitiveOrJniType<JArrayInt>, jintArray>();
1455:   StaticAssertSame<PrimitiveOrJniType<jint>, jint>();
1456:   StaticAssertSame<PrimitiveOrJniType<TestThing>, TestThing::javaobject>();
1457: 
1458:   registerNatives(jaccess_class_name, {
1459:       makeNativeMethod("nativeTestClassResolution", TestClassResolution),
1460:       makeNativeMethod("nativeTestLazyClassResolution", TestLazyClassResolution),
1461:       makeNativeMethod("nativeCreateInstanceOf", TestCreateInstanceOf),
1462:       makeNativeMethod("nativeTestVirtualMethodResolution_I", TestVirtualMethodResolution_I),
1463:       makeNativeMethod("nativeTestTypeDescriptors", TestTypeDescriptors),
1464:       makeNativeMethod(
1465:           "nativeTestVirtualMethodResolution_arrB",
1466:           TestVirtualMethodResolution_arrB),
1467:       makeNativeMethod(
1468:           "nativeTestVirtualMethodResolution_S_arrS",
1469:           TestVirtualMethodResolution_S_arrS),
1470:       makeNativeMethod(
1471:           "nativeTestVirtualMethodResolution_arrarrS",
1472:           TestVirtualMethodResolution_arrarrS),
1473:       makeNativeMethod(
1474:           "nativeTestVirtualMethodResolution_arrarrI",
1475:           TestVirtualMethodResolution_arrarrI),
1476:       makeNativeMethod(
1477:           "nativeTestLazyVirtualMethodResolution_I",
1478:           TestLazyVirtualMethodResolution_I),
1479:       makeNativeMethod(
1480:           "nativeTestJMethodCallbacks",
1481:           "(Lcom/facebook/jni/FBJniTests$Callbacks;)V",
1482:           TestJMethodCallbacks),
1483:       makeNativeMethod("nativeTestJStaticMethodCallbacks", TestJStaticMethodCallbacks),
1484:       makeNativeMethod("nativeTestIsAssignableFrom", TestIsAssignableFrom),
1485:       makeNativeMethod("nativeTestIsInstanceOf", TestIsInstanceOf),
1486:       makeNativeMethod("nativeTestIsSameObject", TestIsSameObject),
1487:       makeNativeMethod("nativeTestGetSuperclass", TestGetSuperclass),
1488:       makeNativeMethod("nativeTestWeakRefs", TestWeakRefs),
1489:       makeNativeMethod("nativeTestAlias", TestAlias),
1490:       makeNativeMethod("nativeTestAutoAliasRefReturningVoid", TestAutoAliasRefReturningVoid),
1491:       makeNativeMethod("nativeTestAliasRefConversions", testAliasRefConversions),
1492:       makeNativeMethod("nativeTestCreatingReferences", testCreatingReferences),
1493:       makeNativeMethod(
1494:           "nativeTestAssignmentAndCopyConstructors",
1495:           testAssignmentAndCopyConstructors),
1496:       makeNativeMethod(
1497:           "nativeTestAssignmentAndCopyCrossTypes",
1498:           testAssignmentAndCopyCrossTypes),
1499:       makeNativeMethod("nativeTestNullReferences", testNullReferences),
1500:       makeNativeMethod("nativeTestFieldAccess", TestFieldAccess),
1501:       makeNativeMethod("nativeTestStringFieldAccess", TestStringFieldAccess),
1502:       makeNativeMethod("nativeTestReferenceFieldAccess", TestReferenceFieldAccess),
1503:       makeNativeMethod("nativeTestStaticFieldAccess", TestStaticFieldAccess),
1504:       makeNativeMethod("nativeTestStaticStringFieldAccess", TestStaticStringFieldAccess),
1505:       makeNativeMethod("nativeTestStaticReferenceFieldAccess", TestStaticReferenceFieldAccess),
1506:       makeNativeMethod("nativeTestNonVirtualMethod", TestNonVirtualMethod),
1507:       makeNativeMethod("nativeTestArrayCreation", TestArrayCreation),
1508:       makeNativeMethod("nativeTestMultidimensionalObjectArray", TestMultidimensionalObjectArray),
1509:       makeNativeMethod("nativeTestMultidimensionalPrimitiveArray", TestMultidimensionalPrimitiveArray),
1510:       makeNativeMethod("nativeTestBuildStringArray", TestBuildStringArray),
1511:       makeNativeMethod("testHandleJavaCustomExceptionNative", TestHandleJavaCustomException),
1512:       makeNativeMethod("testHandleNullExceptionMessageNative", TestHandleNullExceptionMessage),
1513:       makeNativeMethod("nativeTestHandleNestedException", TestHandleNestedException),
1514:       makeNativeMethod("nativeTestHandleNoRttiException", TestHandleNoRttiException),
1515:       makeNativeMethod("nativeTestCopyConstructor", TestCopyConstructor),
1516:       makeNativeMethod(
1517:           "nativeTestMoveConstructorWithEmptyWhat",
1518:           TestMoveConstructorWithEmptyWhat),
1519:       makeNativeMethod(
1520:           "nativeTestMoveConstructorWithPopulatedWhat",
````
- EN: Implements callable logic such as `StaticAssertSame`, `RegisterFbjniTests`.
- CN: 实现可调用逻辑，例如 `StaticAssertSame`, `RegisterFbjniTests`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1521-1555
````cpp
1521:           TestMoveConstructorWithPopulatedWhat),
1522:       makeNativeMethod("nativeTestHandleCppRuntimeError", TestHandleCppRuntimeError),
1523:       makeNativeMethod("nativeTestHandleCppIOBaseFailure", TestHandleCppIOBaseFailure),
1524:       makeNativeMethod("nativeTestHandleCppSystemError", TestHandleCppSystemError),
1525:       makeNativeMethod("nativeTestInterDsoExceptionHandlingA", TestInterDsoExceptionHandlingA),
1526:       makeNativeMethod("nativeTestInterDsoExceptionHandlingB", TestInterDsoExceptionHandlingB),
1527:       makeNativeMethod("nativeTestHandleNonStdExceptionThrow", TestHandleNonStdException),
1528:       makeNativeMethod("nativeTestHandleCppIntThrow", TestHandleCppIntThrow),
1529:       makeNativeMethod("nativeTestHandleCppCharPointerThrow", TestHandleCppCharPointerThrow),
1530:       makeNativeMethod("nativeTestThrowJavaExceptionByName", TestThrowJavaExceptionByName),
1531:       makeNativeMethod("nativeTestJThread", TestJThread),
1532:       makeNativeMethod("nativeTestThreadScopeGuard", TestThreadScopeGuard),
1533:       makeNativeMethod("nativeTestNestedThreadScopeGuard", TestNestedThreadScopeGuard),
1534:       makeNativeMethod("nativeTestClassLoadInWorker", TestClassLoadInWorker),
1535:       makeNativeMethod("nativeTestClassLoadWorkerFastPath", TestClassLoadWorkerFastPath),
1536:       makeNativeMethod("nativeTestHandleCppCharPointerThrow", TestHandleCppCharPointerThrow),
1537:       makeNativeMethod("nativeTestToString", testToString),
1538:       makeNativeMethod("nativeTestThrowJavaExceptionByName", TestThrowJavaExceptionByName),
1539:       makeNativeMethod("nativeTestThreadScopeGuard", TestThreadScopeGuard),
1540:       makeNativeMethod("nativeTestNestedThreadScopeGuard", TestNestedThreadScopeGuard),
1541:       makeNativeMethod("nativeTestNullJString", testNullJString),
1542:       makeNativeMethod("nativeTestSwap", testSwap),
1543:       makeNativeMethod("nativeTestEqualOperator", testEqualOperator),
1544:       makeNativeMethod("nativeTestReleaseAlias", testReleaseAlias),
1545:       makeNativeMethod("nativeTestLockingWeakReferences", testLockingWeakReferences),
1546:       makeNativeMethod("nativeStaticCastAliasRefToString", StaticCastAliasRefToString),
1547:       makeNativeMethod("nativeDynamicCastAliasRefToThrowable", DynamicCastAliasRefToThrowable),
1548:       makeNativeMethod("nativeStaticCastLocalRefToString", StaticCastLocalRefToString),
1549:       makeNativeMethod("nativeDynamicCastLocalRefToString", DynamicCastLocalRefToString),
1550:       makeNativeMethod("nativeStaticCastGlobalRefToString", StaticCastGlobalRefToString),
1551:       makeNativeMethod("nativeDynamicCastGlobalRefToString", DynamicCastGlobalRefToString),
1552:       makeNativeMethod("testMethodResolutionWithCxxTypesNative", TestMethodResolutionWithCxxTypes),
1553:       makeCriticalNativeMethod_DO_NOT_USE_OR_YOU_WILL_BE_FIRED("nativeCriticalNativeMethodBindsAndCanBeInvoked", testCriticalNativeMethodBindsAndCanBeInvoked),
1554:   });
1555: }
````
- EN: This range contributes implementation details for the file goal: Implements JNI bridge abstractions used by Android-facing native code.
- CN: 该范围为文件目标提供实现细节：实现供 Android 原生代码使用的 JNI 桥接抽象。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Validation and test coverage / 校验与测试覆盖
- Symbol `Callbacks` / 符号 `Callbacks`
- Symbol `TestThing` / 符号 `TestThing`
- Symbol `NonStdException` / 符号 `NonStdException`
- Symbol `RefType` / 符号 `RefType`

## Dependencies / 依赖关系
- C/C++ includes: `ios`, `stdexcept`, `system_error`, `thread`, `chrono`, `fbjni/fbjni.h`, `fbjni/JThread.h`, `expect.h`, `no_rtti.h`, `inter_dso_exception_test_2/Test.h`
- C/C++ 头文件: `ios`, `stdexcept`, `system_error`, `thread`, `chrono`, `fbjni/fbjni.h`, `fbjni/JThread.h`, `expect.h`, `no_rtti.h`, `inter_dso_exception_test_2/Test.h`
