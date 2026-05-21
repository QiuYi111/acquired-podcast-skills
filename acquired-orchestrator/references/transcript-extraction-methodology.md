# Acquired Transcript Extraction Methodology

## Source: acquired.fm (Official)

**Status: WORKING (tested 2025-05-20)**

podscripts.co does NOT reliably have Acquired transcripts — Google series pages redirect to generic pages with no actual content. Use acquired.fm instead.

### Extraction Steps

1. **Navigate to episode page**: `browser_navigate("https://www.acquired.fm/episodes/<slug>")`
2. **Click TRANSCRIPT button**: The button has ref like `@e30`, labeled "TRANSCRIPT". Use `browser_click`.
3. **Wait for load**: The transcript div (`#transcript`) starts with class `display-none` and toggles to visible. Verify with:
   ```javascript
   (() => {
     const t = document.getElementById('transcript');
     return { class: t?.className, textLen: t?.textContent?.length || 0 };
   })()
   ```
4. **Extract in chunks**: The transcript is in `.episode-rich-text` with `<p>` elements. Each paragraph starts with speaker label ("Ben:" or "David:" etc). Extract 200-300 paragraphs at a time via `browser_console`:
   ```javascript
   (() => {
     const richText = document.querySelector('.episode-rich-text');
     const paragraphs = richText.querySelectorAll('p');
     const lines = [];
     for (let i = START; i < Math.min(END, paragraphs.length); i++) {
       lines.push(paragraphs[i].textContent);
     }
     return lines.join('\n\n');
   })()
   ```
5. **Save to file**: Write extracted text to `/tmp/` for analysis.

### Episode Data (Google Series)

| Episode | Slug | Season | Date | Duration |
|---------|------|--------|------|----------|
| Google: The Origin of Search | `google` | Summer 2025 E2 | 2025-06-30 | ~4h |
| Google: The AI Company | `google-the-ai-company` | Fall 2025 E1 | 2025-10-06 | ~4h |
| Alphabet Inc. | `alphabet-inc` | Summer 2025 | TBD | TBD |

### Typical Transcript Metrics

- P2 ("Google: The AI Company"): 1159 paragraphs, ~219,000 chars, ~4h audio
- Speaker balance is remarkably even (P1: Ben 399 lines vs David 395 lines)

## Structural Findings from Real Episodes

### P2 Episode Anatomy (Google: The AI Company)

1. **Cold open / banter** (~2 min) — informal, often about studio/equipment
2. **Opening monologue** (Ben) — hooks with a dilemma or provocative framing (Innovator's Dilemma analogy)
3. **Sponsor reads** — JP Morgan Payments (presenting partner)
4. **Historical narrative** — David drives with timeline-based storytelling, heavy book/source citations (In the Plex, Genius Makers, Supremacy)
5. **Guest insertions** — brief clips from interviews (e.g., Chris Cox: "No way.")
6. **Business analysis segments** — Ben drives: TPU economics, Waymo market sizing, token throughput data
7. **Bull/Bear analysis** — structured debate on company's future
8. **Seven Powers** (Hamilton Helmer) — which powers apply, scoped to AI specifically
9. **Quintessence** — each host's distilled takeaway
10. **Carve Outs** — personal recommendations (products, media, games)
11. **Thank yous / credits** — extensive list of research sources and interviewees

### Key Role Patterns

- **David**: Narrative historian. Tells chronological stories with specific dates, names, book citations. Uses "As we talked about in..." cross-references frequently.
- **Ben**: Business analyst + interpreter. Asks clarifying questions ("Oh, is this in one of Google's micro kitchens?"), makes analogies, drives financial analysis and strategic frameworks.
- **Both**: Drop "we spoke with [person] to prep for this episode" to establish authority and access.

### Humor and Chemistry Patterns

- **Jeff Dean Facts** — Chuck Norris-style jokes about Google engineer Jeff Dean
- **Running gags** — "No AI, thank you very much", "best business of all time"
- **Reaction patterns** — "Wow", "That's amazing", "Is that right?", "Oh my God"
- **Competitive banter** — Ben: "Did you hire a Hollywood scriptwriting consultant?" / David: "I love it"
- **Domino meme storytelling** — connecting causality chains across decades (Ilya leaves → OpenAI → ChatGPT → Google not broken up)

### Source Citation Patterns

David frequently names sources mid-narrative:
- "This is from In the Plex" / "This is in Cade Metz's book Genius Makers"
- "Parmy Olson's book Supremacy" / "Stephen Levy wrote in Wired"
- "We talked to [person] for this episode" — Sundar, Demis, Sebastian Thrun, Jeff Dean, etc.

This source-dropping is a signature Acquired style element that adds credibility and depth.
