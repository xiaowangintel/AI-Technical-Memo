# NativePeer.java — Code Analysis / 代码分析

## Source / 来源
- File: `android/pytorch_android/src/main/java/org/pytorch/NativePeer.java`
- Repository: `pytorch`
- Purpose (EN): Supports Android/mobile runtime integration in the PyTorch repository.
- 用途 (CN): 为 PyTorch 仓库中的 Android/移动运行时集成提供支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
````java
 1: package org.pytorch;
 2: 
 3: import com.facebook.jni.HybridData;
 4: import com.facebook.jni.annotations.DoNotStrip;
 5: import com.facebook.soloader.nativeloader.NativeLoader;
 6: import java.util.Map;
 7: 
 8: class NativePeer implements INativePeer {
 9:   static {
10:     NativeLoader.loadLibrary("pytorch_jni");
11:     PyTorchCodegenLoader.loadNativeLibs();
12:   }
13: 
14:   private final HybridData mHybridData;
15: 
16:   @DoNotStrip
17:   private static native HybridData initHybrid(
18:       String moduleAbsolutePath, Map<String, String> extraFiles, int deviceJniCode);
19: 
20:   @DoNotStrip
````
- EN: Handles module imports such as `com.facebook.jni.HybridData;`, `com.facebook.jni.annotations.DoNotStrip;`, `com.facebook.soloader.nativeloader.NativeLoader;`, `java.util.Map;`.
- CN: 处理模块导入，例如 `com.facebook.jni.HybridData;`, `com.facebook.jni.annotations.DoNotStrip;`, `com.facebook.soloader.nativeloader.NativeLoader;`, `java.util.Map;`。
- EN: Imports Java types such as `com.facebook.jni.HybridData`, `com.facebook.jni.annotations.DoNotStrip`, `com.facebook.soloader.nativeloader.NativeLoader`, `java.util.Map`.
- CN: 导入 Java 类型，例如 `com.facebook.jni.HybridData`, `com.facebook.jni.annotations.DoNotStrip`, `com.facebook.soloader.nativeloader.NativeLoader`, `java.util.Map`。
- EN: Declares or extends types including `NativePeer`.
- CN: 声明或扩展类型，包括 `NativePeer`。
- EN: Implements callable logic such as `initHybrid`.
- CN: 实现可调用逻辑，例如 `initHybrid`。

### Lines 21-40
````java
21:   private static native HybridData initHybridAndroidAsset(
22:       String assetName, /* android.content.res.AssetManager */
23:       Object androidAssetManager,
24:       int deviceJniCode);
25: 
26:   NativePeer(String moduleAbsolutePath, Map<String, String> extraFiles, Device device) {
27:     mHybridData = initHybrid(moduleAbsolutePath, extraFiles, device.jniCode);
28:   }
29: 
30:   NativePeer(
31:       String assetName, /* android.content.res.AssetManager */
32:       Object androidAssetManager,
33:       Device device) {
34:     mHybridData = initHybridAndroidAsset(assetName, androidAssetManager, device.jniCode);
35:   }
36: 
37:   public void resetNative() {
38:     mHybridData.resetNative();
39:   }
40: 
````
- EN: Implements callable logic such as `initHybridAndroidAsset`, `resetNative`.
- CN: 实现可调用逻辑，例如 `initHybridAndroidAsset`, `resetNative`。

### Lines 41-46
````java
41:   @DoNotStrip
42:   public native IValue forward(IValue... inputs);
43: 
44:   @DoNotStrip
45:   public native IValue runMethod(String methodName, IValue... inputs);
46: }
````
- EN: Implements callable logic such as `forward`, `runMethod`.
- CN: 实现可调用逻辑，例如 `forward`, `runMethod`。

## Key Concepts / 关键概念
- Android integration / Android 集成
- Symbol `NativePeer` / 符号 `NativePeer`
- Symbol `initHybrid` / 符号 `initHybrid`
- Symbol `initHybridAndroidAsset` / 符号 `initHybridAndroidAsset`
- Symbol `resetNative` / 符号 `resetNative`

## Dependencies / 依赖关系
- Java imports: `com.facebook.jni.HybridData`, `com.facebook.jni.annotations.DoNotStrip`, `com.facebook.soloader.nativeloader.NativeLoader`, `java.util.Map`
- Java 导入: `com.facebook.jni.HybridData`, `com.facebook.jni.annotations.DoNotStrip`, `com.facebook.soloader.nativeloader.NativeLoader`, `java.util.Map`
