# Google Verification Test — Session Notes

> First end-to-end run of the acquired-orchestrator skill, 2026-05-20.
> Company: Google (covered by Acquired S16E1/S16E2). Mode: recreation / verification test.

## Pipeline Execution

### Phase 1: Topic Evaluation
- **Hero Protagonist**: Larry Page & Sergey Brin — Stanford PhDs → rejected by Excite/Yahoo → $1.7T empire. ✅
- **Secret in Plain Sight**: GSP auction economics + Quality Score flywheel. Users never pay, advertisers compete for relevance, better ads win. ✅
- **Importance**: Most valuable information gatekeeper in history. ✅
- **Verdict**: 🟢 GREEN LIGHT

### Phase 2: Parallel Research
Two batches of 3 subagents each:

**Batch 1** (origin story, financials, canonical works):
- Origin story: high quality (8,882 bytes) — PageRank → Stanford → Andy Bechtolsheim's check → garage
- Financials: partial — hit rate limits, got some data
- Canonical works: partial — rate limited

**Batch 2** (product mechanics, 7 Powers, culture + secret):
- Product mechanics: good (3,708 bytes) — GSP auction, Quality Score, three flywheels
- 7 Powers: good (3,942 bytes) — Scale Economies, Network Effects, Counter-Positioning identified
- Culture + secret: good (4,419 bytes) — "perpetual motion machine" thesis

**Lesson**: Batch 2 (product/framework/culture) produced consistently better results than Batch 1 (financials/canonical). Financial data gathering is more rate-limit-prone.

### Phase 3: Production Meeting
Core theme identified: **"Google built a perpetual motion machine"**
- Three interlocking flywheels: data → users → advertisers → money → better products → more data
- GSP auction economics as the "secret in plain sight"
- 12 sections planned: Cold Open → Founding → Excite rejection → Garage → Eric Schmidt → AdWords flywheel → The Secret → IPO → 7 Powers → Culture → Bear case → Carve-outs

### Phase 4: Script Generation
Generated in 3 parts due to output length limits:
- Part 1: Cold Open → Section IV (19,618 bytes)
- Part 2: Section V → Section VIII (19,796 bytes)
- Part 3: Section IX → Section XII + Carve-outs (19,104 bytes)

Merged via `execute_code` → `output/google-acquired-episode.md` (9,562 words, 62KB)

**Lesson**: ~10K words is achievable in a single session run. To reach 25K+, would need either: (a) deeper per-section generation, (b) multiple passes with expansion, or (c) accepting 10-15K as the practical ceiling for one-shot generation.

### Phase 5: Quality Review
All 4 tests passed:
- Timelessness ✅ — Narrative-driven, not time-sensitive
- Secret ✅ — GSP vs VCG auction, AdSense long tail
- Hero's Journey ✅ — Larry/Sergey from rejection to trillion-dollar empire
- Acquired Standard ✅ — Cold open hook + deep research + carve-outs

## Output Files

```
PER-120/
├── output/
│   ├── google-acquired-episode.md      # Merged full script (9,562 words)
│   ├── google-episode-part1.md         # Cold Open → Section IV
│   ├── google-episode-part2.md         # Section V → Section VIII
│   └── google-episode-part3.md         # Section IX → Carve-outs
├── references/sources/
│   ├── 01-product-mechanics.md         # GSP, Quality Score, flywheels
│   ├── 03-origin-story.md              # PageRank → Bechtolsheim → garage
│   ├── 04-seven-powers.md              # Helmer framework analysis
│   ├── 05-culture-and-secret.md        # Perpetual motion machine thesis
│   └── production-meeting-notes.md     # Core theme + section plan
└── Context.md                          # Updated with progress
```

## Key Takeaways for Skill Improvement

1. **Multi-part generation is mandatory** — Cannot generate 25K+ words in one pass. The 3-part split pattern worked well.
2. **Research quality varies by dimension** — Origin stories and culture produce richer output than financials (which are more data-heavy and rate-limit-prone).
3. **Production meeting is the keystone** — Getting the core theme right here ("perpetual motion machine") made Phase 4 coherent.
4. **9-10K words is the practical ceiling** for a single-session run. Future iterations should consider multi-pass expansion.
