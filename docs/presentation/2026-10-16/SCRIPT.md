[Package home](README.md) · [Presentation index](../README.md)

# Final phone-reading script

**16 October 2026, 11:00 AM. Read the spoken paragraphs verbatim.** Headings and bracketed **DO NOT READ ALOUD** directions are instructions only. Both presenters save this script offline on their own phones, enlarge the text and enable Do Not Disturb. Vismay keeps the clips ready in a separate app on his own phone and returns to the marked RESUME paragraph after playback.

**Split:** Vismay opens and demonstrates M1, then narrates M3 and M4. Darshan explains M2, presents the Order A/B result and closes. Only prepare your own milestone for technical Q&A. Aim for 3:40–3:55 including about 33 seconds of clips; confirm with one timed rehearsal.

## Slide 1 — Vismay: opening and separation

[DO NOT READ ALOUD: Start on slide 1.]

Good morning. We are team vismAI. Our project, whospoke, identifies who spoke what and when in noisy Hindi–English conversations.

This simulated test conversation combines real Hindi speech, overlapping voices and market noise.

I handled Milestone One, separation. First, the mixed recording. Then, the two separated voices from an overlapping stretch within it.

[DO NOT READ ALOUD: Play Mixed recording first, about 18 seconds. Then play Speaker A and Speaker B, about 7.5 seconds each. If PC audio fails, read the fallback sentence below, then play the downloaded clips from YOUR OWN phone. Keep slide 1 visible. Do not replay a clip that already worked. Return here after playback. If both playback methods fail, continue immediately.]

[DO NOT READ ALOUD: Say the following sentence ONLY if switching to phone audio.]

We'll play the same clips from the phone.

[DO NOT READ ALOUD: RESUME HERE after playback.]

Separation improves S-I-S-D-R by nine point six decibels across two hundred and forty-five test overlap regions. Darshan will explain the speaker timeline.

## Slide 1 — Darshan: your M2 output

I handled Milestone Two: who spoke when. This actual timeline labels Speaker A, then B, then A again, with start and end times.

We detect speech, extract short-window voice embeddings, and cluster similar embeddings into speaker groups. The labels are anonymous.

Our spectral configuration has twenty point three percent diarization error. The pyannote baseline gets eighteen point four; we do not beat it.

Our design decision was to diarize first, then separate detected overlaps. The next slide shows the difference.

[DO NOT READ ALOUD: Advance to slide 2. Vismay speaks next.]

## Slide 2 — Vismay: transcript output

Abhinav handled Milestone Three. These saved Hinglish lines retain speakers and timestamps. We also output Devanagari and subtitles.

For speech recognition alone, IndicConformer gets twenty-one percent word error on four hundred Vaani clips, compared with thirty-eight percent for IndicWav2Vec.

Darshan will explain the full-pipeline comparison on the right.

## Slide 2 — Darshan: the main result

We tested both orders on the same seventy-two simulated conversations, about eighty-three minutes across eight speaker groups.

Our metric, C-P-W-E-R, measures word errors and speaker attribution errors. Lower is better.

Separating first gives fifty-five point eight percent. Diarizing first gives forty-nine point five, and wins in fifty of the seventy-two conversations.

The order matters. We implemented both and chose using measured outputs. Errors remain substantial; the problem is not solved.

[DO NOT READ ALOUD: Advance to slide 3. Vismay speaks next.]

## Slide 3 — Vismay: report output and its limits

Shrivaths handled Milestone Four. Airavata produces summaries, keywords and translated dialogue. These saved excerpts concern a lost ID card and office hours.

Speaker labels and times stay unchanged. But transcript repair adds zero point nine C-P-W-E-R points on average across sixty-three conversations. No accepted action items remain in the two hundred and fifty-two evaluated reports.

So the original transcript remains the record, and the report needs checking.

## Slide 3 — Darshan: closing

Our next steps are to improve Hindi speaker embeddings, evaluate a larger language model for action items, and retrain separation on Indian voices and noise.

These are proposed experiments. Today we have shown every stage's output, the measured benefit of changing the order, and the remaining limitations. Thank you.

[DO NOT READ ALOUD: End. Vismay answers M1 questions, Darshan M2, Abhinav M3 and Shrivaths M4.]

## Before the talk — do not read aloud

- Download the latest PPT, PDF and this script. Save the script for offline phone reading; do not rely on mobile internet.
- Vismay independently downloads all three clips onto his own phone: [Mixed recording](../demo-video/whospoke_demo.mp4), [Speaker A](../demo-video/speaker_a.mp4), then [Speaker B](../demo-video/speaker_b.mp4). Test sound offline, disable unintended Bluetooth output, and rehearse returning to RESUME HERE. No coordination with Darshan is needed to obtain or play the files.
- Use all three embedded clips in the numbered order. The mixed clip lasts about 18 seconds; each separated clip lasts about 7.5 seconds.
- Practise the speaker switches once. They keep each person speaking in complete sections and let Darshan explain his own stage and the main result.
- Darshan studies M2 only. For the result comparison, understand that both orders used the same cases and that cpWER includes speaker attribution. The closing does not require learning the other implementations.
- Vismay studies M1 only for technical Q&A; the M3/M4 paragraphs above are the exact results narration. Abhinav and Shrivaths own deeper questions about those stages.
- The PDF has no playable audio. Vismay can play the same phone clips alongside it.

## Sources

All content comes from repository commit `8465f0bb54b50024315b01fd167ee8bc0139e24e` and the local explainers. The presentation date and Vismay spelling come from your instructions.

Slide 1: [results/demo/timeline.json](../../../results/demo/timeline.json), [results/separation_test_summary.csv](../../../results/separation_test_summary.csv), [results/eval_test_indicconformer.csv](../../../results/eval_test_indicconformer.csv), [docs/presentation/demo-video/make_speaker_clips.py](../demo-video/make_speaker_clips.py), [results/tuned_params.json](../../../results/tuned_params.json).

Slide 2: [results/demo/transcript.json](../../../results/demo/transcript.json), [results/asr_comparison.csv](../../../results/asr_comparison.csv), [results/eval_test_indicconformer.csv](../../../results/eval_test_indicconformer.csv), [docs/DATASETS.md](../../DATASETS.md).

Slide 3: [results/demo/report.md](../../../results/demo/report.md), [results/eval_postprocess.csv](../../../results/eval_postprocess.csv), [docs/RESULTS_TABLES.md](../../RESULTS_TABLES.md), [docs/presentation/demo-video/deck/slides/next.html](../demo-video/deck/slides/next.html).
