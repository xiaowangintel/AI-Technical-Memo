# NativeRunnable.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/NativeRunnable.h`
- Repository: `pytorch`
- Purpose (EN): Implements JNI bridge abstractions used by Android-facing native code.
- 用途 (CN): 实现供 Android 原生代码使用的 JNI 桥接抽象。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
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
19: #include <fbjni/fbjni.h>
20: 
````
- EN: Pulls in native headers such as `fbjni/fbjni.h`.
- CN: 引入原生头文件，例如 `fbjni/fbjni.h`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 21-40
````cpp
21: #include <functional>
22: 
23: namespace facebook {
24: namespace jni {
25: 
26: struct JRunnable : public JavaClass<JRunnable> {
27:   static auto constexpr kJavaDescriptor = "Ljava/lang/Runnable;";
28: };
29: 
30: struct JNativeRunnable : public HybridClass<JNativeRunnable, JRunnable> {
31:  public:
32:   static auto constexpr kJavaDescriptor = "Lcom/facebook/jni/NativeRunnable;";
33: 
34:   JNativeRunnable(std::function<void()>&& runnable) : runnable_(std::move(runnable)) {}
35: 
36:   static void OnLoad() {
37:     registerHybrid({
38:         makeNativeMethod("run", JNativeRunnable::run),
39:       });
40:   }
````
- EN: Pulls in native headers such as `functional`.
- CN: 引入原生头文件，例如 `functional`。
- EN: Declares or extends types including `JRunnable`, `JNativeRunnable`.
- CN: 声明或扩展类型，包括 `JRunnable`, `JNativeRunnable`。
- EN: Implements callable logic such as `JNativeRunnable`, `OnLoad`.
- CN: 实现可调用逻辑，例如 `JNativeRunnable`, `OnLoad`。

### Lines 41-52
````cpp
41: 
42:   void run() {
43:     runnable_();
44:   }
45: 
46:  private:
47:   std::function<void()> runnable_;
48: };
49: 
50: 
51: } // namespace jni
52: } // namespace facebook
````
- EN: Implements callable logic such as `run`.
- CN: 实现可调用逻辑，例如 `run`。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `JRunnable` / 符号 `JRunnable`
- Symbol `JNativeRunnable` / 符号 `JNativeRunnable`
- Symbol `OnLoad` / 符号 `OnLoad`
- Symbol `run` / 符号 `run`

## Dependencies / 依赖关系
- C/C++ includes: `fbjni/fbjni.h`, `functional`
- C/C++ 头文件: `fbjni/fbjni.h`, `functional`
