# Flash talk — 16 October 2026, 11:00 AM

[Project home](../../../README.md) · [Repository index](../../../INDEX.md) · [Presentation history](../README.md)

Team **vismAI**. Three slides, 3–4 minutes. Vismay and Darshan present; all four members attend and answer questions on their own milestones.

## Presentation files

| File | Purpose |
|---|---|
| [vismAI.pptx](vismAI.pptx) | Editable presentation with three embedded clips: mixed recording and two separated voices, embedded fonts and speaker notes |
| [vismAI.pdf](vismAI.pdf) | Static backup; audio does not play in the PDF |
| [SCRIPT.md](SCRIPT.md) | Final verbatim phone script, balanced speaking and playback cues |
| [DATA_AND_SOURCES.md](DATA_AND_SOURCES.md) | Simple guide to the slide evidence, datasets and model sources |

## Study your own milestone

Each guide starts with ELI5, then covers the implementation, actual outputs, results, a board explanation, likely questions and a self-test. Nobody is assigned all four guides.

| Owner | Guide | Responsibility |
|---|---|---|
| Vismay | [M1_PREP_VISMAY.md](M1_PREP_VISMAY.md) | Separation; opening/audio; M3/M4 results narration |
| Darshan | [M2_PREP.md](M2_PREP.md) | Diarization; Order A/B result; closing |
| Abhinav | [M3_PREP_ABHINAV.md](M3_PREP_ABHINAV.md) | Transcription and Hinglish questions |
| Shrivaths | [M4_PREP_SHRIVATHS.md](M4_PREP_SHRIVATHS.md) | LLM report and guardrail questions |

Both presenters read the final script verbatim from their own phones. Vismay handles the opening/audio and M3/M4 output walkthrough. Darshan handles M2, the Order A/B result and the closing. Vismay keeps the clips in another app and rehearses returning to the marked RESUME paragraph.

Darshan should read M2 sections 1–5, practise the section-6 board explanation, rehearse the M2 questions, and then practise his speaking script with Vismay. Each other member follows the same structure in their own guide.

## What belongs in this package

This directory contains only the presentation, static backup, speaking script, four individual guides, one data-and-sources guide and this index. The guides already contain stage-specific Q&A and corrections to older material. Combined Q&A, duplicate start-here notes and the WhatsApp coordination draft are local conveniences, not maintained repository documents. Existing audio clips are linked below rather than duplicated. Build intermediates and local delivery copies stay out of Git.

## Phone-audio fallback: Vismay owns it

The original media are already tracked in the historical presentation directory. Reuse those files instead of maintaining duplicate copies:

1. [Mixed recording, about 18 seconds](../demo-video/whospoke_demo.mp4)
2. [Speaker A, about 7.5 seconds](../demo-video/speaker_a.mp4)
3. [Speaker B, about 7.5 seconds](../demo-video/speaker_b.mp4)

Vismay downloads and tests all three on his own phone before the talk, keeps them ready in mixed-then-A-then-B order, and plays them if the PC fails. Keep slide 1 visible. Do not replay a clip that already worked. Full checklist: [Vismay's guide, section 10](M1_PREP_VISMAY.md#10-your-job-phone-audio-backup).

## Evidence and scope

The deck uses saved repository outputs and results, not a fresh inference run. Its project evidence comes from commit `8465f0bb54b50024315b01fd167ee8bc0139e24e`; scheduling and presentation assignments follow the team's instructions.

| Slide | Evidence |
|---|---|
| 1: separation and timeline | [Saved timeline](../../../results/demo/timeline.json), [separation scores](../../../results/separation_test_summary.csv), [diarization configuration](../../../results/tuned_params.json), [clip generator](../demo-video/make_speaker_clips.py) |
| 2: transcript and comparison | [Saved transcript](../../../results/demo/transcript.json), [ASR comparison](../../../results/asr_comparison.csv), [full-pipeline results](../../../results/eval_test_indicconformer.csv) |
| 3: report and next steps | [Saved report](../../../results/demo/report.md), [report caveats](../../../results/demo/README.md), [Stage-4 evaluation](../../../results/eval_postprocess.csv), [previously proposed next steps](../demo-video/deck/slides/next.html) |

The prep guides correct overstatements in older explainers: library components are reused in clustering; selected B-spectral overlap-label assignment is disabled; the LLM's repair adds errors; no accepted action items remain. Future improvements are labelled as proposals.

The PPTX is the editable source for this edition. Older HTML sources and generators remain under [demo-video](../demo-video/) and [main](../main/); they build the historical editions, not this one. Local build intermediates and delivery copies are ignored by Git. No older presentation or project result was removed.
