---
name: h3-prompt-writing
description: Write MiniMax H3 video generation prompts for T2VA, I2VA, FL2VA, L2VA, and Ref2VA. Use when rewriting multimodal requests into H3 prompt structures, composing integrated_multimodal_description, overall_soundscape, and non_diegetic_music, aligning keyframes, or defining reference labels for images, videos, and audio.
compatibility: Portable to any agent that can read local files — no external API calls, MiniMax Hub tools, or proprietary runtime required. The agents/openai.yaml file only adds optional ChatGPT/Codex UI metadata; it does not restrict the skill to OpenAI agents.
---

# H3 Prompt Writing

## Workflow

1. Identify the input mode: T2VA, I2VA, FL2VA, L2VA, or full-reference Ref2VA.
2. For base text/keyframe modes, read `references/base-en.txt` and follow its final prompt structure.
3. For full-reference mode, read `references/ref-en.txt` and follow its six-section rewrite format.
4. Preserve the exact field names, section order, labels, and timing notation from the selected guide.

## Base Modes

- T2VA: build the full audiovisual timeline from text.
- I2VA: start from the first frame and develop forward from it.
- FL2VA: describe the continuous path between the first and last frames.
- L2VA: infer a plausible opening and converge to the supplied last frame.

Use `integrated_multimodal_description`, `overall_soundscape`, and `non_diegetic_music` in the order shown in `references/base-en.txt`.

## Full-Reference Mode

Ref2VA rewrites use `subject_definitions`, `summary`, `retention_analysis`, `detailed_description`, `overall_soundscape`, and `non_diegetic_music` in that order. Reference labels stay consistent across all sections.

Read `references/ref-en.txt` for label rules, retention analysis, and complete examples.

## Audio Timeline Alignment (Ref2VA with dialogue audio)

When the input includes dialogue/voice audio (`<Audio N>`), **transcribe the audio first, then set shot timestamps** — this is the most common failure point in Ref2VA-with-audio: speaker mismatch (character A is on screen speaking while the audio has already switched to character B).

- **Audio-first:** Run Whisper (`faster-whisper`, model `small`) to get per-utterance start/end timestamps and speaker IDs (S1/S2 when distinguishable). Merge consecutive utterances by the same speaker into "turns"; the gaps between turns are the natural cut points. Anchor each `[Shot N] At MM:SS.mmm` to these turn boundaries — do not guess blindly. Note: Whisper may mishear numbers/names (e.g. "心之星" → "星之星"); timestamps are reliable, use the user's original text for content.
- **Rough alignment, not tight:** Shot cut points should roughly match speaker-switch boundaries in the audio (±0.5–1s is fine), but do NOT bind frame-precisely. Cutting on every intra-sentence pause, or removing all timestamps and letting the model free-run, both make rhythm worse.
- **Hard rule — no speaker mismatch:** Never let a shot where character A is on screen speaking overlap an audio segment where character B is actually talking. Verify each `[Shot N]` time range matches the speaker (S1/S2) that is actually vocalizing in that range.

## Output Rules

- **Dialogue/lyrics MUST use the `<d>[Language] ... </d>` tag (iron rule).** Place the speaker's identifying phrase, ID, action, and delivery OUTSIDE `<d>`; inside `<d>` include only the language tag and the verbatim original spoken content — preserve every original word and punctuation mark, do NOT translate, rewrite, or append an English translation. Examples: `<d>[Chinese] 今天的星星，好像比昨天暗了一点点。</d>`, `<d>[English] I get off at the next station.</d>`. This is mandatory for both the script block and every shot description.
- **Audio-reuse mode (fully_copy): never re-synthesize.** When `<Audio N>` is reused as the final track, the audio is the sole voice source — describe only WHO speaks, WHEN, and that lip-sync follows the audio. Do NOT add tone / prosody / pause / emotion instructions (e.g. "voice trails off", "scholarly tone"), which trigger re-synthesis and cause garbled speech.
- Write rewrite sections in English; preserve dialogue, lyrics, and visible scene text in their original language.
- Describe each shot by composition, subjects, environment, actions, camera, sound, and the exact point where referenced content appears.
- Avoid plot summaries, unresolved reference labels, and timing that does not match the requested duration.
