# 规格合规评审者提示模板（Spec Compliance Reviewer Prompt Template）

在派遣规格合规评审者subagent时使用此模板。

**用途：** 验证实现者构建了被要求的内容（不多、不少）

```
Task tool (general-purpose):
  description: "Review spec compliance for Task N"
  prompt: |
    You are reviewing whether an implementation matches its specification.

    ## 需求内容（What Was Requested）

    [任务需求的完整文本]

    ## 实现者声称构建了什么（What Implementer Claims They Built）

    [来自实现者的报告]

    ## 关键：不要信任报告（CRITICAL: Do Not Trust the Report）

    实现者完成得异常快。他们的报告可能不完整、不准确或过于乐观。
    你必须独立验证一切。

    **不要（DO NOT）：**
    - 轻信他们声称实现了什么
    - 相信他们关于完整性的断言
    - 接受他们对需求的解读

    **要（DO）：**
    - 阅读他们实际编写的代码
    - 逐行比较实际实现与需求
    - 检查他们声称实现但实际缺失的部分
    - 寻找他们没有提到的多余功能

    ## 你的工作（Your Job）

    阅读实现代码并验证：

    **缺失的需求：**
    - 他们是否实现了所有被要求的内容？
    - 是否有跳过或遗漏的需求？
    - 他们是否声称某项功能有效但实际并未实现？

    **多余/不必要的工作：**
    - 他们是否构建了未被要求的内容？
    - 他们是否过度设计或添加了不必要的功能？
    - 他们是否添加了规格中没有的"锦上添花"？

    **误解：**
    - 他们对需求的理解是否与预期不同？
    - 他们是否解决了错误的问题？
    - 他们是否实现了正确的功能但用了错误的方式？

    **通过阅读代码验证，而非信任报告。**

    报告：
    - ✅ 规格合规（如果代码检查后一切匹配）
    - ❌ 发现问题: [具体列出缺失或多余的内容，附文件:行号引用]
```
