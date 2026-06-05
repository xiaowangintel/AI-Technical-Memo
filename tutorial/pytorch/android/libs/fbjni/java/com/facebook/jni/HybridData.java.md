# HybridData.java — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/java/com/facebook/jni/HybridData.java`
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
19: import com.facebook.jni.annotations.DoNotStrip;
20: import com.facebook.soloader.nativeloader.NativeLoader;
21: 
22: /**
23:  * This object holds a native C++ member for hybrid Java/C++ objects.
24:  *
25:  * <p>NB: THREAD SAFETY
26:  *
27:  * <p>{@link #resetNative} deletes the corresponding native object synchronously on whatever thread
28:  * the method is called on. Otherwise, deletion will occur on the {@link DestructorThread} thread.
29:  */
30: @DoNotStrip
31: public class HybridData {
32: 
33:   static {
34:     NativeLoader.loadLibrary("fbjni");
35:   }
36: 
37:   @DoNotStrip private Destructor mDestructor = new Destructor(this);
38: 
39:   /**
40:    * To explicitly delete the instance, call resetNative(). If the C++ instance is referenced after
````
- EN: Handles module imports such as `com.facebook.jni.annotations.DoNotStrip;`, `com.facebook.soloader.nativeloader.NativeLoader;`.
- CN: 处理模块导入，例如 `com.facebook.jni.annotations.DoNotStrip;`, `com.facebook.soloader.nativeloader.NativeLoader;`。
- EN: Imports Java types such as `com.facebook.jni.annotations.DoNotStrip`, `com.facebook.soloader.nativeloader.NativeLoader`.
- CN: 导入 Java 类型，例如 `com.facebook.jni.annotations.DoNotStrip`, `com.facebook.soloader.nativeloader.NativeLoader`。
- EN: Declares or extends types including `HybridData`.
- CN: 声明或扩展类型，包括 `HybridData`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````java
41:    * this is called, a NullPointerException will be thrown. resetNative() may be called multiple
42:    * times safely. Because the {@link DestructorThread} also calls resetNative, the instance will
43:    * not leak if this is not called, but timing of deletion and the thread the C++ dtor is called on
44:    * will be at the whim of the Java GC. If you want to control the thread and timing of the
45:    * destructor, you should call resetNative() explicitly.
46:    */
47:   public synchronized void resetNative() {
48:     mDestructor.destruct();
49:   }
50: 
51:   /**
52:    * N.B. Thread safety. If you call isValid from a different thread than {@link #resetNative()}
53:    * then be sure to do so while synchronizing on the hybrid. For example:
54:    *
55:    * <pre><code>
56:    * synchronized(hybrid) {
57:    *   if (hybrid.isValid) {
58:    *     // Do stuff.
59:    *   }
60:    * }
61:    * </code></pre>
62:    */
63:   public boolean isValid() {
64:     return mDestructor.mNativePointer != 0;
65:   }
66: 
67:   public static class Destructor extends DestructorThread.Destructor {
68: 
69:     // Private C++ instance
70:     @DoNotStrip private volatile long mNativePointer;
71: 
72:     Destructor(Object referent) {
73:       super(referent);
74:     }
75: 
76:     @Override
77:     protected final void destruct() {
78:       // When invoked from the DestructorThread instead of resetNative,
79:       // the DestructorThread has exclusive ownership of the HybridData
80:       // so synchronization is not necessary.
````
- EN: Declares or extends types including `Destructor`.
- CN: 声明或扩展类型，包括 `Destructor`。
- EN: Implements callable logic such as `resetNative`, `isValid`, `destruct`.
- CN: 实现可调用逻辑，例如 `resetNative`, `isValid`, `destruct`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-87
````java
81:       deleteNative(mNativePointer);
82:       mNativePointer = 0;
83:     }
84: 
85:     static native void deleteNative(long pointer);
86:   }
87: }
````
- EN: Implements callable logic such as `deleteNative`.
- CN: 实现可调用逻辑，例如 `deleteNative`。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `HybridData` / 符号 `HybridData`
- Symbol `Destructor` / 符号 `Destructor`
- Symbol `resetNative` / 符号 `resetNative`
- Symbol `isValid` / 符号 `isValid`

## Dependencies / 依赖关系
- Java imports: `com.facebook.jni.annotations.DoNotStrip`, `com.facebook.soloader.nativeloader.NativeLoader`
- Java 导入: `com.facebook.jni.annotations.DoNotStrip`, `com.facebook.soloader.nativeloader.NativeLoader`
