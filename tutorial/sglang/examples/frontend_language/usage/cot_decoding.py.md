# cot_decoding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/frontend_language/usage/cot_decoding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example demonstrates cot decoding within the SGLang repository. / 该示例展示了在 SGLang 仓库中如何完成 cot decoding。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Import dependencies and runtime symbols
````python
from math import exp
from pprint import pformat

import sglang as sgl
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 6-9: Set top-level configuration
````python
YELLOW = "\033[1;33m"
GREEN = "\033[1;32m"
BLUE = "\033[1;34m"
CLEAR = "\033[1;0m"
````
**EN:** This block establishes constants or shared objects that the rest of the file relies on.
**CN:** 该代码块建立了后续逻辑依赖的常量或共享对象。

### Lines 12-105: Define cot_decoding prompt program
````python
@sgl.function
def cot_decoding(s, question, get_top_k, is_chat_model, verbose):
    """CoT Decoding: http://arxiv.org/abs/2402.10200"""

    if is_chat_model:
        s += sgl.user("Question: " + question + "\nAnswer:")
        s += sgl.assistant_begin()
    else:
        s += "Question: " + question + "\nAnswer:"

    step_0 = s.fork(1)[0]
    forks = s.fork(get_top_k)
    answer_forks = s.fork(get_top_k)

    # decoding step 0
    step_0 += sgl.gen(
        "get_top_k",
        max_tokens=0,
        return_logprob=True,
        top_logprobs_num=get_top_k,
        return_text_in_logprobs=True,
    )
    logprobs = step_0.get_meta_info("get_top_k")["output_top_logprobs"][0]

    print("Decoding step 0:", ", ".join(pformat(token[2]) for token in logprobs))
    for idx, (f, token) in enumerate(zip(forks, logprobs)):
        logprob, token_id, text = token
        f += text

        if text == "<|end_of_text|>":
            print(
                f"{YELLOW}Path #{idx} {pformat(text)}[{exp(logprob):.3f}] (score=nan, answer=nan){CLEAR}"
            )
            continue

        # continue greedy decoding
        f += sgl.gen(
            "answer",
            temperature=0,
            max_tokens=1024,
            return_logprob=True,
            top_logprobs_num=2,
            return_text_in_logprobs=True,
        )

        # calculate probability disparity between the top and secondary tokens
        x1s = [exp(xt[0][0]) for xt in f.get_meta_info("answer")["output_top_logprobs"]]
        x2s = [exp(xt[1][0]) for xt in f.get_meta_info("answer")["output_top_logprobs"]]
        tokens = [xt[0][2] for xt in f.get_meta_info("answer")["output_top_logprobs"]]
        delta = (sum(x1s) - sum(x2s)) / len(x1s)

        # extract the answer span (without the '<|end_of_text|>' token)
        answer_forks[idx] += text + f["answer"] + "\nSo the answer is"
        answer_forks[idx] += sgl.gen(
            "answer_span",
            temperature=0,
            max_tokens=64,
            return_logprob=True,
            top_logprobs_num=2,
            return_text_in_logprobs=True,
        )
        answer = answer_forks[idx]["answer_span"].replace("\n", " ").strip(":")
        print(
            f"{YELLOW}Path #{idx} {pformat(text)}[{exp(logprob):.3f}] (score={delta}, answer={answer}){CLEAR}"
        )
        generated_text = str(answer_forks[idx])[len("ProgramState(") : -1]
        print(f"{BLUE}{pformat(generated_text)}{CLEAR}")

        if verbose:
            answer_tokens = [
                xt[0][2]
                for xt in answer_forks[idx].get_meta_info("answer_span")[
                    "output_top_logprobs"
                ]
            ]
            answer_x1s = [
                exp(xt[0][0])
                for xt in answer_forks[idx].get_meta_info("answer_span")[
                    "output_top_logprobs"
                ]
            ]
            answer_x2s = [
                exp(xt[1][0])
                for xt in answer_forks[idx].get_meta_info("answer_span")[
                    "output_top_logprobs"
                ]
            ]

            for token, x1, x2 in zip(tokens, x1s, x2s):
                print(f" {GREEN}{pformat(token)}{CLEAR}({x1:.3f}-{x2:.3f})", end="")
            print("\n===========")
            for token, x1, x2 in zip(answer_tokens, answer_x1s, answer_x2s):
                print(f" {GREEN}{pformat(token)}{CLEAR}({x1:.3f}-{x2:.3f})", end="")
            print()
````
**EN:** This function defines an SGLang prompt program that appends turns to the state object and reserves one or more generation slots for model output.
**CN:** 该函数定义了一个 SGLang 提示程序：它会向状态对象追加对话或指令内容，并预留一个或多个生成槽位来接收模型输出。

### Lines 108-115: Configure the default backend
````python
sgl.set_default_backend(sgl.RuntimeEndpoint("http://localhost:30000"))

state = cot_decoding.run(
    question=r"Claire makes a 3 egg omelet every morning for breakfast. How many dozens of eggs will she eat in 4 weeks?",
    get_top_k=10,
    is_chat_model=True,
    verbose=False,
)
````
**EN:** The example selects a concrete backend once so later prompt executions automatically use the same model endpoint.
**CN:** 该示例先统一选择具体后端，这样后续提示执行都会自动使用同一个模型端点。

## Key Concepts / 关键概念
- **Prompt programming / 提示程序编排**: The file builds prompts with SGLang state transitions. / 该文件使用 SGLang 状态转换来构造提示程序。

## Dependencies / 依赖关系
- **Standard library / 标准库**: math.exp, pprint.pformat
- **Project-specific / 项目相关**: sglang
