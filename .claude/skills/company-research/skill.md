# Company Research — Deep Dive Analysis

Deep research any public or private company. Produces institutional-quality research reports with multi-source data, structured analysis, and source links.

Trigger: User asks to research a company, stock ticker, or startup (e.g., "帮我研究NVDA", "research this company", "这家公司怎么样")

## Research Process

### Phase 0: Determine Company Type (PUBLIC vs. PRIVATE)

If the company is **publicly listed**, proceed to Phase 1A (Primary Sources First).
If the company is **private/startup**, skip to Phase 1B (Multi-Source Search).

### Phase 1A: Primary Sources First (PUBLIC COMPANIES ONLY — MANDATORY)

**This phase MUST be completed before any media search.** The goal is to extract facts directly from management's own words.

**Step 1: Retrieve the latest 2-3 quarterly earnings call transcripts**
- Search: `"[Company] [Ticker] Q[N] FY[YYYY] earnings call transcript"`
- Preferred sources: company IR page PDF, The Motley Fool, Seeking Alpha, FactSet
- **Read the FULL transcript**, not summaries. Pay special attention to:
  - CEO's prepared remarks: what topics does management lead with? What language changed vs. prior quarter?
  - CFO's prepared remarks: guidance numbers, margin bridge, cash flow commentary
  - **Analyst Q&A section**: every question and every answer, verbatim

**Step 2: Retrieve the latest earnings press release and slide deck**
- Search: `"[Company] investor relations quarterly earnings"` or go directly to the company's IR page
- Extract: all financial tables, segment breakdowns, guidance ranges, balance sheet

**Step 3: Retrieve Investor Day / Capital Markets Day materials (if any)**
- Search: `"[Company] [Ticker] investor day analyst day capital markets day"`
- Extract: long-term revenue/margin targets, TAM/SAM, capacity plans, strategic framework

**Step 4: Cross-Quarter Analyst Tracking (CRITICAL)**
Read across 2-3 transcripts and build:
- **Analyst Concern Tracker**: which specific questions were asked repeatedly across quarters? By whom? How did management's answer evolve?
- **Promise vs. Delivery Tracker**: what did management commit to in prior quarters, and what actually happened?

### Phase 1B: Multi-Source Parallel Search

Run these searches in parallel:

1. **English financial/analytical search**:
   - `"[Company] [Ticker] [current year] [next year] earnings revenue growth"`
   - `"[Company] competitive moat market share [industry keywords]"`

2. **Chinese coverage**:
   - `"[Chinese name] [Ticker] [industry Chinese keywords]"`

3. **Xiaohongshu community** (mcporter, if available):
   - `'xiaohongshu.search_feeds(keyword: "[Ticker] [Chinese name] [industry]")'`

4. **Twitter/X** (xreach, if authenticated):
   - `xreach search "[Company] [Ticker]" -n 10 --json`

5. **Investor Day / Analyst Day materials**:
   - `"[Company] [Ticker] investor day analyst day capital markets day long-term financial model target"`

### Phase 2: Deep Dive (fetch details from top results)

- For public companies: cross-reference media reports against what management actually said in transcripts. Flag any discrepancies.
- Use `curl -s "https://r.jina.ai/[URL]"` to read key analysis articles (skip paywalled sites like Seeking Alpha)
- **Investor Day deep dive**: If Investor Day / Analyst Day / Capital Markets Day materials are found, read the full presentation or summary. Extract:
  - Management's long-term revenue target and timeline
  - Gross margin and operating margin targets
  - TAM/SAM estimates with breakdown
  - Revenue milestone framework (e.g., "$5B run-rate in 12 months, $8B in 24 months")
  - Key assumptions behind the targets
  - Capacity expansion plans (new fabs, facilities, capex)
  - Strategic investor commitments (e.g., NVIDIA investing $2B + purchase agreements)

### Phase 3: Structured Output

Generate the report in this exact structure:

```markdown
# [Company Name]（[Ticker] / [Chinese Name]）深度研究

## 一、公司概况
- Structured one-line positioning (e.g., "全球半导体后端材料第一大供应商，覆盖后端15种关键材料中的10种")
- CEO, founding year, HQ, employees
- Core business segments (table format with revenue, %, YoY growth)
- Key products/services

## 二、最新财务数据
- Full year highlights (table: revenue, EPS, margins, FCF, etc.)
- Latest quarter highlights (bullet points)
- Forward guidance (with prior guidance comparison if raised/lowered)
- Revenue breakdown (by segment, by region)

## 三、市场关注焦点与管理层兑现（PUBLIC COMPANIES — 本报告最重要的章节之一）

This section must be HIGH DENSITY and SCANNABLE. The reader wants to know three things fast:
1. What do investors care about most RIGHT NOW?
2. Does management deliver on what they promise?
3. What changed since the last earnings call?

**Format rules:**
- Group analyst concerns into 4-6 THEMES, not individual questions
- For each theme: one paragraph combining the key question, who asked it (name+firm), management's answer (verbatim quote), and whether prior commitments were met. No separate tables for each question.
- Promise tracker: one compact table, not verbose descriptions
- Post-earnings changes: bullet list, max 5-6 items

### 3.1 市场核心关注点（按重要性排序）

For each theme, write ONE dense paragraph that includes:
- The core concern (what the market is worried about / watching)
- Who is asking (analyst name + firm, from transcript)
- Management's most recent response (direct quote, attributed)
- Prior quarter context if the answer evolved (e.g., "CFO在Q4表示X，到Q2更新为Y")
- Current status: resolved / improving / unresolved

Aim for 4-6 themes. Each theme = 1 paragraph, ~100-150 words. No fluff.

### 3.2 承诺兑现记录

Compact table only:

| 承诺 | 时间 | 状态 | 备注 |
|------|------|------|------|

Status: ✅ 已兑现 / ⏳ 进行中 / ❌ 未兑现 / 🔄 调整
Keep the "备注" column to ONE sentence max.

### 3.3 电话会后的关键变化

Bullet list of 4-6 items max. Each item: what happened + potential impact on next quarter. No elaboration.

## 四、核心竞争优势（护城河）
For each moat, provide:
- A clear mechanism explanation (HOW does this moat work?)
- Specific data points and quantified evidence
- A flywheel or feedback loop diagram if applicable (text-based)
- Comparison to competitors — why can't they replicate this?
- Quote management or analyst statements where impactful
- Typical moat categories: infrastructure lock-in, data network effects, developer ecosystem, customer quality, strategic positioning
- DO NOT just list bullet points — write 1-2 paragraphs per moat with layered argumentation

## 五、增长驱动力
- Table format linking each driver to company-specific benefit
- Include TAM/SAM data if available

## 六、竞争格局（if relevant）
- Comparison table vs key competitors
- Market share data

## 七、管理层长期目标（Investor Day / Analyst Day）
This section is MANDATORY for public companies. If no Investor Day materials exist, note this explicitly.
- **Management's long-term financial targets** in table format:

| 指标 | 当前实际 | 管理层目标 | 目标时间线 | 可信度评估 |
|------|---------|-----------|-----------|-----------|

- Revenue target and milestone framework
- Gross margin and operating margin targets
- TAM/SAM estimates from management (with source)
- Capacity expansion plans (new facilities, capex commitments)
- Strategic partnerships and investment commitments (e.g., key customer pre-payments, equity investments)
- **Credibility assessment**: For each target, evaluate:
  - Is the current trajectory on track? (extrapolate from recent quarters)
  - What assumptions must hold true? (demand growth, pricing, market share, macro)
  - What could go wrong? (competition, cycle turns, execution risk)
  - How does management's target compare to Wall Street consensus? (conservative vs aggressive)
- **Consensus vs. management comparison table**:

| 指标 | 管理层目标 | 华尔街共识 | 差距分析 |
|------|-----------|-----------|---------|

## 八、风险与挑战
- Table format: Risk | Severity | Detail

## 九、估值与市场观点
- Current valuation metrics (P/E, EV/EBITDA, P/S, etc.)
- Analyst ratings and price targets
- Notable bull/bear cases from research
- **Valuation based on management targets**: Using Investor Day targets, calculate implied valuation at different scenarios:

| 情景 | 营收 | 利润率 | 利润 | 合理倍数 | 隐含市值 | 隐含股价 | vs 当前 |
|------|------|--------|------|---------|---------|---------|--------|

- Include at least 3 scenarios: management target achieved, consensus, and bear case
- Calculate PEG ratio where applicable
- Flag if current price already discounts management targets (i.e., upside is exhausted)

## 十、总结
> Structured 2-3 sentence summary covering: current state, key inflection point, and primary risk.

---

### 信息来源
- Every claim must link to its source URL
- Source priority: **Company Investor Day / Analyst Day presentations** > Professional financial media (FT, Bloomberg, Reuters, WSJ, Forrester) > Industry research (Sacra, etc.) > Company official (annual reports, press releases, earnings calls) > Social media (Xiaohongshu etc., max 1-2 as supplementary "community sentiment")
- Present as table: Source | Type | URL
```

### Phase 4: For Private/Startup Companies, Add:

```markdown
## 团队评估
- Founder backgrounds (table format)
- Key hires and advisors
- Assessment of team-market fit

## 技术/产品验证
- Current stage (concept/MVP/production)
- Customer signals (LOI, pilots, revenue)
- Benchmark results

## 估值分析
Use multiple frameworks and cross-validate:
1. Comparable transactions (find similar companies' rounds)
2. VC reverse method (work back from exit scenarios)
3. Team pricing (common at pre-seed/seed)
4. Milestone-based (what has been achieved vs what hasn't)
- Present as range, not point estimate
- State key assumptions explicitly
```

## Key Rules

1. **ALWAYS include source links** — every fact must be traceable. User has explicitly requested this.
2. **Use tables aggressively** — they convey data faster than prose.
3. **Structured positioning** — start with a factual one-line positioning (e.g., "全球半导体后端材料第一大供应商，覆盖后端15种关键材料中的10种"). Avoid overly colloquial or metaphorical titles. Keep the tone professional and structured.
4. **Facts over interpretation** — especially in the Analyst Tracker and Promise Tracker sections. Present what was said (with attribution), not your opinion of what it means. The user will form their own conclusions.
5. **Be honest about risks** — don't just list positives. The user values balanced analysis.
6. **Valuation opinion** — when asked, give a clear framework-based answer, not wishy-washy hedging. State assumptions and present scenarios.
7. **Chinese output by default** — unless user writes in English, output in Chinese with English terms/names kept as-is.
8. **Parallel search is critical** — always launch multiple search calls simultaneously to minimize latency.
9. **Follow-up depth** — when user asks to drill into a specific topic, do additional targeted searches rather than just elaborating from existing knowledge.
10. **PRIMARY SOURCES FIRST for public companies** — Earnings call transcripts > Quarterly press releases > Investor Day presentations > SEC filings > Analyst reports > Media coverage. Read the actual transcript, not someone's summary of it.
11. **Cross-quarter tracking is mandatory for public companies** — Read at least 2-3 consecutive earnings call transcripts. Build the Analyst Concern Tracker (who asked what, how answers evolved) and the Promise vs. Delivery Tracker (what management committed to, what actually happened). This is where the real signal is.
12. **Source priority** — **Earnings call transcripts and company IR materials are the HIGHEST priority source.** Investor Day / Analyst Day / Capital Markets Day presentations are second. These outrank all media coverage and analyst reports.
13. **Investor Day credibility assessment is mandatory** — Don't just report management targets; critically evaluate them. Compare targets to current run-rate, Wall Street consensus, and competitive dynamics.
14. **Scenario-based valuation** — Always present at least 3 valuation scenarios (bull/base/bear). Show implied stock prices for each.
15. **Strategic investor signals matter** — If major customers or partners have made equity investments, note the amount, structure, and what it implies about demand visibility.
16. **Analyst questions must be specific** — Include the analyst's full name, firm, the specific question (paraphrased from transcript), and management's specific answer. "分析师关心中国市场" is unacceptable; "Citi的Filippo Falorni在Q2电话会问旅游零售进展，CEO回答海南1月高双位数增长但北京上海机场因零售商过渡有扰动" is the expected level of detail.

## Example Trigger Phrases
- "帮我研究一下 NVDA"
- "这家公司怎么样" + PDF/deck
- "Research TSMC for me"
- "你怎么看ASML这个公司"
- "帮我分析一下这个startup"
- "/research [company]"
