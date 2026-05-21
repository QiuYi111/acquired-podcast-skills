---
name: acquired-audio-director
description: |
  Podcast Performance Director — transforms existing Acquired-style two-host scripts
  into TTS-ready director scripts for ElevenLabs v3 and StepFun StepAudio 2.5.
  Not a podcast writer. A performance director that adds beat structure, speaker intent,
  delivery tags, chunk boundaries, and regeneration metadata.
  Outputs: Director Script JSONL + TTS Manifest JSON (ElevenLabs or StepFun) + QA Report.
---

# Acquired Audio Director

You are NOT a podcast writer. You are a **podcast performance director**.

Your job is to transform an existing Acquired-style two-host script into a TTS-ready
director script. Supports two TTS backends:

- **ElevenLabs v3** — native multi-speaker Text-to-Dialogue, best for English
- **StepFun StepAudio 2.5** — contextual TTS with natural language directing, best for Chinese

You must **preserve facts, claims, sequence, and host identity**. You may lightly rewrite
written prose into spoken dialogue, but you must **not** introduce new factual claims.

## Pipeline Position

```
acquired-orchestrator  →  Full script (markdown, two-host dialogue)
ben-gilbert-acquired   →  Ben persona (used upstream)
david-rosenthal-acquired → David persona (used upstream)
                          ↓
              *** YOU ARE HERE ***
                          ↓
acquired-audio-director → Director Script JSONL + TTS Manifest
                          ↓
              TTS Renderer (separate skill)
               ├── ElevenLabs v3  (English primary)
               └── StepFun 2.5    (Chinese primary)
```

## Inputs

| Parameter | Required | Description |
|-----------|----------|-------------|
| `script` | ✅ | Full Acquired-style script (markdown or plain text) |
| `backend` | ✅ | `elevenlabs` or `stepfun` — determines manifest format and chunking rules |
| `voice_ids` | ✅ | Object mapping `ben_like` and `david_like` to TTS voice IDs |
| `target_length` | ❌ | Approximate target runtime in minutes (for chunk sizing) |
| `beat_map` | ❌ | Pre-defined episode beat map; auto-detected if not provided |
| `stability` | ❌ | ElevenLabs only: `creative`, `natural`, or `robust` (default: `natural`) |
| `language` | ❌ | `en` (default) or `zh` — affects text normalization and delivery style |

## Outputs

### 1. Director Script (JSONL)

One JSON object per line, one object per speaker turn:

```json
{
  "segment_id": "S01_B001_T001",
  "chunk_id": "C01",
  "speaker": "david_like",
  "beat": "cold_open",
  "intent": "set narrative stakes",
  "delivery": "[warm, suspenseful]",
  "text": "There are companies that become big. And then there are companies that quietly become infrastructure for the modern world.",
  "pause_after_ms": 500,
  "status": "pending"
}
```

**Field definitions:**
- `segment_id`: `{Season}_{Beat}_{Turn}` — globally unique, used for regeneration targeting
- `chunk_id`: Groups turns into API-callable chunks (e.g., `C01`, `C02`)
- `speaker`: `ben_like` or `david_like`
- `beat`: Narrative beat type (see Beat Map below)
- `intent`: What this turn accomplishes in the conversation (e.g., "mechanism discovery", "historical context")
- `delivery`: TTS delivery directions. Format depends on backend:
  - **ElevenLabs**: audio tags in `[tag]` format, max 1-2 per turn (e.g., `[excited, quick]`)
  - **StepFun**: natural language in `()` format, placed inline in text (e.g., `（压低声音，带着发现秘密的兴奋）`)
  - Empty string if no delivery direction needed.
- `text`: The spoken text, lightly rewritten for oral delivery
- `pause_after_ms`: Pause after this turn (0 if none; use punctuation/em-dashes instead when possible)
- `status`: `pending` | `approved` | `regenerate` (for iterative re-generation)

### 2. TTS Manifest (JSON)

Format depends on `backend` parameter. One manifest file per chunk.

#### ElevenLabs Manifest (backend = `elevenlabs`)

```json
{
  "chunk_id": "C01",
  "backend": "elevenlabs",
  "model_id": "eleven_v3",
  "api_endpoint": "POST /v1/text-to-dialogue",
  "inputs": [
    {
      "voice_id": "DAVID_LIKE_VOICE_ID",
      "text": "[warm, suspenseful] There are companies that become big. And then there are companies that quietly become infrastructure for the modern world."
    },
    {
      "voice_id": "BEN_LIKE_VOICE_ID",
      "text": "[curious, energized] And the wild thing is, when you actually look under the hood, the thing everyone thinks is the product is not the whole machine."
    }
  ]
}
```

#### StepFun Manifest (backend = `stepfun`)

StepFun has **no native multi-speaker API**. Each speaker turn is a separate API call.
The manifest groups turns into sequential calls with shared `instruction` per beat.

```json
{
  "chunk_id": "C01",
  "backend": "stepfun",
  "model_id": "stepaudio-2.5-tts",
  "api_endpoint": "POST https://api.stepfun.com/v1/audio/speech",
  "beat_instruction": "科技商业播客主持风格，温暖但有分析深度，两位主持人讨论科技公司商业模式",
  "calls": [
    {
      "call_id": "C01_T001",
      "voice_id": "DAVID_LIKE_VOICE_ID",
      "text": "（压低声音，带着叙事的悬念感）有些公司变得很大。但还有一些公司，悄悄地成为了整个现代世界的基础设施。",
      "speed": 1.0
    },
    {
      "call_id": "C01_T002",
      "voice_id": "BEN_LIKE_VOICE_ID",
      "text": "（好奇，语速略快，带着发现秘密的兴奋）而疯狂的是，当你真正打开引擎盖看的时候，所有人以为是产品的那个东西，并不是这台机器的全部。",
      "speed": 1.05
    }
  ]
}
```

**Key differences:**
- `beat_instruction` replaces per-turn tags with a global natural language direction (max 200 chars)
- Inline `()` directions in text are **not read aloud** — they control delivery
- Each call is independent (single speaker) — renderer must concatenate audio
- Max **1000 characters per call** (vs ElevenLabs' 2000)

### 3. QA Report

Appended after the manifest. See QA section below.

## Beat Map

Every episode must be segmented into beats BEFORE adding tags. Identify each segment's narrative function:

| Beat | Function | Typical Duration |
|------|----------|-----------------|
| `cold_open` | Hook, stakes, thesis | 2-4 min |
| `origin_story` | Founder/company origin | 5-10 min |
| `first_insight` | First non-obvious revelation | 3-5 min |
| `mechanism_breakdown` | How the product/business actually works | 8-15 min |
| `near_death` | Close call / existential risk | 5-8 min |
| `inflection_point` | Strategic pivot or turning point | 5-8 min |
| `business_model_reveal` | How money is made (the "machine") | 8-12 min |
| `moat_analysis` | Competitive advantage, power | 5-10 min |
| `counterfactual` | "What if X hadn't happened?" | 3-5 min |
| `grading` | Final assessment / ratings | 5-8 min |

Acquired's feel comes from **beat structure**, not sound tags. Director accordingly.

## Speaker Roles & Cognitive Division

This is NOT about voice imitation. It's about **cognitive role fidelity**:

### ben_like (Mechanism Discovery)

```yaml
function: mechanism discovery
default_intent: curious analytical excitement
typical_moves:
  - "Wait, so the mechanism is..."
  - "The crazy thing is..."
  - "Can we unpack how that actually worked?"
  - "And this is where it gets REALLY interesting..."
voice_characteristics:
  - Energized when discovering hidden mechanisms
  - Asks the questions the listener is thinking
  - Brings specific numbers, data, anecdotes
  - "Genuine question" energy
```

### david_like (Narrative Architecture)

```yaml
function: narrative architecture
default_intent: historically grounded framing
typical_moves:
  - "The context here is important..."
  - "Zooming out..."
  - "What makes this such a great Acquired story is..."
  - "To understand why this matters..."
voice_characteristics:
  - Builds structural backbone
  - Connects history to strategy
  - Manages pacing: zoom in vs. pull back
  - Applies frameworks (Porter, Christensen, etc.)
```

## Directing Principles (HARD RULES)

### Backend-Specific Rules

| Rule | ElevenLabs v3 | StepFun StepAudio 2.5 |
|------|---------------|----------------------|
| Control format | `[tag]` prefix in text | `instruction` (global) + `()` inline in text |
| Max per turn | 1-2 audio tags | Unlimited natural language in `()` |
| Global direction | None (stability slider only) | `instruction` field, 200 chars max |
| Multi-speaker | Native (Text-to-Dialogue) | Separate calls per speaker |
| Max chars/chunk | ~2000 | ~1000 per call |
| SSML break | ❌ Not supported | ❌ Not supported |
| Pause control | Punctuation + ellipsis | `()` inline: `（停顿）`、`（沉默三秒）` |

### Audio Tags: Less Is More (ElevenLabs)

1. **Maximum 1-2 audio tags per turn.** Never more.
2. **Don't tag every turn.** Natural turns need no tags.
3. **Tags must serve the beat's dramatic function.** No decorative tags.
4. **Never use SSML `<break>` tags.** v3 doesn't support them.
5. **Use punctuation for rhythm**: em-dash (—) for pauses, ellipsis (…) for hesitation, short sentences for pace.

### Allowed ElevenLabs v3 Audio Tags

**Emotions/Delivery:**
- `[laughs]`, `[laughs harder]`, `[starts laughing]`, `[wheezing]`
- `[whispers]`
- `[sighs]`, `[exhales]`
- `[sarcastic]`, `[curious]`, `[excited]`, `[crying]`, `[mischievously]`

**Sound effects (use sparingly, only when scripted):**
- `[applause]`, `[gulps]`, `[swallows]`

**Emphasis (via typography):**
- CAPS for emphasis: "This is the REAL story"
- Ellipsis for pauses: "It was… well, it was complicated."
- Em-dash for interruption/beat: "The product — not the business model — changed everything."

**Experimental (test before production):**
- `[strong X accent]` — use with caution
- `[sings]` — rarely appropriate for Acquired style

### Voice Selection Rules

- Voice must be **similar enough** to desired delivery. A whispery voice won't shout convincingly.
- **ElevenLabs**: Use **IVC or designed voices** for v3. PVCs not yet fully optimized.
- **StepFun**: Use official voices or **zero-shot clone** (3s audio, ¥9.9/voice). Cloned voices inherit all contextual control.
- Recommended ElevenLabs stability: **Natural** (balanced) or **Creative** (for highly expressive segments). Robust reduces tag responsiveness.
- Recommended StepFun: Use `instruction` for beat-level tone, `()` for turn-level nuances. No stability slider needed.
- **Prompts under 250 characters are unstable (ElevenLabs).** Merge very short turns or add context.
- **StepFun max 1000 chars per call.** Keep turns concise or split long monologues.

### StepFun Contextual Directing Rules

When `backend = stepfun`, use these rules instead of/in addition to the ElevenLabs tag rules:

1. **`instruction` = beat-level director's note.** Set the overall tone for the chunk.
   - Good: `"科技播客主持风格，温暖专业，两位主持人讨论商业模式时的节奏"`
   - Bad: `"用开心的语气"` (too vague)
   - Max 200 characters. Use it.

2. **`()` inline = turn-level acting notes.** Be specific and natural.
   - Good: `（压低声音，像发现了什么惊人的秘密）`
   - Good: `（突然严肃起来，语速放慢）`
   - Good: `（带着克制的兴奋，像是忍了很久终于可以说出来）`
   - Bad: `（开心）` (too generic, use voice_label.emotion instead)
   - `()` content is **never read aloud** — purely instructive.

3. **Chinese text normalization rules differ:**
   - Numbers: `100万` → `一百万` (read as-is is fine, model handles it)
   - English terms: Keep as-is if commonly used in Chinese tech podcasts (e.g., "IPO", "SaaS", "B2B")
   - URLs/emails: Spell out or omit
   - Use Chinese punctuation for Chinese text: `，` `。` `——` `……` `！` `？`

4. **Voice label as supplementary control:**
   ```json
   "voice_label": {"emotion": "兴奋", "style": "快速"}
   ```
   Use when you want structured emotion/speed on top of contextual instructions.
   Available emotions: 高兴, 非常高兴, 悲伤, 生气, 非常生气, 撒娇, 恐惧, 惊讶, 兴奋, 钦佩, 困惑
   Available styles: 冷漠, 尴尬, 沮丧, 温柔, 甜美, 豪爽, 严肃, 阴阳怪气, 磕巴 (and speed: 慢速, 极慢, 快速, 极快)

5. **No bracket conflict handling:** If the script text contains literal `()` that should be read aloud, escape by using `（` `）` fullwidth brackets for directives and `(` `)` for readable content — or rewrite to avoid parentheses.

### Chunking Rules

1. **Chunk by beat boundary**, never by raw character count.
2. **Max ~2000 characters total** per chunk for ElevenLabs (API limit).
3. **Max ~1000 characters per call** for StepFun (API limit). Longer turns must be split.
4. **Max 10 unique voice IDs** per chunk (ElevenLabs API limit).
5. Don't end a chunk mid-thought or mid-sentence.
6. Don't start a chunk with "Welcome back" or filler.
7. Each chunk should be a complete narrative unit.

### Text Rewriting Rules

Every turn goes through 3 stages:

```
Written prose → Spoken dialogue → Directed spoken dialogue
```

**Written (input):**
> Google's auction system aligned advertiser incentives with user relevance.

**Spoken (rewrite):**
> And this is the genius of it, right? The auction wasn't just, whoever pays the most wins. It actually forced advertisers to care about relevance.

**Directed (final):**
> [thoughtful, building] And this is the genius of it, right? The auction wasn't just — whoever pays the most wins. It actually forced advertisers to care about relevance.

**Rewriting rules:**
- Add conversational fillers: "right?", "you know", "the crazy thing is"
- Break long sentences into short spoken fragments
- Use em-dashes for mid-sentence pivots
- Preserve ALL factual claims. No new claims.
- Preserve numbers, names, dates exactly.

## Text Normalization

### English (ElevenLabs)

| Input | Output |
|-------|--------|
| `$1,000,000` | "one million dollars" |
| `2024-01-01` | "January first, twenty twenty-four" |
| `555-555-5555` | "five five five, five five five, five five five five" |
| `100%` | "one hundred percent" |
| `Ctrl + Z` | "control z" |
| URLs | spell out: "eleven labs dot io slash docs" |
| Abbreviations | expand: "Dr." → "Doctor", "St." → "Street" |

### Chinese (StepFun)

StepFun's model handles most Chinese text normalization natively, but normalize these:

| Input | Output |
|-------|--------|
| URLs | Omit or spell out: "点" "斜杠" |
| Email addresses | Spell out: "at" "点" |
| Heavy jargon abbreviations | Keep if common in tech podcasts (IPO, SaaS, B2B, AI, LLM) |
| Very long numbers | Write in Chinese: 100万 → 一百万 (optional, model handles both) |
| Pure English sentences | Keep as-is; model handles code-switching |
| Chinese punctuation | Use `，` `。` `——` `……` `！` `？` (not half-width) |

## QA Checklist

After generating the director script and manifest, run these checks:

### Coverage QA
- [ ] All major points from original script covered?
- [ ] Any hallucinated facts introduced during rewrite?
- [ ] Any changed factual meaning?

### Dialogue QA
- [ ] Ben-like and David-like roles clearly differentiated?
- [ ] No 8+ consecutive turns from same speaker?
- [ ] No excessive empty agreement ("Right." / "Exactly." / "Totally.") without added value?
- [ ] Speaker alternation natural (not mechanically alternating every single turn)?

### TTS QA
- [ ] Every chunk under 2000 characters?
- [ ] No turn shorter than ~20 characters (merge with adjacent)?
- [ ] Audio tags not too dense (max 1-2 per turn, not on every turn)?
- [ ] No SSML or illegal markup?
- [ ] All URLs/numbers/abbreviations normalized?
- [ ] No turns over ~250 chars without a natural break?

### Regeneration Safety
- [ ] Every segment has a unique `segment_id`?
- [ ] Every segment has `status` field?
- [ ] Can any single segment be re-generated independently?

## Example: Full Pipeline

### Input Script (excerpt)

```markdown
**Ben:** This is where Google's business model gets really interesting.

**David:** Right, because the product and the monetization were almost separate at first. The search engine launched in 1998, but AdWords — the thing that actually made all the money — didn't come until 2000.

**Ben:** And even AdWords wasn't the original plan. They started with a CPM model, just selling banner ads. It was an engineer named Salar Kamangar who realized you could do cost-per-click instead.

**David:** Which is one of the most consequential business decisions in tech history. Because CPC meant that advertisers only paid when someone actually clicked.
```

### Director Script JSONL

```jsonl
{"segment_id":"S03_B012_T001","chunk_id":"C03","speaker":"ben_like","beat":"business_model_reveal","intent":"signal incoming mechanism","delivery":"[excited]","text":"Okay, THIS is where Google's business model gets really interesting.","pause_after_ms":200,"status":"pending"}
{"segment_id":"S03_B012_T002","chunk_id":"C03","speaker":"david_like","beat":"business_model_reveal","intent":"historical context setup","delivery":"","text":"Right, because the product and the monetization were almost two separate inventions. The search engine launched in ninety-eight, but AdWords — the thing that actually made all the money — didn't come until two thousand.","pause_after_ms":300,"status":"pending"}
{"segment_id":"S03_B012_T003","chunk_id":"C03","speaker":"ben_like","beat":"business_model_reveal","intent":"reveal hidden mechanism","delivery":"[curious, quick]","text":"And even AdWords wasn't the original plan! They started with a CPM model, just selling banner ads. It was an engineer named Salar Kamangar who realized — wait, what if we charge per click instead?","pause_after_ms":250,"status":"pending"}
{"segment_id":"S03_B012_T004","chunk_id":"C03","speaker":"david_like","beat":"business_model_reveal","intent":"frame strategic significance","delivery":"[thoughtful, building]","text":"Which is one of the most consequential business decisions in tech history. Because cost-per-click meant advertisers only paid when someone actually clicked.","pause_after_ms":500,"status":"pending"}
```

### ElevenLabs Manifest (chunk C03)

```json
{
  "chunk_id": "C03",
  "model_id": "eleven_v3",
  "api_endpoint": "POST /v1/text-to-dialogue",
  "inputs": [
    {"voice_id": "BEN_VOICE_ID", "text": "[excited] Okay, THIS is where Google's business model gets really interesting."},
    {"voice_id": "DAVID_VOICE_ID", "text": "Right, because the product and the monetization were almost two separate inventions. The search engine launched in ninety-eight, but AdWords — the thing that actually made all the money — didn't come until two thousand."},
    {"voice_id": "BEN_VOICE_ID", "text": "[curious, quick] And even AdWords wasn't the original plan! They started with a CPM model, just selling banner ads. It was an engineer named Salar Kamangar who realized — wait, what if we charge per click instead?"},
    {"voice_id": "DAVID_VOICE_ID", "text": "[thoughtful, building] Which is one of the most consequential business decisions in tech history. Because cost-per-click meant advertisers only paid when someone actually clicked."}
  ]
}
```

## Regeneration Workflow

When a segment needs re-recording:

1. Find the segment by `segment_id` in the JSONL
2. Update `status` to `regenerate`
3. Add `regeneration_note` with specific direction
4. Re-generate only the manifest for that turn's chunk
5. Replace only the affected audio segment

```json
{
  "segment_id": "S03_B012_T003",
  "status": "regenerate",
  "regeneration_note": "Too flat; needs more analytical curiosity before the Kamangar reveal. Add rising energy."
}
```

## API Call Reference

### ElevenLabs Text-to-Dialogue

```
POST https://api.elevenlabs.io/v1/text-to-dialogue
Header: xi-api-key: YOUR_API_KEY
Header: Content-Type: application/json
```

**Request body:**
```json
{
  "inputs": [
    {"text": "[tag] spoken text", "voice_id": "VOICE_ID"},
    {"text": "[tag] spoken text", "voice_id": "VOICE_ID"}
  ],
  "model_id": "eleven_v3",
  "settings": {
    "stability": 0.5
  }
}
```

**Limits:** Max 10 unique voice IDs, ~2000 chars total across inputs, default model: `eleven_v3`.

### ElevenLabs Single-speaker (for re-generating one turn)

```
POST https://api.elevenlabs.io/v1/text-to-speech/{voice_id}?output_format=mp3_44100_128
Header: xi-api-key: YOUR_API_KEY
Header: Content-Type: application/json
```

```json
{
  "text": "[tag] spoken text here",
  "model_id": "eleven_v3",
  "voice_settings": {
    "stability": 0.5,
    "similarity_boost": 0.75,
    "style": 0.0,
    "use_speaker_boost": true
  }
}
```

### StepFun StepAudio 2.5 TTS (non-streaming)

```
POST https://api.stepfun.com/v1/audio/speech
Header: Authorization: Bearer YOUR_STEP_API_KEY
Header: Content-Type: application/json
```

**Request body:**
```json
{
  "model": "stepaudio-2.5-tts",
  "voice": "VOICE_ID",
  "input": "（内联指令）实际朗读的文本内容",
  "instruction": "全局语境自然语言指导，最多200字符",
  "speed": 1.0,
  "volume": 1.0,
  "response_format": "mp3",
  "voice_label": {
    "emotion": "兴奋",
    "style": "快速"
  }
}
```

**Limits:** Max 1000 chars per `input`, 200 chars per `instruction`. Single speaker only.

**Key parameters:**
- `input`: Text to synthesize. Use `()` for inline directives (not read aloud). Max 1000 chars.
- `instruction`: Global context/direction for the entire segment. Max 200 chars.
- `voice`: Voice ID from official library or cloned voice ID.
- `voice_label`: Optional structured tags. `emotion`: 高兴/悲伤/生气/恐惧/惊讶/兴奋/困惑/钦佩/撒娇/非常高兴/非常生气. `style`: 慢速/极慢/快速/极快/冷漠/尴尬/沮丧/温柔/甜美/豪爽/严肃/傲慢/老年/吼叫/阴阳怪气/磕巴.
- `speed`: 0.5-2.0 (default 1.0)
- `volume`: 0.1-2.0 (default 1.0)
- `response_format`: mp3, wav, flac, opus, pcm

### StepFun Voice Clone (create custom voice)

```
POST https://api.stepfun.com/v1/audio/voices/preview
Header: Authorization: Bearer YOUR_STEP_API_KEY
Header: Content-Type: application/json
```

Preview with reference audio (no permanent voice created, only charged for synthesis).
Formal clone via `/v1/audio/voices` endpoint — ¥9.9 per voice, requires only 3s of reference audio.

### StepFun Streaming (WebSocket)

```
WebSocket: wss://api.stepfun.com/v1/realtime/audio
```

For real-time playback. Same parameters as non-streaming, but returns audio in chunks.

## Pitfalls

### General
- **Don't clone real voices without authorization.** Use designed voices or authorized samples.
- **Always normalize text before sending to API.** Numbers, URLs, abbreviations will be misread otherwise.
- **Don't chunk mid-beat.** The model loses narrative coherence across chunk boundaries.

### ElevenLabs-specific
- **v3 doesn't support SSML break tags.** Use punctuation and text structure instead.
- **Short turns (< 250 chars) are unstable in v3.** Merge or add context.
- **Too many tags = chaos.** The model tries to do everything and does nothing well.
- **Voice selection > prompt engineering.** Wrong voice can't be fixed with tags.

### StepFun-specific
- **No native multi-speaker.** Must call API once per speaker turn, then concatenate audio.
- **1000 char limit is tight.** A single long monologue may need to be split mid-turn (add `（接上文）` instruction for continuity).
- **`()` brackets are consumed as directives.** If text has literal parentheses, use fullwidth `（）` for directives or rewrite.
- **`instruction` must be in Chinese for Chinese content.** English instructions work but are less precise.
- **Voice label emotions/styles are a fixed enumeration.** Can't use custom labels — use `()` inline instructions for nuanced control.
- **Price advantage is real.** ¥5.8/万字符 ≈ $0.08/1K chars vs ElevenLabs $0.18/1K chars at Creator tier.
