# OnLoad.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/OnLoad.cpp`
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
17: #include <fbjni/fbjni.h>
18: #include <fbjni/NativeRunnable.h>
19: 
20: using namespace facebook::jni;
````
- EN: Pulls in native headers such as `fbjni/fbjni.h`, `fbjni/NativeRunnable.h`.
- CN: 引入原生头文件，例如 `fbjni/fbjni.h`, `fbjni/NativeRunnable.h`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 21-28
````cpp
21: 
22: JNIEXPORT jint JNI_OnLoad(JavaVM* vm, void* reserved) {
23:   return facebook::jni::initialize(vm, [] {
24:     HybridDataOnLoad();
25:     JNativeRunnable::OnLoad();
26:     ThreadScope::OnLoad();
27:   });
28: }
````
- EN: Implements callable logic such as `JNI_OnLoad`.
- CN: 实现可调用逻辑，例如 `JNI_OnLoad`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `JNI_OnLoad` / 符号 `JNI_OnLoad`

## Dependencies / 依赖关系
- C/C++ includes: `fbjni/fbjni.h`, `fbjni/NativeRunnable.h`
- C/C++ 头文件: `fbjni/fbjni.h`, `fbjni/NativeRunnable.h`
