# Environment.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/Environment.h`
- Repository: `pytorch`
- Purpose (EN): Implements JNI bridge abstractions used by Android-facing native code.
- 用途 (CN): 实现供 Android 原生代码使用的 JNI 桥接抽象。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
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
17: #pragma once
18: #include <functional>
19: #include <string>
20: #include <jni.h>
21: 
22: 
23: namespace facebook {
24: namespace jni {
25: 
26: // Keeps a thread-local reference to the current thread's JNIEnv.
27: struct Environment {
28:   // Throws a std::runtime_error if this thread isn't attached to the JVM
29:   // TODO(T6594868) Benchmark against raw JNI access
30:   static JNIEnv* current();
31:   static void initialize(JavaVM* vm);
32: 
33:   // There are subtle issues with calling the next functions directly. It is
34:   // much better to always use a ThreadScope to manage attaching/detaching for
35:   // you.
36:   static JNIEnv* ensureCurrentThreadIsAttached();
37: 
38:   // To check if a Java VM is available at all in this environment.
39:   // Note that this doesn't check if it is attached to this thread,
40:   // it checks whether one is available at all.
````
- EN: Pulls in native headers such as `functional`, `string`, `jni.h`.
- CN: 引入原生头文件，例如 `functional`, `string`, `jni.h`。
- EN: Declares or extends types including `Environment`.
- CN: 声明或扩展类型，包括 `Environment`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````cpp
41:   static bool isGlobalJvmAvailable();
42: };
43: 
44: namespace detail {
45: 
46: // This will return null the thread isn't attached to the VM, or if
47: // fbjni has never been initialized with a VM at all.  You probably
48: // shouldn't be using this.
49: JNIEnv* currentOrNull();
50: 
51: /**
52:  * If there's thread-local data, it's a pointer to one of these.  The
53:  * instance is a member of JniEnvCacher or ThreadScope, and lives on
54:  * the stack.
55:  */
56: struct TLData {
57:   // This is modified only by JniEnvCacher, and is guaranteed to be
58:   // valid if set, and refer to an env which originated from a JNI
59:   // call into C++.
60:   JNIEnv* env;
61:   // This is modified only by ThreadScope, and is set only if an
62:   // instance of ThreadScope which attached is on the stack.
63:   bool attached;
64: };
65: 
66: /**
67:  * RAII object which manages a cached JNIEnv* value.  A Value is only
68:  * cached if it is guaranteed safe, which means when C++ is called
69:  * from a registered fbjni function.
70:  */
71: class JniEnvCacher {
72: public:
73:   JniEnvCacher(JNIEnv* env);
74:   JniEnvCacher(JniEnvCacher&) = delete;
75:   JniEnvCacher(JniEnvCacher&&) = default;
76:   JniEnvCacher& operator=(JniEnvCacher&) = delete;
77:   JniEnvCacher& operator=(JniEnvCacher&&) = delete;
78:   ~JniEnvCacher();
79: 
80: private:
````
- EN: Declares or extends types including `TLData`, `JniEnvCacher`.
- CN: 声明或扩展类型，包括 `TLData`, `JniEnvCacher`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````cpp
 81:   // If this flag is set, then, this object needs to clear the cache.
 82:   bool thisCached_;
 83: 
 84:   // The thread local pointer may point here.
 85:   detail::TLData data_;
 86: };
 87: 
 88: }
 89: 
 90: /**
 91:  * RAII Object that attaches a thread to the JVM. Failing to detach from a thread before it
 92:  * exits will cause a crash, as will calling Detach an extra time, and this guard class helps
 93:  * keep that straight. In addition, it remembers whether it performed the attach or not, so it
 94:  * is safe to nest it with itself or with non-fbjni code that manages the attachment correctly.
 95:  *
 96:  * Potential concerns:
 97:  *  - Attaching to the JVM is fast (~100us on MotoG), but ideally you would attach while the
 98:  *    app is not busy.
 99:  *  - Having a thread detach at arbitrary points is not safe in Dalvik; you need to be sure that
100:  *    there is no Java code on the current stack or you run the risk of a crash like:
101:  *      ERROR: detaching thread with interp frames (count=18)
102:  *    (More detail at https://groups.google.com/forum/#!topic/android-ndk/2H8z5grNqjo)
103:  *    ThreadScope won't do a detach if the thread was already attached before the guard is
104:  *    instantiated, but there's probably some usage that could trip this up.
105:  *  - Newly attached C++ threads only get the bootstrap class loader -- i.e. java language
106:  *    classes, not any of our application's classes. This will be different behavior than threads
107:  *    that were initiated on the Java side. A workaround is to pass a global reference for a
108:  *    class or instance to the new thread; this bypasses the need for the class loader.
109:  *    (See http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/invocation.html#attach_current_thread)
110:  *    If you need access to the application's classes, you can use ThreadScope::WithClassLoader.
111:  *  - If fbjni has never been initialized, there will be no JavaVM object to attach with.
112:  *    In that case, a std::runtime_error will be thrown.  This is only likely to happen in a
113:  *    standalone C++ application, or if Environment::initialize is not used.
114:  */
115: class ThreadScope {
116:  public:
117:   ThreadScope();
118:   ThreadScope(ThreadScope&) = delete;
119:   ThreadScope(ThreadScope&&) = default;
120:   ThreadScope& operator=(ThreadScope&) = delete;
````
- EN: Declares or extends types including `helps`, `loader`, `or`, `ThreadScope`.
- CN: 声明或扩展类型，包括 `helps`, `loader`, `or`, `ThreadScope`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-143
````cpp
121:   ThreadScope& operator=(ThreadScope&&) = delete;
122:   ~ThreadScope();
123: 
124:   /**
125:    * This runs the closure in a scope with fbjni's classloader. This should be
126:    * the same classloader as the rest of the application and thus anything
127:    * running in the closure will have access to the same classes as in a normal
128:    * java-create thread.
129:    */
130:   static void WithClassLoader(std::function<void()>&& runnable);
131: 
132:   static void OnLoad();
133: 
134:  private:
135:   // If this flag is set, then this object needs to detach.
136:   bool thisAttached_;
137: 
138:   // The thread local pointer may point here.
139:   detail::TLData data_;
140: };
141: 
142: }
143: }
````
- EN: This range contributes implementation details for the file goal: Implements JNI bridge abstractions used by Android-facing native code.
- CN: 该范围为文件目标提供实现细节：实现供 Android 原生代码使用的 JNI 桥接抽象。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `Environment` / 符号 `Environment`
- Symbol `TLData` / 符号 `TLData`
- Symbol `JniEnvCacher` / 符号 `JniEnvCacher`
- Symbol `helps` / 符号 `helps`

## Dependencies / 依赖关系
- C/C++ includes: `functional`, `string`, `jni.h`
- C/C++ 头文件: `functional`, `string`, `jni.h`
