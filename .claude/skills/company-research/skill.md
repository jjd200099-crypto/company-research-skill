# Company Research — Deep Dive Analysis

Deep research any public or private company. Produces institutional-quality research reports with multi-source data, structured analysis, and source links.

Trigger: User asks to research a company, stock ticker, or startup (e.g., "帮我研究NVDA", "research this company", "这家公司怎么样")

## Research Process

### Phase 0: Determine Company Type (PUBLIC vs. PRIVATE)

If the company is **publicly listed**, proceed to Phase 1A (Primary Sources First).
If the company is **private/startup**, skip to Phase 1B (Multi-Source Search).

### Phase 1A: Primary Sources First (PUBLIC COMPANIES ONLY — MANDATORY)

**This phase MUST be completed before any media search.** The goal is to extract facts directly from management's own words, covering the **past 2 years (8 quarters)** of earnings calls.

#### Architecture: Parallel Subagent Transcript Processing

Reading 8 full transcripts in the main context is prohibitively expensive. Instead, use the **Agent tool** to spawn parallel subagents — one per quarter. Each subagent reads the FULL transcript independently and returns a compressed, structured extraction. The main agent then synthesizes across all 8 quarters.

**Step 1: Identify all transcript URLs (main agent)**

Search for all 8 quarterly earnings call transcripts from the past 2 years:
- Search: `"[Company] [Ticker] Q[N] FY[YYYY] earnings call transcript"` for each quarter
- Preferred sources: company IR page PDF, The Motley Fool, Seeking Alpha, FactSet
- Collect all transcript URLs before spawning subagents

**Step 2: Spawn parallel subagents (one per quarter, up to 8 concurrent)**

For each transcript, launch an Agent with this prompt template:

```
You are a financial transcript analyst. Read the FULL earnings call transcript at [URL] for [Company] [Ticker] [Quarter] [Year].

Read the complete transcript using WebFetch or curl. Do NOT summarize — read every word first, then extract.

Return a JSON object with EXACTLY this structure:

{
  "quarter": "Q[N] FY[YYYY]",
  "date": "YYYY-MM-DD",
  "ceo_key_topics": ["topic1", "topic2", ...],
  "ceo_notable_quotes": ["verbatim quote 1", "verbatim quote 2"],
  "ceo_language_shifts": "What topics/framing changed vs how a CEO typically presents (note new emphases, dropped topics, tone changes)",
  "cfo_guidance": {
    "revenue": "guidance range or number",
    "eps": "guidance range or number",
    "gross_margin": "guidance range or number",
    "op_margin": "guidance range or number",
    "fcf": "guidance or commentary",
    "other": "any other quantitative guidance"
  },
  "cfo_margin_bridge": "Key drivers of margin expansion/contraction mentioned",
  "segment_performance": [
    {"segment": "name", "revenue": "$X", "yoy": "+X%", "commentary": "key points"}
  ],
  "analyst_qa": [
    {
      "analyst_name": "Full Name",
      "firm": "Firm Name",
      "question_topic": "brief topic",
      "question_detail": "specific question paraphrased",
      "management_answer": "verbatim or near-verbatim key quote",
      "follow_up": "if any"
    }
  ],
  "forward_commitments": ["specific promise or target management made for future quarters"],
  "risks_mentioned": ["risk 1", "risk 2"],
  "surprise_moments": "anything unexpected — guidance cut/raise, strategic pivot, acquisition hint, unusual analyst pushback",
  "source_url": "[URL]"
}
```

**Step 3: Retrieve the latest earnings press release and slide deck (main agent, in parallel with Step 2)**
- Search: `"[Company] investor relations quarterly earnings"` or go directly to the company's IR page
- Extract: all financial tables, segment breakdowns, guidance ranges, balance sheet

**Step 4: Retrieve Investor Day / Capital Markets Day materials (main agent, in parallel with Step 2)**
- Search: `"[Company] [Ticker] investor day analyst day capital markets day"`
- Extract: long-term revenue/margin targets, TAM/SAM, capacity plans, strategic framework

**Step 5: Cross-Quarter Synthesis (main agent, AFTER all subagents return)**

Once all 8 subagent results are collected, the main agent builds:

- **Narrative Arc**: How has management's story evolved over 2 years? What topics rose/fell in prominence? Map the CEO's key topics quarter by quarter to show strategic pivots.
- **Guidance Trajectory**: Table tracking guidance numbers across all 8 quarters — did they consistently beat-and-raise, or were there cuts?
- **Analyst Concern Tracker**: Which specific questions were asked repeatedly across quarters? By whom? How did management's answer evolve? (Cross-reference `analyst_qa` across all 8 extractions)
- **Promise vs. Delivery Tracker**: What did management commit to (`forward_commitments`) in Q-N, and what actually happened in Q-N+1/N+2? Track fulfillment rate.
- **Language Shift Analysis**: Combine all `ceo_language_shifts` and `surprise_moments` to identify inflection points in the company narrative.

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

## 九、估值分析

**Act as a professional sector analyst.** Do NOT default to P/E for every company. Choose the valuation method that best fits the industry and company stage. The reader expects you to think like a specialist, not a generalist.

### 9.1 选择适当的估值方法

First, explicitly state which valuation methods you are using and WHY they are appropriate for this company/industry. Different industries demand different primary metrics:

| 行业 | 首选方法 | 次选方法 | 避免使用 |
|------|---------|---------|---------|
| 矿业/资源 | EV/资源量, EV/产能, NAV(DCF矿山寿命), P/NAV | EV/EBITDA | P/E（利润受商品价格周期扭曲） |
| 银行/金融 | P/TBV, P/BV, ROTCE, 股息率 | P/E(周期调整) | EV/EBITDA（资本结构特殊）, P/S |
| SaaS/软件 | EV/Revenue, Rule of 40, EV/ARR | EV/EBITDA, P/FCF | P/E（早期亏损公司不适用） |
| 消费品/品牌 | EV/EBITDA, P/E(周期调整), DCF | P/S, 品牌价值评估 | 单一P/E（忽略品牌溢价结构） |
| 半导体/周期 | EV/EBITDA(mid-cycle), P/E(normalized), P/BV | 前瞻P/E | 历史P/E（周期扭曲） |
| 医药/生物 | rNPV(风险调整NPV), EV/Pipeline, P/E(盈利期) | DCF | P/S（研发期不适用） |
| 房地产 | NAV, P/NAV, 股息率, Cap Rate | P/BV | P/E（折旧扭曲） |

State your choice explicitly: "本公司属于[行业]，主要使用[方法1]和[方法2]进行估值，因为[原因]。"

### 9.2 同行业可比估值

**MANDATORY.** Every valuation must include a peer comparison table. Select 3-5 closest peers (by business model, size, geography, or growth stage).

| 公司 | [核心指标1] | [核心指标2] | [核心指标3] | 备注 |
|------|-----------|-----------|-----------|------|

Then position the target company: is it trading at a premium, discount, or inline with peers? State WHY any premium/discount exists (growth differential, quality, risk).

### 9.3 情景分析

Always present at least 3 scenarios with the chosen valuation method:

| 情景 | 关键假设 | [核心指标] | 合理倍数 | 隐含价值 | vs 当前 |
|------|---------|-----------|---------|---------|--------|

- Bull / Base / Bear with explicit assumptions for each
- For commodity companies: sensitivity to commodity price (e.g., "锂价每变动1万元/吨 → 利润变动X亿")
- Flag if current price already discounts the bull case

### 9.4 分析师共识与分歧

- Consensus rating and average target price
- Notable bull/bear cases from specific analysts (name + firm + thesis)
- Where consensus may be wrong (your assessment based on primary source analysis)

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
11. **Cross-quarter tracking is mandatory for public companies** — Read the past 2 years (8 quarters) of earnings call transcripts using parallel subagents (one Agent per quarter). Each subagent reads the FULL transcript and returns structured JSON. The main agent then synthesizes across all 8 quarters to build the Narrative Arc, Analyst Concern Tracker, Promise vs. Delivery Tracker, and Language Shift Analysis. This is where the real signal is.
12. **Source priority** — **Earnings call transcripts and company IR materials are the HIGHEST priority source.** Investor Day / Analyst Day / Capital Markets Day presentations are second. These outrank all media coverage and analyst reports.
13. **Investor Day credibility assessment is mandatory** — Don't just report management targets; critically evaluate them. Compare targets to current run-rate, Wall Street consensus, and competitive dynamics.
14. **Industry-appropriate valuation** — Do NOT default to P/E for every company. Think like a specialist sector analyst: mining uses EV/resource and NAV; banks use P/TBV and ROTCE; SaaS uses EV/ARR and Rule of 40; consumer brands use EV/EBITDA. Explicitly state which method you chose and why. Always include a peer comparison table with 3-5 comparable companies.
15. **Scenario-based valuation** — Always present at least 3 valuation scenarios (bull/base/bear) using the industry-appropriate method. Show implied values for each. Include commodity/price sensitivity where applicable.
16. **Strategic investor signals matter** — If major customers or partners have made equity investments, note the amount, structure, and what it implies about demand visibility.
16. **Analyst questions must be specific** — Include the analyst's full name, firm, the specific question (paraphrased from transcript), and management's specific answer. "分析师关心中国市场" is unacceptable; "Citi的Filippo Falorni在Q2电话会问旅游零售进展，CEO回答海南1月高双位数增长但北京上海机场因零售商过渡有扰动" is the expected level of detail.

## Example Trigger Phrases
- "帮我研究一下 NVDA"
- "这家公司怎么样" + PDF/deck
- "Research TSMC for me"
- "你怎么看ASML这个公司"
- "帮我分析一下这个startup"
- "/research [company]"
