# Module.java — Code Analysis / 代码分析

## Source / 来源
- File: `android/pytorch_android/src/main/java/org/pytorch/Module.java`
- Repository: `pytorch`
- Purpose (EN): Supports Android/mobile runtime integration in the PyTorch repository.
- 用途 (CN): 为 PyTorch 仓库中的 Android/移动运行时集成提供支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
````java
 1: // Copyright 2004-present Facebook. All Rights Reserved.
 2: 
 3: package org.pytorch;
 4: 
 5: import com.facebook.soloader.nativeloader.NativeLoader;
 6: import com.facebook.soloader.nativeloader.SystemDelegate;
 7: import java.util.Map;
 8: 
 9: /** Java wrapper for torch::jit::Module. */
10: public class Module {
11: 
12:   private INativePeer mNativePeer;
13: 
14:   /**
15:    * Loads a serialized TorchScript module from the specified path on the disk to run on specified
16:    * device.
17:    *
18:    * @param modelPath path to file that contains the serialized TorchScript module.
19:    * @param extraFiles map with extra files names as keys, content of them will be loaded to values.
20:    * @param device {@link org.pytorch.Device} to use for running specified module.
````
- EN: Handles module imports such as `com.facebook.soloader.nativeloader.NativeLoader;`, `com.facebook.soloader.nativeloader.SystemDelegate;`, `java.util.Map;`.
- CN: 处理模块导入，例如 `com.facebook.soloader.nativeloader.NativeLoader;`, `com.facebook.soloader.nativeloader.SystemDelegate;`, `java.util.Map;`。
- EN: Imports Java types such as `com.facebook.soloader.nativeloader.NativeLoader`, `com.facebook.soloader.nativeloader.SystemDelegate`, `java.util.Map`.
- CN: 导入 Java 类型，例如 `com.facebook.soloader.nativeloader.NativeLoader`, `com.facebook.soloader.nativeloader.SystemDelegate`, `java.util.Map`。
- EN: Declares or extends types including `Module`.
- CN: 声明或扩展类型，包括 `Module`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 21-40
````java
21:    * @return new {@link org.pytorch.Module} object which owns torch::jit::Module.
22:    */
23:   public static Module load(
24:       final String modelPath, final Map<String, String> extraFiles, final Device device) {
25:     if (!NativeLoader.isInitialized()) {
26:       NativeLoader.init(new SystemDelegate());
27:     }
28:     return new Module(new NativePeer(modelPath, extraFiles, device));
29:   }
30: 
31:   /**
32:    * Loads a serialized TorchScript module from the specified path on the disk to run on CPU.
33:    *
34:    * @param modelPath path to file that contains the serialized TorchScript module.
35:    * @return new {@link org.pytorch.Module} object which owns torch::jit::Module.
36:    */
37:   public static Module load(final String modelPath) {
38:     return load(modelPath, null, Device.CPU);
39:   }
40: 
````
- EN: Implements callable logic such as `load`.
- CN: 实现可调用逻辑，例如 `load`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 41-60
````java
41:   Module(INativePeer nativePeer) {
42:     this.mNativePeer = nativePeer;
43:   }
44: 
45:   /**
46:    * Runs the 'forward' method of this module with the specified arguments.
47:    *
48:    * @param inputs arguments for the TorchScript module's 'forward' method.
49:    * @return return value from the 'forward' method.
50:    */
51:   public IValue forward(IValue... inputs) {
52:     return mNativePeer.forward(inputs);
53:   }
54: 
55:   /**
56:    * Runs the specified method of this module with the specified arguments.
57:    *
58:    * @param methodName name of the TorchScript method to run.
59:    * @param inputs arguments that will be passed to TorchScript method.
60:    * @return return value from the method.
````
- EN: Implements callable logic such as `forward`.
- CN: 实现可调用逻辑，例如 `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 61-75
````java
61:    */
62:   public IValue runMethod(String methodName, IValue... inputs) {
63:     return mNativePeer.runMethod(methodName, inputs);
64:   }
65: 
66:   /**
67:    * Explicitly destroys the native torch::jit::Module. Calling this method is not required, as the
68:    * native object will be destroyed when this object is garbage-collected. However, the timing of
69:    * garbage collection is not guaranteed, so proactively calling {@code destroy} can free memory
70:    * more quickly. See {@link com.facebook.jni.HybridData#resetNative}.
71:    */
72:   public void destroy() {
73:     mNativePeer.resetNative();
74:   }
75: }
````
- EN: Implements callable logic such as `runMethod`, `destroy`.
- CN: 实现可调用逻辑，例如 `runMethod`, `destroy`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- Symbol `Module` / 符号 `Module`
- Symbol `load` / 符号 `load`
- Symbol `forward` / 符号 `forward`
- Symbol `runMethod` / 符号 `runMethod`

## Dependencies / 依赖关系
- Java imports: `com.facebook.soloader.nativeloader.NativeLoader`, `com.facebook.soloader.nativeloader.SystemDelegate`, `java.util.Map`
- Java 导入: `com.facebook.soloader.nativeloader.NativeLoader`, `com.facebook.soloader.nativeloader.SystemDelegate`, `java.util.Map`
