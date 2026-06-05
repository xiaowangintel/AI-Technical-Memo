# parallel_sample.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/frontend_language/usage/parallel_sample.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example demonstrates parallel sample within the SGLang repository. / 该示例展示了在 SGLang 仓库中如何完成 parallel sample。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Module overview and usage
````python
"""
Usage:
python3 parallel_sample.py
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 6-6: Import dependencies and runtime symbols
````python
import sglang as sgl
````
**EN:** The file only needs the SGLang frontend/runtime API, which it imports under the `sgl` alias.
**CN:** 该文件只依赖 SGLang 前端/运行时 API，并以 `sgl` 别名导入。

### Lines 9-26: Define parallel_sample prompt program
````python
@sgl.function
def parallel_sample(s, question, n):
    s += (
        "Question: Compute 1 + 2 + 3\n"
        "Reasoning: I need to use a calculator.\n"
        "Tool: calculator\n"
        "Answer: 6\n"
        "Question: Compute 3 + 2 + 2\n"
        "Reasoning: I will try a calculator.\n"
        "Tool: calculator\n"
        "Answer: 7\n"
    )
    s += "Question: " + question + "\n"
    forks = s.fork(n)
    forks += "Reasoning:" + sgl.gen("reasoning", stop="\n") + "\n"
    forks += "Tool:" + sgl.gen("tool", choices=["calculator", "browser"]) + "\n"
    forks += "Answer:" + sgl.gen("answer", stop="\n") + "\n"
    forks.join()
````
**EN:** This function defines an SGLang prompt program that appends turns to the state object and reserves one or more generation slots for model output.
**CN:** 该函数定义了一个 SGLang 提示程序：它会向状态对象追加对话或指令内容，并预留一个或多个生成槽位来接收模型输出。

### Lines 29-40: Configure the default backend
````python
sgl.set_default_backend(sgl.OpenAI("gpt-3.5-turbo-instruct"))
# sgl.set_default_backend(sgl.RuntimeEndpoint("http://localhost:30000"))

state = parallel_sample.run(question="Compute 5 + 2 + 4.", n=5, temperature=1.0)

for i in range(5):
    obj = {
        "reasoning": state["reasoning"][i],
        "tool": state["tool"][i],
        "answer": state["answer"][i],
    }
    print(f"[{i}], {obj}")
````
**EN:** The example selects a concrete backend once so later prompt executions automatically use the same model endpoint.
**CN:** 该示例先统一选择具体后端，这样后续提示执行都会自动使用同一个模型端点。

## Key Concepts / 关键概念
- **Prompt programming / 提示程序编排**: The file builds prompts with SGLang state transitions. / 该文件使用 SGLang 状态转换来构造提示程序。
- **Structured generation / 结构化生成**: Schemas or constrained decoding keep outputs machine-readable. / 模式约束或受限解码让输出保持机器可读。

## Dependencies / 依赖关系
- **Project-specific / 项目相关**: sglang
