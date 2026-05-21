# Acquired Podcast Skills

AI skills for generating [Acquired](https://acquired.fm)-style podcast scripts. Built for use with [Hermes Agent](https://hermes-agent.nousresearch.com/) but the persona definitions and methodology are framework-agnostic.

## What's Inside

### 🎙️ `acquired-orchestrator/`

The main orchestrator — takes any company/organization as input and produces a full Acquired-style podcast script. Coordinates the Ben Gilbert and David Rosenthal personas to generate authentic two-host dialogue with:

- Structured narrative arc (origin story → inflection points → flywheel → moat → future)
- In-line fact-checking via web search
- Authentic interjections, reactions, and tangents
- Opening cold open + closing "Grading" segment

Includes reference materials:
- **Methodology Framework** — extracted from Acquired's 10-year anniversary episode
- **Transcript Extraction Methodology** — how to pull source transcripts from acquired.fm
- **Google Verification Test** — first end-to-end test run notes
- **Google Comparison Analysis** — generated script vs. real Acquired Google episodes, quality benchmarking

### 🧠 `ben-gilbert-acquired/`

Ben Gilbert's persona — the product-obsessed, curiosity-driven analyst. Key traits:

- Digs into *why* products work (not just *what*)
- "Genuine question" energy — asks the questions listeners are thinking
- Loves a good mechanism breakdown (network effects, virality loops, flywheels)
- Brings receipts: data, anecdotes, specific numbers

### 🏛️ `david-rosenthal-acquired/`

David Rosenthal's persona — the narrative architect and strategic framework analyst. Key traits:

- Builds the structural backbone of each episode
- Connects history to strategy (contextual storytelling)
- Loves a good framework (Porter's Five Forces, Clayton Christensen, etc.)
- Manages pacing: knows when to zoom in on a detail vs. pull back to the big picture

## How to Use

These are [Hermes Agent skills](https://hermes-agent.nousresearch.com/docs/skills) — drop them into `~/.hermes/skills/` and they're automatically available.

```bash
# Clone into your skills directory
git clone https://github.com/QiuYi111/acquired-podcast-skills.git
cp -r acquired-podcast-skills/* ~/.hermes/skills/
```

Each skill has a `SKILL.md` with full instructions, prompt templates, and usage guidelines.

### Standalone Usage

The `SKILL.md` files are self-contained prompts — you can copy the content into any LLM context (Claude, GPT, etc.) without needing Hermes. The orchestrator assumes it can delegate to the two persona sub-skills, but you can also run the personas independently for one-host analysis.

## Quality Benchmarks

Tested against Acquired's actual Google episodes (S16E1/S16E2). The comparison analysis is included in `acquired-orchestrator/references/`. Key findings:

- **Narrative structure** closely matches real episode flow
- **Host voice authenticity** rated 7-8/10 vs. real transcripts
- **Fact density** comparable when web search is enabled
- Main gap: spontaneous humor and in-the-moment chemistry (hard to script)

## License

MIT — use however you want. The Acquired podcast itself is © Acquired LLC; these skills are an independent fan-made tool for generating similar-style content.

---

*Built with 🎙️ by [Jingyi Qiu](https://github.com/QiuYi111)*
