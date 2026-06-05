# PaperJury 修改对照表(中文版)

**输入(初稿):** `original_draft.pdf`(origin 初稿)
**处理:** 用 Codex 适配版 PaperJury 跑一轮 AUTO 评审
**输出:** `revised_draft.pdf`(21 页，编译 0 error，16 个 overfull hbox warnings)

账本:12 条 reviewer issue -> 5 条已应用 / 1 条 queued / 6 条 author-required。

## 表一:可修复缺陷 F1-F6

| 问题定位 | 修复结果 | 人工验证 |
|---|---|---|
| **F1** section 8 并发数:正文写 `8`、表格写 `16`，自相矛盾 | revised draft 中已统一为 `16` | 已核对，自相矛盾已解决 |
| **F2** section 7.2 clerk 合并阈值:正文写 `simThreshold=0.7`、相邻公式写 `0.8` | **未修复:被 bounded-aggressive policy 放入 queued；revised draft 仍是 `0.7`** | 待作者或 policy override 处理，正文仍和公式冲突 |
| **F3** section 5 升级陪审团:写成 `jurySize=10`，全文别处为 `12` | 已改成 `12` | 已核对，不一致已解决 |
| **F4** section 2 isolation invariant 被篡改，写成 reviewers 会拿到 cumulative ledger | **未修复:被反转的 isolation 句子仍在 revised draft 中** | 待作者或下一轮 edit 处理 |
| **F5** section 1 C5 术语写成 `registrar`，别处叫 `clerk` | **未修复:`registrar` 仍在 revised draft 中** | 待作者或下一轮 edit 处理 |
| **F6** section 4 assignment 段落带有多余外部标记 | 已移除该标记，句子保留 | 已核对，build check 通过 |

## 表二:伪造断言 A1-A3(缺少支撑的断言)

| 问题定位 | 修复结果 | 人工验证 |
|---|---|---|
| **A1** abstract 声称 "94% router agreement ... confirming ... in practice"(无实验支撑) | **未修复:被路由为 author-required；该断言仍在 revised draft 中** | 待作者处理，软化或删除 |
| **A2** section 4 声称 "in our runs ... order of magnitude fewer agents ... strictly higher precision"(无实验支撑) | **未修复:被路由为 author-required；该断言仍在 revised draft 中** | 待作者处理，软化或删除 |
| **A3** section 3 声称 "reaches this fixed point within three rounds"(无数据支撑) | **未修复:Codex ledger 漏检；该句仍在 revised draft 中** | 待作者或下一轮 review 处理 |

## 表三:诱饵 B1-B2(看似缺陷、实则可辩护，应保持不动)

| 问题定位 | 修复结果 | 人工验证 |
|---|---|---|
| **B1** section 4 "no per-section reviewer assignment and no per-section coverage quota"(看似覆盖漏洞) | 无需修复，原文保留 | 已核对，正确保持不动，无 false positive |
| **B2** section 7 gate "evaluated over the same ledger state the engine's own steps write"(看似循环论证) | 无需修复，原文保留 | 已核对，正确保持不动，无 false positive |

## 小结
- 可修复类里，**F1、F3 和 F6 已解决**；F2/F4/F5 仍待处理。
- **仍有 6 项待作者或下一轮 edit 处理:** F2、F4、F5、A1、A2、A3。
- 诱饵 B1/B2 原文保留，零 false positive。
- 输出 `revised_draft.pdf` 编译为 21 页，0 error；仍有 16 个 overfull hbox warnings。
