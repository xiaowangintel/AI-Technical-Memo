# HybridTests.java — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/test/HybridTests.java`
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
20: 
21: import com.facebook.jni.annotations.DoNotStrip;
22: import org.junit.Test;
23: 
24: public class HybridTests extends BaseFBJniTests {
25:   static class TestHybridClass {
26:     // Hybrid classes must include a member which manages the C++ object.  It
27:     // will be initialized from C++.  It must be declared exactly with this
28:     // type and name, so JNI can find it, and initialized once in the ctor.
29:     // The annotation is necessary to keep proguard from renaming it, or else JNI
30:     // won't be able to find it.
31:     @DoNotStrip private final HybridData mHybridData;
32: 
33:     // This is the method which creates the C++ instance and initializes
34:     // mHybridData.  Conventionally, it should be named initHybrid, and invoked
35:     // from the constructor.  This must be called only once.  If the C++
36:     // instance is referenced before this is called, a NullPointerException
37:     // will be thrown.
38:     private native HybridData initHybrid(int i, String s, boolean b);
39: 
40:     // You can have more than one, which may be useful if the ctor is
41:     // overloaded.  This will call the default C++ ctor.
42:     private native HybridData initHybrid();
43: 
44:     // Implements factory-style initialization.  You shouldn't usually
45:     // need both styles in one class.  Here we do it for testing and
46:     // demo purposes.
47:     private native HybridData initHybrid(String s, int i, boolean b);
48: 
49:     // Java ctor must invoke initHybrid().  This just passes arguments through,
50:     // but the ctor can do whatever work it wants, as long as it calls
51:     // initHybrid() before any native methods.
52:     public TestHybridClass(int i, String s, boolean b) {
53:       mHybridData = initHybrid(i, s, b);
54:     }
55: 
56:     // This behaves the same as the ctor above, I just wanted a different
57:     // signature to demonstrate factory-style initialization.
58:     public TestHybridClass(String s, int i, boolean b) {
59:       mHybridData = initHybrid(s, i, b);
60:     }
````
- EN: Handles module imports such as `static org.fest.assertions.api.Assertions.assertThat;`, `com.facebook.jni.annotations.DoNotStrip;`, `org.junit.Test;`.
- CN: 处理模块导入，例如 `static org.fest.assertions.api.Assertions.assertThat;`, `com.facebook.jni.annotations.DoNotStrip;`, `org.junit.Test;`。
- EN: Imports Java types such as `com.facebook.jni.annotations.DoNotStrip`, `org.junit.Test`.
- CN: 导入 Java 类型，例如 `com.facebook.jni.annotations.DoNotStrip`, `org.junit.Test`。
- EN: Declares or extends types including `HybridTests`, `TestHybridClass`.
- CN: 声明或扩展类型，包括 `HybridTests`, `TestHybridClass`。
- EN: Implements callable logic such as `initHybrid`.
- CN: 实现可调用逻辑，例如 `initHybrid`。

### Lines 61-120
````java
 61: 
 62:     // This is the simplest case.  Even if everything is default, initHybrid()
 63:     // must still be called.
 64:     public TestHybridClass() {
 65:       mHybridData = initHybrid();
 66:     }
 67: 
 68:     // Java ctor used by C++ newObjectCxxArgs.  Note this is private.
 69:     private TestHybridClass(HybridData hd) {
 70:       mHybridData = hd;
 71:     }
 72: 
 73:     public void doneUsingIt() {
 74:       mHybridData.resetNative();
 75:     }
 76: 
 77:     // Some C++ methods.
 78:     public native void setBoth(int i, String s);
 79: 
 80:     public native int getInt();
 81: 
 82:     public native String getString();
 83: 
 84:     public native String getCharString();
 85: 
 86:     public native boolean copy1(TestHybridClass other);
 87: 
 88:     public native boolean copy2(TestHybridClass other);
 89: 
 90:     public native void oops();
 91: 
 92:     public native void setGlobal(String s);
 93: 
 94:     public native String getGlobal1();
 95: 
 96:     public native String getGlobal2();
 97: 
 98:     public static native TestHybridClass makeWithTwo();
 99: 
100:     public static native TestHybridClass makeWithThree();
101: 
102:     public static native void autoconvertMany();
103:   }
104: 
105:   @Test
106:   public void testHybridClass() {
107:     TestHybridClass thc1 = new TestHybridClass();
108:     assertThat(thc1.getInt()).isEqualTo(0);
109:     assertThat(thc1.getString()).isEqualTo("");
110: 
111:     thc1.setBoth(1, "one");
112:     assertThat(thc1.getInt()).isEqualTo(1);
113:     assertThat(thc1.getString()).isEqualTo("one");
114: 
115:     TestHybridClass thc2 = TestHybridClass.makeWithTwo();
116:     assertThat(thc2.getInt()).isEqualTo(2);
117:     assertThat(thc2.getString()).isEqualTo("two");
118: 
119:     thc2.doneUsingIt();
120: 
````
- EN: Implements callable logic such as `doneUsingIt`, `setBoth`, `getInt`, `getString`.
- CN: 实现可调用逻辑，例如 `doneUsingIt`, `setBoth`, `getInt`, `getString`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-180
````java
121:     thrown.expect(NullPointerException.class);
122:     thc2.getInt();
123:   }
124: 
125:   @Test
126:   public void testHybridAutoconversion() {
127:     TestHybridClass thc3 = TestHybridClass.makeWithThree();
128:     assertThat(thc3.copy1(new TestHybridClass(3, "three", false))).isTrue();
129:     assertThat(thc3.getInt()).isEqualTo(3);
130:     assertThat(thc3.getString()).isEqualTo("three");
131: 
132:     TestHybridClass thc4 = new TestHybridClass();
133:     thc4.copy1(new TestHybridClass("four", 4, false));
134:     assertThat(thc4.getInt()).isEqualTo(4);
135:     assertThat(thc4.getString()).isEqualTo("four");
136:     assertThat(thc4.getCharString()).isEqualTo("four");
137: 
138:     TestHybridClass thc5 = new TestHybridClass();
139:     assertThat(thc5.copy2(new TestHybridClass(5, "five", false))).isTrue();
140:     assertThat(thc5.getInt()).isEqualTo(5);
141:     assertThat(thc5.getString()).isEqualTo("five");
142:   }
143: 
144:   @Test
145:   public void testReturnGlobalRef() {
146:     TestHybridClass thc = new TestHybridClass();
147:     thc.setGlobal("global_ref");
148:     assertThat(thc.getGlobal1()).isEqualTo("global_ref");
149:     assertThat(thc.getGlobal2()).isEqualTo("global_ref");
150:   }
151: 
152:   @Test
153:   public void testLocalLeak() {
154:     TestHybridClass.autoconvertMany();
155:   }
156: 
157:   @Test
158:   public void testExceptionMapping() {
159:     TestHybridClass thc1 = new TestHybridClass();
160:     thrown.expect(ArrayStoreException.class);
161:     thc1.oops();
162:   }
163: 
164:   abstract static class AbstractTestHybrid {
165:     @DoNotStrip private final HybridData mHybridData;
166: 
167:     private int mAbstractNum;
168: 
169:     protected AbstractTestHybrid(HybridData hybridData, int an) {
170:       mHybridData = hybridData;
171:       mAbstractNum = an;
172:     }
173: 
174:     public int abstractNum() {
175:       return mAbstractNum;
176:     }
177: 
178:     public native int nativeNum();
179: 
180:     public abstract int concreteNum();
````
- EN: Declares or extends types including `AbstractTestHybrid`.
- CN: 声明或扩展类型，包括 `AbstractTestHybrid`。
- EN: Implements callable logic such as `testHybridAutoconversion`, `testReturnGlobalRef`, `testLocalLeak`, `testExceptionMapping`.
- CN: 实现可调用逻辑，例如 `testHybridAutoconversion`, `testReturnGlobalRef`, `testLocalLeak`, `testExceptionMapping`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````java
181: 
182:     public abstract int sum();
183:   }
184: 
185:   static class ConcreteTestHybrid extends AbstractTestHybrid {
186:     public ConcreteTestHybrid(int an, int nn, int cn) {
187:       super(initHybrid(nn, cn), an);
188:     }
189: 
190:     private static native HybridData initHybrid(int nn, int cn);
191: 
192:     // overrides can be native
193:     @Override
194:     public native int concreteNum();
195: 
196:     // overrides can be java
197:     @Override
198:     public int sum() {
199:       return nativeNum() + abstractNum() + concreteNum();
200:     }
201:   }
202: 
203:   @Test
204:   public void testHybridInheritance() {
205:     AbstractTestHybrid ath = new ConcreteTestHybrid(1, 2, 3);
206:     assertThat(ath.abstractNum()).isEqualTo(1);
207:     assertThat(ath.nativeNum()).isEqualTo(2);
208:     assertThat(ath.concreteNum()).isEqualTo(3);
209:     assertThat(ath.sum()).isEqualTo(6);
210:   }
211: 
212:   public static native boolean cxxTestInheritance(AbstractTestHybrid ath);
213: 
214:   public static native AbstractTestHybrid makeAbstractHybrid();
215: 
216:   @Test
217:   public void testHybridCxx() {
218:     AbstractTestHybrid ath = new ConcreteTestHybrid(4, 5, 6);
219:     assertThat(cxxTestInheritance(ath)).isTrue();
220: 
221:     AbstractTestHybrid ath2 = makeAbstractHybrid();
222:     assertThat(ath2 instanceof ConcreteTestHybrid).isTrue();
223:     assertThat(ath2.abstractNum()).isEqualTo(7);
224:     assertThat(ath2.nativeNum()).isEqualTo(8);
225:     assertThat(ath2.concreteNum()).isEqualTo(9);
226:     assertThat(ath2.sum()).isEqualTo(24);
227:   }
228: 
229:   static class Base {}
230: 
231:   static class Derived extends Base {
232:     @DoNotStrip private final HybridData mHybridData;
233: 
234:     private Derived(HybridData hybridData) {
235:       mHybridData = hybridData;
236:     }
237:   }
238: 
239:   public static native boolean cxxTestDerivedJavaClass();
240: 
````
- EN: Declares or extends types including `ConcreteTestHybrid`, `Base`, `Derived`.
- CN: 声明或扩展类型，包括 `ConcreteTestHybrid`, `Base`, `Derived`。
- EN: Implements callable logic such as `sum`, `initHybrid`, `concreteNum`, `testHybridInheritance`.
- CN: 实现可调用逻辑，例如 `sum`, `initHybrid`, `concreteNum`, `testHybridInheritance`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````java
241:   @Test
242:   public void testDerivedJavaClassCxx() {
243:     assertThat(cxxTestDerivedJavaClass()).isTrue();
244:   }
245: 
246:   static class TestHybridClassBase extends HybridClassBase {
247:     protected native void initHybrid();
248: 
249:     private native void initHybrid(int i);
250: 
251:     protected TestHybridClassBase() {
252:       // No initHybrid() here!
253:       // Otherwise factory construction will set native pointer twice and process will crash.
254:     }
255: 
256:     public TestHybridClassBase(int i) {
257:       initHybrid(i);
258:     }
259: 
260:     // Some C++ methods.
261:     public native void setInt(int i);
262: 
263:     public native int getInt();
264: 
265:     public static native TestHybridClassBase makeWithThree();
266:   }
267: 
268:   static class TestHybridClassBaseDefaultCtor extends TestHybridClassBase {
269:     public TestHybridClassBaseDefaultCtor() {
270:       initHybrid();
271:     }
272:   }
273: 
274:   @Test
275:   public void testHybridBaseDefaultCtor() {
276:     TestHybridClassBaseDefaultCtor base = new TestHybridClassBaseDefaultCtor();
277:     assertThat(base.getInt()).isZero();
278: 
279:     base.setInt(58);
280:     assertThat(base.getInt()).isEqualTo(58);
281:   }
282: 
283:   @Test
284:   public void testHybridBaseConstructorArgs() {
285:     TestHybridClassBase base = new TestHybridClassBase(42);
286:     assertThat(base.getInt()).isEqualTo(42);
287:   }
288: 
289:   @Test
290:   public void testHybridBaseFactoryConstruction() {
291:     TestHybridClassBase base = TestHybridClassBase.makeWithThree();
292:     assertThat(base.getInt()).isEqualTo(3);
293:   }
294: 
295:   static class Destroyable {
296:     @DoNotStrip private final HybridData mHybridData;
297: 
298:     private Destroyable(HybridData hybridData) {
299:       mHybridData = hybridData;
300:     }
````
- EN: Declares or extends types including `TestHybridClassBase`, `TestHybridClassBaseDefaultCtor`, `Destroyable`.
- CN: 声明或扩展类型，包括 `TestHybridClassBase`, `TestHybridClassBaseDefaultCtor`, `Destroyable`。
- EN: Implements callable logic such as `testDerivedJavaClassCxx`, `initHybrid`, `setInt`, `getInt`.
- CN: 实现可调用逻辑，例如 `testDerivedJavaClassCxx`, `initHybrid`, `setInt`, `getInt`。

### Lines 301-309
````java
301:   }
302: 
303:   public static native boolean cxxTestHybridDestruction();
304: 
305:   @Test
306:   public void testHybridDestuction() {
307:     assertThat(cxxTestHybridDestruction()).isTrue();
308:   }
309: }
````
- EN: Implements callable logic such as `cxxTestHybridDestruction`, `testHybridDestuction`.
- CN: 实现可调用逻辑，例如 `cxxTestHybridDestruction`, `testHybridDestuction`。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Validation and test coverage / 校验与测试覆盖
- Symbol `HybridTests` / 符号 `HybridTests`
- Symbol `TestHybridClass` / 符号 `TestHybridClass`
- Symbol `AbstractTestHybrid` / 符号 `AbstractTestHybrid`
- Symbol `ConcreteTestHybrid` / 符号 `ConcreteTestHybrid`

## Dependencies / 依赖关系
- Java imports: `com.facebook.jni.annotations.DoNotStrip`, `org.junit.Test`
- Java 导入: `com.facebook.jni.annotations.DoNotStrip`, `org.junit.Test`
