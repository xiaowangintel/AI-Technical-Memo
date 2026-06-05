# notes.md — Documentation Analysis / 文档分析

## Source / 来源
- **Path:** `/root/xw/triton/docs/meetups/05-01-2025/notes.md`
- **EN:** Meeting notes for the Triton community meetup held on 05-01-2025.
- **CN:** Triton 社区会议 05-01-2025 的会议记录。

## Content Analysis / 内容分析
### Agenda
**EN:** This section lays out the planned agenda for the meetup or event. Key listed points include What are the plans for existing block pointer programming model? (Context: Intel GPU backend relies heavily on it an will need time to fully move to tensor descriptor programming model) - Jianhui Li (Intel), Infrastructure for Triton performance tests - Sayce Falk (Google), and What talks/tutorials/open discussions would you like to see at the 2025 Triton Developers' Summit? How can we help? Adnan Aziz (Meta).
**CN:** 本节列出了会议或活动的计划议程。 列出的重点包括 What are the plans for existing block pointer programming model? (Context: Intel GPU backend relies heavily on it an will need time to fully move to tensor descriptor programming model) - Jianhui Li (Intel)、Infrastructure for Triton performance tests - Sayce Falk (Google)、What talks/tutorials/open discussions would you like to see at the 2025 Triton Developers' Summit? How can we help? Adnan Aziz (Meta)。 其中反复出现的技术关键词包括 What、programming、model、Intel、Triton、plans。

### Notes
**EN:** This section provides structure for Notes.
**CN:** 本节为 Notes 提供结构化内容。

### What are the plans for existing block pointer programming model? (Context: Intel GPU backend relies heavily on it an will need time to fully move to tensor descriptor programming model)
**EN:** This section discusses What are the plans for existing block pointer programming model? (Context: Intel GPU backend relies heavily on it an will need time to fully move to tensor descriptor programming model) in the context of the source document. Key listed points include Glad to see Triton moving toward generic tensor descriptor vs vendor-specific TMA. and Intel is still relying on older block pointer programming model. Will take some time to migrate to new tensor descriptor model. Notable prose emphasis: Speakers: Jianhui Li (Intel), Keren Zhou (George Mason Univ)
**CN:** 本节围绕 What are the plans for existing block pointer programming model? (Context: Intel GPU backend relies heavily on it an will need time to fully move to tensor descriptor programming model) 讨论该文档中的相关内容。 列出的重点包括 Glad to see Triton moving toward generic tensor descriptor vs vendor-specific TMA.、Intel is still relying on older block pointer programming model. Will take some time to migrate to new tensor descriptor model。 其中反复出现的技术关键词包括 Intel、tensor、descriptor、Speakers、Jianhui、Keren。

### Questions
**EN:** This section provides structure for Questions. Key listed points include Q> What is timeline for deprecation of block pointer?, Q> Looked at code examples. Two flavors of tensor descriptor. We'd prefer keeping one: CreateTensorDescriptorFromHost Why are there two flavors? WHy not just keep the device side one?, A> You want to know why we have one device side and one host side., Q> Ok to have tensor descriptors in global memory. We want tensor descriptors to reside on the device., A> We have descriptor API on device because when you update the descriptor from the kernel and not from the device., and Q> Performance. Would like to limit choices to programmer. Don't need to enable other programming models. Makes it easier to support triton on other platforms..
**CN:** 本节为 Questions 提供结构化内容。 列出的重点包括 Q> What is timeline for deprecation of block pointer?、Q> Looked at code examples. Two flavors of tensor descriptor. We'd prefer keeping one: CreateTensorDescriptorFromHost Why are there two flavors? WHy not just keep the device side one?、A> You want to know why we have one device side and one host side.、Q> Ok to have tensor descriptors in global memory. We want tensor descriptors to reside on the device.、A> We have descriptor API on device because when you update the descriptor from the kernel and not from the device.、Q> Performance. Would like to limit choices to programmer. Don't need to enable other programming models. Makes it easier to support triton on other platforms.。 其中反复出现的技术关键词包括 block、tensor、descriptor、pointers、pointer、device。

### Infrastructure for Triton performance tests
**EN:** This section discusses Infrastructure for Triton performance tests in the context of the source document. Key listed points include Q> Any near term plans for setting up public benchmarks for Nvidia's newest hardware? Maybe through PyTorch or TorchBench., A> Cicie Wang (Meta): Meta discussed with Nvidia about running TritonBench on B200. Nvidia suggested working with OpenAI (OpenAI has hardware). We now have hardware. Jason from Nvidia working on setting up CI. First steps: get TritonBench running on this hardware., Q> Need devops/infra side to setup devrunners (complexity/security of setting up these machines is high). Possible to use existing GB200 triton runner in triton CI., Q> You want to run torchbench? Is this on the triton main project?, A> Possibly using the facebookexperimental/triton repo. Maybe a second repo. Maybe the PyTorch repo?, and A> Also looking at the AMD MI300x and AMD MI350x.. Notable prose emphasis: Speaker: Sayce Falk (Google), Cicie Wang (Meta), Jason Knight (Nvidia), Keren Zhou (George Mason University), Areg Melik-Adamyan (Intel)
**CN:** 本节围绕 Infrastructure for Triton performance tests 讨论该文档中的相关内容。 列出的重点包括 Q> Any near term plans for setting up public benchmarks for Nvidia's newest hardware? Maybe through PyTorch or TorchBench.、A> Cicie Wang (Meta): Meta discussed with Nvidia about running TritonBench on B200. Nvidia suggested working with OpenAI (OpenAI has hardware). We now have hardware. Jason from Nvidia working on setting up CI. First steps: get TritonBench running on this hardware.、Q> Need devops/infra side to setup devrunners (complexity/security of setting up these machines is high). Possible to use existing GB200 triton runner in triton CI.、Q> You want to run torchbench? Is this on the triton main project?、A> Possibly using the facebookexperimental/triton repo. Maybe a second repo. Maybe the PyTorch repo?、A> Also looking at the AMD MI300x and AMD MI350x.。 其中反复出现的技术关键词包括 Intel、Meta、Maybe、triton、Nvidia、PyTorch。

### What talks/tutorials/open discussions would you like to see at the 2025 Triton Developers' Summit? How can we help?
**EN:** This section discusses What talks/tutorials/open discussions would you like to see at the 2025 Triton Developers' Summit? How can we help? in the context of the source document. Key listed points include Phil, Elena Mithra & Adnan Aziz pulled together last year's Triton Developers' Summit., Mlir tutorials, keynotes, closed-end backends, OSS projects, Intel triton efforts., Heterogeneous hardware., Over 500 people attended!, Microsoft running it in 2025., and Ideas. Notable prose emphasis: Speaker: Adnan Aziz (Meta)
**CN:** 本节围绕 What talks/tutorials/open discussions would you like to see at the 2025 Triton Developers' Summit? How can we help? 讨论该文档中的相关内容。 列出的重点包括 Phil, Elena Mithra & Adnan Aziz pulled together last year's Triton Developers' Summit.、Mlir tutorials, keynotes, closed-end backends, OSS projects, Intel triton efforts.、Heterogeneous hardware.、Over 500 people attended!、Microsoft running it in 2025.、Ideas。 其中反复出现的技术关键词包括 triton、Microsoft、users、Adnan、Aziz、give。

### Minutes
**EN:** This section records outcomes, decisions, and follow-up points from the discussion. Notable prose emphasis: Recording link [here](https://youtu.be/W16BrXc5BYE)
**CN:** 本节记录讨论产生的结论、决策和后续事项。 其中反复出现的技术关键词包括 Recording、here、https、youtu.be、W16BrXc5BYE。

## Key Concepts / 关键概念
- **EN:** Agenda  **CN:** 议程
- **EN:** Notes  **CN:** Notes
- **EN:** What are the plans for existing block pointer programming model? (Context: Intel GPU backend relies heavily on it an will need time to fully move to tensor descriptor programming model)  **CN:** What are the plans for existing block pointer programming model? (Context: Intel GPU backend relies heavily on it an will need time to fully move to tensor descriptor programming model)
- **EN:** Questions  **CN:** Questions
- **EN:** Infrastructure for Triton performance tests  **CN:** Infrastructure for Triton performance tests
- **EN:** What talks/tutorials/open discussions would you like to see at the 2025 Triton Developers' Summit? How can we help?  **CN:** What talks/tutorials/open discussions would you like to see at the 2025 Triton Developers' Summit? How can we help?

## Related Files / 相关文件
- `/root/xw/triton/docs/meetups/04-02-2024/notes.md`
- `/root/xw/triton/docs/meetups/05-05-2026/notes.md`
- `/root/xw/triton/docs/meetups/for_moderators/README.md`
