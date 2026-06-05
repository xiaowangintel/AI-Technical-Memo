# jni.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `.ci/docker/java/jni.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Declares native interfaces, helper types, or constants consumed by nearby C++ implementation files. Java/native interop constraints influence the interface and data-marshalling choices here. The file header summarizes the intent as: "Copyright (C) 2006 The Android Open Source Project Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0 Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.."
- **Purpose (CN)**: 声明供附近 C++ 实现文件使用的原生接口、辅助类型或常量。 Java/原生互操作约束会影响这里的接口与数据封送方式。 文件头部将其意图概括为：“Copyright (C) 2006 The Android Open Source Project Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0 Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.”。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
/*
 * Copyright (C) 2006 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 17-27 / 第 17-27 行

```cpp
/*
 * JNI specification, as defined by Sun:
 * http://java.sun.com/javase/6/docs/technotes/guides/jni/spec/jniTOC.html
 *
 * Everything here is expected to be VM-neutral.
 */

#ifndef JNI_H_
#define JNI_H_

#include <stdarg.h>
```

- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 28-39 / 第 28-39 行

```cpp
#include <stdint.h>

/* Primitive types that match up with Java equivalents. */
typedef uint8_t  jboolean; /* unsigned 8 bits */
typedef int8_t   jbyte;    /* signed 8 bits */
typedef uint16_t jchar;    /* unsigned 16 bits */
typedef int16_t  jshort;   /* signed 16 bits */
typedef int32_t  jint;     /* signed 32 bits */
typedef int64_t  jlong;    /* signed 64 bits */
typedef float    jfloat;   /* 32-bit IEEE 754 */
typedef double   jdouble;  /* 64-bit IEEE 754 */

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 40-50 / 第 40-50 行

```cpp
/* "cardinal indices and sizes" */
typedef jint     jsize;

#ifdef __cplusplus
/*
 * Reference types, in C++
 */
class _jobject {};
class _jclass : public _jobject {};
class _jstring : public _jobject {};
class _jarray : public _jobject {};
```

- **EN:** Important local symbols in this block include _jobject, _jclass, _jstring, _jarray.
- **CN:** 该代码块中的重要局部符号包括 _jobject、_jclass、_jstring、_jarray。

### Lines 51-61 / 第 51-61 行

```cpp
class _jobjectArray : public _jarray {};
class _jbooleanArray : public _jarray {};
class _jbyteArray : public _jarray {};
class _jcharArray : public _jarray {};
class _jshortArray : public _jarray {};
class _jintArray : public _jarray {};
class _jlongArray : public _jarray {};
class _jfloatArray : public _jarray {};
class _jdoubleArray : public _jarray {};
class _jthrowable : public _jobject {};

```

- **EN:** Important local symbols in this block include _jobjectArray, _jbooleanArray, _jbyteArray, _jcharArray, _jshortArray, _jintArray.
- **CN:** 该代码块中的重要局部符号包括 _jobjectArray、_jbooleanArray、_jbyteArray、_jcharArray、_jshortArray、_jintArray。

### Lines 62-77 / 第 62-77 行

```cpp
typedef _jobject*       jobject;
typedef _jclass*        jclass;
typedef _jstring*       jstring;
typedef _jarray*        jarray;
typedef _jobjectArray*  jobjectArray;
typedef _jbooleanArray* jbooleanArray;
typedef _jbyteArray*    jbyteArray;
typedef _jcharArray*    jcharArray;
typedef _jshortArray*   jshortArray;
typedef _jintArray*     jintArray;
typedef _jlongArray*    jlongArray;
typedef _jfloatArray*   jfloatArray;
typedef _jdoubleArray*  jdoubleArray;
typedef _jthrowable*    jthrowable;
typedef _jobject*       jweak;

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 78-99 / 第 78-99 行

```cpp

#else /* not __cplusplus */

/*
 * Reference types, in C.
 */
typedef void*           jobject;
typedef jobject         jclass;
typedef jobject         jstring;
typedef jobject         jarray;
typedef jarray          jobjectArray;
typedef jarray          jbooleanArray;
typedef jarray          jbyteArray;
typedef jarray          jcharArray;
typedef jarray          jshortArray;
typedef jarray          jintArray;
typedef jarray          jlongArray;
typedef jarray          jfloatArray;
typedef jarray          jdoubleArray;
typedef jobject         jthrowable;
typedef jobject         jweak;

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 100-121 / 第 100-121 行

```cpp
#endif /* not __cplusplus */

struct _jfieldID;                       /* opaque structure */
typedef struct _jfieldID* jfieldID;     /* field IDs */

struct _jmethodID;                      /* opaque structure */
typedef struct _jmethodID* jmethodID;   /* method IDs */

struct JNIInvokeInterface;

typedef union jvalue {
    jboolean    z;
    jbyte       b;
    jchar       c;
    jshort      s;
    jint        i;
    jlong       j;
    jfloat      f;
    jdouble     d;
    jobject     l;
} jvalue;

```

- **EN:** Important local symbols in this block include _jfieldID, _jmethodID, JNIInvokeInterface.
- **CN:** 该代码块中的重要局部符号包括 _jfieldID、_jmethodID、JNIInvokeInterface。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 122-134 / 第 122-134 行

```cpp
typedef enum jobjectRefType {
    JNIInvalidRefType = 0,
    JNILocalRefType = 1,
    JNIGlobalRefType = 2,
    JNIWeakGlobalRefType = 3
} jobjectRefType;

typedef struct {
    const char* name;
    const char* signature;
    void*       fnPtr;
} JNINativeMethod;

```

- **EN:** Important local symbols in this block include jobjectRefType.
- **CN:** 该代码块中的重要局部符号包括 jobjectRefType。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 135-146 / 第 135-146 行

```cpp
struct _JNIEnv;
struct _JavaVM;
typedef const struct JNINativeInterface* C_JNIEnv;

#if defined(__cplusplus)
typedef _JNIEnv JNIEnv;
typedef _JavaVM JavaVM;
#else
typedef const struct JNINativeInterface* JNIEnv;
typedef const struct JNIInvokeInterface* JavaVM;
#endif

```

- **EN:** Important local symbols in this block include _JNIEnv, _JavaVM, JNINativeInterface, JNIInvokeInterface.
- **CN:** 该代码块中的重要局部符号包括 _JNIEnv、_JavaVM、JNINativeInterface、JNIInvokeInterface。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 147-157 / 第 147-157 行

```cpp
/*
 * Table of interface function pointers.
 */
struct JNINativeInterface {
    void*       reserved0;
    void*       reserved1;
    void*       reserved2;
    void*       reserved3;

    jint        (*GetVersion)(JNIEnv *);

```

- **EN:** Important local symbols in this block include JNINativeInterface, jint.
- **CN:** 该代码块中的重要局部符号包括 JNINativeInterface、jint。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 158-169 / 第 158-169 行

```cpp
    jclass      (*DefineClass)(JNIEnv*, const char*, jobject, const jbyte*,
                        jsize);
    jclass      (*FindClass)(JNIEnv*, const char*);

    jmethodID   (*FromReflectedMethod)(JNIEnv*, jobject);
    jfieldID    (*FromReflectedField)(JNIEnv*, jobject);
    /* spec doesn't show jboolean parameter */
    jobject     (*ToReflectedMethod)(JNIEnv*, jclass, jmethodID, jboolean);

    jclass      (*GetSuperclass)(JNIEnv*, jclass);
    jboolean    (*IsAssignableFrom)(JNIEnv*, jclass, jclass);

```

- **EN:** Important local symbols in this block include jclass, jmethodID, jfieldID, jobject, jboolean.
- **CN:** 该代码块中的重要局部符号包括 jclass、jmethodID、jfieldID、jobject、jboolean。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 170-182 / 第 170-182 行

```cpp
    /* spec doesn't show jboolean parameter */
    jobject     (*ToReflectedField)(JNIEnv*, jclass, jfieldID, jboolean);

    jint        (*Throw)(JNIEnv*, jthrowable);
    jint        (*ThrowNew)(JNIEnv *, jclass, const char *);
    jthrowable  (*ExceptionOccurred)(JNIEnv*);
    void        (*ExceptionDescribe)(JNIEnv*);
    void        (*ExceptionClear)(JNIEnv*);
    void        (*FatalError)(JNIEnv*, const char*);

    jint        (*PushLocalFrame)(JNIEnv*, jint);
    jobject     (*PopLocalFrame)(JNIEnv*, jobject);

```

- **EN:** Important local symbols in this block include jobject, jint, jthrowable, void.
- **CN:** 该代码块中的重要局部符号包括 jobject、jint、jthrowable、void。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 183-195 / 第 183-195 行

```cpp
    jobject     (*NewGlobalRef)(JNIEnv*, jobject);
    void        (*DeleteGlobalRef)(JNIEnv*, jobject);
    void        (*DeleteLocalRef)(JNIEnv*, jobject);
    jboolean    (*IsSameObject)(JNIEnv*, jobject, jobject);

    jobject     (*NewLocalRef)(JNIEnv*, jobject);
    jint        (*EnsureLocalCapacity)(JNIEnv*, jint);

    jobject     (*AllocObject)(JNIEnv*, jclass);
    jobject     (*NewObject)(JNIEnv*, jclass, jmethodID, ...);
    jobject     (*NewObjectV)(JNIEnv*, jclass, jmethodID, va_list);
    jobject     (*NewObjectA)(JNIEnv*, jclass, jmethodID, const jvalue*);

```

- **EN:** Important local symbols in this block include jobject, void, jboolean, jint.
- **CN:** 该代码块中的重要局部符号包括 jobject、void、jboolean、jint。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 196-217 / 第 196-217 行

```cpp
    jclass      (*GetObjectClass)(JNIEnv*, jobject);
    jboolean    (*IsInstanceOf)(JNIEnv*, jobject, jclass);
    jmethodID   (*GetMethodID)(JNIEnv*, jclass, const char*, const char*);

    jobject     (*CallObjectMethod)(JNIEnv*, jobject, jmethodID, ...);
    jobject     (*CallObjectMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    jobject     (*CallObjectMethodA)(JNIEnv*, jobject, jmethodID, const jvalue*);
    jboolean    (*CallBooleanMethod)(JNIEnv*, jobject, jmethodID, ...);
    jboolean    (*CallBooleanMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    jboolean    (*CallBooleanMethodA)(JNIEnv*, jobject, jmethodID, const jvalue*);
    jbyte       (*CallByteMethod)(JNIEnv*, jobject, jmethodID, ...);
    jbyte       (*CallByteMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    jbyte       (*CallByteMethodA)(JNIEnv*, jobject, jmethodID, const jvalue*);
    jchar       (*CallCharMethod)(JNIEnv*, jobject, jmethodID, ...);
    jchar       (*CallCharMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    jchar       (*CallCharMethodA)(JNIEnv*, jobject, jmethodID, const jvalue*);
    jshort      (*CallShortMethod)(JNIEnv*, jobject, jmethodID, ...);
    jshort      (*CallShortMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    jshort      (*CallShortMethodA)(JNIEnv*, jobject, jmethodID, const jvalue*);
    jint        (*CallIntMethod)(JNIEnv*, jobject, jmethodID, ...);
    jint        (*CallIntMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    jint        (*CallIntMethodA)(JNIEnv*, jobject, jmethodID, const jvalue*);
```

- **EN:** Important local symbols in this block include jclass, jboolean, jmethodID, jobject, jbyte, jchar.
- **CN:** 该代码块中的重要局部符号包括 jclass、jboolean、jmethodID、jobject、jbyte、jchar。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 218-230 / 第 218-230 行

```cpp
    jlong       (*CallLongMethod)(JNIEnv*, jobject, jmethodID, ...);
    jlong       (*CallLongMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    jlong       (*CallLongMethodA)(JNIEnv*, jobject, jmethodID, const jvalue*);
    jfloat      (*CallFloatMethod)(JNIEnv*, jobject, jmethodID, ...);
    jfloat      (*CallFloatMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    jfloat      (*CallFloatMethodA)(JNIEnv*, jobject, jmethodID, const jvalue*);
    jdouble     (*CallDoubleMethod)(JNIEnv*, jobject, jmethodID, ...);
    jdouble     (*CallDoubleMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    jdouble     (*CallDoubleMethodA)(JNIEnv*, jobject, jmethodID, const jvalue*);
    void        (*CallVoidMethod)(JNIEnv*, jobject, jmethodID, ...);
    void        (*CallVoidMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    void        (*CallVoidMethodA)(JNIEnv*, jobject, jmethodID, const jvalue*);

```

- **EN:** Important local symbols in this block include jlong, jfloat, jdouble, void.
- **CN:** 该代码块中的重要局部符号包括 jlong、jfloat、jdouble、void。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 231-252 / 第 231-252 行

```cpp
    jobject     (*CallNonvirtualObjectMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    jobject     (*CallNonvirtualObjectMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list);
    jobject     (*CallNonvirtualObjectMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, const jvalue*);
    jboolean    (*CallNonvirtualBooleanMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    jboolean    (*CallNonvirtualBooleanMethodV)(JNIEnv*, jobject, jclass,
                         jmethodID, va_list);
    jboolean    (*CallNonvirtualBooleanMethodA)(JNIEnv*, jobject, jclass,
                         jmethodID, const jvalue*);
    jbyte       (*CallNonvirtualByteMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    jbyte       (*CallNonvirtualByteMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list);
    jbyte       (*CallNonvirtualByteMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, const jvalue*);
    jchar       (*CallNonvirtualCharMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    jchar       (*CallNonvirtualCharMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list);
```

- **EN:** Important local symbols in this block include jobject, jboolean, jbyte, jchar.
- **CN:** 该代码块中的重要局部符号包括 jobject、jboolean、jbyte、jchar。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 253-274 / 第 253-274 行

```cpp
    jchar       (*CallNonvirtualCharMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, const jvalue*);
    jshort      (*CallNonvirtualShortMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    jshort      (*CallNonvirtualShortMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list);
    jshort      (*CallNonvirtualShortMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, const jvalue*);
    jint        (*CallNonvirtualIntMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    jint        (*CallNonvirtualIntMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list);
    jint        (*CallNonvirtualIntMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, const jvalue*);
    jlong       (*CallNonvirtualLongMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    jlong       (*CallNonvirtualLongMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list);
    jlong       (*CallNonvirtualLongMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, const jvalue*);
    jfloat      (*CallNonvirtualFloatMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
```

- **EN:** Important local symbols in this block include jchar, jshort, jint, jlong, jfloat.
- **CN:** 该代码块中的重要局部符号包括 jchar、jshort、jint、jlong、jfloat。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 275-291 / 第 275-291 行

```cpp
    jfloat      (*CallNonvirtualFloatMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list);
    jfloat      (*CallNonvirtualFloatMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, const jvalue*);
    jdouble     (*CallNonvirtualDoubleMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    jdouble     (*CallNonvirtualDoubleMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list);
    jdouble     (*CallNonvirtualDoubleMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, const jvalue*);
    void        (*CallNonvirtualVoidMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    void        (*CallNonvirtualVoidMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list);
    void        (*CallNonvirtualVoidMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, const jvalue*);

```

- **EN:** Important local symbols in this block include jfloat, jdouble, void.
- **CN:** 该代码块中的重要局部符号包括 jfloat、jdouble、void。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 292-303 / 第 292-303 行

```cpp
    jfieldID    (*GetFieldID)(JNIEnv*, jclass, const char*, const char*);

    jobject     (*GetObjectField)(JNIEnv*, jobject, jfieldID);
    jboolean    (*GetBooleanField)(JNIEnv*, jobject, jfieldID);
    jbyte       (*GetByteField)(JNIEnv*, jobject, jfieldID);
    jchar       (*GetCharField)(JNIEnv*, jobject, jfieldID);
    jshort      (*GetShortField)(JNIEnv*, jobject, jfieldID);
    jint        (*GetIntField)(JNIEnv*, jobject, jfieldID);
    jlong       (*GetLongField)(JNIEnv*, jobject, jfieldID);
    jfloat      (*GetFloatField)(JNIEnv*, jobject, jfieldID);
    jdouble     (*GetDoubleField)(JNIEnv*, jobject, jfieldID);

```

- **EN:** Important local symbols in this block include jfieldID, jobject, jboolean, jbyte, jchar, jshort.
- **CN:** 该代码块中的重要局部符号包括 jfieldID、jobject、jboolean、jbyte、jchar、jshort。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 304-315 / 第 304-315 行

```cpp
    void        (*SetObjectField)(JNIEnv*, jobject, jfieldID, jobject);
    void        (*SetBooleanField)(JNIEnv*, jobject, jfieldID, jboolean);
    void        (*SetByteField)(JNIEnv*, jobject, jfieldID, jbyte);
    void        (*SetCharField)(JNIEnv*, jobject, jfieldID, jchar);
    void        (*SetShortField)(JNIEnv*, jobject, jfieldID, jshort);
    void        (*SetIntField)(JNIEnv*, jobject, jfieldID, jint);
    void        (*SetLongField)(JNIEnv*, jobject, jfieldID, jlong);
    void        (*SetFloatField)(JNIEnv*, jobject, jfieldID, jfloat);
    void        (*SetDoubleField)(JNIEnv*, jobject, jfieldID, jdouble);

    jmethodID   (*GetStaticMethodID)(JNIEnv*, jclass, const char*, const char*);

```

- **EN:** Important local symbols in this block include void, jmethodID.
- **CN:** 该代码块中的重要局部符号包括 void、jmethodID。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 316-337 / 第 316-337 行

```cpp
    jobject     (*CallStaticObjectMethod)(JNIEnv*, jclass, jmethodID, ...);
    jobject     (*CallStaticObjectMethodV)(JNIEnv*, jclass, jmethodID, va_list);
    jobject     (*CallStaticObjectMethodA)(JNIEnv*, jclass, jmethodID, const jvalue*);
    jboolean    (*CallStaticBooleanMethod)(JNIEnv*, jclass, jmethodID, ...);
    jboolean    (*CallStaticBooleanMethodV)(JNIEnv*, jclass, jmethodID,
                        va_list);
    jboolean    (*CallStaticBooleanMethodA)(JNIEnv*, jclass, jmethodID, const jvalue*);
    jbyte       (*CallStaticByteMethod)(JNIEnv*, jclass, jmethodID, ...);
    jbyte       (*CallStaticByteMethodV)(JNIEnv*, jclass, jmethodID, va_list);
    jbyte       (*CallStaticByteMethodA)(JNIEnv*, jclass, jmethodID, const jvalue*);
    jchar       (*CallStaticCharMethod)(JNIEnv*, jclass, jmethodID, ...);
    jchar       (*CallStaticCharMethodV)(JNIEnv*, jclass, jmethodID, va_list);
    jchar       (*CallStaticCharMethodA)(JNIEnv*, jclass, jmethodID, const jvalue*);
    jshort      (*CallStaticShortMethod)(JNIEnv*, jclass, jmethodID, ...);
    jshort      (*CallStaticShortMethodV)(JNIEnv*, jclass, jmethodID, va_list);
    jshort      (*CallStaticShortMethodA)(JNIEnv*, jclass, jmethodID, const jvalue*);
    jint        (*CallStaticIntMethod)(JNIEnv*, jclass, jmethodID, ...);
    jint        (*CallStaticIntMethodV)(JNIEnv*, jclass, jmethodID, va_list);
    jint        (*CallStaticIntMethodA)(JNIEnv*, jclass, jmethodID, const jvalue*);
    jlong       (*CallStaticLongMethod)(JNIEnv*, jclass, jmethodID, ...);
    jlong       (*CallStaticLongMethodV)(JNIEnv*, jclass, jmethodID, va_list);
    jlong       (*CallStaticLongMethodA)(JNIEnv*, jclass, jmethodID, const jvalue*);
```

- **EN:** Important local symbols in this block include jobject, jboolean, jbyte, jchar, jshort, jint.
- **CN:** 该代码块中的重要局部符号包括 jobject、jboolean、jbyte、jchar、jshort、jint。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 338-350 / 第 338-350 行

```cpp
    jfloat      (*CallStaticFloatMethod)(JNIEnv*, jclass, jmethodID, ...);
    jfloat      (*CallStaticFloatMethodV)(JNIEnv*, jclass, jmethodID, va_list);
    jfloat      (*CallStaticFloatMethodA)(JNIEnv*, jclass, jmethodID, const jvalue*);
    jdouble     (*CallStaticDoubleMethod)(JNIEnv*, jclass, jmethodID, ...);
    jdouble     (*CallStaticDoubleMethodV)(JNIEnv*, jclass, jmethodID, va_list);
    jdouble     (*CallStaticDoubleMethodA)(JNIEnv*, jclass, jmethodID, const jvalue*);
    void        (*CallStaticVoidMethod)(JNIEnv*, jclass, jmethodID, ...);
    void        (*CallStaticVoidMethodV)(JNIEnv*, jclass, jmethodID, va_list);
    void        (*CallStaticVoidMethodA)(JNIEnv*, jclass, jmethodID, const jvalue*);

    jfieldID    (*GetStaticFieldID)(JNIEnv*, jclass, const char*,
                        const char*);

```

- **EN:** Important local symbols in this block include jfloat, jdouble, void, jfieldID.
- **CN:** 该代码块中的重要局部符号包括 jfloat、jdouble、void、jfieldID。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 351-370 / 第 351-370 行

```cpp
    jobject     (*GetStaticObjectField)(JNIEnv*, jclass, jfieldID);
    jboolean    (*GetStaticBooleanField)(JNIEnv*, jclass, jfieldID);
    jbyte       (*GetStaticByteField)(JNIEnv*, jclass, jfieldID);
    jchar       (*GetStaticCharField)(JNIEnv*, jclass, jfieldID);
    jshort      (*GetStaticShortField)(JNIEnv*, jclass, jfieldID);
    jint        (*GetStaticIntField)(JNIEnv*, jclass, jfieldID);
    jlong       (*GetStaticLongField)(JNIEnv*, jclass, jfieldID);
    jfloat      (*GetStaticFloatField)(JNIEnv*, jclass, jfieldID);
    jdouble     (*GetStaticDoubleField)(JNIEnv*, jclass, jfieldID);

    void        (*SetStaticObjectField)(JNIEnv*, jclass, jfieldID, jobject);
    void        (*SetStaticBooleanField)(JNIEnv*, jclass, jfieldID, jboolean);
    void        (*SetStaticByteField)(JNIEnv*, jclass, jfieldID, jbyte);
    void        (*SetStaticCharField)(JNIEnv*, jclass, jfieldID, jchar);
    void        (*SetStaticShortField)(JNIEnv*, jclass, jfieldID, jshort);
    void        (*SetStaticIntField)(JNIEnv*, jclass, jfieldID, jint);
    void        (*SetStaticLongField)(JNIEnv*, jclass, jfieldID, jlong);
    void        (*SetStaticFloatField)(JNIEnv*, jclass, jfieldID, jfloat);
    void        (*SetStaticDoubleField)(JNIEnv*, jclass, jfieldID, jdouble);

```

- **EN:** Important local symbols in this block include jobject, jboolean, jbyte, jchar, jshort, jint.
- **CN:** 该代码块中的重要局部符号包括 jobject、jboolean、jbyte、jchar、jshort、jint。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 371-384 / 第 371-384 行

```cpp
    jstring     (*NewString)(JNIEnv*, const jchar*, jsize);
    jsize       (*GetStringLength)(JNIEnv*, jstring);
    const jchar* (*GetStringChars)(JNIEnv*, jstring, jboolean*);
    void        (*ReleaseStringChars)(JNIEnv*, jstring, const jchar*);
    jstring     (*NewStringUTF)(JNIEnv*, const char*);
    jsize       (*GetStringUTFLength)(JNIEnv*, jstring);
    /* JNI spec says this returns const jbyte*, but that's inconsistent */
    const char* (*GetStringUTFChars)(JNIEnv*, jstring, jboolean*);
    void        (*ReleaseStringUTFChars)(JNIEnv*, jstring, const char*);
    jsize       (*GetArrayLength)(JNIEnv*, jarray);
    jobjectArray (*NewObjectArray)(JNIEnv*, jsize, jclass, jobject);
    jobject     (*GetObjectArrayElement)(JNIEnv*, jobjectArray, jsize);
    void        (*SetObjectArrayElement)(JNIEnv*, jobjectArray, jsize, jobject);

```

- **EN:** Important local symbols in this block include jstring, jsize, void, jobjectArray, jobject.
- **CN:** 该代码块中的重要局部符号包括 jstring、jsize、void、jobjectArray、jobject。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 385-402 / 第 385-402 行

```cpp
    jbooleanArray (*NewBooleanArray)(JNIEnv*, jsize);
    jbyteArray    (*NewByteArray)(JNIEnv*, jsize);
    jcharArray    (*NewCharArray)(JNIEnv*, jsize);
    jshortArray   (*NewShortArray)(JNIEnv*, jsize);
    jintArray     (*NewIntArray)(JNIEnv*, jsize);
    jlongArray    (*NewLongArray)(JNIEnv*, jsize);
    jfloatArray   (*NewFloatArray)(JNIEnv*, jsize);
    jdoubleArray  (*NewDoubleArray)(JNIEnv*, jsize);

    jboolean*   (*GetBooleanArrayElements)(JNIEnv*, jbooleanArray, jboolean*);
    jbyte*      (*GetByteArrayElements)(JNIEnv*, jbyteArray, jboolean*);
    jchar*      (*GetCharArrayElements)(JNIEnv*, jcharArray, jboolean*);
    jshort*     (*GetShortArrayElements)(JNIEnv*, jshortArray, jboolean*);
    jint*       (*GetIntArrayElements)(JNIEnv*, jintArray, jboolean*);
    jlong*      (*GetLongArrayElements)(JNIEnv*, jlongArray, jboolean*);
    jfloat*     (*GetFloatArrayElements)(JNIEnv*, jfloatArray, jboolean*);
    jdouble*    (*GetDoubleArrayElements)(JNIEnv*, jdoubleArray, jboolean*);

```

- **EN:** Important local symbols in this block include jbooleanArray, jbyteArray, jcharArray, jshortArray, jintArray, jlongArray.
- **CN:** 该代码块中的重要局部符号包括 jbooleanArray、jbyteArray、jcharArray、jshortArray、jintArray、jlongArray。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 403-419 / 第 403-419 行

```cpp
    void        (*ReleaseBooleanArrayElements)(JNIEnv*, jbooleanArray,
                        jboolean*, jint);
    void        (*ReleaseByteArrayElements)(JNIEnv*, jbyteArray,
                        jbyte*, jint);
    void        (*ReleaseCharArrayElements)(JNIEnv*, jcharArray,
                        jchar*, jint);
    void        (*ReleaseShortArrayElements)(JNIEnv*, jshortArray,
                        jshort*, jint);
    void        (*ReleaseIntArrayElements)(JNIEnv*, jintArray,
                        jint*, jint);
    void        (*ReleaseLongArrayElements)(JNIEnv*, jlongArray,
                        jlong*, jint);
    void        (*ReleaseFloatArrayElements)(JNIEnv*, jfloatArray,
                        jfloat*, jint);
    void        (*ReleaseDoubleArrayElements)(JNIEnv*, jdoubleArray,
                        jdouble*, jint);

```

- **EN:** Important local symbols in this block include void.
- **CN:** 该代码块中的重要局部符号包括 void。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 420-436 / 第 420-436 行

```cpp
    void        (*GetBooleanArrayRegion)(JNIEnv*, jbooleanArray,
                        jsize, jsize, jboolean*);
    void        (*GetByteArrayRegion)(JNIEnv*, jbyteArray,
                        jsize, jsize, jbyte*);
    void        (*GetCharArrayRegion)(JNIEnv*, jcharArray,
                        jsize, jsize, jchar*);
    void        (*GetShortArrayRegion)(JNIEnv*, jshortArray,
                        jsize, jsize, jshort*);
    void        (*GetIntArrayRegion)(JNIEnv*, jintArray,
                        jsize, jsize, jint*);
    void        (*GetLongArrayRegion)(JNIEnv*, jlongArray,
                        jsize, jsize, jlong*);
    void        (*GetFloatArrayRegion)(JNIEnv*, jfloatArray,
                        jsize, jsize, jfloat*);
    void        (*GetDoubleArrayRegion)(JNIEnv*, jdoubleArray,
                        jsize, jsize, jdouble*);

```

- **EN:** Important local symbols in this block include void.
- **CN:** 该代码块中的重要局部符号包括 void。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 437-454 / 第 437-454 行

```cpp
    /* spec shows these without const; some jni.h do, some don't */
    void        (*SetBooleanArrayRegion)(JNIEnv*, jbooleanArray,
                        jsize, jsize, const jboolean*);
    void        (*SetByteArrayRegion)(JNIEnv*, jbyteArray,
                        jsize, jsize, const jbyte*);
    void        (*SetCharArrayRegion)(JNIEnv*, jcharArray,
                        jsize, jsize, const jchar*);
    void        (*SetShortArrayRegion)(JNIEnv*, jshortArray,
                        jsize, jsize, const jshort*);
    void        (*SetIntArrayRegion)(JNIEnv*, jintArray,
                        jsize, jsize, const jint*);
    void        (*SetLongArrayRegion)(JNIEnv*, jlongArray,
                        jsize, jsize, const jlong*);
    void        (*SetFloatArrayRegion)(JNIEnv*, jfloatArray,
                        jsize, jsize, const jfloat*);
    void        (*SetDoubleArrayRegion)(JNIEnv*, jdoubleArray,
                        jsize, jsize, const jdouble*);

```

- **EN:** Important local symbols in this block include void.
- **CN:** 该代码块中的重要局部符号包括 void。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 455-467 / 第 455-467 行

```cpp
    jint        (*RegisterNatives)(JNIEnv*, jclass, const JNINativeMethod*,
                        jint);
    jint        (*UnregisterNatives)(JNIEnv*, jclass);
    jint        (*MonitorEnter)(JNIEnv*, jobject);
    jint        (*MonitorExit)(JNIEnv*, jobject);
    jint        (*GetJavaVM)(JNIEnv*, JavaVM**);

    void        (*GetStringRegion)(JNIEnv*, jstring, jsize, jsize, jchar*);
    void        (*GetStringUTFRegion)(JNIEnv*, jstring, jsize, jsize, char*);

    void*       (*GetPrimitiveArrayCritical)(JNIEnv*, jarray, jboolean*);
    void        (*ReleasePrimitiveArrayCritical)(JNIEnv*, jarray, void*, jint);

```

- **EN:** Important local symbols in this block include jint, void.
- **CN:** 该代码块中的重要局部符号包括 jint、void。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 468-479 / 第 468-479 行

```cpp
    const jchar* (*GetStringCritical)(JNIEnv*, jstring, jboolean*);
    void        (*ReleaseStringCritical)(JNIEnv*, jstring, const jchar*);

    jweak       (*NewWeakGlobalRef)(JNIEnv*, jobject);
    void        (*DeleteWeakGlobalRef)(JNIEnv*, jweak);

    jboolean    (*ExceptionCheck)(JNIEnv*);

    jobject     (*NewDirectByteBuffer)(JNIEnv*, void*, jlong);
    void*       (*GetDirectBufferAddress)(JNIEnv*, jobject);
    jlong       (*GetDirectBufferCapacity)(JNIEnv*, jobject);

```

- **EN:** Important local symbols in this block include void, jweak, jboolean, jobject, jlong.
- **CN:** 该代码块中的重要局部符号包括 void、jweak、jboolean、jobject、jlong。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 480-490 / 第 480-490 行

```cpp
    /* added in JNI 1.6 */
    jobjectRefType (*GetObjectRefType)(JNIEnv*, jobject);
};

/*
 * C++ object wrapper.
 *
 * This is usually overlaid on a C struct whose first element is a
 * JNINativeInterface*.  We rely somewhat on compiler behavior.
 */
struct _JNIEnv {
```

- **EN:** Important local symbols in this block include whose, _JNIEnv, jobjectRefType.
- **CN:** 该代码块中的重要局部符号包括 whose、_JNIEnv、jobjectRefType。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 491-502 / 第 491-502 行

```cpp
    /* do not rename this; it does not seem to be entirely opaque */
    const struct JNINativeInterface* functions;

#if defined(__cplusplus)

    jint GetVersion()
    { return functions->GetVersion(this); }

    jclass DefineClass(const char *name, jobject loader, const jbyte* buf,
        jsize bufLen)
    { return functions->DefineClass(this, name, loader, buf, bufLen); }

```

- **EN:** Important local symbols in this block include JNINativeInterface, GetVersion, DefineClass.
- **CN:** 该代码块中的重要局部符号包括 JNINativeInterface、GetVersion、DefineClass。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 503-514 / 第 503-514 行

```cpp
    jclass FindClass(const char* name)
    { return functions->FindClass(this, name); }

    jmethodID FromReflectedMethod(jobject method)
    { return functions->FromReflectedMethod(this, method); }

    jfieldID FromReflectedField(jobject field)
    { return functions->FromReflectedField(this, field); }

    jobject ToReflectedMethod(jclass cls, jmethodID methodID, jboolean isStatic)
    { return functions->ToReflectedMethod(this, cls, methodID, isStatic); }

```

- **EN:** Important local symbols in this block include FindClass, FromReflectedMethod, FromReflectedField, ToReflectedMethod.
- **CN:** 该代码块中的重要局部符号包括 FindClass、FromReflectedMethod、FromReflectedField、ToReflectedMethod。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 515-526 / 第 515-526 行

```cpp
    jclass GetSuperclass(jclass clazz)
    { return functions->GetSuperclass(this, clazz); }

    jboolean IsAssignableFrom(jclass clazz1, jclass clazz2)
    { return functions->IsAssignableFrom(this, clazz1, clazz2); }

    jobject ToReflectedField(jclass cls, jfieldID fieldID, jboolean isStatic)
    { return functions->ToReflectedField(this, cls, fieldID, isStatic); }

    jint Throw(jthrowable obj)
    { return functions->Throw(this, obj); }

```

- **EN:** Important local symbols in this block include GetSuperclass, IsAssignableFrom, ToReflectedField, Throw.
- **CN:** 该代码块中的重要局部符号包括 GetSuperclass、IsAssignableFrom、ToReflectedField、Throw。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 527-538 / 第 527-538 行

```cpp
    jint ThrowNew(jclass clazz, const char* message)
    { return functions->ThrowNew(this, clazz, message); }

    jthrowable ExceptionOccurred()
    { return functions->ExceptionOccurred(this); }

    void ExceptionDescribe()
    { functions->ExceptionDescribe(this); }

    void ExceptionClear()
    { functions->ExceptionClear(this); }

```

- **EN:** Important local symbols in this block include ThrowNew, ExceptionOccurred, ExceptionDescribe, ExceptionClear.
- **CN:** 该代码块中的重要局部符号包括 ThrowNew、ExceptionOccurred、ExceptionDescribe、ExceptionClear。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 539-550 / 第 539-550 行

```cpp
    void FatalError(const char* msg)
    { functions->FatalError(this, msg); }

    jint PushLocalFrame(jint capacity)
    { return functions->PushLocalFrame(this, capacity); }

    jobject PopLocalFrame(jobject result)
    { return functions->PopLocalFrame(this, result); }

    jobject NewGlobalRef(jobject obj)
    { return functions->NewGlobalRef(this, obj); }

```

- **EN:** Important local symbols in this block include FatalError, PushLocalFrame, PopLocalFrame, NewGlobalRef.
- **CN:** 该代码块中的重要局部符号包括 FatalError、PushLocalFrame、PopLocalFrame、NewGlobalRef。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 551-562 / 第 551-562 行

```cpp
    void DeleteGlobalRef(jobject globalRef)
    { functions->DeleteGlobalRef(this, globalRef); }

    void DeleteLocalRef(jobject localRef)
    { functions->DeleteLocalRef(this, localRef); }

    jboolean IsSameObject(jobject ref1, jobject ref2)
    { return functions->IsSameObject(this, ref1, ref2); }

    jobject NewLocalRef(jobject ref)
    { return functions->NewLocalRef(this, ref); }

```

- **EN:** Important local symbols in this block include DeleteGlobalRef, DeleteLocalRef, IsSameObject, NewLocalRef.
- **CN:** 该代码块中的重要局部符号包括 DeleteGlobalRef、DeleteLocalRef、IsSameObject、NewLocalRef。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 563-575 / 第 563-575 行

```cpp
    jint EnsureLocalCapacity(jint capacity)
    { return functions->EnsureLocalCapacity(this, capacity); }

    jobject AllocObject(jclass clazz)
    { return functions->AllocObject(this, clazz); }

    jobject NewObject(jclass clazz, jmethodID methodID, ...)
    {
        va_list args;
        va_start(args, methodID);
        jobject result = functions->NewObjectV(this, clazz, methodID, args);
        va_end(args);
        return result;
```

- **EN:** Important local symbols in this block include EnsureLocalCapacity, AllocObject, NewObject, va_start, va_end.
- **CN:** 该代码块中的重要局部符号包括 EnsureLocalCapacity、AllocObject、NewObject、va_start、va_end。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 576-586 / 第 576-586 行

```cpp
    }

    jobject NewObjectV(jclass clazz, jmethodID methodID, va_list args)
    { return functions->NewObjectV(this, clazz, methodID, args); }

    jobject NewObjectA(jclass clazz, jmethodID methodID, const jvalue* args)
    { return functions->NewObjectA(this, clazz, methodID, args); }

    jclass GetObjectClass(jobject obj)
    { return functions->GetObjectClass(this, obj); }

```

- **EN:** Important local symbols in this block include NewObjectV, NewObjectA, GetObjectClass.
- **CN:** 该代码块中的重要局部符号包括 NewObjectV、NewObjectA、GetObjectClass。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 587-602 / 第 587-602 行

```cpp
    jboolean IsInstanceOf(jobject obj, jclass clazz)
    { return functions->IsInstanceOf(this, obj, clazz); }

    jmethodID GetMethodID(jclass clazz, const char* name, const char* sig)
    { return functions->GetMethodID(this, clazz, name, sig); }

#define CALL_TYPE_METHOD(_jtype, _jname)                                    \
    _jtype Call##_jname##Method(jobject obj, jmethodID methodID, ...)       \
    {                                                                       \
        _jtype result;                                                      \
        va_list args;                                                       \
        va_start(args, methodID);                                           \
        result = functions->Call##_jname##MethodV(this, obj, methodID,      \
                    args);                                                  \
        va_end(args);                                                       \
        return result;                                                      \
```

- **EN:** Important local symbols in this block include IsInstanceOf, GetMethodID, va_start, va_end.
- **CN:** 该代码块中的重要局部符号包括 IsInstanceOf、GetMethodID、va_start、va_end。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 603-617 / 第 603-617 行

```cpp
    }
#define CALL_TYPE_METHODV(_jtype, _jname)                                   \
    _jtype Call##_jname##MethodV(jobject obj, jmethodID methodID,           \
        va_list args)                                                       \
    { return functions->Call##_jname##MethodV(this, obj, methodID, args); }
#define CALL_TYPE_METHODA(_jtype, _jname)                                   \
    _jtype Call##_jname##MethodA(jobject obj, jmethodID methodID,           \
                                 const jvalue* args)                        \
    { return functions->Call##_jname##MethodA(this, obj, methodID, args); }

#define CALL_TYPE(_jtype, _jname)                                           \
    CALL_TYPE_METHOD(_jtype, _jname)                                        \
    CALL_TYPE_METHODV(_jtype, _jname)                                       \
    CALL_TYPE_METHODA(_jtype, _jname)

```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 618-639 / 第 618-639 行

```cpp
    CALL_TYPE(jobject, Object)
    CALL_TYPE(jboolean, Boolean)
    CALL_TYPE(jbyte, Byte)
    CALL_TYPE(jchar, Char)
    CALL_TYPE(jshort, Short)
    CALL_TYPE(jint, Int)
    CALL_TYPE(jlong, Long)
    CALL_TYPE(jfloat, Float)
    CALL_TYPE(jdouble, Double)

    void CallVoidMethod(jobject obj, jmethodID methodID, ...)
    {
        va_list args;
        va_start(args, methodID);
        functions->CallVoidMethodV(this, obj, methodID, args);
        va_end(args);
    }
    void CallVoidMethodV(jobject obj, jmethodID methodID, va_list args)
    { functions->CallVoidMethodV(this, obj, methodID, args); }
    void CallVoidMethodA(jobject obj, jmethodID methodID, const jvalue* args)
    { functions->CallVoidMethodA(this, obj, methodID, args); }

```

- **EN:** Important local symbols in this block include va_start, va_end, CallVoidMethodV, CallVoidMethodA.
- **CN:** 该代码块中的重要局部符号包括 va_start、va_end、CallVoidMethodV、CallVoidMethodA。

### Lines 640-650 / 第 640-650 行

```cpp
#define CALL_NONVIRT_TYPE_METHOD(_jtype, _jname)                            \
    _jtype CallNonvirtual##_jname##Method(jobject obj, jclass clazz,        \
        jmethodID methodID, ...)                                            \
    {                                                                       \
        _jtype result;                                                      \
        va_list args;                                                       \
        va_start(args, methodID);                                           \
        result = functions->CallNonvirtual##_jname##MethodV(this, obj,      \
                    clazz, methodID, args);                                 \
        va_end(args);                                                       \
        return result;                                                      \
```

- **EN:** Important local symbols in this block include va_start, va_end.
- **CN:** 该代码块中的重要局部符号包括 va_start、va_end。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 651-662 / 第 651-662 行

```cpp
    }
#define CALL_NONVIRT_TYPE_METHODV(_jtype, _jname)                           \
    _jtype CallNonvirtual##_jname##MethodV(jobject obj, jclass clazz,       \
        jmethodID methodID, va_list args)                                   \
    { return functions->CallNonvirtual##_jname##MethodV(this, obj, clazz,   \
        methodID, args); }
#define CALL_NONVIRT_TYPE_METHODA(_jtype, _jname)                           \
    _jtype CallNonvirtual##_jname##MethodA(jobject obj, jclass clazz,       \
        jmethodID methodID, const jvalue* args)                             \
    { return functions->CallNonvirtual##_jname##MethodA(this, obj, clazz,   \
        methodID, args); }

```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 663-677 / 第 663-677 行

```cpp
#define CALL_NONVIRT_TYPE(_jtype, _jname)                                   \
    CALL_NONVIRT_TYPE_METHOD(_jtype, _jname)                                \
    CALL_NONVIRT_TYPE_METHODV(_jtype, _jname)                               \
    CALL_NONVIRT_TYPE_METHODA(_jtype, _jname)

    CALL_NONVIRT_TYPE(jobject, Object)
    CALL_NONVIRT_TYPE(jboolean, Boolean)
    CALL_NONVIRT_TYPE(jbyte, Byte)
    CALL_NONVIRT_TYPE(jchar, Char)
    CALL_NONVIRT_TYPE(jshort, Short)
    CALL_NONVIRT_TYPE(jint, Int)
    CALL_NONVIRT_TYPE(jlong, Long)
    CALL_NONVIRT_TYPE(jfloat, Float)
    CALL_NONVIRT_TYPE(jdouble, Double)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 678-692 / 第 678-692 行

```cpp
    void CallNonvirtualVoidMethod(jobject obj, jclass clazz,
        jmethodID methodID, ...)
    {
        va_list args;
        va_start(args, methodID);
        functions->CallNonvirtualVoidMethodV(this, obj, clazz, methodID, args);
        va_end(args);
    }
    void CallNonvirtualVoidMethodV(jobject obj, jclass clazz,
        jmethodID methodID, va_list args)
    { functions->CallNonvirtualVoidMethodV(this, obj, clazz, methodID, args); }
    void CallNonvirtualVoidMethodA(jobject obj, jclass clazz,
        jmethodID methodID, const jvalue* args)
    { functions->CallNonvirtualVoidMethodA(this, obj, clazz, methodID, args); }

```

- **EN:** Important local symbols in this block include CallNonvirtualVoidMethod, va_start, va_end, CallNonvirtualVoidMethodV, CallNonvirtualVoidMethodA.
- **CN:** 该代码块中的重要局部符号包括 CallNonvirtualVoidMethod、va_start、va_end、CallNonvirtualVoidMethodV、CallNonvirtualVoidMethodA。

### Lines 693-714 / 第 693-714 行

```cpp
    jfieldID GetFieldID(jclass clazz, const char* name, const char* sig)
    { return functions->GetFieldID(this, clazz, name, sig); }

    jobject GetObjectField(jobject obj, jfieldID fieldID)
    { return functions->GetObjectField(this, obj, fieldID); }
    jboolean GetBooleanField(jobject obj, jfieldID fieldID)
    { return functions->GetBooleanField(this, obj, fieldID); }
    jbyte GetByteField(jobject obj, jfieldID fieldID)
    { return functions->GetByteField(this, obj, fieldID); }
    jchar GetCharField(jobject obj, jfieldID fieldID)
    { return functions->GetCharField(this, obj, fieldID); }
    jshort GetShortField(jobject obj, jfieldID fieldID)
    { return functions->GetShortField(this, obj, fieldID); }
    jint GetIntField(jobject obj, jfieldID fieldID)
    { return functions->GetIntField(this, obj, fieldID); }
    jlong GetLongField(jobject obj, jfieldID fieldID)
    { return functions->GetLongField(this, obj, fieldID); }
    jfloat GetFloatField(jobject obj, jfieldID fieldID)
    { return functions->GetFloatField(this, obj, fieldID); }
    jdouble GetDoubleField(jobject obj, jfieldID fieldID)
    { return functions->GetDoubleField(this, obj, fieldID); }

```

- **EN:** Important local symbols in this block include GetFieldID, GetObjectField, GetBooleanField, GetByteField, GetCharField, GetShortField.
- **CN:** 该代码块中的重要局部符号包括 GetFieldID、GetObjectField、GetBooleanField、GetByteField、GetCharField、GetShortField。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 715-733 / 第 715-733 行

```cpp
    void SetObjectField(jobject obj, jfieldID fieldID, jobject value)
    { functions->SetObjectField(this, obj, fieldID, value); }
    void SetBooleanField(jobject obj, jfieldID fieldID, jboolean value)
    { functions->SetBooleanField(this, obj, fieldID, value); }
    void SetByteField(jobject obj, jfieldID fieldID, jbyte value)
    { functions->SetByteField(this, obj, fieldID, value); }
    void SetCharField(jobject obj, jfieldID fieldID, jchar value)
    { functions->SetCharField(this, obj, fieldID, value); }
    void SetShortField(jobject obj, jfieldID fieldID, jshort value)
    { functions->SetShortField(this, obj, fieldID, value); }
    void SetIntField(jobject obj, jfieldID fieldID, jint value)
    { functions->SetIntField(this, obj, fieldID, value); }
    void SetLongField(jobject obj, jfieldID fieldID, jlong value)
    { functions->SetLongField(this, obj, fieldID, value); }
    void SetFloatField(jobject obj, jfieldID fieldID, jfloat value)
    { functions->SetFloatField(this, obj, fieldID, value); }
    void SetDoubleField(jobject obj, jfieldID fieldID, jdouble value)
    { functions->SetDoubleField(this, obj, fieldID, value); }

```

- **EN:** Important local symbols in this block include SetObjectField, SetBooleanField, SetByteField, SetCharField, SetShortField, SetIntField.
- **CN:** 该代码块中的重要局部符号包括 SetObjectField、SetBooleanField、SetByteField、SetCharField、SetShortField、SetIntField。

### Lines 734-747 / 第 734-747 行

```cpp
    jmethodID GetStaticMethodID(jclass clazz, const char* name, const char* sig)
    { return functions->GetStaticMethodID(this, clazz, name, sig); }

#define CALL_STATIC_TYPE_METHOD(_jtype, _jname)                             \
    _jtype CallStatic##_jname##Method(jclass clazz, jmethodID methodID,     \
        ...)                                                                \
    {                                                                       \
        _jtype result;                                                      \
        va_list args;                                                       \
        va_start(args, methodID);                                           \
        result = functions->CallStatic##_jname##MethodV(this, clazz,        \
                    methodID, args);                                        \
        va_end(args);                                                       \
        return result;                                                      \
```

- **EN:** Important local symbols in this block include GetStaticMethodID, va_start, va_end.
- **CN:** 该代码块中的重要局部符号包括 GetStaticMethodID、va_start、va_end。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 748-759 / 第 748-759 行

```cpp
    }
#define CALL_STATIC_TYPE_METHODV(_jtype, _jname)                            \
    _jtype CallStatic##_jname##MethodV(jclass clazz, jmethodID methodID,    \
        va_list args)                                                       \
    { return functions->CallStatic##_jname##MethodV(this, clazz, methodID,  \
        args); }
#define CALL_STATIC_TYPE_METHODA(_jtype, _jname)                            \
    _jtype CallStatic##_jname##MethodA(jclass clazz, jmethodID methodID,    \
                                       const jvalue* args)                  \
    { return functions->CallStatic##_jname##MethodA(this, clazz, methodID,  \
        args); }

```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 760-774 / 第 760-774 行

```cpp
#define CALL_STATIC_TYPE(_jtype, _jname)                                    \
    CALL_STATIC_TYPE_METHOD(_jtype, _jname)                                 \
    CALL_STATIC_TYPE_METHODV(_jtype, _jname)                                \
    CALL_STATIC_TYPE_METHODA(_jtype, _jname)

    CALL_STATIC_TYPE(jobject, Object)
    CALL_STATIC_TYPE(jboolean, Boolean)
    CALL_STATIC_TYPE(jbyte, Byte)
    CALL_STATIC_TYPE(jchar, Char)
    CALL_STATIC_TYPE(jshort, Short)
    CALL_STATIC_TYPE(jint, Int)
    CALL_STATIC_TYPE(jlong, Long)
    CALL_STATIC_TYPE(jfloat, Float)
    CALL_STATIC_TYPE(jdouble, Double)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 775-786 / 第 775-786 行

```cpp
    void CallStaticVoidMethod(jclass clazz, jmethodID methodID, ...)
    {
        va_list args;
        va_start(args, methodID);
        functions->CallStaticVoidMethodV(this, clazz, methodID, args);
        va_end(args);
    }
    void CallStaticVoidMethodV(jclass clazz, jmethodID methodID, va_list args)
    { functions->CallStaticVoidMethodV(this, clazz, methodID, args); }
    void CallStaticVoidMethodA(jclass clazz, jmethodID methodID, const jvalue* args)
    { functions->CallStaticVoidMethodA(this, clazz, methodID, args); }

```

- **EN:** Important local symbols in this block include CallStaticVoidMethod, va_start, va_end, CallStaticVoidMethodV, CallStaticVoidMethodA.
- **CN:** 该代码块中的重要局部符号包括 CallStaticVoidMethod、va_start、va_end、CallStaticVoidMethodV、CallStaticVoidMethodA。

### Lines 787-808 / 第 787-808 行

```cpp
    jfieldID GetStaticFieldID(jclass clazz, const char* name, const char* sig)
    { return functions->GetStaticFieldID(this, clazz, name, sig); }

    jobject GetStaticObjectField(jclass clazz, jfieldID fieldID)
    { return functions->GetStaticObjectField(this, clazz, fieldID); }
    jboolean GetStaticBooleanField(jclass clazz, jfieldID fieldID)
    { return functions->GetStaticBooleanField(this, clazz, fieldID); }
    jbyte GetStaticByteField(jclass clazz, jfieldID fieldID)
    { return functions->GetStaticByteField(this, clazz, fieldID); }
    jchar GetStaticCharField(jclass clazz, jfieldID fieldID)
    { return functions->GetStaticCharField(this, clazz, fieldID); }
    jshort GetStaticShortField(jclass clazz, jfieldID fieldID)
    { return functions->GetStaticShortField(this, clazz, fieldID); }
    jint GetStaticIntField(jclass clazz, jfieldID fieldID)
    { return functions->GetStaticIntField(this, clazz, fieldID); }
    jlong GetStaticLongField(jclass clazz, jfieldID fieldID)
    { return functions->GetStaticLongField(this, clazz, fieldID); }
    jfloat GetStaticFloatField(jclass clazz, jfieldID fieldID)
    { return functions->GetStaticFloatField(this, clazz, fieldID); }
    jdouble GetStaticDoubleField(jclass clazz, jfieldID fieldID)
    { return functions->GetStaticDoubleField(this, clazz, fieldID); }

```

- **EN:** Important local symbols in this block include GetStaticFieldID, GetStaticObjectField, GetStaticBooleanField, GetStaticByteField, GetStaticCharField, GetStaticShortField.
- **CN:** 该代码块中的重要局部符号包括 GetStaticFieldID、GetStaticObjectField、GetStaticBooleanField、GetStaticByteField、GetStaticCharField、GetStaticShortField。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 809-827 / 第 809-827 行

```cpp
    void SetStaticObjectField(jclass clazz, jfieldID fieldID, jobject value)
    { functions->SetStaticObjectField(this, clazz, fieldID, value); }
    void SetStaticBooleanField(jclass clazz, jfieldID fieldID, jboolean value)
    { functions->SetStaticBooleanField(this, clazz, fieldID, value); }
    void SetStaticByteField(jclass clazz, jfieldID fieldID, jbyte value)
    { functions->SetStaticByteField(this, clazz, fieldID, value); }
    void SetStaticCharField(jclass clazz, jfieldID fieldID, jchar value)
    { functions->SetStaticCharField(this, clazz, fieldID, value); }
    void SetStaticShortField(jclass clazz, jfieldID fieldID, jshort value)
    { functions->SetStaticShortField(this, clazz, fieldID, value); }
    void SetStaticIntField(jclass clazz, jfieldID fieldID, jint value)
    { functions->SetStaticIntField(this, clazz, fieldID, value); }
    void SetStaticLongField(jclass clazz, jfieldID fieldID, jlong value)
    { functions->SetStaticLongField(this, clazz, fieldID, value); }
    void SetStaticFloatField(jclass clazz, jfieldID fieldID, jfloat value)
    { functions->SetStaticFloatField(this, clazz, fieldID, value); }
    void SetStaticDoubleField(jclass clazz, jfieldID fieldID, jdouble value)
    { functions->SetStaticDoubleField(this, clazz, fieldID, value); }

```

- **EN:** Important local symbols in this block include SetStaticObjectField, SetStaticBooleanField, SetStaticByteField, SetStaticCharField, SetStaticShortField, SetStaticIntField.
- **CN:** 该代码块中的重要局部符号包括 SetStaticObjectField、SetStaticBooleanField、SetStaticByteField、SetStaticCharField、SetStaticShortField、SetStaticIntField。

### Lines 828-839 / 第 828-839 行

```cpp
    jstring NewString(const jchar* unicodeChars, jsize len)
    { return functions->NewString(this, unicodeChars, len); }

    jsize GetStringLength(jstring string)
    { return functions->GetStringLength(this, string); }

    const jchar* GetStringChars(jstring string, jboolean* isCopy)
    { return functions->GetStringChars(this, string, isCopy); }

    void ReleaseStringChars(jstring string, const jchar* chars)
    { functions->ReleaseStringChars(this, string, chars); }

```

- **EN:** Important local symbols in this block include NewString, GetStringLength, GetStringChars, ReleaseStringChars.
- **CN:** 该代码块中的重要局部符号包括 NewString、GetStringLength、GetStringChars、ReleaseStringChars。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 840-851 / 第 840-851 行

```cpp
    jstring NewStringUTF(const char* bytes)
    { return functions->NewStringUTF(this, bytes); }

    jsize GetStringUTFLength(jstring string)
    { return functions->GetStringUTFLength(this, string); }

    const char* GetStringUTFChars(jstring string, jboolean* isCopy)
    { return functions->GetStringUTFChars(this, string, isCopy); }

    void ReleaseStringUTFChars(jstring string, const char* utf)
    { functions->ReleaseStringUTFChars(this, string, utf); }

```

- **EN:** Important local symbols in this block include NewStringUTF, GetStringUTFLength, GetStringUTFChars, ReleaseStringUTFChars.
- **CN:** 该代码块中的重要局部符号包括 NewStringUTF、GetStringUTFLength、GetStringUTFChars、ReleaseStringUTFChars。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 852-862 / 第 852-862 行

```cpp
    jsize GetArrayLength(jarray array)
    { return functions->GetArrayLength(this, array); }

    jobjectArray NewObjectArray(jsize length, jclass elementClass,
        jobject initialElement)
    { return functions->NewObjectArray(this, length, elementClass,
        initialElement); }

    jobject GetObjectArrayElement(jobjectArray array, jsize index)
    { return functions->GetObjectArrayElement(this, array, index); }

```

- **EN:** Important local symbols in this block include GetArrayLength, NewObjectArray, GetObjectArrayElement.
- **CN:** 该代码块中的重要局部符号包括 GetArrayLength、NewObjectArray、GetObjectArrayElement。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 863-882 / 第 863-882 行

```cpp
    void SetObjectArrayElement(jobjectArray array, jsize index, jobject value)
    { functions->SetObjectArrayElement(this, array, index, value); }

    jbooleanArray NewBooleanArray(jsize length)
    { return functions->NewBooleanArray(this, length); }
    jbyteArray NewByteArray(jsize length)
    { return functions->NewByteArray(this, length); }
    jcharArray NewCharArray(jsize length)
    { return functions->NewCharArray(this, length); }
    jshortArray NewShortArray(jsize length)
    { return functions->NewShortArray(this, length); }
    jintArray NewIntArray(jsize length)
    { return functions->NewIntArray(this, length); }
    jlongArray NewLongArray(jsize length)
    { return functions->NewLongArray(this, length); }
    jfloatArray NewFloatArray(jsize length)
    { return functions->NewFloatArray(this, length); }
    jdoubleArray NewDoubleArray(jsize length)
    { return functions->NewDoubleArray(this, length); }

```

- **EN:** Important local symbols in this block include SetObjectArrayElement, NewBooleanArray, NewByteArray, NewCharArray, NewShortArray, NewIntArray.
- **CN:** 该代码块中的重要局部符号包括 SetObjectArrayElement、NewBooleanArray、NewByteArray、NewCharArray、NewShortArray、NewIntArray。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 883-899 / 第 883-899 行

```cpp
    jboolean* GetBooleanArrayElements(jbooleanArray array, jboolean* isCopy)
    { return functions->GetBooleanArrayElements(this, array, isCopy); }
    jbyte* GetByteArrayElements(jbyteArray array, jboolean* isCopy)
    { return functions->GetByteArrayElements(this, array, isCopy); }
    jchar* GetCharArrayElements(jcharArray array, jboolean* isCopy)
    { return functions->GetCharArrayElements(this, array, isCopy); }
    jshort* GetShortArrayElements(jshortArray array, jboolean* isCopy)
    { return functions->GetShortArrayElements(this, array, isCopy); }
    jint* GetIntArrayElements(jintArray array, jboolean* isCopy)
    { return functions->GetIntArrayElements(this, array, isCopy); }
    jlong* GetLongArrayElements(jlongArray array, jboolean* isCopy)
    { return functions->GetLongArrayElements(this, array, isCopy); }
    jfloat* GetFloatArrayElements(jfloatArray array, jboolean* isCopy)
    { return functions->GetFloatArrayElements(this, array, isCopy); }
    jdouble* GetDoubleArrayElements(jdoubleArray array, jboolean* isCopy)
    { return functions->GetDoubleArrayElements(this, array, isCopy); }

```

- **EN:** Important local symbols in this block include GetBooleanArrayElements, GetByteArrayElements, GetCharArrayElements, GetShortArrayElements, GetIntArrayElements, GetLongArrayElements.
- **CN:** 该代码块中的重要局部符号包括 GetBooleanArrayElements、GetByteArrayElements、GetCharArrayElements、GetShortArrayElements、GetIntArrayElements、GetLongArrayElements。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 900-921 / 第 900-921 行

```cpp
    void ReleaseBooleanArrayElements(jbooleanArray array, jboolean* elems,
        jint mode)
    { functions->ReleaseBooleanArrayElements(this, array, elems, mode); }
    void ReleaseByteArrayElements(jbyteArray array, jbyte* elems,
        jint mode)
    { functions->ReleaseByteArrayElements(this, array, elems, mode); }
    void ReleaseCharArrayElements(jcharArray array, jchar* elems,
        jint mode)
    { functions->ReleaseCharArrayElements(this, array, elems, mode); }
    void ReleaseShortArrayElements(jshortArray array, jshort* elems,
        jint mode)
    { functions->ReleaseShortArrayElements(this, array, elems, mode); }
    void ReleaseIntArrayElements(jintArray array, jint* elems,
        jint mode)
    { functions->ReleaseIntArrayElements(this, array, elems, mode); }
    void ReleaseLongArrayElements(jlongArray array, jlong* elems,
        jint mode)
    { functions->ReleaseLongArrayElements(this, array, elems, mode); }
    void ReleaseFloatArrayElements(jfloatArray array, jfloat* elems,
        jint mode)
    { functions->ReleaseFloatArrayElements(this, array, elems, mode); }
    void ReleaseDoubleArrayElements(jdoubleArray array, jdouble* elems,
```

- **EN:** Important local symbols in this block include ReleaseBooleanArrayElements, ReleaseByteArrayElements, ReleaseCharArrayElements, ReleaseShortArrayElements, ReleaseIntArrayElements, ReleaseLongArrayElements.
- **CN:** 该代码块中的重要局部符号包括 ReleaseBooleanArrayElements、ReleaseByteArrayElements、ReleaseCharArrayElements、ReleaseShortArrayElements、ReleaseIntArrayElements、ReleaseLongArrayElements。

### Lines 922-943 / 第 922-943 行

```cpp
        jint mode)
    { functions->ReleaseDoubleArrayElements(this, array, elems, mode); }

    void GetBooleanArrayRegion(jbooleanArray array, jsize start, jsize len,
        jboolean* buf)
    { functions->GetBooleanArrayRegion(this, array, start, len, buf); }
    void GetByteArrayRegion(jbyteArray array, jsize start, jsize len,
        jbyte* buf)
    { functions->GetByteArrayRegion(this, array, start, len, buf); }
    void GetCharArrayRegion(jcharArray array, jsize start, jsize len,
        jchar* buf)
    { functions->GetCharArrayRegion(this, array, start, len, buf); }
    void GetShortArrayRegion(jshortArray array, jsize start, jsize len,
        jshort* buf)
    { functions->GetShortArrayRegion(this, array, start, len, buf); }
    void GetIntArrayRegion(jintArray array, jsize start, jsize len,
        jint* buf)
    { functions->GetIntArrayRegion(this, array, start, len, buf); }
    void GetLongArrayRegion(jlongArray array, jsize start, jsize len,
        jlong* buf)
    { functions->GetLongArrayRegion(this, array, start, len, buf); }
    void GetFloatArrayRegion(jfloatArray array, jsize start, jsize len,
```

- **EN:** Important local symbols in this block include GetBooleanArrayRegion, GetByteArrayRegion, GetCharArrayRegion, GetShortArrayRegion, GetIntArrayRegion, GetLongArrayRegion.
- **CN:** 该代码块中的重要局部符号包括 GetBooleanArrayRegion、GetByteArrayRegion、GetCharArrayRegion、GetShortArrayRegion、GetIntArrayRegion、GetLongArrayRegion。

### Lines 944-965 / 第 944-965 行

```cpp
        jfloat* buf)
    { functions->GetFloatArrayRegion(this, array, start, len, buf); }
    void GetDoubleArrayRegion(jdoubleArray array, jsize start, jsize len,
        jdouble* buf)
    { functions->GetDoubleArrayRegion(this, array, start, len, buf); }

    void SetBooleanArrayRegion(jbooleanArray array, jsize start, jsize len,
        const jboolean* buf)
    { functions->SetBooleanArrayRegion(this, array, start, len, buf); }
    void SetByteArrayRegion(jbyteArray array, jsize start, jsize len,
        const jbyte* buf)
    { functions->SetByteArrayRegion(this, array, start, len, buf); }
    void SetCharArrayRegion(jcharArray array, jsize start, jsize len,
        const jchar* buf)
    { functions->SetCharArrayRegion(this, array, start, len, buf); }
    void SetShortArrayRegion(jshortArray array, jsize start, jsize len,
        const jshort* buf)
    { functions->SetShortArrayRegion(this, array, start, len, buf); }
    void SetIntArrayRegion(jintArray array, jsize start, jsize len,
        const jint* buf)
    { functions->SetIntArrayRegion(this, array, start, len, buf); }
    void SetLongArrayRegion(jlongArray array, jsize start, jsize len,
```

- **EN:** Important local symbols in this block include GetDoubleArrayRegion, SetBooleanArrayRegion, SetByteArrayRegion, SetCharArrayRegion, SetShortArrayRegion, SetIntArrayRegion.
- **CN:** 该代码块中的重要局部符号包括 GetDoubleArrayRegion、SetBooleanArrayRegion、SetByteArrayRegion、SetCharArrayRegion、SetShortArrayRegion、SetIntArrayRegion。

### Lines 966-978 / 第 966-978 行

```cpp
        const jlong* buf)
    { functions->SetLongArrayRegion(this, array, start, len, buf); }
    void SetFloatArrayRegion(jfloatArray array, jsize start, jsize len,
        const jfloat* buf)
    { functions->SetFloatArrayRegion(this, array, start, len, buf); }
    void SetDoubleArrayRegion(jdoubleArray array, jsize start, jsize len,
        const jdouble* buf)
    { functions->SetDoubleArrayRegion(this, array, start, len, buf); }

    jint RegisterNatives(jclass clazz, const JNINativeMethod* methods,
        jint nMethods)
    { return functions->RegisterNatives(this, clazz, methods, nMethods); }

```

- **EN:** Important local symbols in this block include SetFloatArrayRegion, SetDoubleArrayRegion, RegisterNatives.
- **CN:** 该代码块中的重要局部符号包括 SetFloatArrayRegion、SetDoubleArrayRegion、RegisterNatives。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 979-990 / 第 979-990 行

```cpp
    jint UnregisterNatives(jclass clazz)
    { return functions->UnregisterNatives(this, clazz); }

    jint MonitorEnter(jobject obj)
    { return functions->MonitorEnter(this, obj); }

    jint MonitorExit(jobject obj)
    { return functions->MonitorExit(this, obj); }

    jint GetJavaVM(JavaVM** vm)
    { return functions->GetJavaVM(this, vm); }

```

- **EN:** Important local symbols in this block include UnregisterNatives, MonitorEnter, MonitorExit, GetJavaVM.
- **CN:** 该代码块中的重要局部符号包括 UnregisterNatives、MonitorEnter、MonitorExit、GetJavaVM。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 991-1002 / 第 991-1002 行

```cpp
    void GetStringRegion(jstring str, jsize start, jsize len, jchar* buf)
    { functions->GetStringRegion(this, str, start, len, buf); }

    void GetStringUTFRegion(jstring str, jsize start, jsize len, char* buf)
    { return functions->GetStringUTFRegion(this, str, start, len, buf); }

    void* GetPrimitiveArrayCritical(jarray array, jboolean* isCopy)
    { return functions->GetPrimitiveArrayCritical(this, array, isCopy); }

    void ReleasePrimitiveArrayCritical(jarray array, void* carray, jint mode)
    { functions->ReleasePrimitiveArrayCritical(this, array, carray, mode); }

```

- **EN:** Important local symbols in this block include GetStringRegion, GetStringUTFRegion, GetPrimitiveArrayCritical, ReleasePrimitiveArrayCritical.
- **CN:** 该代码块中的重要局部符号包括 GetStringRegion、GetStringUTFRegion、GetPrimitiveArrayCritical、ReleasePrimitiveArrayCritical。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 1003-1014 / 第 1003-1014 行

```cpp
    const jchar* GetStringCritical(jstring string, jboolean* isCopy)
    { return functions->GetStringCritical(this, string, isCopy); }

    void ReleaseStringCritical(jstring string, const jchar* carray)
    { functions->ReleaseStringCritical(this, string, carray); }

    jweak NewWeakGlobalRef(jobject obj)
    { return functions->NewWeakGlobalRef(this, obj); }

    void DeleteWeakGlobalRef(jweak obj)
    { functions->DeleteWeakGlobalRef(this, obj); }

```

- **EN:** Important local symbols in this block include GetStringCritical, ReleaseStringCritical, NewWeakGlobalRef, DeleteWeakGlobalRef.
- **CN:** 该代码块中的重要局部符号包括 GetStringCritical、ReleaseStringCritical、NewWeakGlobalRef、DeleteWeakGlobalRef。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 1015-1026 / 第 1015-1026 行

```cpp
    jboolean ExceptionCheck()
    { return functions->ExceptionCheck(this); }

    jobject NewDirectByteBuffer(void* address, jlong capacity)
    { return functions->NewDirectByteBuffer(this, address, capacity); }

    void* GetDirectBufferAddress(jobject buf)
    { return functions->GetDirectBufferAddress(this, buf); }

    jlong GetDirectBufferCapacity(jobject buf)
    { return functions->GetDirectBufferCapacity(this, buf); }

```

- **EN:** Important local symbols in this block include ExceptionCheck, NewDirectByteBuffer, GetDirectBufferAddress, GetDirectBufferCapacity.
- **CN:** 该代码块中的重要局部符号包括 ExceptionCheck、NewDirectByteBuffer、GetDirectBufferAddress、GetDirectBufferCapacity。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 1027-1037 / 第 1027-1037 行

```cpp
    /* added in JNI 1.6 */
    jobjectRefType GetObjectRefType(jobject obj)
    { return functions->GetObjectRefType(this, obj); }
#endif /*__cplusplus*/
};


/*
 * JNI invocation interface.
 */
struct JNIInvokeInterface {
```

- **EN:** Important local symbols in this block include JNIInvokeInterface, GetObjectRefType.
- **CN:** 该代码块中的重要局部符号包括 JNIInvokeInterface、GetObjectRefType。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 1038-1048 / 第 1038-1048 行

```cpp
    void*       reserved0;
    void*       reserved1;
    void*       reserved2;

    jint        (*DestroyJavaVM)(JavaVM*);
    jint        (*AttachCurrentThread)(JavaVM*, JNIEnv**, void*);
    jint        (*DetachCurrentThread)(JavaVM*);
    jint        (*GetEnv)(JavaVM*, void**, jint);
    jint        (*AttachCurrentThreadAsDaemon)(JavaVM*, JNIEnv**, void*);
};

```

- **EN:** Important local symbols in this block include jint.
- **CN:** 该代码块中的重要局部符号包括 jint。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 1049-1067 / 第 1049-1067 行

```cpp
/*
 * C++ version.
 */
struct _JavaVM {
    const struct JNIInvokeInterface* functions;

#if defined(__cplusplus)
    jint DestroyJavaVM()
    { return functions->DestroyJavaVM(this); }
    jint AttachCurrentThread(JNIEnv** p_env, void* thr_args)
    { return functions->AttachCurrentThread(this, p_env, thr_args); }
    jint DetachCurrentThread()
    { return functions->DetachCurrentThread(this); }
    jint GetEnv(void** env, jint version)
    { return functions->GetEnv(this, env, version); }
    jint AttachCurrentThreadAsDaemon(JNIEnv** p_env, void* thr_args)
    { return functions->AttachCurrentThreadAsDaemon(this, p_env, thr_args); }
#endif /*__cplusplus*/
};
```

- **EN:** Important local symbols in this block include _JavaVM, JNIInvokeInterface, DestroyJavaVM, AttachCurrentThread, DetachCurrentThread, GetEnv.
- **CN:** 该代码块中的重要局部符号包括 _JavaVM、JNIInvokeInterface、DestroyJavaVM、AttachCurrentThread、DetachCurrentThread、GetEnv。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 1068-1084 / 第 1068-1084 行

```cpp

struct JavaVMAttachArgs {
    jint        version;    /* must be >= JNI_VERSION_1_2 */
    const char* name;       /* NULL or name of thread as modified UTF-8 str */
    jobject     group;      /* global ref of a ThreadGroup object, or NULL */
};
typedef struct JavaVMAttachArgs JavaVMAttachArgs;

/*
 * JNI 1.2+ initialization.  (As of 1.6, the pre-1.2 structures are no
 * longer supported.)
 */
typedef struct JavaVMOption {
    const char* optionString;
    void*       extraInfo;
} JavaVMOption;

```

- **EN:** Important local symbols in this block include JavaVMAttachArgs, JavaVMOption.
- **CN:** 该代码块中的重要局部符号包括 JavaVMAttachArgs、JavaVMOption。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 1085-1104 / 第 1085-1104 行

```cpp
typedef struct JavaVMInitArgs {
    jint        version;    /* use JNI_VERSION_1_2 or later */

    jint        nOptions;
    JavaVMOption* options;
    jboolean    ignoreUnrecognized;
} JavaVMInitArgs;

#ifdef __cplusplus
extern "C" {
#endif
/*
 * VM initialization functions.
 *
 * Note these are the only symbols exported for JNI by the VM.
 */
jint JNI_GetDefaultJavaVMInitArgs(void*);
jint JNI_CreateJavaVM(JavaVM**, JNIEnv**, void*);
jint JNI_GetCreatedJavaVMs(JavaVM**, jsize, jsize*);

```

- **EN:** Important local symbols in this block include JavaVMInitArgs, JNI_GetDefaultJavaVMInitArgs, JNI_CreateJavaVM, JNI_GetCreatedJavaVMs.
- **CN:** 该代码块中的重要局部符号包括 JavaVMInitArgs、JNI_GetDefaultJavaVMInitArgs、JNI_CreateJavaVM、JNI_GetCreatedJavaVMs。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 1105-1115 / 第 1105-1115 行

```cpp
#define JNIIMPORT
#define JNIEXPORT  __attribute__ ((visibility ("default")))
#define JNICALL

/*
 * Prototypes for functions exported by loadable shared libs.  These are
 * called by JNI, not provided by JNI.
 */
JNIEXPORT jint JNI_OnLoad(JavaVM* vm, void* reserved);
JNIEXPORT void JNI_OnUnload(JavaVM* vm, void* reserved);

```

- **EN:** Important local symbols in this block include JNI_OnLoad, JNI_OnUnload.
- **CN:** 该代码块中的重要局部符号包括 JNI_OnLoad、JNI_OnUnload。
- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 1116-1126 / 第 1116-1126 行

```cpp
#ifdef __cplusplus
}
#endif


/*
 * Manifest constants.
 */
#define JNI_FALSE   0
#define JNI_TRUE    1

```

- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 1127-1139 / 第 1127-1139 行

```cpp
#define JNI_VERSION_1_1 0x00010001
#define JNI_VERSION_1_2 0x00010002
#define JNI_VERSION_1_4 0x00010004
#define JNI_VERSION_1_6 0x00010006

#define JNI_OK          (0)         /* no error */
#define JNI_ERR         (-1)        /* generic error */
#define JNI_EDETACHED   (-2)        /* thread detached from the VM */
#define JNI_EVERSION    (-3)        /* JNI version error */
#define JNI_ENOMEM      (-4)        /* Out of memory */
#define JNI_EEXIST      (-5)        /* VM already created */
#define JNI_EINVAL      (-6)        /* Invalid argument */

```

- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

### Lines 1140-1143 / 第 1140-1143 行

```cpp
#define JNI_COMMIT      1           /* copy content, do not free buffer */
#define JNI_ABORT       2           /* free buffer w/o copying back */

#endif  /* JNI_H_ */
```

- **EN:** JNI-facing code appears here, indicating data marshaling and Java/native boundary checks.
- **CN:** 这里出现 JNI 相关代码，说明重点包含数据封送以及 Java/原生边界检查。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **Android integration** — 聚焦 Android 构建、打包或运行时集成细节。
- **JNI bridge** — 把面向 Java/Kotlin 的 Android 代码与原生 C++ 运行时连接起来。
- **Representative symbols: _jobject, _jclass, _jstring, _jarray, _jobjectArray, _jbooleanArray, _jbyteArray, _jcharArray** — 代表性符号：_jobject、_jclass、_jstring、_jarray、_jobjectArray、_jbooleanArray、_jbyteArray、_jcharArray

## Dependencies / 依赖关系

- `stdarg.h`
- `stdint.h`
