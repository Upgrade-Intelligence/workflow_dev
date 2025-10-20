## Talkbook Universal Context

- Audience: age-agnostic. Some templates target children; others are general or adult-facing. Each template must enforce its own guardrails (reading level, tone, safety). Always avoid PII.
- Stages pipeline: stage1 (Educational Theme Integrator) → stage2 (Story Generator) → stage3 (Branching Orchestrator) → stage4 (Storyboarder).
- Interaction model (current batch): only two node types are supported:
  - Linear static narrative pages (Type: linear)
  - Choice pages (Type: choice_question with choice_outcome nodes)
  No other interaction types yet (inputs, tools, mini-games). Design prompts and outputs accordingly.
- Shareability defaults: content should be easy to share and consume out of context.
  - Open with a strong hook; keep each page self-contained and quotable.
  - Prefer concise, vivid copy; avoid insider jargon and region-specific references.
  - Visual briefs should stand alone (clear subject, setting, mood) and be social-friendly.
  - Keep language inclusive; no PII; avoid sensitive personal details.
- Safety: COPPA-safe when children-focused; gentle tone; no graphic detail; no PII.
- Narrative structure: setup → rising action → climax → warm resolution. Stage 3 preserves chronology; Stage 4 preserves IDs and routing.
- Visual guidance: concise briefs consistent with character_bible and settings; do not introduce new facts in Stage 4 imagery.
- Variables: use `{{inputs.*}}` for all user-supplied fields; stage inputs map from template inputs and stage outputs.
- TTS: narrator voice comes from `inputs.narrator_voice` when provided; keep voice consistent across pages.
- Template constants: templates may define `constants.word_band` and `constants.pagination`; Stage 2 and Stage 3 reference these constants rather than asking the user.
