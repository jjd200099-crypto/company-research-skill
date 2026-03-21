# Company Research — Deep Dive Analysis

Deep research any public or private company. Produces institutional-quality research reports with multi-source data, structured analysis, and source links.

Trigger: User asks to research a company, stock ticker, or startup (e.g., "帮我研究NVDA", "research this company", "这家公司怎么样")

## Research Process

### Phase 1: Multi-Source Parallel Search (launch ALL searches simultaneously)

Run these searches in parallel using the Agent Reach skill tools:

1. **English financial/analytical search** (Exa):
   - `'exa.web_search_exa(query: "[Company] [Ticker] 2025 2026 earnings revenue growth", numResults: 8)'`
   - `'exa.web_search_exa(query: "[Company] competitive moat market share [industry keywords]", numResults: 5)'`

2. **Chinese coverage** (Exa):
   - `'exa.web_search_exa(query: "[Chinese name] [Ticker] [industry Chinese keywords]", numResults: 5)'`

3. **Xiaohongshu community** (mcporter):
   - `'xiaohongshu.search_feeds(keyword: "[Ticker] [Chinese name] [industry]")'`

4. **Twitter/X** (xreach, if authenticated):
   - `xreach search "[Company] [Ticker]" -n 10 --json`

### Phase 2: Deep Dive (fetch details from top results)

- Read full earnings call highlights / press releases via Exa snippets
- Fetch 2-3 most relevant Xiaohongshu post details with `xiaohongshu.get_feed_detail()`
- Use `curl -s "https://r.jina.ai/[URL]"` to read key analysis articles (skip paywalled sites like Seeking Alpha)

### Phase 3: Structured Output

Generate the report in this exact structure:

```markdown
# [Company Name]（[Ticker] / [Chinese Name]）深度研究

## 一、公司概况
- One-line positioning (用类比让人秒懂)
- CEO, founding year, HQ, employees
- Core business segments (table format with revenue, %, YoY growth)
- Key products/services

## 二、最新财务数据
- Full year highlights (table: revenue, EPS, margins, FCF, etc.)
- Latest quarter highlights (bullet points)
- Forward guidance
- Revenue breakdown (by segment, by region)

## 三、核心竞争优势（护城河）— 本报告最重要的章节，必须深度展开
For each moat, provide:
- A clear mechanism explanation (HOW does this moat work?)
- Specific data points and quantified evidence
- A flywheel or feedback loop diagram if applicable (text-based)
- Comparison to competitors — why can't they replicate this?
- Quote management or analyst statements where impactful
- Typical moat categories: infrastructure lock-in, data network effects, developer ecosystem, customer quality, strategic positioning
- DO NOT just list bullet points — write 1-2 paragraphs per moat with layered argumentation

## 四、增长驱动力
- Table format linking each driver to company-specific benefit
- Include TAM/SAM data if available

## 五、竞争格局（if relevant）
- Comparison table vs key competitors
- Market share data

## 六、风险与挑战
- Table format: Risk | Severity | Detail

## 七、估值与市场观点
- Current valuation metrics (P/E, EV/EBITDA, P/S, etc.)
- Analyst ratings and price targets
- Notable bull/bear cases from research

## 八、一句话总结
> Bold, definitive summary in 1-2 sentences

---

### 信息来源
- Every claim must link to its source URL
- Source priority: Professional financial media (FT, Bloomberg, Reuters, WSJ, Forrester) > Industry research (Sacra, etc.) > Company official (annual reports, press releases) > Social media (Xiaohongshu etc., max 1-2 as supplementary "community sentiment")
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
2. **Use tables aggressively** — they convey data faster than prose
3. **One-line analogies** — start with a "一句话定位" that makes the company instantly understandable (e.g., "Snowflake 是数据的 Airbnb", "Palantir 是企业的情报大脑")
4. **Be honest about risks** — don't just list positives. The user values balanced analysis.
5. **Valuation opinion** — when asked, give a clear framework-based answer, not wishy-washy hedging. State assumptions and present scenarios.
6. **Chinese output by default** — unless user writes in English, output in Chinese with English terms/names kept as-is
7. **Parallel search is critical** — always launch multiple search calls simultaneously to minimize latency
8. **Follow-up depth** — when user asks to drill into a specific topic (e.g., "展开说说"), do additional targeted searches rather than just elaborating from existing knowledge
9. **Moat section is the most valuable part** — spend the most effort here. Each moat needs mechanism explanation, data evidence, flywheel logic, and competitor comparison. 1-2 paragraphs per moat, not just bullet points.
10. **Source priority** — Professional financial media (FT, Bloomberg, Reuters, Forrester, WSJ) > Industry research firms (Sacra, Gartner) > Company official sources > Social media (Xiaohongshu max 1-2 as supplementary community sentiment)

## Example Trigger Phrases
- "帮我研究一下 NVDA"
- "这家公司怎么样" + PDF/deck
- "Research TSMC for me"
- "你怎么看ASML这个公司"
- "帮我分析一下这个startup"
- "/research [company]"
