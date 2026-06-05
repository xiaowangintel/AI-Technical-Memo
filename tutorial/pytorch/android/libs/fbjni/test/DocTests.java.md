# DocTests.java — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/test/DocTests.java`
- Repository: `pytorch`
- Purpose (EN): Implements JNI bridge abstractions used by Android-facing native code.
- 用途 (CN): 实现供 Android 原生代码使用的 JNI 桥接抽象。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
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
20: import static org.fest.assertions.api.Assertions.failBecauseExceptionWasNotThrown;
21: 
22: import com.facebook.soloader.nativeloader.NativeLoader;
23: import java.nio.ByteBuffer;
24: import java.util.Arrays;
25: import java.util.List;
26: import java.util.Map;
27: import java.util.TreeMap;
28: import org.junit.BeforeClass;
29: import org.junit.Test;
30: 
31: public class DocTests extends BaseFBJniTests {
32:   @BeforeClass
33:   public static void setup() {
34:     BaseFBJniTests.setup();
35:     NativeLoader.loadLibrary("doc_tests");
36:   }
37: 
38:   public String toString() {
39:     return "instance of DocTests";
40:   }
````
- EN: Handles module imports such as `static org.fest.assertions.api.Assertions.assertThat;`, `static org.fest.assertions.api.Assertions.failBecauseExceptionWasNotThrown;`, `com.facebook.soloader.nativeloader.NativeLoader;`, `java.nio.ByteBuffer;`.
- CN: 处理模块导入，例如 `static org.fest.assertions.api.Assertions.assertThat;`, `static org.fest.assertions.api.Assertions.failBecauseExceptionWasNotThrown;`, `com.facebook.soloader.nativeloader.NativeLoader;`, `java.nio.ByteBuffer;`。
- EN: Imports Java types such as `com.facebook.soloader.nativeloader.NativeLoader`, `java.nio.ByteBuffer`, `java.util.Arrays`, `java.util.List`.
- CN: 导入 Java 类型，例如 `com.facebook.soloader.nativeloader.NativeLoader`, `java.nio.ByteBuffer`, `java.util.Arrays`, `java.util.List`。
- EN: Declares or extends types including `DocTests`.
- CN: 声明或扩展类型，包括 `DocTests`。
- EN: Implements callable logic such as `setup`, `toString`.
- CN: 实现可调用逻辑，例如 `setup`, `toString`。

### Lines 41-80
````java
41: 
42:   // SECTION basic_methods
43:   native void nativeVoidMethod();
44:   static native void staticNativeVoidMethod();
45:   void voidMethod() {}
46:   static void staticVoidMethod() {}
47:   // END
48: 
49:   @Test
50:   public void testVoids() {
51:     nativeVoidMethod();
52:     staticNativeVoidMethod();
53:   }
54: 
55:   // SECTION primitives
56:   static native long addSomeNumbers(byte b, short s, int i);
57:   static long doubler(int i) { return i + i; }
58:   // END
59: 
60:   @Test
61:   public void testNumbers() {
62:     assertThat(addSomeNumbers((byte) 1, (short) 2, 3)).isEqualTo(14);
63:   }
64: 
65:   // SECTION strings
66:   // Java methods used by the C++ code below.
67:   static native String fancyCat(String s1, String s2);
68:   static native String getCString();
69:   static String doubler(String s) { return s + s; }
70:   // END
71: 
72:   @Test
73:   public void testStrings() {
74:     assertThat(fancyCat("a", "b")).isEqualTo("aaabbbb");
75:     assertThat(getCString()).isEqualTo("Watch your memory.");
76:   }
77: 
78:   // SECTION primitive_arrays
79:   static native int[] primitiveArrays(int[] arr);
80:   // END
````
- EN: Implements callable logic such as `nativeVoidMethod`, `staticNativeVoidMethod`, `staticVoidMethod`, `testVoids`.
- CN: 实现可调用逻辑，例如 `nativeVoidMethod`, `staticNativeVoidMethod`, `staticVoidMethod`, `testVoids`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````java
 81: 
 82:   @Test
 83:   public void testPrimitiveArrays() {
 84:     assertThat(primitiveArrays(new int[] {1, 2})).contains(1, 2, 3);
 85:   }
 86: 
 87:   static native Object convertReferences(MyDerivedClass derived);
 88: 
 89:   @Test
 90:   public void testConvertReferences() {
 91:     MyDerivedClass derived = new MyDerivedClass();
 92:     Object obj = convertReferences(derived);
 93:     assertThat(obj).isSameAs(derived);
 94:   }
 95: 
 96:   static native void castReferences(MyBaseClass base);
 97: 
 98:   @Test
 99:   public void testCastReferences() {
100:     castReferences(new MyDerivedClass());
101:     try {
102:       castReferences(new MyBaseClass());
103:       failBecauseExceptionWasNotThrown(ClassCastException.class);
104:     } catch (ClassCastException e) {
105:       assertThat(e).hasMessageContaining("MyBaseClass");
106:       assertThat(e).hasMessageContaining("MyDerivedClass");
107:     }
108:   }
109: 
110:   static native DataHolder runConstructor();
111: 
112:   @Test
113:   public void testRunConstructor() {
114:     DataHolder d = runConstructor();
115:     assertThat(d.i).isEqualTo(1);
116:     assertThat(d.s).isEqualTo("hi");
117:   }
118: 
119:   static native void callGetAndSetFields(DataHolder data);
120: 
````
- EN: Implements callable logic such as `testPrimitiveArrays`, `convertReferences`, `testConvertReferences`, `castReferences`.
- CN: 实现可调用逻辑，例如 `testPrimitiveArrays`, `convertReferences`, `testConvertReferences`, `castReferences`。

### Lines 121-160
````java
121:   @Test
122:   public void testCallGetAndSetFields() {
123:     synchronized (DataHolder.class) {
124:       DataHolder dh1 = new DataHolder(1, "1");
125:       DataHolder dh2 = new DataHolder(3, "3");
126:       DataHolder.someInstance = null;
127:       callGetAndSetFields(dh1);
128:       assertThat(dh1.i).isEqualTo(2);
129:       assertThat(dh1.s).isEqualTo("11");
130:       assertThat(DataHolder.someInstance).isSameAs(dh1);
131:       callGetAndSetFields(dh2);
132:       assertThat(dh2.i).isEqualTo(4);
133:       assertThat(dh2.s).isEqualTo("31");
134:       assertThat(DataHolder.someInstance).isSameAs(dh1);
135:       DataHolder.someInstance = null;
136:     }
137:   }
138: 
139:   static native String showJObject(Object obj, DataHolder data);
140: 
141:   @Test
142:   public void testShowJObject() {
143:     String str = showJObject(new Object(), new DataHolder(1, "hi"));
144:     assertThat(str).startsWith("data=com.facebook.jni.DataHolder@");
145:   }
146: 
147:   // SECTION boxed
148:   static native Double scaleUp(Integer number);
149:   // END
150: 
151:   @Test
152:   public void testScaleUp() {
153:     assertThat(scaleUp(5)).isEqualTo(7.5);
154:   }
155: 
156:   // SECTION iterables
157:   static native String concatMatches(List<Integer> values, Map<String, Integer> names);
158:   // END
159: 
160:   @Test
````
- EN: Implements callable logic such as `testCallGetAndSetFields`, `showJObject`, `testShowJObject`, `scaleUp`.
- CN: 实现可调用逻辑，例如 `testCallGetAndSetFields`, `showJObject`, `testShowJObject`, `scaleUp`。

### Lines 161-200
````java
161:   public void testConcatMatches() {
162:     Map<String, Integer> names = new TreeMap<>();
163:     names.put("a", 1);
164:     names.put("b", 3);
165:     names.put("c", 3);
166:     names.put("d", 7);
167:     assertThat(concatMatches(Arrays.asList(1, 2), names)).isEqualTo("bc");
168:   }
169: 
170:   static native void catchAndThrow();
171: 
172:   @Test
173:   public void testCatchAndThrow() {
174:     try {
175:       catchAndThrow();
176:       failBecauseExceptionWasNotThrown(RuntimeException.class);
177:     } catch (RuntimeException e) {
178:       assertThat(e)
179:           .hasMessageStartingWith("Caught 'java.lang.NoSuchMethodError:")
180:           .hasMessageContaining("doesNotExist")
181:           ;
182:     }
183:   }
184: 
185:   // SECTION byte_buffer
186:   static native ByteBuffer transformBuffer(ByteBuffer data);
187:   static void receiveBuffer(ByteBuffer buffer) {
188:     assertThat(buffer.capacity()).isEqualTo(2);
189:     assertThat(buffer.get(0)).isEqualTo((byte)2);
190:     assertThat(buffer.get(1)).isEqualTo((byte)3);
191:   }
192:   @Test
193:   public void testByteBuffers() {
194:     ByteBuffer data = ByteBuffer.allocateDirect(2);
195:     data.put(new byte[] {1, 2});
196:     ByteBuffer transformed = transformBuffer(data);
197:     receiveBuffer(transformed);
198:   }
199:   // END
200: }
````
- EN: Implements callable logic such as `testConcatMatches`, `catchAndThrow`, `testCatchAndThrow`, `transformBuffer`.
- CN: 实现可调用逻辑，例如 `testConcatMatches`, `catchAndThrow`, `testCatchAndThrow`, `transformBuffer`。

### Lines 201-223
````java
201: 
202: // SECTION inheritance
203: class MyBaseClass {}
204: class MyDerivedClass extends MyBaseClass {}
205: // END
206: 
207: // SECTION nested_class
208: class Outer {
209:   class Nested {}
210: }
211: // END
212: 
213: // SECTION constructor
214: class DataHolder {
215:   int i;
216:   String s;
217:   DataHolder(int i, String s) {
218:     this.i = i;
219:     this.s = s;
220:   }
221:   static DataHolder someInstance;
222: }
223: // END
````
- EN: Declares or extends types including `MyBaseClass`, `MyDerivedClass`, `Outer`, `Nested`.
- CN: 声明或扩展类型，包括 `MyBaseClass`, `MyDerivedClass`, `Outer`, `Nested`。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Validation and test coverage / 校验与测试覆盖
- Symbol `DocTests` / 符号 `DocTests`
- Symbol `MyBaseClass` / 符号 `MyBaseClass`
- Symbol `MyDerivedClass` / 符号 `MyDerivedClass`
- Symbol `Outer` / 符号 `Outer`

## Dependencies / 依赖关系
- Java imports: `com.facebook.soloader.nativeloader.NativeLoader`, `java.nio.ByteBuffer`, `java.util.Arrays`, `java.util.List`, `java.util.Map`, `java.util.TreeMap`, `org.junit.BeforeClass`, `org.junit.Test`
- Java 导入: `com.facebook.soloader.nativeloader.NativeLoader`, `java.nio.ByteBuffer`, `java.util.Arrays`, `java.util.List`, `java.util.Map`, `java.util.TreeMap`, `org.junit.BeforeClass`, `org.junit.Test`
