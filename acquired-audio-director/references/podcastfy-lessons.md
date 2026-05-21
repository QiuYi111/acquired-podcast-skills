# Podcastfy Lessons

> Key takeaways from Podcastfy (open-source podcast generation) applicable to director scripting.

## Conversation Configuration

Podcastfy's approach: configure the *conversation architecture* before generating content.

```yaml
conversation_style: [deeply researched, narrative, analytical, warm, intellectually excited]
roles_person1: [mechanism discovery, product analysis, data-driven, curious]
roles_person2: [narrative architecture, historical context, strategic framing]
dialogue_structure: [cold open → thesis → evidence → counterpoint → synthesis]
engagement_techniques: [rhetorical questions, "the crazy thing is", callback references]
```

**Lesson:** Don't generate dialogue without first defining the cognitive division of labor. This is especially critical for Acquired style where Ben and David have distinct analytical roles.

## Long-Form Handling

Podcastfy splits long content into parts and carries conversation context forward:

1. Chunk the source material
2. For each part, include context from previous conversation
3. Require natural continuation (no "welcome back" between parts)
4. Maintain speaker alternation patterns

**Lesson for Director Script:**
- Chunk by narrative beat, not character count
- Carry emotional arc across chunks
- Never insert filler transitions between chunks
- The API handles speaker alternation within a chunk

## Style Transfer

Podcastfy uses a "conversation style" parameter that controls the entire tone. This is superior to per-turn tagging because it maintains consistency.

**Lesson:** The director script's `beat` and `intent` fields serve this purpose. They ensure that all turns within a beat share the same emotional register, rather than tagging each turn independently.

## What Podcastfy Gets Wrong (for our use case)

- No concept of "beat" or narrative arc — it treats all content equally
- No regeneration support — must re-run entire generation
- No TTS-specific formatting — outputs raw dialogue text
- No QA layer — trusts the LLM output
- No cognitive role fidelity — speakers are interchangeable

Our director skill must address all of these.
