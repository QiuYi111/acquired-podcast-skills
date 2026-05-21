# ElevenLabs v3 Directing Notes

> Extracted from official docs: https://elevenlabs.io/docs/overview/capabilities/text-to-speech/best-practices

## Voice Selection (MOST IMPORTANT)

- Voice must be **similar to desired delivery**. A whispery voice won't shout with `[shout]`.
- Use **IVC or designed voices** for v3. PVCs not yet fully optimized.
- Voices from library may produce more variable results in v3 vs v2/v2.5.
- Recommended: 22+ curated v3-ready voices in ElevenLabs library.

## Stability Settings

| Mode | Stability | Use Case |
|------|-----------|----------|
| Creative | Low | Maximum expressiveness, accepts audio tags. Prone to hallucinations. |
| Natural | Medium (0.5) | Closest to original voice. Balanced. Best for most podcast use. |
| Robust | High | Very stable, but ignores most directional prompts. Like v2. |

**For Acquired-style:** Use **Natural** as default. Switch to **Creative** for emotionally intense beats (near-death, grading). Never use Robust for directed content.

## Audio Tags

### Voice-related (emotions)
```
[laughs] [laughs harder] [starts laughing] [wheezing]
[whispers]
[sighs] [exhales]
[sarcastic] [curious] [excited] [crying] [snorts] [mischievously]
```

### Sound effects
```
[gunshot] [applause] [clapping] [explosion]
[swallows] [gulps]
```

### Experimental
```
[strong X accent]  — e.g., [strong French accent]
[sings] [woo] [fart]
```

## Punctuation as Direction

| Technique | Effect | Example |
|-----------|--------|---------|
| Ellipsis (…) | Hesitation, pause, weight | "It was… well, it was complicated." |
| Em-dash (—) | Interruption, pivot, beat | "The product — not the business — changed everything." |
| CAPS | Emphasis | "This is the REAL story." |
| Short sentences | Quick pace, urgency | "And then? Everything changed." |
| `!` | Energy | "That's incredible!" |
| `?` | Rising tone, curiosity | "But why does this matter?" |

## What v3 Does NOT Support

- **SSML `<break>` tags** — use punctuation, ellipsis, em-dashes instead
- `<break time="1.5s" />` — will be read as literal text or cause errors
- Phoneme tags — only Flash v2 and English v1

## Multi-Speaker Dialogue

v3 handles multi-voice prompts natively. Use the Text-to-Dialogue API:

```
POST /v1/text-to-dialogue
```

Each turn has its own `voice_id` and `text`. Tags go inside the `text` field:

```json
{
  "inputs": [
    {"text": "[excitedly] Check this out!", "voice_id": "VOICE_A"},
    {"text": "[curiously] What is it?", "voice_id": "VOICE_B"}
  ]
}
```

**Limits:** Max 10 unique voice IDs, ~2000 chars total per request.

## Text Normalization

Always normalize before sending to API:

| Input | Normalized |
|-------|-----------|
| `$1,000,000` | "one million dollars" |
| `2024-01-01` | "January first, twenty twenty-four" |
| `555-555-5555` | "five five five, five five five, five five five five" |
| `100%` | "one hundred percent" |
| `3.14` | "three point one four" |
| `Ctrl + Z` | "control z" |
| `elevenlabs.io/docs` | "eleven labs dot io slash docs" |
| `Dr.` | "Doctor" |
| `St. Patrick` | "Saint Patrick" (keep as-is) |
| `123 Main St` | "one two three Main Street" |

## Prompt Length

- **Prompts under ~250 characters are unstable.** Merge short turns.
- Longer prompts give the model more context for consistent delivery.
- But keep total per chunk ≤2000 chars (API limit).

## Enhance Feature (ElevenLabs UI)

The UI has an "Enhance" button that uses an LLM to add audio tags. The underlying system prompt:
1. Adds audio tags from a curated list
2. Does NOT alter original text
3. Adds emphasis via CAPS, punctuation
4. Places tags before or after relevant dialogue
5. Avoids non-auditory tags (`[standing]`, `[grinning]`, `[music]`)

This is essentially what our director skill does, but with Acquired-specific role awareness.
