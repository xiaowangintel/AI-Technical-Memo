# Exceptions.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/Exceptions.h`
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
17: /**
18:  * @file Exceptions.h
19:  *
20:  * After invoking a JNI function that can throw a Java exception, the macro
21:  * @ref FACEBOOK_JNI_THROW_PENDING_EXCEPTION() or @ref FACEBOOK_JNI_THROW_EXCEPTION_IF()
22:  * should be invoked.
23:  *
24:  * IMPORTANT! IMPORTANT! IMPORTANT! IMPORTANT! IMPORTANT! IMPORTANT! IMPORTANT! IMPORTANT!
25:  * To use these methods you MUST call initExceptionHelpers() when your library is loaded.
26:  */
27: 
28: #pragma once
29: 
30: #include <stdexcept>
31: #include <string>
32: 
33: #include <jni.h>
34: 
35: #include "Common.h"
36: #include "References.h"
37: #include "CoreClasses.h"
38: 
39: #if defined(__ANDROID__) && defined(__ARM_ARCH_5TE__) && !defined(FBJNI_NO_EXCEPTION_PTR)
40: // ARMv5 NDK does not support exception_ptr so we cannot use that when building for it.
````
- EN: Pulls in native headers such as `stdexcept`, `string`, `jni.h`, `Common.h`.
- CN: 引入原生头文件，例如 `stdexcept`, `string`, `jni.h`, `Common.h`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````cpp
41: #define FBJNI_NO_EXCEPTION_PTR
42: #endif
43: 
44: namespace facebook {
45: namespace jni {
46: 
47: class JThrowable;
48: 
49: class JCppException : public JavaClass<JCppException, JThrowable> {
50:  public:
51:   static auto constexpr kJavaDescriptor = "Lcom/facebook/jni/CppException;";
52: 
53:   static local_ref<JCppException> create(const char* str) {
54:     return newInstance(make_jstring(str));
55:   }
56: 
57:   static local_ref<JCppException> create(const std::exception& ex) {
58:     return newInstance(make_jstring(ex.what()));
59:   }
60: };
61: 
62: // JniException ////////////////////////////////////////////////////////////////////////////////////
63: 
64: /**
65:  * This class wraps a Java exception into a C++ exception; if the exception is routed back
66:  * to the Java side, it can be unwrapped and just look like a pure Java interaction. The class
67:  * is resilient to errors while creating the exception, falling back to some pre-allocated
68:  * exceptions if a new one cannot be allocated or populated.
69:  *
70:  * Note: the what() method of this class is not thread-safe (t6900503).
71:  */
72: class JniException : public std::exception {
73:  public:
74:   JniException();
75:   ~JniException() override;
76: 
77:   explicit JniException(alias_ref<jthrowable> throwable);
78: 
79:   JniException(JniException &&rhs);
80: 
````
- EN: Declares or extends types including `JThrowable`, `JCppException`, `wraps`, `is`.
- CN: 声明或扩展类型，包括 `JThrowable`, `JCppException`, `wraps`, `is`。
- EN: Implements callable logic such as `create`.
- CN: 实现可调用逻辑，例如 `create`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````cpp
 81:   JniException(const JniException &other);
 82: 
 83:   local_ref<JThrowable> getThrowable() const noexcept;
 84: 
 85:   const char* what() const noexcept override;
 86: 
 87:   void setJavaException() const noexcept;
 88: 
 89:  private:
 90:   global_ref<JThrowable> throwable_;
 91:   mutable std::string what_;
 92:   mutable bool isMessageExtracted_;
 93:   const static std::string kExceptionMessageFailure_;
 94: 
 95:   void populateWhat() const noexcept;
 96: };
 97: 
 98: // Exception throwing & translating functions //////////////////////////////////////////////////////
 99: 
100: // Functions that throw C++ exceptions
101: 
102: static const int kMaxExceptionMessageBufferSize = 512;
103: 
104: // These methods are the preferred way to throw a Java exception from
105: // a C++ function.  They create and throw a C++ exception which wraps
106: // a Java exception, so the C++ flow is interrupted. Then, when
107: // translatePendingCppExceptionToJavaException is called at the
108: // topmost level of the native stack, the wrapped Java exception is
109: // thrown to the java caller.
110: template<typename... Args>
111: [[noreturn]] void throwNewJavaException(const char* throwableName, const char* fmt, Args... args) {
112:   char msg[kMaxExceptionMessageBufferSize];
113:   snprintf(msg, kMaxExceptionMessageBufferSize, fmt, args...);
114:   throwNewJavaException(throwableName, msg);
115: }
116: 
117: // Identifies any pending C++ exception and throws it as a Java exception. If the exception can't
118: // be thrown, it aborts the program.
119: void translatePendingCppExceptionToJavaException();
120: 
````
- EN: Implements callable logic such as `throwNewJavaException`.
- CN: 实现可调用逻辑，例如 `throwNewJavaException`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-137
````cpp
121: #ifndef FBJNI_NO_EXCEPTION_PTR
122: local_ref<JThrowable> getJavaExceptionForCppException(std::exception_ptr ptr);
123: #endif
124: 
125: /***
126:  * The stack returned may include build ids.  It may be beneficial to
127:  * call lyra::setLibraryIdentifierFunction before calling this if
128:  * build ids are desirable.
129:  */
130: local_ref<JThrowable> getJavaExceptionForCppBackTrace();
131: 
132: local_ref<JThrowable> getJavaExceptionForCppBackTrace(const char* msg);
133: 
134: // For convenience, some exception names in java.lang are available here.
135: const char* const gJavaLangIllegalArgumentException = "java/lang/IllegalArgumentException";
136: 
137: }}
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `JThrowable` / 符号 `JThrowable`
- Symbol `JCppException` / 符号 `JCppException`
- Symbol `wraps` / 符号 `wraps`
- Symbol `is` / 符号 `is`

## Dependencies / 依赖关系
- C/C++ includes: `stdexcept`, `string`, `jni.h`, `Common.h`, `References.h`, `CoreClasses.h`
- C/C++ 头文件: `stdexcept`, `string`, `jni.h`, `Common.h`, `References.h`, `CoreClasses.h`
