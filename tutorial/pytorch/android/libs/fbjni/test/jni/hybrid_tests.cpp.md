# hybrid_tests.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/test/jni/hybrid_tests.cpp`
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
17: #include <fbjni/fbjni.h>
18: #include <mutex>
19: #include <condition_variable>
20: 
21: using namespace facebook::jni;
22: 
23: class TestException : public std::runtime_error {
24: public:
25:   TestException() : std::runtime_error("fail") {}
26: };
27: 
28: // The C++ half of a hybrid class must extend HybridClass like this.
29: // HybridClass is default constructible.
30: class TestHybridClass : public facebook::jni::HybridClass<TestHybridClass> {
31: public:
32:   static constexpr const char* const kJavaDescriptor =
33:     "Lcom/facebook/jni/HybridTests$TestHybridClass;";
34: 
35:   TestHybridClass() {}
36: 
37:   TestHybridClass(int i, std::string s, bool b)
38:     : i_(i)
39:     , s_(std::move(s))
40:     , b_(b) {}
41: 
42:   // The C++ implementation of initHybrid must be a static C++ method,
43:   // since it has no C++ instance (it creates it).  But it's a normal
44:   // java method, because it gets a java this passed in (which is
45:   // often unused).  It must call makeCxxInstance and return its
46:   // result.
47:   static jhybriddata initHybrid(alias_ref<jhybridobject>, jint i, jstring s, jboolean b) {
48:     // The arguments will be forwarded to the ctor, and the result
49:     // will be saved in mHybridPointer in the java object.
50:     return makeCxxInstance(i, wrap_alias(s)->toStdString(), b).release();
51:   }
52:   static local_ref<jhybriddata> initHybrid2(alias_ref<jhybridobject>) {
53:     return makeCxxInstance();
54:   }
55:   // It's also ok to make initHybrid work like a factory, which
56:   // eliminates the need to define a ctor and reduces boilerplate
57:   // code.  This is a member, so fields can be initialized directly.
58:   static local_ref<jhybriddata> initHybrid3(alias_ref<jhybridobject>,
59:                                             std::string s,
60:                                             int i,
````
- EN: Pulls in native headers such as `fbjni/fbjni.h`, `mutex`, `condition_variable`.
- CN: 引入原生头文件，例如 `fbjni/fbjni.h`, `mutex`, `condition_variable`。
- EN: Declares or extends types including `TestException`, `must`, `TestHybridClass`.
- CN: 声明或扩展类型，包括 `TestException`, `must`, `TestHybridClass`。
- EN: Implements callable logic such as `TestException`, `TestHybridClass`, `instance`, `initHybrid2`.
- CN: 实现可调用逻辑，例如 `TestException`, `TestHybridClass`, `instance`, `initHybrid2`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````cpp
 61:                                             bool b) {
 62:     auto cxxPart = std::unique_ptr<TestHybridClass>(new TestHybridClass);
 63:     cxxPart->s_ = std::move(s);
 64:     cxxPart->i_ = i;
 65:     cxxPart->b_ = b;
 66:     return makeHybridData(std::move(cxxPart));
 67:   }
 68: 
 69:   jint getInt() {
 70:     return i_;
 71:   }
 72: 
 73:   std::string getString() {
 74:     return s_;
 75:   }
 76: 
 77:   const char* getCharString() {
 78:     return s_.c_str();
 79:   }
 80: 
 81:   void setBoth(int i, std::string s) {
 82:     i_ = i;
 83:     s_ = s;
 84:   }
 85: 
 86:   bool copy1(alias_ref<jhybridobject> other) {
 87:     i_ = cthis(other)->i_;
 88:     s_ = cthis(other)->s_;
 89:     return true;
 90:   }
 91: 
 92:   // This returns a boolean to test more code paths, not for any
 93:   // functional reason.
 94:   jboolean copy2(TestHybridClass* other) {
 95:     i_ = other->i_;
 96:     s_ = other->s_;
 97:     return true;
 98:   }
 99: 
100:   void oops() {
101:     throw TestException();
102:   }
103: 
104:   void setGlobal(alias_ref<jstring> str) {
105:     global_ = make_global(str);
106:   }
107: 
108:   global_ref<jstring> getGlobal1() {
109:     return global_;
110:   }
111: 
112:   const global_ref<jstring>& getGlobal2() {
113:     return global_;
114:   }
115: 
116:   static jhybridobject makeWithTwo(alias_ref<jclass> jthis) {
117:     // The args are passed to java, with autoconversion.
118:     return newObjectJavaArgs(2, "two", true).release();
119:   }
120: 
````
- EN: Implements callable logic such as `getInt`, `getString`, `getCharString`, `setBoth`.
- CN: 实现可调用逻辑，例如 `getInt`, `getString`, `getCharString`, `setBoth`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````cpp
121:   static local_ref<jhybridobject> makeWithThree(alias_ref<jclass> jthis) {
122:     // The args are passed to C++ initialization directly.
123:    return newObjectCxxArgs(3, "three", true);
124:   }
125: 
126:   static void mapException(const std::exception& ex) {
127:     if (dynamic_cast<const TestException*>(&ex) != 0) {
128:       throwNewJavaException("java/lang/ArrayStoreException", "");
129:     }
130:   }
131: 
132:   static void autoconvertMany(alias_ref<jclass>) {
133:     // Autoconversion to jstring creates a local ref.  Make sure is is
134:     // released properly.  The default local table size is 512.
135:     for (int i = 0; i < 1000; ++i) {
136:       newObjectJavaArgs(1000, "thousand", false);
137:       newObjectJavaArgs(1001, make_jstring("thousand+1"), false);
138:     }
139:   }
140: 
141:   // Declaring a register function for each class makes it easy to
142:   // manage the process.
143:   static void registerNatives() {
144:     registerHybrid({
145:         // Test registration of static method returning void.
146:         makeNativeMethod("initHybrid", TestHybridClass::initHybrid),
147:         // overloaded members are ugly to get pointers to.  Use a
148:         // different name for each C++ method to keep things readable.
149:         makeNativeMethod("initHybrid", TestHybridClass::initHybrid2),
150:         makeNativeMethod("initHybrid", TestHybridClass::initHybrid3),
151:         // C++ members can be registered directly.  The C++ instance
152:         // created by makeCxxInstance will be retrieved from the
153:         // HybridData, and the method will be invoked with the
154:         // arguments passed to java.  These test registration of
155:         // method pointers returning non-void.
156:         makeNativeMethod("getInt", TestHybridClass::getInt),
157:         makeNativeMethod("getString", TestHybridClass::getString),
158:         makeNativeMethod("getCharString", TestHybridClass::getCharString),
159:         makeNativeMethod("setBoth", TestHybridClass::setBoth),
160:         makeNativeMethod("copy1", TestHybridClass::copy1),
161:         makeNativeMethod("copy2", TestHybridClass::copy2),
162:         makeNativeMethod("oops", TestHybridClass::oops),
163:         makeNativeMethod("setGlobal", TestHybridClass::setGlobal),
164:         makeNativeMethod("getGlobal1", TestHybridClass::getGlobal1),
165:         makeNativeMethod("getGlobal2", TestHybridClass::getGlobal2),
166:         makeNativeMethod("makeWithTwo", TestHybridClass::makeWithTwo),
167:         makeNativeMethod("makeWithThree", TestHybridClass::makeWithThree),
168:         makeNativeMethod("autoconvertMany", TestHybridClass::autoconvertMany),
169:       });
170:   }
171: 
172: private:
173:   friend HybridBase;
174: 
175:   int i_ = 0;
176:   std::string s_;
177:   bool b_ = false;
178:   global_ref<jstring> global_;
179: };
180: 
````
- EN: Declares or extends types including `makes`.
- CN: 声明或扩展类型，包括 `makes`。
- EN: Implements callable logic such as `makeWithThree`, `mapException`, `autoconvertMany`, `registerNatives`.
- CN: 实现可调用逻辑，例如 `makeWithThree`, `mapException`, `autoconvertMany`, `registerNatives`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````cpp
181: class AbstractTestHybrid : public facebook::jni::HybridClass<AbstractTestHybrid> {
182: public:
183:   static constexpr const char* const kJavaDescriptor =
184:     "Lcom/facebook/jni/HybridTests$AbstractTestHybrid;";
185: 
186:   AbstractTestHybrid(int nn)
187:       : nativeNum_(nn) {}
188: 
189:   int nativeNum() { return nativeNum_; }
190:   // This is different than the java method!
191:   virtual int sum() = 0;
192: 
193:   static void registerNatives() {
194:     registerHybrid({
195:       makeNativeMethod("nativeNum", AbstractTestHybrid::nativeNum),
196:     });
197:   }
198: 
199:  private:
200:   int nativeNum_;
201: };
202: 
203: class ConcreteTestHybrid
204:     : public facebook::jni::HybridClass<ConcreteTestHybrid, AbstractTestHybrid> {
205:  public:
206:   static constexpr const char* const kJavaDescriptor =
207:     "Lcom/facebook/jni/HybridTests$ConcreteTestHybrid;";
208: 
209:   static local_ref<jhybriddata> initHybrid(alias_ref<jclass>, int nn, int cn) {
210:     return makeCxxInstance(nn, cn);
211:   }
212: 
213:   int concreteNum() {
214:     return concreteNum_;
215:   }
216:   int sum() override {
217:     return nativeNum() + concreteNum();
218:   }
219: 
220:   static void registerNatives() {
221:     registerHybrid({
222:       makeNativeMethod("initHybrid", ConcreteTestHybrid::initHybrid),
223:       makeNativeMethod("concreteNum", ConcreteTestHybrid::concreteNum),
224:     });
225:   }
226: 
227:  private:
228:   friend HybridBase;
229:   ConcreteTestHybrid(int nn, int cn)
230:       : HybridBase(nn)
231:       , concreteNum_(cn) {}
232: 
233:   int concreteNum_;
234: };
235: 
236: static jboolean cxxTestInheritance(alias_ref<jclass>, AbstractTestHybrid* ath) {
237:   bool ret = true;
238: 
239:   ret &= ath->nativeNum() == 5;
240:   ret &= ath->sum() == 11;
````
- EN: Declares or extends types including `AbstractTestHybrid`, `ConcreteTestHybrid`.
- CN: 声明或扩展类型，包括 `AbstractTestHybrid`, `ConcreteTestHybrid`。
- EN: Implements callable logic such as `AbstractTestHybrid`, `nativeNum`, `registerNatives`, `initHybrid`.
- CN: 实现可调用逻辑，例如 `AbstractTestHybrid`, `nativeNum`, `registerNatives`, `initHybrid`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````cpp
241: 
242:   auto cth = dynamic_cast<ConcreteTestHybrid*>(ath);
243: 
244:   ret &= cth != nullptr;
245:   ret &= cth->concreteNum() == 6;
246: 
247:   return ret;
248: }
249: 
250: static local_ref<AbstractTestHybrid::jhybridobject> makeAbstractHybrid(alias_ref<jclass>) {
251:   auto cth = ConcreteTestHybrid::newObjectJavaArgs(0,0,0);
252:   weak_ref<ConcreteTestHybrid::jhybridobject> wcth = make_weak(cth);
253:   weak_ref<AbstractTestHybrid::jhybridobject> wath = wcth;
254: 
255:   local_ref<AbstractTestHybrid::jhybridobject> ath = cth;
256:   return ConcreteTestHybrid::newObjectJavaArgs(cthis(ath)->nativeNum() + 7, 8, 9);
257: }
258: 
259: struct Base : public JavaClass<Base> {
260:   static constexpr const char* const kJavaDescriptor = "Lcom/facebook/jni/HybridTests$Base;";
261: };
262: 
263: class Derived : public HybridClass<Derived, Base> {
264:  public:
265:   static constexpr const char* const kJavaDescriptor = "Lcom/facebook/jni/HybridTests$Derived;";
266: 
267:  private:
268:   friend HybridBase;
269:   Derived() {}
270: };
271: 
272: struct Destroyable : public HybridClass<Destroyable> {
273:  public:
274:   static constexpr const char* const kJavaDescriptor = "Lcom/facebook/jni/HybridTests$Destroyable;";
275: 
276:   ~Destroyable() override {
277:     std::lock_guard<std::mutex> lk(*mtx_);
278:     *dead_ = true;
279:     cv_->notify_one();
280:   }
281: 
282:   using HybridBase::makeCxxInstance;
283: 
284:  private:
285:   friend HybridBase;
286:   Destroyable(std::shared_ptr<std::mutex> mtx,
287:       std::shared_ptr<bool> dead,
288:       std::shared_ptr<std::condition_variable> cv)
289:     : mtx_(mtx), dead_(dead), cv_(cv) {}
290: 
291:   std::shared_ptr<std::mutex> mtx_;
292:   std::shared_ptr<bool> dead_;
293:   std::shared_ptr<std::condition_variable> cv_;
294: };
295: 
296: static jboolean cxxTestHybridDestruction(alias_ref<jclass>) {
297:   auto mtx = std::make_shared<std::mutex>();
298:   auto dead = std::make_shared<bool>(false);
299:   auto cv = std::make_shared<std::condition_variable>();
300:   Destroyable::makeCxxInstance(mtx, dead, cv);
````
- EN: Declares or extends types including `Base`, `Derived`, `Destroyable`.
- CN: 声明或扩展类型，包括 `Base`, `Derived`, `Destroyable`。
- EN: Implements callable logic such as `makeAbstractHybrid`, `Derived`, `~Destroyable`, `Destroyable`.
- CN: 实现可调用逻辑，例如 `makeAbstractHybrid`, `Derived`, `~Destroyable`, `Destroyable`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````cpp
301:   std::unique_lock<std::mutex> lk(*mtx);
302:   cv->wait(lk, [&] { return dead; });
303:   return JNI_TRUE;
304: }
305: 
306: 
307: static jboolean cxxTestDerivedJavaClass(alias_ref<jclass>) {
308:   bool ret = true;
309: 
310:   auto derivedJava = Derived::newObjectCxxArgs();
311: 
312:   ret &= derivedJava ? true : false;
313:   ret &= derivedJava->isInstanceOf(Derived::javaClassLocal());
314:   ret &= derivedJava->isInstanceOf(Base::javaClassLocal());
315: 
316:   auto javaPtr = derivedJava.get();
317: 
318:   ret &= dynamic_cast<Derived::javaobject>(javaPtr) != nullptr;
319:   ret &= dynamic_cast<Base::javaobject>(javaPtr) != nullptr;
320:   ret &= dynamic_cast<jobject>(javaPtr) != nullptr;
321: 
322:   return ret;
323: }
324: 
325: class TestHybridClassBase
326: : public facebook::jni::HybridClass<TestHybridClassBase> {
327:   public:
328:     static constexpr const char* const kJavaDescriptor =
329:       "Lcom/facebook/jni/HybridTests$TestHybridClassBase;";
330: 
331:     TestHybridClassBase() {}
332: 
333:     explicit TestHybridClassBase(int i)
334:       : i_(i) {}
335: 
336:     static void initHybrid(alias_ref<jhybridobject> o, jint i) {
337:       // The arguments will be forwarded to the ctor, and the result
338:       // will be saved in mHybridPointer in the java object.
339:       setCxxInstance(o, i);
340:     }
341: 
342:     static void initHybrid2(alias_ref<jhybridobject> o) {
343:       setCxxInstance(o);
344:     }
345: 
346:     virtual jint getInt() {
347:       return i_;
348:     }
349: 
350:     void setInt(int i) {
351:       i_ = i;
352:     }
353: 
354:     static local_ref<jhybridobject> makeWithThree(alias_ref<jclass> /* unused */) {
355:       // The args are passed to C++ initialization directly.
356:       return newObjectCxxArgs(3);
357:     }
358: 
359:     static void registerNatives() {
360:       registerHybrid({
````
- EN: Declares or extends types including `TestHybridClassBase`.
- CN: 声明或扩展类型，包括 `TestHybridClassBase`。
- EN: Implements callable logic such as `cxxTestDerivedJavaClass`, `TestHybridClassBase`, `initHybrid`, `initHybrid2`.
- CN: 实现可调用逻辑，例如 `cxxTestDerivedJavaClass`, `TestHybridClassBase`, `initHybrid`, `initHybrid2`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-384
````cpp
361:           makeNativeMethod("initHybrid", TestHybridClassBase::initHybrid),
362:           makeNativeMethod("initHybrid", TestHybridClassBase::initHybrid2),
363:           makeNativeMethod("getInt", TestHybridClassBase::getInt),
364:           makeNativeMethod("setInt", TestHybridClassBase::setInt),
365:           makeNativeMethod("makeWithThree", TestHybridClassBase::makeWithThree),
366:           });
367:     }
368:   private:
369:     int i_ = 0;
370: };
371: 
372: void RegisterTestHybridClass() {
373:   TestHybridClass::registerNatives();
374:   AbstractTestHybrid::registerNatives();
375:   ConcreteTestHybrid::registerNatives();
376:   TestHybridClassBase::registerNatives();
377: 
378:   registerNatives("com/facebook/jni/HybridTests", {
379:     makeNativeMethod("cxxTestInheritance", cxxTestInheritance),
380:     makeNativeMethod("makeAbstractHybrid", makeAbstractHybrid),
381:     makeNativeMethod("cxxTestDerivedJavaClass", cxxTestDerivedJavaClass),
382:     makeNativeMethod("cxxTestHybridDestruction", cxxTestHybridDestruction),
383:   });
384: }
````
- EN: Implements callable logic such as `RegisterTestHybridClass`.
- CN: 实现可调用逻辑，例如 `RegisterTestHybridClass`。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Validation and test coverage / 校验与测试覆盖
- Symbol `TestException` / 符号 `TestException`
- Symbol `must` / 符号 `must`
- Symbol `TestHybridClass` / 符号 `TestHybridClass`
- Symbol `makes` / 符号 `makes`

## Dependencies / 依赖关系
- C/C++ includes: `fbjni/fbjni.h`, `mutex`, `condition_variable`
- C/C++ 头文件: `fbjni/fbjni.h`, `mutex`, `condition_variable`
