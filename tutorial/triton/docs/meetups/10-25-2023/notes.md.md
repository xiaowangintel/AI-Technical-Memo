# notes.md — Documentation Analysis / 文档分析

## Source / 来源
- **Path:** `/root/xw/triton/docs/meetups/10-25-2023/notes.md`
- **EN:** Meeting notes for the Triton community meetup held on 10-25-2023.
- **CN:** Triton 社区会议 10-25-2023 的会议记录。

## Content Analysis / 内容分析
### Agenda
**EN:** This section lays out the planned agenda for the meetup or event.
**CN:** 本节列出了会议或活动的计划议程。

### Items
**EN:** This section lists the detailed discussion topics that drive the meeting. Key listed points include H100 updates, Triton-Shared layer updates, Intel update, and Open discussion.
**CN:** 本节列出推动会议展开的详细议题。 列出的重点包括 H100 updates、Triton-Shared layer updates、Intel update、Open discussion。 其中反复出现的技术关键词包括 updates、H100、Triton-Shared、layer、Intel、update。

### Minutes
**EN:** This section records outcomes, decisions, and follow-up points from the discussion. Key listed points include H100 updates, Enabled WGMMA by default, now any matmul can reuse it., fp8 formats enabled – 1.3 Petaflops on dense matmul on H100 (gemm performance), Enabled Flash Attention using wgmma, resulting in 450 teraflop on fwd pass and 250 on backward pass – still working on perf for flash attention, fp8 numbers with flash attention running in fp8 with matmul is tricky, because the fp8 layout is significantly different than what is returned by wgmma, still wip, and Triton-Shared layer. Notable prose emphasis: Recording link [here](https://youtu.be/KZAzpKx1ebI)
**CN:** 本节记录讨论产生的结论、决策和后续事项。 列出的重点包括 H100 updates、Enabled WGMMA by default, now any matmul can reuse it.、fp8 formats enabled – 1.3 Petaflops on dense matmul on H100 (gemm performance)、Enabled Flash Attention using wgmma, resulting in 450 teraflop on fwd pass and 250 on backward pass – still working on perf for flash attention、fp8 numbers with flash attention running in fp8 with matmul is tricky, because the fp8 layout is significantly different than what is returned by wgmma, still wip、Triton-Shared layer。 其中反复出现的技术关键词包括 fp8、matmul、H100、Enabled、can、wgmma。

## Key Concepts / 关键概念
- **EN:** Agenda  **CN:** 议程
- **EN:** Items  **CN:** 议题
- **EN:** Minutes  **CN:** 会议纪要
- **EN:** H100 updates  **CN:** H100 updates
- **EN:** Triton-Shared layer updates  **CN:** Triton-Shared layer updates
- **EN:** Intel update  **CN:** Intel update

## Related Files / 相关文件
- `/root/xw/triton/docs/meetups/09-03-2025/notes.md`
- `/root/xw/triton/docs/meetups/11-05-2025/notes.md`
- `/root/xw/triton/docs/meetups/for_moderators/README.md`
