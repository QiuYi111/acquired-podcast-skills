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

### 🎬 `acquired-audio-director/` *(NEW)*

The performance director — transforms finished scripts into TTS-ready director scripts for ElevenLabs v3. This is NOT a podcast writer; it's a **podcast performance director** that:

- Segments the script into **narrative beats** (cold open → origin → mechanism → business model → grading)
- Adds **speaker intent** and **delivery direction** per turn (not just raw emotion tags)
- Rewrites written prose into **spoken dialogue** while preserving all facts
- Outputs **Director Script JSONL** (one turn per line, with segment IDs for regeneration)
- Compiles into **ElevenLabs Dialogue Manifest JSON** (ready for Text-to-Dialogue API)
- Runs a **QA checklist** (coverage, dialogue balance, TTS limits, text normalization)

Includes:
- **JSON schemas** for both director script and ElevenLabs manifest (validate before calling API)
- **ElevenLabs v3 directing notes** — extracted from official docs: audio tags, punctuation as direction, stability settings, voice selection rules
- **Podcastfy lessons** — conversation configuration, long-form chunking, style transfer patterns
- **NeuralNoise lessons** — segment-level regeneration, manual editing workflow, audio stitching

#### Pipeline

```
acquired-orchestrator → Full script
                          ↓
         acquired-audio-director → Director Script JSONL + ElevenLabs Manifest
                          ↓
              elevenlabs-renderer → Audio segments (separate skill, TBD)
```

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
