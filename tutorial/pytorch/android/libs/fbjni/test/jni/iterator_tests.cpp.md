# iterator_tests.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/test/jni/iterator_tests.cpp`
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
17: #include <algorithm>
18: #include <unordered_map>
19: #include <unordered_set>
20: #include <vector>
21: 
22: #include <fbjni/fbjni.h>
23: 
24: #include "expect.h"
25: 
26: using namespace facebook::jni;
27: 
28: namespace {
29: 
30: template <typename K, typename V>
31: struct JHashMap : public JavaClass<JHashMap<K,V>, JMap<K,V>> {
32:   constexpr static auto kJavaDescriptor = "Ljava/util/HashMap;";
33: };
34: 
35: }
36: 
37: jboolean nativeTestListIterator(
38:     alias_ref<jclass>,
39:     alias_ref<JList<jstring>> jlist) {
40:   EXPECT(jlist);
````
- EN: Pulls in native headers such as `algorithm`, `unordered_map`, `unordered_set`, `vector`.
- CN: 引入原生头文件，例如 `algorithm`, `unordered_map`, `unordered_set`, `vector`。
- EN: Declares or extends types including `JHashMap`.
- CN: 声明或扩展类型，包括 `JHashMap`。
- EN: Implements callable logic such as `nativeTestListIterator`.
- CN: 实现可调用逻辑，例如 `nativeTestListIterator`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````cpp
41: 
42:   EXPECT(jlist->size() == 3);
43: 
44:   std::vector<std::string> vs1;
45:   for (const auto& elem : *jlist) {
46:     vs1.push_back(elem->toStdString());
47:   }
48: 
49:   EXPECT(vs1.size() == 3);
50:   EXPECT(vs1[0] == "red");
51:   EXPECT(vs1[1] == "green");
52:   EXPECT(vs1[2] == "blue");
53: 
54:   std::vector<std::string> vs2;
55:   std::transform(jlist->begin(), jlist->end(), std::back_inserter(vs2),
56:                  [](local_ref<jstring> elem) { return elem->toStdString(); });
57: 
58:   EXPECT(vs1 == vs2);
59: 
60:   std::vector<std::string> vs3 = { "red", "green", "blue" };
61: 
62:   EXPECT(vs1 == vs3);
63: 
64:   static auto iteratorMethod =
65:     JIterable<jstring>::javaClassStatic()->getMethod<JIterator<jstring>()>("iterator");
66:   auto iter = iteratorMethod(jlist);
67: 
68:   EXPECT(std::equal(iter->begin(), iter->end(), jlist->begin()));
69:   EXPECT(std::equal(iter->begin(), iter->end(), vs3.begin(),
70:                     [](const local_ref<jstring>& a, const std::string& b) {
71:                       return a->toStdString() == b;
72:                     }));
73: 
74:   return JNI_TRUE;
75: }
76: 
77: jboolean nativeTestMapIterator(
78:     alias_ref<jclass>,
79:     alias_ref<JMap<jstring, JInteger>> jmap) {
80:   EXPECT(jmap);
````
- EN: Implements callable logic such as `std::transform`, `EXPECT`, `nativeTestMapIterator`.
- CN: 实现可调用逻辑，例如 `std::transform`, `EXPECT`, `nativeTestMapIterator`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````cpp
 81: 
 82:   EXPECT(jmap->size() == 3);
 83: 
 84:   std::unordered_map<std::string, int> umap;
 85: 
 86:   for (const auto& entry : *jmap) {
 87:     umap[entry.first->toStdString()] = entry.second->intValue();
 88:   }
 89: 
 90:   EXPECT(umap.size() == 3);
 91: 
 92:   EXPECT(umap["one"] == 1);
 93:   EXPECT(umap["two"] == 2);
 94:   EXPECT(umap["four"] == 4);
 95: 
 96:   // For an empty map, any types will do; the cast will only happen on null
 97:   // pointers, which will always succeed.
 98:   typedef JHashMap<jclass, jthrowable> TestMap;
 99: 
100:   static auto testmapCtor = TestMap::javaClassStatic()->
101:     getConstructor<TestMap::javaobject()>();
102:   auto emptyMap = TestMap::javaClassStatic()->newObject(testmapCtor);
103:   EXPECT(emptyMap->size() == 0);
104: 
105:   JHashMap<jclass, jthrowable>::Iterator i1 = emptyMap->begin();
106:   JHashMap<jclass, jthrowable>::Iterator i2 = emptyMap->end();
107: 
108:   EXPECT(i1 == i2);
109: 
110:   return JNI_TRUE;
111: }
112: 
113: jboolean nativeTestIterateWrongType(
114:     alias_ref<jclass>,
115:     alias_ref<JMap<jstring, JInteger::javaobject>> jmap) {
116:   EXPECT(jmap);
117: 
118:   EXPECT(jmap->size() == 3);
119: 
120:   for (const auto& entry : *jmap) {
````
- EN: Implements callable logic such as `nativeTestIterateWrongType`.
- CN: 实现可调用逻辑，例如 `nativeTestIterateWrongType`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-160
````cpp
121:     (void) entry;
122:   }
123: 
124:   // The above should throw an exception.
125:   EXPECT(false);
126: 
127:   return JNI_FALSE;
128: }
129: 
130: jboolean nativeTestIterateNullKey(
131:     alias_ref<jclass>,
132:     alias_ref<JMap<jstring, JInteger>> jmap) {
133:   EXPECT(jmap);
134: 
135:   EXPECT(jmap->size() == 3);
136: 
137:   std::unordered_map<std::string, int> umap;
138:   std::unordered_set<int> nullValues;
139: 
140:   for (const auto& entry : *jmap) {
141:     if (entry.first) {
142:       umap[entry.first->toStdString()] = entry.second->intValue();
143:     } else {
144:       nullValues.insert(entry.second->intValue());
145:     }
146:   }
147: 
148:   EXPECT(umap.size() == 2);
149: 
150:   EXPECT(umap["one"] == 1);
151:   EXPECT(umap["four"] == 4);
152: 
153:   EXPECT(nullValues.size() == 1);
154:   EXPECT(*nullValues.begin() == -99);
155: 
156:   return JNI_TRUE;
157: }
158: 
159: jboolean nativeTestLargeMapIteration(
160:     alias_ref<jclass>,
````
- EN: Implements callable logic such as `nativeTestIterateNullKey`.
- CN: 实现可调用逻辑，例如 `nativeTestIterateNullKey`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 161-181
````cpp
161:     alias_ref<JMap<jstring, jstring>> jmap) {
162:   EXPECT(jmap);
163:   EXPECT(jmap->size() == 3000);
164: 
165:   for (const auto& entry : *jmap) {
166:     if (!entry.first) {
167:       return JNI_FALSE;
168:     }
169:   }
170:   return JNI_TRUE;
171: }
172: 
173: void RegisterIteratorTests() {
174:   registerNatives("com/facebook/jni/IteratorTests", {
175:     makeNativeMethod("nativeTestListIterator", nativeTestListIterator),
176:     makeNativeMethod("nativeTestMapIterator", nativeTestMapIterator),
177:     makeNativeMethod("nativeTestIterateWrongType", nativeTestIterateWrongType),
178:     makeNativeMethod("nativeTestIterateNullKey", nativeTestIterateNullKey),
179:     makeNativeMethod("nativeTestLargeMapIteration", nativeTestLargeMapIteration),
180:   });
181: }
````
- EN: Implements callable logic such as `RegisterIteratorTests`.
- CN: 实现可调用逻辑，例如 `RegisterIteratorTests`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Validation and test coverage / 校验与测试覆盖
- Symbol `JHashMap` / 符号 `JHashMap`
- Symbol `nativeTestListIterator` / 符号 `nativeTestListIterator`
- Symbol `std::transform` / 符号 `std::transform`
- Symbol `EXPECT` / 符号 `EXPECT`

## Dependencies / 依赖关系
- C/C++ includes: `algorithm`, `unordered_map`, `unordered_set`, `vector`, `fbjni/fbjni.h`, `expect.h`
- C/C++ 头文件: `algorithm`, `unordered_map`, `unordered_set`, `vector`, `fbjni/fbjni.h`, `expect.h`
