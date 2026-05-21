---
name: acquired-orchestrator
description: |
  Acquired 播客脚本生成 Orchestrator — 协调 Ben Gilbert + David Rosenthal 两位 host persona，
  对任意公司/组织生成 Acquired 风格的 podcast 脚本。
  支持两个模式：(1) 已被 Acquired cover 过的公司 → 复现验证 (2) 未 cover 过的公司 → 原创生成。
  触发词：「acquired episode」「生成节目」「acquired script」「帮我做一个 acquired」「acquired style」。
version: 1.0.0
author: PER-120
license: MIT
metadata:
  hermes:
    tags: [podcast, writing, analysis, research, storytelling]
    related_skills: [ben-gilbert-acquired, david-rosenthal-acquired]
platforms: [macos, linux]
requires_toolsets: [web, terminal, file, browser]
---

# Acquired Orchestrator · 脚本生成引擎

> "We make end-of-one products. Only we can do. We'll only be great." — Acquired Philosophy

## 概述

本 Orchestrator 协调两个 host persona skill：
- **ben-gilbert-acquired** — 分析担当，产品/运营/财务深挖
- **david-rosenthal-acquired** — 叙事担当，故事架构/战略框架/情感节拍

**输入**：公司/组织名称（或 Acquired episode 编号，用于复现模式）
**输出**：完整的 Acquired 风格 podcast 脚本（3-4 小时播客级别，约 25,000-40,000 字）

## 依赖

- `ben-gilbert-acquired` skill — Ben 的人格和调研方法论
- `david-rosenthal-acquired` skill — David 的人格和叙事方法论
- 方法论框架：`references/methodology-framework.md`

## 执行流程

### Phase 1: 选题评估（Topic Selection）

**输入**：公司/组织名称

**评估标准（三要素模型）**：

1. **Hero Protagonist（英雄主角）** — 是否有从默默无闻到 ubiquity 的故事？
   - 创始人/关键人物的英雄之旅是否足够 compelling？
   - 是否有关键转折点（near-death experience → triumph）？

2. **Secret in Plain Sight（显而不见的秘密）** — 是否存在人人都知道但没人真正理解的东西？
   - Costco 的运营芭蕾、Hermès 的等候名单策略、Google 的激励结构
   - 如果找不到，能否从研究中发现？

3. **Importance（重要性）** — 这个公司是否值得 Acquired 的舞台？
   - 对行业/社会/文化的影响是否足够深远？
   - 是否足够 durable（5-10 年后仍然重要）？

**决策**：
- 三要素全满足 → Green light，进入 Phase 2
- 满足 2/3 → Yellow light，需要额外论证
- 满足 1/3 → Red light，考虑 kill 或换角度

**输出**：选题评估文档（1-2 页），包含三要素分析和选题决策

### Phase 2: 并行调研（Parallel Research）

**模拟 Acquired 的独立调研模式**：两个研究 track 完全并行、不共享中间结果。

#### 获取参考 Transcript（如需对比分析）

当需要获取真实 Acquired episode transcript 作为对比参考时，使用 `references/transcript-extraction-methodology.md` 中记录的浏览器提取方法。**podscripts.co 不可靠**（部分 episode 页面重定向到通用页面无实际内容），应使用 acquired.fm 官方网站。

关键步骤：`browser_navigate` → 点击 TRANSCRIPT 按钮 → 验证 `#transcript` 不再 `display-none` → 通过 `browser_console` 从 `.episode-rich-text` 分段提取 `<p>` 元素（每批 200-300 段）。

#### 研究充足性验证门（Research Sufficiency Gate）

Phase 2 的研究必须通过机械化验证才能进入 Phase 3。这不是建议，是硬性门控。

**验证清单 — 每个维度必须通过：**

1. **来源多样性检查（Source Diversity）**
   - 每个研究维度至少引用 3 个独立来源
   - 不能仅依赖单一来源（如只看 Wikipedia 或只看一篇长文）
   - 至少包含 1 个一手来源（原始访谈、SEC filing、创始人演讲/信件、学术论文）
   - 检查方法：研究报告中列出 [Source: ...] 标注，orchestrator 清点

2. **关键事实交叉验证（Cross-Validation）**
   - 核心数字（营收、估值、市场份额、用户数）必须至少 2 个来源确认
   - 矛盾信息必须标注并选择更可信来源
   - 无法验证的数字标注 [UNVERIFIED]

3. **叙事覆盖度检查（Narrative Coverage）**
   - Origin story: 创始人背景 + 创立契机 + 早期挣扎 — 三个要素齐全
   - Key decisions: 至少识别 3 个关键战略决策及其决策背景
   - Bet-the-company moment: 至少 1 个（如果没有，标注 [NO BET-THE-COMPANY] 需额外研究）
   - "Who did it first" story: 至少 1 个创新来源追溯（GoTo→AdWords 式）
   - 检查方法：对照以下清单逐项确认

4. **分析深度检查（Analysis Depth）**
   - 产品机制：核心产品/服务如何运作，有具体的运营数据支撑
   - 财务数据：至少覆盖 revenue、margin、growth rate 三个维度
   - 竞争格局：至少识别 2-3 个主要竞争对手和差异化因素
   - 检查方法：研究报告中的数据密度 — 如果某段超过 500 词没有具体数字/人名/日期，标记为 [THIN]

5. **盲区标记（Blind Spot Flagging）**
   - 研究过程中发现但未能充分调查的线索标记为 [BLINDSPOT: topic]
   - Phase 3 Production Meeting 中必须讨论所有 BLINDSPOT
   - 如果关键维度有 > 2 个 BLINDSPOT，补充研究轮次后再进入 Phase 3

**执行方式：**
- 研究完成后，orchestrator 执行验证清单（不是 subagent 自己声称完成）
- 每个维度产出 `pass` / `fail (reason)` / `partial (what's missing)` 
- 任何维度 `fail` → 补充研究，不能直接进入 Phase 3
- 验证结果记录到 `references/sources/00-research-gate-check.md`

#### 机械化验证执行流程

**这不是可选的优化，是硬性门控。** 按以下步骤严格执行：

1. **收集研究报告** — 所有 subagent 返回后，将每个维度的研究保存到 `references/sources/<NN>-<topic>.md`
2. **运行 Gate Check 脚本** — 用 `execute_code` 执行以下验证：

```python
# Gate Check Script Pattern — paste into execute_code
import re

# Read all research files
research_files = [...]  # glob references/sources/*.md
all_text = "\n".join([open(f).read() for f in research_files])

checks = {
    "source_diversity": {
        "test": lambda: len(re.findall(r'\[Source:', all_text)) >= 15,  # 3 sources × 5+ dimensions
        "fail_msg": "Need at least 15 [Source: ...] citations across all research. Each dimension needs 3+."
    },
    "cross_validation": {
        "test": lambda: len(re.findall(r'\[UNVERIFIED\]', all_text)) <= 3,
        "fail_msg": "Too many unverifiable claims. Core numbers need 2+ source confirmation."
    },
    "origin_story": {
        "test": lambda: all(kw in all_text.lower() for kw in ['founder', 'founded', 'early']),
        "fail_msg": "Origin story incomplete — need founder background + founding moment + early struggles."
    },
    "bet_the_company": {
        "test": lambda: '[NO BET-THE-COMPANY]' not in all_text or 'bet' in all_text.lower(),
        "fail_msg": "No bet-the-company moment identified. Need deeper research into critical junctures."
    },
    "who_did_it_first": {
        "test": lambda: any(kw in all_text.lower() for kw in ['borrowed', 'inspired by', 'copied', 'prior art', 'origin of the idea']),
        "fail_msg": "No 'who did it first' story found. Every major innovation has a precursor — find it."
    },
    "analysis_depth": {
        "test": lambda: len(re.findall(r'\[\$[\d,]+[BMK]?\]', all_text)) >= 5,
        "fail_msg": "Insufficient financial data density. Need specific dollar figures."
    },
    "thin_sections": {
        "test": lambda: len(re.findall(r'\[THIN\]', all_text)) <= 2,
        "fail_msg": "Too many thin sections — paragraphs without specific numbers/names/dates."
    },
    "blindspots": {
        "test": lambda: len(re.findall(r'\[BLINDSPOT:', all_text)) <= 3,
        "fail_msg": "Too many blind spots — need supplementary research round before Phase 3."
    }
}

results = {}
for name, check in checks.items():
    passed = check["test"]()
    results[name] = "PASS" if passed else f"FAIL: {check['fail_msg']}"
    
    all_pass = all(v == "PASS" for v in results.values())
print(f"Gate Check Result: {'ALL PASS ✅' if all_pass else 'GATE BLOCKED ❌'}")
for k, v in results.items():
    print(f"  {k}: {v}")
```

3. **Gate blocked → 补充研究** — 针对 fail 的维度，启动针对性补充 agent（1-2 个，focused on the gap）
4. **Gate passed → 记录通过** — 将完整的 gate check 结果写入 `00-research-gate-check.md`，包含通过时间和每个维度的状态
5. **进入 Phase 3** — 只有在 gate check ALL PASS 后才进入 Production Meeting

#### ⚠️ Operational Constraints

1. **Max 3 concurrent subagents** — `delegate_task` caps at 3 children. Phase 2 has 2 tracks × multiple dimensions each. **Batch into rounds of 3 max.** Recommended pattern: launch 3 research agents first, then 3 more when the first batch completes.
2. **Subagents return summaries, not files** — `delegate_task` children cannot reliably write to project files. The orchestrator must collect each agent's summary and persist it to `references/sources/<NN>-<topic>.md` before proceeding to Phase 3.
3. **Research output is large (5,000-8,000 words per dimension)** — Don't hold all research in context. Save each to disk as it returns, then load selectively for Phase 3/4.
4. **6-dimension research pattern works well** — In practice, launching 6 specialized research agents (canonical works, financials, primary sources, competition, culture, "the secret") produces richer, more structured output than 2 broad-track agents. Use `delegate_task` with specific dimension goals and `toolsets: ["web"]`.
5. **Research agents may hit rate limits** — Web search/scraping subagents can get throttled, especially for financial data and canonical works. If a batch returns thin results, retry that specific dimension in the next batch or fall back to a single focused agent. Origin stories and culture dimensions tend to be more resilient.

#### Track A: Ben's Research（分析线）
使用 `ben-gilbert-acquired` persona：

1. **Canonical work 扫描** — 找到最权威的 3-5 个现有分析/书籍/深度报道
2. **Financial deep-dive** — 收集关键财务数据（revenue growth, margins, unit economics, competitive dynamics）
3. **Product/mechanics research** — 理解核心产品/服务是如何运作的（"how does this work?"）
4. **7 Powers analysis** — 用 Helmer 框架分析 competitive moat
5. **Pattern matching** — 与其他伟大公司做对比，找出共性模式

**输出**：Ben's Insight Document（~4,000 字）
- 关键财务洞察
- 产品机制分析
- 7 Powers 评估
- 与其他公司的模式对比
- 给 David 的"惊喜发现"列表

#### Track B: David's Research（叙事线）
使用 `david-rosenthal-acquired` persona：

1. **Origin story** — 创始人背景、创立契机、早期挣扎
2. **Key characters** — 所有重要人物及其动机/激励
3. **Timeline construction** — 构建完整的 chronological timeline
4. **Narrative arc identification** — 找到英雄之旅的弧线
5. **Cultural/historical context** — 公司所处时代和文化背景

**输出**：David's Narrative Script（~39 页 / ~10,000 字叙事骨架）
- 完整时间线
- 关键场景和对话
- 转折点和情感节拍
- 冷开场设计
- 主线主题

### Phase 3: 资料整合（Production Meeting）

**模拟 Acquired 的 production meeting**：合并两个 track 的发现。

0. **研究验证门复查** — 确认 Phase 2 的验证清单全部 pass。如有 fail 或 BLINDSPOT，先补研究。

1. **Topic alignment** — 确认 episode 的核心主题（"这是关于什么的？"）
2. **Structure agreement** — 同意叙事结构（几个 section，每个 section 的重点）
3. **Surprise planning** — 规划 Ben 和 David 各自会"惊讶"对方的点
4. **Analysis placement** — 决定分析段落（7 Powers、bull/bear）穿插在哪些位置
5. **Kill decisions** — 砍掉不够好的素材（"the too hard pile"）
6. **七力辩论预规划** — 为 Phase 4 的七力辩论确定：
   - 哪几个 Power 最可能有分歧？（至少选 3 个）
   - Ben 的怀疑论点和 David 的支持论点各自的基础
   - 每个 Power 的辩论焦点问题（如 "Counter-Positioning: 新行业真的有 CP 吗？"）

**输出**：Production Meeting Notes
- 核心主题（一句话）
- Section 列表（含每个 section 的 focus）
- Ben 的 surprise points
- David 的 surprise points
- 被砍掉的素材和原因
- 七力辩论焦点（至少 3 个 Power 的辩论话题）

### Phase 4: 脚本生成（Script Generation）

**这是核心创作阶段**。两个 persona 交替写作，保持即兴感和真实感。

#### 格式规范

```
## [Section 标题]

### Cold Open（如适用）
[戏剧性开场画面/引语]

---

**David**: [叙事内容...]

**Ben**: [分析/追问/惊讶...]

**David**: [继续叙事/回应...]

[循环交替]

---

### Analysis Break
**Ben**: [7 Powers 分析 / 财务深挖]
**David**: [战略框架 / 叙事回扣]

---

### Carve-outs
**Ben**: [推荐]
**David**: [推荐]
```

#### 写作原则
1. **Improv energy** — 保持「第一次听到」的新鲜感，不要写成报告
2. **Emotional truth** — 用「我刚才搞明白了 X 是怎么运作的，这太酷了」的语气
3. **Specific > general** — "1997 年 8 月，Steve Jobs 回到 Apple 时..." 比 "Jobs 回归后..." 好 100 倍
4. **Show the work** — 不只是给结论，展示推理过程
5. **Earn the insight** — 先用叙事建立理解基础，再给分析
6. **Host chemistry** — Ben 和 David 要有真实的互动：惊讶、追问、补充、偶尔的分歧
7. **Banter density targets (quantified)** — Reaction words must hit these per-1000-word floors:
   - "Oh" / "Whoa" / "Wait": ≥ 1.5/k (original hits 2.0/k)
   - "Right" / "Yeah" / "Yep": ≥ 3.0/k (original hits 4.2/k)
   - "I didn't know that" / "stumping me": ≥ 0.3/k
   - If the script reads like two AI voices taking turns, the density is too low
8. **"Not the hero" stories** — For every major innovation by the subject company, research who they *borrowed from*. GoTo/Overture→AdWords, Xerox→Macintosh, etc. These "origin of the idea" stories are Acquired's signature depth marker.
9. **Bet-the-company moments** — Identify at least one moment where the founders risked everything. These create the narrative tension that separates Acquired from corporate Wikipedia pages.

#### 多段生成策略（Multi-Part Generation）

LLM 输出长度有限，无法一次生成 25,000+ 字。必须分段生成后合并：

1. **按 section 拆分** — 将 Phase 3 确定的 section 列表分为 3-4 组（每组 3-4 个 section）
2. **逐段生成** — 每段作为独立的 `write_file` 调用，存为 `output/<topic>-episode-part1.md`, `part2.md`, ...
3. **合并** — 用 `execute_code` 读取所有部分，去除重复的前置元数据，合并为 `output/<topic>-acquired-episode.md`
4. **连续性检查** — 合并后检查段间过渡是否自然，section 间有无断裂

每段约 5,000-8,000 词（3-4 个 section），最终合并产出 15,000-30,000 词。

#### 目标篇幅
- 总字数：15,000-30,000 词（多段合并后）
- 单段字数：5,000-8,000 词
- Section 数量：8-15 个主要 section
- Analysis breaks：3-5 个
- 对应播客时长：3-4 小时

### Phase 5: 审校输出（Quality Check）

#### 时间性测试（Timelessness Test）
- [ ] 5 年后听这个 episode，还有 80% 的价值吗？
- [ ] 去掉所有时效性信息后，核心叙事仍然成立吗？

#### 秘密测试（Secret Test）
- [ ] 有一个「显而不见的秘密」被揭示了吗？
- [ ] 听众听完会说「我以前不知道这个」吗？

#### 英雄之旅测试（Hero's Journey Test）
- [ ] 有清晰的起点（obscurity）吗？
- [ ] 有转折点/危机吗？
- [ ] 有高潮（dominance/success）吗？
- [ ] 主角足够 compelling 吗？

#### Acquired 标准测试
- [ ] 两 host 有真实的互动感吗（不是念稿）？
- [ ] 分析深度够吗（7 Powers、financial、strategic）？
- [ ] 叙事足够引人入胜吗（不是 Wikipedia 摘要）？
- [ ] 有足够的细节和具体场景吗？
- [ ] Ben 和 David 的分工清晰吗（分析 vs 叙事）？

**如果任何一项测试不通过，回到 Phase 4 修改。**

## 脚本模板

```markdown
# Acquired: [公司名]

## Cold Open
[60-90 秒的戏剧性开场。一个具体场景、一个惊人数字、或一个反直觉的陈述。]

---

**David**: Welcome to season [X], episode [Y] of Acquired, the podcast about great companies. I'm David Rosenthal.

**Ben**: And I'm Ben Gilbert. Today we are telling the story of [Company].

**David**: [Company] is one of the most [adjective] companies in the world. [Hook — why this is worth 4 hours of your time.]

**Ben**: [Additional hook from analytical angle.]

**David**: As always, this is not investment advice. We are not financial advisors.

---

## I. Origins

**David**: [创始人背景，时代背景，创立契机...]

**Ben**: [对早期策略的分析/惊讶...]

## II. The Early Struggles

**David**: [near-death experience，关键危机...]

**Ben**: [财务/运营层面的分析...]

## III. The Pivot / The Insight

**David**: [那个改变一切的洞察或决策...]

**Ben**: [why this worked — 产品/运营/激励分析...]

## IV. The Secret Revealed

**David**: [揭示 "secret in plain sight"...]

**Ben**: [用数据/框架验证...]

## V. The Climb to Dominance

**David**: [从转折点到行业主导的过程...]

**Ben**: [关键决策分析，7 Powers 开始显现...]

## VI. The Modern Era

**David**: [当代的战略和竞争格局...]

**Ben**: [详细财务/竞争分析...]

## VII. 7 Powers Analysis — Debate Format

**核心原则：七力分析不是记分卡，是辩论。** Acquired 原版中，Ben 和 David 对每个 Power 是否成立有真正的分歧、质疑、和最终（有时不）达成共识。

### 辩论角色

- **Ben = Skeptical Analyst（怀疑论者）** — 默认"prove it"。质疑定义是否成立，数据是否充分，是否有更简单的替代解释。倾向低估。
- **David = Strategic Optimist（战略乐观派）** — 寻找 Power 成立的证据。构建论证链，提出类比和框架。倾向高估。

### 辩论规则

1. **至少 3 个 Power 必须有实质性分歧** — 不能 7 个都顺利通过
2. **至少 1 个 Power 必须被否决** — "Doesn't Apply" 或 "Weak at best"
3. **怀疑论者必须提出具体反驳** — 不是 "I'm not sure"，是 "here's why this doesn't fit the definition"
4. **评级必须比直觉更低** — 觉得 "strong"？先论证为什么不是 moderate
5. **Counter-Positioning 对新行业几乎不存在** — "better product" ≠ CP
6. **"Brand awareness" ≠ Branding Power** — 需要 demonstrable willingness to pay premium
7. **最终评级可以分歧** — Ben 标 Moderate、David 标 Strong 是正常的，记录分歧

### 脚本中的辩论模板

```
**Ben**: Alright, 7 Powers time. I've got my scorecard ready — and I suspect we're going to disagree on a few of these.

**David**: Good. Let's go power by power. I'll start with the argument FOR, you argue against, and we'll see where we land.

---

**Ben**: [Power 1 — states skeptical rating with evidence, challenges the definition]
So let's talk about [Power]. The question is: does [Company] actually have [Power]? My initial take is [skeptical position]. Here's why: [specific concern about definition/threshold].

**David**: I actually think [opposing view], and here's the case: [supporting argument with specific evidence].
Think about [analogy to another company]...

**Ben**: But that's different from [Power] because [nuance/definition concern]. [Reference to specific data point that complicates the case]

**David**: Fair, but what about [counter-evidence]? The way I'd frame it is [reframing].

**Ben**: [Concedes: "OK, I'll give you that one" / "I'll give you a moderate"]
OR [Holds ground: "I'm still not convinced. I'd put this at weak/nonexistent."]

**David**: [Final position — agree to disagree or reach consensus]

[Repeat for all 7 Powers, with genuine back-and-forth on at least 3-4 powers]

---

**Ben**: Alright, final scorecard.

| Power | Ben | David | Consensus |
|-------|-----|-------|-----------|
| Scale Economies | [rating] | [rating] | [agree/split] |
| Network Effects | [rating] | [rating] | [agree/split] |
| Counter-Positioning | [rating] | [rating] | [agree/split] |
| Switching Costs | [rating] | [rating] | [agree/split] |
| Branding | [rating] | [rating] | [agree/split] |
| Cornered Resource | [rating] | [rating] | [agree/split] |
| Process Power | [rating] | [rating] | [agree/split] |

**Biggest disagreement**: [which power and why]

**David**: The thing that strikes me is [big-picture observation about the powers as a system — not just a list, but how they interact]
```

## VIII. Lessons & Playbook

**David**: [提炼出的战略教训...]
**Ben**: [可操作的商业洞察...]

## Carve-outs

**Ben**: [个人推荐 — 书/产品/体验]
**David**: [个人推荐 — 书/产品/体验]

---

**David**: If you want more Acquired, [closing CTA...]
**Ben**: [Final thought / callback to cold open]
```

## 输出格式

最终脚本以 Markdown 格式输出，保存到项目目录。

文件头部包含元数据：
```yaml
---
company: [公司名]
generated: [日期]
mode: [recreation | original]
target_duration: 3-4 hours
word_count: [字数]
phases_completed: [1-5]
quality_checks: [pass/fail details]
---
```

## 验收标准

### 模式一：复现模式（已被 Acquired cover 的公司）
- [ ] 核心叙事线与原 episode 一致（但不参考原 transcript）
- [ ] 关键洞察和分析角度匹配
- [ ] 两 host 的互动风格接近原 episode
- [ ] 篇幅至少达到原 episode 的 60%

### 模式二：原创模式（未被 cover 的公司）
- [ ] 调研资料集足够详实（经过用户审查）
- [ ] 满足选题评估三要素
- [ ] 通过所有 Phase 5 质量测试
- [ ] 两 host 有明确的分工和互动
- [ ] 篇幅达到 15,000+ 词

### 模式三：验证测试（Verification Test）
用于检验 skill 本身的质量。选择一个 Acquired 已 cover 的公司，**不参考原始 transcript**，走完全部 5 个 Phase，然后由用户审查输出质量。

额外要求：
- [ ] 在脚本文件头标注 `mode: recreation (verification test)`
- [ ] 产出所有中间文件（research、production meeting notes、分段脚本）
- [ ] 用户审查后，将反馈记录到 `Context.md` 作为 skill 改进输入
- [ ] 比对维度：叙事结构、分析深度、host 化学反应、细节密度

## Pitfalls

1. **不要写成报告** — Acquired 是故事，不是 Wikipedia。用叙事驱动，不是用分析驱动。
2. **不要忽略情感** — "One of your secret sauces is emotion."（Michael Lewis 评价）
3. **不要让两 host 说话风格一样** — Ben 问「how」，David 问「why」
4. **不要跳过细节** — 具体的日期、数字、人名、场景是 Acquired 的标志
5. **不要用 AI 口吻** — 避免 "It's worth noting that..."、"Furthermore..."等 AI 高频词
6. **不要在分析部分太浅** — Acquired 的分析是 300+ 小时研究的浓缩，不是表面评论
7. **不要忘记 cold open** — 每个 Acquired episode 都有一个精心设计的开场
8. **不要省略 disagreement** — 两 host 有分歧是好事情，增加真实性
9. **不要追热点** — 选择 durable 的主题，不是当下流行的
10. **不要直接参考 Acquired transcript**（复现模式）— 基于公开资料独立研究，不能抄袭
11. **不要尝试一次生成完整脚本** — 输出长度限制决定了必须分 3-4 段生成再合并，每段聚焦 3-4 个 section
12. **不要在段间丢失连续性** — 合并后检查 section 间过渡、host 对话的上下文引用是否完整
13. **不要在七力分析上过于慷慨** — 这是最大的问题。辩论格式是硬性要求：(a) 至少 3 个 Power 有分歧；(b) 至少 1 个被否决；(c) Counter-Positioning 对新行业几乎不存在；(d) "better product" ≠ CP；(e) 最终评级可以分歧。宁可低估也不要高估。
14. **不要遗漏"谁先做的"故事线** — Acquired 的标志性深度在于追溯创意来源（GoTo→AdWords, Xerox→Mac）。如果只讲主角发明了什么而忽略了前人，叙事就少了 Acquired 最有价值的层次。
15. **不要跳过 bet-the-company 时刻** — 每个伟大公司都有至少一次"押上全部"的决策（Google+ AOL $100M guarantee）。这些时刻是叙事张力的核心，不能只列在 timeline 里一笔带过。
16. **不要写成干净的双人对话** — 真实播客有重叠、打断、跑题、闲聊。每 2-3 个 section 应有一次"不对等等"或"天哪"级别的即时反应。Pre-show banter (~200词) 和至少 2 次赞助商朗读是必选项，不是可选项。
17. **不要遗漏多集覆盖** — 如果 Acquired 用多集 cover 一家公司（如 Google 两集），脚本必须覆盖所有集的内容。Google Ep1 是搜索+广告，Ep2 是 AI+Transformer 历史。只覆盖一部分等于只做了一半。
18. **不要缺少一手信源引用** — 原版 Acquired 大量引用独家采访："I talked to Anna Patterson" / "from Bill Gross directly" / "I was having dinner with X"。脚本中必须有类似的信源归属（即使是引用公开访谈/演讲/自传），不能只写"据说"。目标：每 5000 词至少 3-5 处具体信源引用。
19. **不要用平铺直叙的 Cold Open** — 原版有两种高杀伤力开场：(a) "框架先行"——先抛一个反直觉的论断（"Google is an AI company"），再用整集论证；(b) "数据震撼"——用 3 个巨大数字开场。纯叙事场景开场（我们的 Bechtolsheim 开场）也不错但缺少框架层。最理想的 Cold Open 同时有数据和框架。
20. **不要忽略 "increasing returns to scale" 叙事线** — Acquired 不只说"规模大=好"，而是构建逻辑链：更多用户 → 更多数据 → 更好的广告匹配 → 更高 CPM → 吸引更多广告主 → 每搜索收入反而上升（不只是成本下降）。这种"超线性回报"叙事比简单的规模经济更有说服力。

## Supporting Files

- `references/methodology-framework.md` — 从 10 周年 episode 提取的完整方法论框架（10 条核心洞察、调研方法论、episode 结构模板、host 分工）
- `references/google-verification-test.md` — 首次端到端验证测试记录（Google episode），包含 pipeline 执行细节、产出文件结构、经验教训
- `references/google-comparison-analysis.md` — 生成脚本 vs 原版 Acquired Google Ep1+Ep2 的系统性对比分析（12KB）。包含：篇幅差距、4 大结构性缺失、host 互动风格量化差距、七力分析差异、TOP 20 改进清单。**每次生成新脚本前应复习此文件。**
- Project file `~/clawd/projects/PER-120/references/research/google-transcript-deep-comparison.md` (14KB) — 逐段深度对比，含 Cold Open 分析、叙事手法拆解、具体遗漏故事线（GoTo/Overture/AOL deal）、原版独有叙事技巧清单。比概览报告更细致。
- `references/transcript-extraction-methodology.md` — 从 acquired.fm 提取真实 transcript 的技术方法（浏览器→TRANSCRIPT按钮→分段提取）。含 Google 三集系列的结构发现、host 角色模式量化数据、幽默/化学模式分析。**需要获取参考 transcript 时使用此方法。**
