# Iterator-inl.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/Iterator-inl.h`
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
19: namespace facebook {
20: namespace jni {
21: 
22: namespace detail {
23: 
24: template <typename E>
25: struct IteratorHelper : public JavaClass<IteratorHelper<E>> {
26:   constexpr static auto kJavaDescriptor = "Lcom/facebook/jni/IteratorHelper;";
27: 
28:   typedef local_ref<E> value_type;
29:   typedef ptrdiff_t difference_type;
30:   typedef value_type* pointer;
31:   typedef value_type& reference;
32:   typedef std::forward_iterator_tag iterator_category;
33: 
34:   typedef JavaClass<IteratorHelper<E>> JavaBase_;
35: 
36:   bool hasNext() const {
37:     static auto hasNextMethod =
38:       JavaBase_::javaClassStatic()->template getMethod<jboolean()>("hasNext");
39:     return hasNextMethod(JavaBase_::self());
40:   }
````
- EN: Declares or extends types including `IteratorHelper`.
- CN: 声明或扩展类型，包括 `IteratorHelper`。
- EN: Implements callable logic such as `hasNext`.
- CN: 实现可调用逻辑，例如 `hasNext`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 41-80
````cpp
41: 
42:   value_type next() {
43:     static auto elementField =
44:       JavaBase_::javaClassStatic()->template getField<jobject>("mElement");
45:     return dynamic_ref_cast<E>(JavaBase_::getFieldValue(elementField));
46:   }
47: 
48:   static void reset(value_type& v) {
49:     v.reset();
50:   }
51: };
52: 
53: template <typename K, typename V>
54: struct MapIteratorHelper : public JavaClass<MapIteratorHelper<K,V>> {
55:   constexpr static auto kJavaDescriptor = "Lcom/facebook/jni/MapIteratorHelper;";
56: 
57:   typedef std::pair<local_ref<K>, local_ref<V>> value_type;
58: 
59:   typedef JavaClass<MapIteratorHelper<K,V>> JavaBase_;
60: 
61:   bool hasNext() const {
62:     static auto hasNextMethod =
63:       JavaBase_::javaClassStatic()->template getMethod<jboolean()>("hasNext");
64:     return hasNextMethod(JavaBase_::self());
65:   }
66: 
67:   value_type next() {
68:     static auto keyField = JavaBase_::javaClassStatic()->template getField<jobject>("mKey");
69:     static auto valueField = JavaBase_::javaClassStatic()->template getField<jobject>("mValue");
70:     return std::make_pair(dynamic_ref_cast<K>(JavaBase_::getFieldValue(keyField)),
71:                           dynamic_ref_cast<V>(JavaBase_::getFieldValue(valueField)));
72:   }
73: 
74:   static void reset(value_type& v) {
75:     v.first.reset();
76:     v.second.reset();
77:   }
78: };
79: 
80: template <typename T>
````
- EN: Declares or extends types including `MapIteratorHelper`.
- CN: 声明或扩展类型，包括 `MapIteratorHelper`。
- EN: Implements callable logic such as `next`, `reset`, `hasNext`.
- CN: 实现可调用逻辑，例如 `next`, `reset`, `hasNext`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````cpp
 81: class Iterator {
 82:  public:
 83:   typedef typename T::value_type value_type;
 84:   typedef ptrdiff_t difference_type;
 85:   typedef value_type* pointer;
 86:   typedef value_type& reference;
 87:   typedef std::input_iterator_tag iterator_category;
 88: 
 89:   // begin ctor
 90:   Iterator(global_ref<typename T::javaobject>&& helper)
 91:       : helper_(std::move(helper))
 92:       , i_(-1) {
 93:     ++(*this);
 94:   }
 95: 
 96:   // end ctor
 97:   Iterator()
 98:       : i_(-1) {}
 99: 
100:   bool operator==(const Iterator& it) const { return i_ == it.i_; }
101:   bool operator!=(const Iterator& it) const { return !(*this == it); }
102:   const value_type& operator*() const { assert(i_ != -1); return entry_; }
103:   const value_type* operator->() const { assert(i_ != -1); return &entry_; }
104:   Iterator& operator++() {  // preincrement
105:     bool hasNext = helper_->hasNext();
106:     if (hasNext) {
107:       ++i_;
108:       entry_ = helper_->next();
109:     } else {
110:       i_ = -1;
111:       helper_->reset(entry_);
112:     }
113:     return *this;
114:   }
115:   Iterator operator++(int) {  // postincrement
116:     Iterator ret;
117:     ret.i_ = i_;
118:     ret.entry_ = std::move(entry_);
119:     ++(*this);
120:     return ret;
````
- EN: Declares or extends types including `Iterator`.
- CN: 声明或扩展类型，包括 `Iterator`。
- EN: Implements callable logic such as `Iterator`.
- CN: 实现可调用逻辑，例如 `Iterator`。
- EN: Includes validation, assertions, or defensive checks.
- CN: 包含校验、断言或防御性检查。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-160
````cpp
121:   }
122: 
123:   global_ref<T> helper_;
124:   // set to -1 at end
125:   std::ptrdiff_t i_;
126:   value_type entry_;
127: };
128: 
129: }
130: 
131: template <typename E>
132: struct JIterator<E>::Iterator : public detail::Iterator<detail::IteratorHelper<E>> {
133:   using detail::Iterator<detail::IteratorHelper<E>>::Iterator;
134: };
135: 
136: template <typename E>
137: typename JIterator<E>::Iterator JIterator<E>::begin() const {
138:   static auto ctor = detail::IteratorHelper<E>::javaClassStatic()->
139:     template getConstructor<typename detail::IteratorHelper<E>::javaobject(
140:                               typename JIterator<E>::javaobject)>();
141:   return Iterator(
142:     make_global(
143:       detail::IteratorHelper<E>::javaClassStatic()->newObject(ctor, this->self())));
144: }
145: 
146: template <typename E>
147: typename JIterator<E>::Iterator JIterator<E>::end() const {
148:   return Iterator();
149: }
150: 
151: template <typename E>
152: struct JIterable<E>::Iterator : public detail::Iterator<detail::IteratorHelper<E>> {
153:   using detail::Iterator<detail::IteratorHelper<E>>::Iterator;
154: };
155: 
156: template <typename E>
157: typename JIterable<E>::Iterator JIterable<E>::begin() const {
158:   static auto ctor = detail::IteratorHelper<E>::javaClassStatic()->
159:     template getConstructor<typename detail::IteratorHelper<E>::javaobject(
160:                               typename JIterable<E>::javaobject)>();
````
- EN: Declares or extends types including `JIterator`, `JIterable`.
- CN: 声明或扩展类型，包括 `JIterator`, `JIterable`。
- EN: Implements callable logic such as `begin`, `end`.
- CN: 实现可调用逻辑，例如 `begin`, `end`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 161-200
````cpp
161:   return Iterator(
162:     make_global(
163:       detail::IteratorHelper<E>::javaClassStatic()->newObject(ctor, this->self())));
164: }
165: 
166: template <typename E>
167: typename JIterable<E>::Iterator JIterable<E>::end() const {
168:   return Iterator();
169: }
170: 
171: template <typename E>
172: size_t JCollection<E>::size() const {
173:   static auto sizeMethod =
174:     JCollection<E>::javaClassStatic()->template getMethod<jint()>("size");
175:   return sizeMethod(this->self());
176: }
177: 
178: template <typename K, typename V>
179: struct JMap<K,V>::Iterator : public detail::Iterator<detail::MapIteratorHelper<K,V>> {
180:   using detail::Iterator<detail::MapIteratorHelper<K,V>>::Iterator;
181: };
182: 
183: template <typename K, typename V>
184: size_t JMap<K,V>::size() const {
185:   static auto sizeMethod =
186:     JMap<K,V>::javaClassStatic()->template getMethod<jint()>("size");
187:   return sizeMethod(this->self());
188: }
189: 
190: template <typename K, typename V>
191: typename JMap<K,V>::Iterator JMap<K,V>::begin() const {
192:   static auto ctor = detail::MapIteratorHelper<K,V>::javaClassStatic()->
193:     template getConstructor<typename detail::MapIteratorHelper<K,V>::javaobject(
194:                               typename JMap<K,V>::javaobject)>();
195:   return Iterator(
196:     make_global(
197:       detail::MapIteratorHelper<K,V>::javaClassStatic()->newObject(ctor, this->self())));
198: }
199: 
200: template <typename K, typename V>
````
- EN: Declares or extends types including `JMap`.
- CN: 声明或扩展类型，包括 `JMap`。
- EN: Implements callable logic such as `end`, `size`, `begin`.
- CN: 实现可调用逻辑，例如 `end`, `size`, `begin`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 201-206
````cpp
201: typename JMap<K,V>::Iterator JMap<K,V>::end() const {
202:   return Iterator();
203: }
204: 
205: }
206: }
````
- EN: Implements callable logic such as `end`.
- CN: 实现可调用逻辑，例如 `end`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `IteratorHelper` / 符号 `IteratorHelper`
- Symbol `MapIteratorHelper` / 符号 `MapIteratorHelper`
- Symbol `Iterator` / 符号 `Iterator`
- Symbol `JIterator` / 符号 `JIterator`

## Dependencies / 依赖关系
- EN: No obvious direct dependency was detected from import/include style statements in this file.
- CN: 未从该文件中的导入/包含语句检测到明显的直接依赖。
