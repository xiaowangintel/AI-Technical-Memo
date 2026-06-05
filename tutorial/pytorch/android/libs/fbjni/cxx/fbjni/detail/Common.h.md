# Common.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/Common.h`
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
17: /** @file Common.h
18:  *
19:  * Defining the stuff that don't deserve headers of their own...
20:  */
21: 
22: #pragma once
23: 
24: #include <functional>
25: 
26: #include <jni.h>
27: 
28: #ifdef FBJNI_DEBUG_REFS
29: # ifdef __ANDROID__
30: #  include <android/log.h>
31: # else
32: #  include <cstdio>
33: # endif
34: #endif
35: 
36: // If a pending JNI Java exception is found, wraps it in a JniException object and throws it as
37: // a C++ exception.
38: #define FACEBOOK_JNI_THROW_PENDING_EXCEPTION() \
39:   ::facebook::jni::throwPendingJniExceptionAsCppException()
40: 
````
- EN: Pulls in native headers such as `functional`, `jni.h`.
- CN: 引入原生头文件，例如 `functional`, `jni.h`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````cpp
41: // If the condition is true, throws a JniException object, which wraps the pending JNI Java
42: // exception if any. If no pending exception is found, throws a JniException object that wraps a
43: // RuntimeException throwable.
44: #define FACEBOOK_JNI_THROW_EXCEPTION_IF(CONDITION) \
45:   ::facebook::jni::throwCppExceptionIf(CONDITION)
46: 
47: /// @cond INTERNAL
48: 
49: namespace facebook {
50: namespace jni {
51: 
52: void throwPendingJniExceptionAsCppException();
53: void throwCppExceptionIf(bool condition);
54: 
55: [[noreturn]] void throwNewJavaException(jthrowable);
56: [[noreturn]] void throwNewJavaException(const char* throwableName, const char* msg);
57: template<typename... Args>
58: [[noreturn]] void throwNewJavaException(const char* throwableName, const char* fmt, Args... args);
59: 
60: 
61: /**
62:  * This needs to be called at library load time, typically in your JNI_OnLoad method.
63:  *
64:  * The intended use is to return the result of initialize() directly
65:  * from JNI_OnLoad and to do nothing else there. Library specific
66:  * initialization code should go in the function passed to initialize
67:  * (which can be, and probably should be, a C++ lambda). This approach
68:  * provides correct error handling and translation errors during
69:  * initialization into Java exceptions when appropriate.
70:  *
71:  * Failure to call this will cause your code to crash in a remarkably
72:  * unhelpful way (typically a segfault) while trying to handle an exception
73:  * which occurs later.
74:  */
75: jint initialize(JavaVM*, std::function<void()>&&) noexcept;
76: 
77: namespace internal {
78: 
79: // Define to get extremely verbose logging of references and to enable reference stats
80: #ifdef FBJNI_DEBUG_LOG_REFS
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-100
````cpp
 81: template<typename... Args>
 82: inline void dbglog(const char* msg, Args... args) {
 83: # ifdef __ANDROID__
 84:   __android_log_print(ANDROID_LOG_VERBOSE, "fbjni_dbg", msg, args...);
 85: # else
 86:   std::fprintf(stderr, msg, args...);
 87: # endif
 88: }
 89: 
 90: #else
 91: 
 92: template<typename... Args>
 93: inline void dbglog(const char*, Args...) {
 94: }
 95: 
 96: #endif
 97: 
 98: }}}
 99: 
100: /// @endcond
````
- EN: Implements callable logic such as `dbglog`.
- CN: 实现可调用逻辑，例如 `dbglog`。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `dbglog` / 符号 `dbglog`

## Dependencies / 依赖关系
- C/C++ includes: `functional`, `jni.h`
- C/C++ 头文件: `functional`, `jni.h`
