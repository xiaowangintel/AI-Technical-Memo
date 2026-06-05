# LiteNativePeer.java — Code Analysis / 代码分析

## Source / 来源
- File: `android/pytorch_android/src/main/java/org/pytorch/LiteNativePeer.java`
- Repository: `pytorch`
- Purpose (EN): Supports Android/mobile runtime integration in the PyTorch repository.
- 用途 (CN): 为 PyTorch 仓库中的 Android/移动运行时集成提供支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
````java
 1: package org.pytorch;
 2: 
 3: import com.facebook.jni.HybridData;
 4: import com.facebook.soloader.nativeloader.NativeLoader;
 5: import com.facebook.soloader.nativeloader.SystemDelegate;
 6: import java.util.Map;
 7: 
 8: class LiteNativePeer implements INativePeer {
 9:   static {
10:     if (!NativeLoader.isInitialized()) {
11:       NativeLoader.init(new SystemDelegate());
12:     }
13:     NativeLoader.loadLibrary("pytorch_jni_lite");
14:     PyTorchCodegenLoader.loadNativeLibs();
15:   }
16: 
17:   private final HybridData mHybridData;
18: 
19:   private static native HybridData initHybrid(
20:       String moduleAbsolutePath, Map<String, String> extraFiles, int deviceJniCode);
````
- EN: Handles module imports such as `com.facebook.jni.HybridData;`, `com.facebook.soloader.nativeloader.NativeLoader;`, `com.facebook.soloader.nativeloader.SystemDelegate;`, `java.util.Map;`.
- CN: 处理模块导入，例如 `com.facebook.jni.HybridData;`, `com.facebook.soloader.nativeloader.NativeLoader;`, `com.facebook.soloader.nativeloader.SystemDelegate;`, `java.util.Map;`。
- EN: Imports Java types such as `com.facebook.jni.HybridData`, `com.facebook.soloader.nativeloader.NativeLoader`, `com.facebook.soloader.nativeloader.SystemDelegate`, `java.util.Map`.
- CN: 导入 Java 类型，例如 `com.facebook.jni.HybridData`, `com.facebook.soloader.nativeloader.NativeLoader`, `com.facebook.soloader.nativeloader.SystemDelegate`, `java.util.Map`。
- EN: Declares or extends types including `LiteNativePeer`.
- CN: 声明或扩展类型，包括 `LiteNativePeer`。
- EN: Implements callable logic such as `initHybrid`.
- CN: 实现可调用逻辑，例如 `initHybrid`。

### Lines 21-40
````java
21: 
22:   private static native HybridData initHybridAndroidAsset(
23:       String assetName, /* android.content.res.AssetManager */
24:       Object androidAssetManager,
25:       int deviceJniCode);
26: 
27:   LiteNativePeer(String moduleAbsolutePath, Map<String, String> extraFiles, Device device) {
28:     mHybridData = initHybrid(moduleAbsolutePath, extraFiles, device.jniCode);
29:   }
30: 
31:   LiteNativePeer(
32:       String assetName, /* android.content.res.AssetManager */
33:       Object androidAssetManager,
34:       Device device) {
35:     mHybridData = initHybridAndroidAsset(assetName, androidAssetManager, device.jniCode);
36:   }
37: 
38:   /**
39:    * Explicitly destroys the native torch::jit::mobile::Module. Calling this method is not required,
40:    * as the native object will be destroyed when this object is garbage-collected. However, the
````
- EN: Implements callable logic such as `initHybridAndroidAsset`.
- CN: 实现可调用逻辑，例如 `initHybridAndroidAsset`。

### Lines 41-60
````java
41:    * timing of garbage collection is not guaranteed, so proactively calling {@code resetNative} can
42:    * free memory more quickly. See {@link com.facebook.jni.HybridData#resetNative}.
43:    */
44:   public void resetNative() {
45:     mHybridData.resetNative();
46:   }
47: 
48:   /**
49:    * Runs the 'forward' method of this module with the specified arguments.
50:    *
51:    * @param inputs arguments for the TorchScript module's 'forward' method.
52:    * @return return value from the 'forward' method.
53:    */
54:   public native IValue forward(IValue... inputs);
55: 
56:   /**
57:    * Runs the specified method of this module with the specified arguments.
58:    *
59:    * @param methodName name of the TorchScript method to run.
60:    * @param inputs arguments that will be passed to TorchScript method.
````
- EN: Implements callable logic such as `resetNative`, `forward`.
- CN: 实现可调用逻辑，例如 `resetNative`, `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 61-64
````java
61:    * @return return value from the method.
62:    */
63:   public native IValue runMethod(String methodName, IValue... inputs);
64: }
````
- EN: Implements callable logic such as `runMethod`.
- CN: 实现可调用逻辑，例如 `runMethod`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- Symbol `LiteNativePeer` / 符号 `LiteNativePeer`
- Symbol `initHybrid` / 符号 `initHybrid`
- Symbol `initHybridAndroidAsset` / 符号 `initHybridAndroidAsset`
- Symbol `resetNative` / 符号 `resetNative`

## Dependencies / 依赖关系
- Java imports: `com.facebook.jni.HybridData`, `com.facebook.soloader.nativeloader.NativeLoader`, `com.facebook.soloader.nativeloader.SystemDelegate`, `java.util.Map`
- Java 导入: `com.facebook.jni.HybridData`, `com.facebook.soloader.nativeloader.NativeLoader`, `com.facebook.soloader.nativeloader.SystemDelegate`, `java.util.Map`
