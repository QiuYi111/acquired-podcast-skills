# NeuralNoise Lessons

> Key takeaways from NeuralNoise (segment-level podcast generation) applicable to director scripting.

## Segment-Level Regeneration

NeuralNoise's core insight: **never re-run the entire pipeline when one segment is wrong.**

Architecture:
1. Script is divided into segments
2. Each segment generates independently
3. Audio is stored per-segment
4. User can edit individual segments
5. Only edited segments are re-generated
6. Final audio is concatenated from all segments

**Implementation in Director Script:**
- Every turn gets a unique `segment_id`
- Every turn has a `status` field (pending/approved/regenerate)
- Audio path stored per segment
- Manifest can be re-generated for a single chunk
- Adjacent segments can be re-generated for context continuity

## Manual Editing Workflow

NeuralNoise allows manual text editing before TTS:

```
Generated Script → User Edits Text → Re-generate Only Changed Segments
```

This is critical because:
1. TTS output reveals issues invisible in text (mispronunciations, wrong emphasis)
2. Long podcasts will always need some manual tweaking
3. Re-running everything costs time and money

**Implementation:**
```jsonl
{"segment_id":"S03_B012_T004","status":"regenerate","regeneration_note":"too flat; add rising energy before Kamangar reveal"}
```

## Segment Stitching

Audio segments must be stitched together seamlessly:
- Short crossfade between segments (~50ms)
- Match volume levels across segments
- Maintain consistent room tone/ambient noise

**Note:** This is handled by the downstream `elevenlabs-renderer` skill, but the director script must ensure clean segment boundaries (no mid-sentence cuts).

## Quality Checkpoints

NeuralNoise validates at multiple stages:
1. **Text validation** — does the script make sense?
2. **TTS validation** — did the model produce clean audio?
3. **Concatenation validation** — do segments flow naturally?

Our QA checklist in the SKILL.md covers #1. The renderer skill should handle #2 and #3.

## What NeuralNoise Gets Wrong (for our use case)

- No concept of speaker roles or cognitive division
- No beat-level narrative structure
- No TTS-specific prompt engineering (audio tags, punctuation as direction)
- No text normalization for TTS
- Generic podcast format, not Acquired-specific

Our director skill adds all of these on top of the segment-level regeneration model.
