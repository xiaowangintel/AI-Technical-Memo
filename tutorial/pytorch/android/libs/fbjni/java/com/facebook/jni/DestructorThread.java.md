# DestructorThread.java — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/java/com/facebook/jni/DestructorThread.java`
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
19: import java.lang.ref.PhantomReference;
20: import java.lang.ref.ReferenceQueue;
21: import java.util.concurrent.atomic.AtomicReference;
22: 
23: /**
24:  * A thread which invokes the "destruct" routine for objects after they have been garbage collected.
25:  *
26:  * <p>An object which needs to be destructed should create a static subclass of {@link Destructor}.
27:  * Once the referent object is garbage collected, the DestructorThread will callback to the {@link
28:  * Destructor#destruct()} method.
29:  *
30:  * <p>The underlying thread in DestructorThread starts when the first Destructor is constructed and
31:  * then runs indefinitely.
32:  */
33: public class DestructorThread {
34: 
35:   /**
36:    * N.B The Destructor <b>SHOULD NOT</b> refer back to its referent object either explicitly or
37:    * implicitly (for example, as a non-static inner class). This will create a reference cycle where
38:    * the referent object will never be garbage collected.
39:    */
40:   public abstract static class Destructor extends PhantomReference<Object> {
````
- EN: Handles module imports such as `java.lang.ref.PhantomReference;`, `java.lang.ref.ReferenceQueue;`, `java.util.concurrent.atomic.AtomicReference;`.
- CN: 处理模块导入，例如 `java.lang.ref.PhantomReference;`, `java.lang.ref.ReferenceQueue;`, `java.util.concurrent.atomic.AtomicReference;`。
- EN: Imports Java types such as `java.lang.ref.PhantomReference`, `java.lang.ref.ReferenceQueue`, `java.util.concurrent.atomic.AtomicReference`.
- CN: 导入 Java 类型，例如 `java.lang.ref.PhantomReference`, `java.lang.ref.ReferenceQueue`, `java.util.concurrent.atomic.AtomicReference`。
- EN: Declares or extends types including `DestructorThread`, `Destructor`.
- CN: 声明或扩展类型，包括 `DestructorThread`, `Destructor`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````java
41: 
42:     private Destructor next;
43:     private Destructor previous;
44: 
45:     public Destructor(Object referent) {
46:       super(referent, sReferenceQueue);
47:       sDestructorStack.push(this);
48:     }
49: 
50:     private Destructor() {
51:       super(null, sReferenceQueue);
52:     }
53: 
54:     /** Callback which is invoked when the original object has been garbage collected. */
55:     protected abstract void destruct();
56:   }
57: 
58:   /** A list to keep all active Destructors in memory confined to the Destructor thread. */
59:   private static final DestructorList sDestructorList;
60:   /** A thread safe stack where new Destructors are placed before being add to sDestructorList. */
61:   private static final DestructorStack sDestructorStack;
62: 
63:   private static final ReferenceQueue sReferenceQueue;
64:   private static final Thread sThread;
65: 
66:   static {
67:     sDestructorStack = new DestructorStack();
68:     sReferenceQueue = new ReferenceQueue();
69:     sDestructorList = new DestructorList();
70:     sThread =
71:         new Thread("HybridData DestructorThread") {
72:           @Override
73:           public void run() {
74:             while (true) {
75:               try {
76:                 Destructor current = (Destructor) sReferenceQueue.remove();
77:                 current.destruct();
78: 
79:                 // If current is in the sDestructorStack,
80:                 // transfer all the Destructors in the stack to the list.
````
- EN: Implements callable logic such as `destruct`, `run`.
- CN: 实现可调用逻辑，例如 `destruct`, `run`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-120
````java
 81:                 if (current.previous == null) {
 82:                   sDestructorStack.transferAllToList();
 83:                 }
 84: 
 85:                 DestructorList.drop(current);
 86:               } catch (InterruptedException e) {
 87:                 // Continue. This thread should never be terminated.
 88:               }
 89:             }
 90:           }
 91:         };
 92: 
 93:     sThread.start();
 94:   }
 95: 
 96:   private static class Terminus extends Destructor {
 97:     @Override
 98:     protected void destruct() {
 99:       throw new IllegalStateException("Cannot destroy Terminus Destructor.");
100:     }
101:   }
102: 
103:   /** This is a thread safe, lock-free Treiber-like Stack of Destructors. */
104:   private static class DestructorStack {
105:     private final AtomicReference<Destructor> mHead = new AtomicReference<>();
106: 
107:     public void push(Destructor newHead) {
108:       Destructor oldHead;
109:       do {
110:         oldHead = mHead.get();
111:         newHead.next = oldHead;
112:       } while (!mHead.compareAndSet(oldHead, newHead));
113:     }
114: 
115:     public void transferAllToList() {
116:       Destructor current = mHead.getAndSet(null);
117:       while (current != null) {
118:         Destructor next = current.next;
119:         sDestructorList.enqueue(current);
120:         current = next;
````
- EN: Declares or extends types including `Terminus`, `DestructorStack`.
- CN: 声明或扩展类型，包括 `Terminus`, `DestructorStack`。
- EN: Implements callable logic such as `destruct`, `push`, `transferAllToList`.
- CN: 实现可调用逻辑，例如 `destruct`, `push`, `transferAllToList`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-148
````java
121:       }
122:     }
123:   }
124: 
125:   /** A doubly-linked list of Destructors. */
126:   private static class DestructorList {
127:     private final Destructor mHead;
128: 
129:     public DestructorList() {
130:       mHead = new Terminus();
131:       mHead.next = new Terminus();
132:       mHead.next.previous = mHead;
133:     }
134: 
135:     public void enqueue(Destructor current) {
136:       current.next = mHead.next;
137:       mHead.next = current;
138: 
139:       current.next.previous = current;
140:       current.previous = mHead;
141:     }
142: 
143:     private static void drop(Destructor current) {
144:       current.next.previous = current.previous;
145:       current.previous.next = current.next;
146:     }
147:   }
148: }
````
- EN: Declares or extends types including `DestructorList`.
- CN: 声明或扩展类型，包括 `DestructorList`。
- EN: Implements callable logic such as `enqueue`, `drop`.
- CN: 实现可调用逻辑，例如 `enqueue`, `drop`。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `DestructorThread` / 符号 `DestructorThread`
- Symbol `Destructor` / 符号 `Destructor`
- Symbol `Terminus` / 符号 `Terminus`
- Symbol `DestructorStack` / 符号 `DestructorStack`

## Dependencies / 依赖关系
- Java imports: `java.lang.ref.PhantomReference`, `java.lang.ref.ReferenceQueue`, `java.util.concurrent.atomic.AtomicReference`
- Java 导入: `java.lang.ref.PhantomReference`, `java.lang.ref.ReferenceQueue`, `java.util.concurrent.atomic.AtomicReference`
