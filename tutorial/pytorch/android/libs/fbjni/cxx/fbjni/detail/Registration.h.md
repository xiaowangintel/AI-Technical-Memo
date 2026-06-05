# Registration.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/Registration.h`
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
18: 
19: #include <jni.h>
20: #include "References.h"
21: 
22: namespace facebook {
23: namespace jni {
24: 
25: namespace detail {
26: 
27: // This uses the real JNI function as a non-type template parameter to
28: // cause a (static member) function to exist with the same signature,
29: // but with try/catch exception translation.
30: template<typename F, F func, typename C, typename R, typename... Args>
31: constexpr void* exceptionWrapJNIMethod(R (*func0)(JNIEnv*, jobject, Args... args));
32: 
33: // Automatically wrap object argument, and don't take env explicitly.
34: template<typename F, F func, typename C, typename R, typename... Args>
35: constexpr void* exceptionWrapJNIMethod(R (*func0)(alias_ref<C>, Args... args));
36: 
37: // Extract C++ instance from object, and invoke given method on it,
38: template<typename M, M method, typename C, typename R, typename... Args>
39: constexpr void* exceptionWrapJNIMethod(R (C::*method0)(Args... args));
40: 
````
- EN: Pulls in native headers such as `jni.h`, `References.h`.
- CN: 引入原生头文件，例如 `jni.h`, `References.h`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````cpp
41: // This uses deduction to figure out the descriptor name if the types
42: // are primitive.
43: //
44: // Subtlety: must return a reference to the underlying constant global
45: // or the call in makeNativeMethod2 is using a pointer to a subobject
46: // of a temporary!
47: template<typename R, typename C, typename... Args>
48: constexpr const auto& /* detail::SimpleFixedString<_> */ makeDescriptor(R (*func)(JNIEnv*, C, Args... args));
49: 
50: // This uses deduction to figure out the descriptor name if the types
51: // are primitive.
52: template<typename R, typename C, typename... Args>
53: constexpr const auto& /* detail::SimpleFixedString<_> */ makeDescriptor(R (*func)(alias_ref<C>, Args... args));
54: 
55: // This uses deduction to figure out the descriptor name if the types
56: // are primitive.
57: template<typename R, typename C, typename... Args>
58: constexpr const auto& /* detail::SimpleFixedString<_> */ makeDescriptor(R (C::*method0)(Args... args));
59: 
60: template<typename F>
61: struct CriticalMethod;
62: 
63: template<typename R, typename ...Args>
64: struct CriticalMethod<R(*)(Args...)> {
65:   template<R(*func)(Args...)>
66:   static R call(alias_ref<jclass>, Args... args) noexcept;
67: 
68:   template<R(*func)(Args...)>
69:   inline static constexpr auto /* detail::SimpleFixedString<_> */ desc();
70: };
71: 
72: }
73: 
74: // We have to use macros here, because the func needs to be used
75: // as both a decltype expression argument and as a non-type template
76: // parameter, since C++ provides no way for translateException
77: // to deduce the type of its non-type template parameter.
78: // The empty string in the macros below ensures that name
79: // is always a string literal (because that syntax is only
80: // valid when name is a string literal).
````
- EN: Declares or extends types including `CriticalMethod`.
- CN: 声明或扩展类型，包括 `CriticalMethod`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````cpp
 81: //
 82: // We need to const_cast because non-Android platforms have
 83: // unnecessarily non-const members in JNINativeMethod.
 84: #define makeNativeMethod2(name, func)                                   \
 85:   { const_cast<char*>(name ""), const_cast<char*>(::facebook::jni::detail::makeDescriptor(&func).c_str()), \
 86:       ::facebook::jni::detail::exceptionWrapJNIMethod<decltype(&func), &func>(&func) }
 87: 
 88: #define makeNativeMethod3(name, desc, func)                             \
 89:   { const_cast<char*>(name ""), const_cast<char*>(desc),                \
 90:       ::facebook::jni::detail::exceptionWrapJNIMethod<decltype(&func), &func>(&func) }
 91: 
 92: // Variadic template hacks to get macros with different numbers of
 93: // arguments. Usage instructions are in CoreClasses.h.
 94: // Extra level of expansion is needed for Visual Studio to treat VA_ARGS as multiple args.
 95: #define FBJNI_MACRO_EXPAND(tokens) tokens
 96: #define makeNativeMethodN(a, b, c, count, ...) makeNativeMethod ## count
 97: #define makeNativeMethod(...) FBJNI_MACRO_EXPAND(makeNativeMethodN(__VA_ARGS__, 3, 2)(__VA_ARGS__))
 98: 
 99: 
100: // FAST CALLS / CRITICAL CALLS
101: // Android up to and including v7 supports "fast calls" by prefixing the method
102: // signature with an exclamation mark.
103: // Android v8+ supports fast calls by annotating methods:
104: // https://source.android.com/devices/tech/dalvik/improvements#faster-native-methods
105: //
106: // YOU ALMOST CERTAINLY DO NOT NEED THIS AND IT IS DANGEROUS.
107: // YOU ALMOST CERTAINLY DO NOT NEED THIS AND IT IS DANGEROUS.
108: // YOU ALMOST CERTAINLY DO NOT NEED THIS AND IT IS DANGEROUS.
109: // YOU ALMOST CERTAINLY DO NOT NEED THIS AND IT IS DANGEROUS.
110: // YOU ALMOST CERTAINLY DO NOT NEED THIS AND IT IS DANGEROUS.
111: //
112: // "Fast" calls are only on the order of a few dozen NANO-seconds faster than
113: // regular JNI calls. If your method does almost aaanything of consequence - if
114: // you loop, if you write to a log, if you call another method, if you even
115: // simply allocate or deallocate - then the method body will significantly
116: // outweigh the method overhead.
117: //
118: // The difference between a regular JNI method and a "FastJNI" method (as
119: // they're called inside the runtime) is that a FastJNI method doesn't mark the
120: // thread as executing native code, and by skipping that avoids the locking and
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-160
````cpp
121: // thread state check overhead of interacting with the Garbage Collector.
122: //
123: // To understand why this is dangerous, you need to understand a bit about the
124: // GC. In order to perform its work the GC needs to have at least one (usually
125: // two in modern implementations) "stop the world" checkpoints where it can
126: // guarantee that all managed-code execution is paused. The VM performs these
127: // checkpoints at allocations, method boundaries, and each backward branch (ie
128: // anytime you loop). When the GC wants to run, it will signal to all managed
129: // threads that they should pause at the next checkpoint, and then it will wait
130: // for every thread in the system to transition from the "runnable" state into a
131: // "waiting" state. Once every thread has stopped, the GC thread can perform the
132: // work it needs to and then it will trigger the execution threads to resume.
133: //
134: // JNI methods fit neatly into the above paradigm: They're still methods, so
135: // they perform GC checkpoints at method entry and method exit. JNI methods also
136: // perform checkpoints at any JNI boundary crossing - ie, any time you call
137: // GetObjectField etc. Because access to managed objects from native code is
138: // tightly controlled, the VM is able to mark threads executing native methods
139: // into a special "native" state which the GC is able to ignore: It knows they
140: // can't touch managed objects (without hitting a checkpoint) so it doesn't care
141: // about them.
142: //
143: // JNI critical methods don't perform that "runnable" -> "native" thread state
144: // transition. Skipping that transition allows them to shave about 20ns off
145: // their total execution time, but it means that the GC has to wait for them to
146: // complete before it can move forward. If a critical method begins blocking,
147: // say on a long loop, or an I/O operation, or on perhaps a mutex, then the GC
148: // will also block, and because the GC is blocking the entire rest of the VM
149: // (which is waiting on the GC) will block. If the critical method is blocking
150: // on a mutex that's already held by the GC - for example, the VM's internal
151: // weak_globals_lock_ which guards modifications to the weak global reference
152: // table (and is required in order to create or free a weak_ref<>) - then you
153: // have a system-wide deadlock.
154: 
155: // prefixes a JNI method signature as android "fast call".
156: #if defined(__ANDROID__) && defined(FBJNI_WITH_FAST_CALLS)
157: #define FBJNI_PREFIX_FAST_CALL(desc) ("!" + desc)
158: #else
159: #define FBJNI_PREFIX_FAST_CALL(desc) (desc)
160: #endif
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 161-189
````cpp
161: 
162: #define makeCriticalNativeMethod3(name, desc, func) (                   \
163:     []() -> JNINativeMethod {                                           \
164:       static constexpr auto descString = FBJNI_PREFIX_FAST_CALL(desc);  \
165:       return makeNativeMethod3(                                         \
166:         name,                                                           \
167:         descString.c_str(),                                             \
168:         ::facebook::jni::detail::CriticalMethod<decltype(&func)>::call<&func>); \
169:         }())
170: 
171: #define makeCriticalNativeMethod2(name, func)                                \
172:   makeCriticalNativeMethod3(                                                 \
173:     name,                                                                    \
174:     ::facebook::jni::detail::CriticalMethod<decltype(&func)>::desc<&func>(), \
175:     func)
176: 
177: #define makeCriticalNativeMethodN(a, b, c, count, ...) makeCriticalNativeMethod ## count
178: 
179: // YOU ALMOST CERTAINLY DO NOT NEED THIS AND IT IS DANGEROUS.
180: // YOU ALMOST CERTAINLY DO NOT NEED THIS AND IT IS DANGEROUS.
181: // YOU ALMOST CERTAINLY DO NOT NEED THIS AND IT IS DANGEROUS.
182: // YOU ALMOST CERTAINLY DO NOT NEED THIS AND IT IS DANGEROUS.
183: // YOU ALMOST CERTAINLY DO NOT NEED THIS AND IT IS DANGEROUS.
184: // See above for an explanation.
185: #define makeCriticalNativeMethod_DO_NOT_USE_OR_YOU_WILL_BE_FIRED(...) FBJNI_MACRO_EXPAND(makeCriticalNativeMethodN(__VA_ARGS__, 3, 2)(__VA_ARGS__))
186: 
187: }}
188: 
189: #include "Registration-inl.h"
````
- EN: Pulls in native headers such as `Registration-inl.h`.
- CN: 引入原生头文件，例如 `Registration-inl.h`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `CriticalMethod` / 符号 `CriticalMethod`

## Dependencies / 依赖关系
- C/C++ includes: `jni.h`, `References.h`, `Registration-inl.h`
- C/C++ 头文件: `jni.h`, `References.h`, `Registration-inl.h`
