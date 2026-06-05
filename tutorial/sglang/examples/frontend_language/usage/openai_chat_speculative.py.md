# openai_chat_speculative.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/frontend_language/usage/openai_chat_speculative.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example demonstrates openai chat speculative within the SGLang repository. / 该示例展示了在 SGLang 仓库中如何完成 openai chat speculative。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Module overview and usage
````python
"""
Usage:
***Note: for speculative execution to work, user must put all "gen" in "assistant".
Show in "assistant" the desired answer format. Each "gen" term should have a stop token.
The stream mode is not supported in speculative execution.

E.g.
correct:
    sgl.assistant("\nName:" + sgl.gen("name", stop="\n") + "\nBirthday:" + sgl.gen("birthday", stop="\n") + "\nJob:" + sgl.gen("job", stop="\n"))
incorrect:
    s += sgl.assistant("\nName:" + sgl.gen("name", stop="\n"))
    s += sgl.assistant("\nBirthday:" + sgl.gen("birthday", stop="\n"))
    s += sgl.assistant("\nJob:" + sgl.gen("job", stop="\n"))

export OPENAI_API_KEY=sk-******
python3 openai_chat_speculative.py
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 19-20: Import dependencies and runtime symbols
````python
import sglang as sgl
from sglang import OpenAI, function, set_default_backend
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 23-38: Generate character spec
````python
@function(num_api_spec_tokens=256)
def gen_character_spec(s):
    s += sgl.system("You are a helpful assistant.")
    s += sgl.user("Construct a character within the following format:")
    s += sgl.assistant(
        "Name: Steve Jobs.\nBirthday: February 24, 1955.\nJob: Apple CEO.\n"
    )
    s += sgl.user("Please generate new Name, Birthday and Job.\n")
    s += sgl.assistant(
        "Name:"
        + sgl.gen("name", stop="\n")
        + "\nBirthday:"
        + sgl.gen("birthday", stop="\n")
        + "\nJob:"
        + sgl.gen("job", stop="\n")
    )
````
**EN:** This function encapsulates the “Generate character spec” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Generate character spec”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 41-51: Generate character spec no few shot
````python
@function(num_api_spec_tokens=256)
def gen_character_spec_no_few_shot(s):
    s += sgl.user("Construct a character. For each field stop with a newline\n")
    s += sgl.assistant(
        "Name:"
        + sgl.gen("name", stop="\n")
        + "\nAge:"
        + sgl.gen("age", stop="\n")
        + "\nJob:"
        + sgl.gen("job", stop="\n")
    )
````
**EN:** This function encapsulates the “Generate character spec no few shot” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Generate character spec no few shot”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 54-58: Define gen_character_normal prompt program
````python
@function
def gen_character_normal(s):
    s += sgl.system("You are a helpful assistant.")
    s += sgl.user("What's the answer of 23 + 8?")
    s += sgl.assistant(sgl.gen("answer", max_tokens=64))
````
**EN:** This function defines an SGLang prompt program that appends turns to the state object and reserves one or more generation slots for model output.
**CN:** 该函数定义了一个 SGLang 提示程序：它会向状态对象追加对话或指令内容，并预留一个或多个生成槽位来接收模型输出。

### Lines 61-77: Handle turn question
````python
@function(num_api_spec_tokens=1024)
def multi_turn_question(s, question_1, question_2):
    s += sgl.system("You are a helpful assistant.")
    s += sgl.user("Answer questions in the following format:")
    s += sgl.user(
        "Question 1: What is the capital of France?\nQuestion 2: What is the population of this city?\n"
    )
    s += sgl.assistant(
        "Answer 1: The capital of France is Paris.\nAnswer 2: The population of Paris in 2024 is estimated to be around 2.1 million for the city proper.\n"
    )
    s += sgl.user("Question 1: " + question_1 + "\nQuestion 2: " + question_2)
    s += sgl.assistant(
        "Answer 1: "
        + sgl.gen("answer_1", stop="\n")
        + "\nAnswer 2: "
        + sgl.gen("answer_2", stop="\n")
    )
````
**EN:** This function encapsulates the “Handle turn question” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle turn question”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 80-90: Handle spec single turn
````python
def test_spec_single_turn():
    backend.token_usage.reset()

    state = gen_character_spec.run()
    for m in state.messages():
        print(m["role"], ":", m["content"])

    print("\n-- name:", state["name"])
    print("-- birthday:", state["birthday"])
    print("-- job:", state["job"])
    print(backend.token_usage)
````
**EN:** This function encapsulates the “Handle spec single turn” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle spec single turn”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 93-100: Handle inaccurate spec single turn
````python
def test_inaccurate_spec_single_turn():
    state = gen_character_spec_no_few_shot.run()
    for m in state.messages():
        print(m["role"], ":", m["content"])

    print("\n-- name:", state["name"])
    print("\n-- age:", state["age"])
    print("\n-- job:", state["job"])
````
**EN:** This function encapsulates the “Handle inaccurate spec single turn” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle inaccurate spec single turn”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 103-106: Handle normal single turn
````python
def test_normal_single_turn():
    state = gen_character_normal.run()
    for m in state.messages():
        print(m["role"], ":", m["content"])
````
**EN:** This function encapsulates the “Handle normal single turn” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle normal single turn”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 109-119: Handle spec multi turn
````python
def test_spec_multi_turn():
    state = multi_turn_question.run(
        question_1="What is the capital of the United States?",
        question_2="List two local attractions in the capital of the United States.",
    )

    for m in state.messages():
        print(m["role"], ":", m["content"])

    print("\n-- answer_1 --\n", state["answer_1"])
    print("\n-- answer_2 --\n", state["answer_2"])
````
**EN:** This function encapsulates the “Handle spec multi turn” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle spec multi turn”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 122-130: Stream incremental output
````python
def test_spec_multi_turn_stream():
    state = multi_turn_question.run(
        question_1="What is the capital of the United States?",
        question_2="List two local attractions.",
        stream=True,
    )

    for out in state.text_iter():
        print(out, end="", flush=True)
````
**EN:** This function enables streaming mode and consumes the result incrementally so the caller can observe partial output as soon as it is produced.
**CN:** 该函数会启用流式模式并逐步消费结果，因此调用方可以在内容产生后立即看到部分输出。

### Lines 133-155: Program entry point
````python
if __name__ == "__main__":
    backend = OpenAI("gpt-4-turbo")
    set_default_backend(backend)

    print("\n========== test spec single turn ==========\n")
    # expect reasonable answer for each field
    test_spec_single_turn()

    print("\n========== test inaccurate spec single turn ==========\n")
    # expect incomplete or unreasonable answers
    test_inaccurate_spec_single_turn()

    print("\n========== test normal single turn ==========\n")
    # expect reasonable answer
    test_normal_single_turn()

    print("\n========== test spec multi turn ==========\n")
    # expect answer with same format as in the few shot
    test_spec_multi_turn()

    print("\n========== test spec multi turn stream ==========\n")
    # expect error in stream_executor: stream is not supported...
    test_spec_multi_turn_stream()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Prompt programming / 提示程序编排**: The file builds prompts with SGLang state transitions. / 该文件使用 SGLang 状态转换来构造提示程序。
- **Streaming output / 流式输出**: Results can be consumed incrementally as tokens arrive. / 结果可以在 token 到达时被逐步消费。

## Dependencies / 依赖关系
- **Project-specific / 项目相关**: sglang, sglang.OpenAI, sglang.function, sglang.set_default_backend
