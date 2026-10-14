[Package home](README.md) · [Presentation index](../README.md)

# whospoke: flash-talk script

**Friday, 16 October 2026, 11:00 AM. Team vismAI.** Three slides, two presenters. Aim for about 3:40–3:55 including the two 7.5-second clips. Rehearse aloud; timing depends on your delivery. The bracketed directions are cues, not spoken words.

## Slide 1: outputs from separation and diarization

### Vismay: 0:00–1:00, including 15 seconds of audio

Good morning. We are team vismAI. Our project, whospoke, identifies who spoke what and when in noisy Hindi–English conversations.

We will show the actual saved outputs. This example combines real Hindi speech with market noise and overlapping voices. It is a simulated test conversation.

I handled Milestone 1, separation. These are two output clips from Conv-TasNet, taken from the same overlapping stretch.

[Vismay: click Speaker A, then Speaker B, about 7.5 seconds each. If PC playback fails, say: “We'll play the same clips from the phone.” Keep slide 1 visible and play the downloaded clips from Darshan's phone, A then B. You own this fallback. Do not replay a clip that already worked. If the phone also fails, continue with the visible outputs.]

Across 245 test overlap regions, the separator improves SI-SDR by 9.6 decibels. That measures separation quality, not transcription accuracy. Darshan will show the speaker timeline.

### Darshan: 1:00–1:45

I handled Milestone 2: who spoke when. The table is directly from our timeline output. It assigns Speaker A from 26.85 to 31.75 seconds, then Speaker B, then Speaker A again.

We detect speech, extract short-window voice embeddings, and cluster similar embeddings into speaker groups. The output is anonymous speaker labels with start and end times.

Our spectral configuration has 20.3 percent diarization error. GMM also gets 20.3, while the complete pyannote baseline gets 18.4. So we do not claim to beat that baseline.

In our chosen order, we diarize the original recording first, then separate the detected overlaps.

[Next slide.]

## Slide 2: actual transcription output

### Darshan: 1:45–2:40

Abhinav handled Milestone 3. These lines are copied from the saved Hinglish transcript, with the predicted speakers and timestamps. The system also saves Devanagari text, subtitles and JSON.

[Point to the first line, then the Speaker B reply. Do not read all three lines.]

For speech recognition alone, IndicConformer gets 21 percent word error on 400 Vaani clips, compared with 38 percent for IndicWav2Vec.

The full-pipeline comparison is a different test: 72 simulated conversations, about 83 minutes, across eight speaker groups. cpWER measures both word errors and speaker attribution errors.

Separating first gives 55.8 percent cpWER. Diarizing first gives 49.5, and wins in 50 of the 72 conversations. Errors remain substantial, especially in assigning speech to the right speaker.

[Next slide.]

## Slide 3: report output and next steps

### Darshan: 2:40–3:40

Shrivaths handled Milestone 4. Airavata produces a report with a summary, keywords and translated dialogue. On the left are actual excerpts: the lost ID card, and the question about office hours.

The report keeps speaker labels and timestamps unchanged. But producing a report does not mean the report is fully reliable. On 63 evaluated conversations, transcript repair adds 0.9 cpWER points on average. No accepted action items remain in the 252 evaluated reports.

So the Stage-3 transcript remains the record, and the report is a reading aid that needs checking.

Our proposed next steps are to improve Hindi speaker embeddings, evaluate a larger LLM for action items, and retrain separation on Indian voices and noise. These are future experiments, not completed improvements. Thank you.

## Rehearsal and fallback

- Vismay presents M1, then hands over on slide 1. Darshan presents M2 and narrates the M3/M4 results. Abhinav and Shrivaths answer questions about their stages.
- Play only the two embedded separation clips during the talk. Their combined duration is 15 seconds. The older 18-second main clip is optional practice material, not part of this script.
- Vismay owns the phone fallback: prepare and test both downloaded clips on Darshan's phone before the talk, keep them ready in A-then-B order, and handle playback if the PC fails. See your section in [TEAM_QA.md](TEAM_QA.md) for the checklist.
- If you exceed four minutes, omit the sentence comparing GMM and pyannote, and shorten the output-format list to “text and subtitles.” Keep the result limitations.
- The PDF is a static backup: audio does not play in it.
- Say “error,” not “accuracy.” Say “simulated conversations made from real speech,” not “72 real conversations.”
- Do not claim the team trained the pretrained models. Do not say the LLM fixes the transcript.
- The current repository defines four milestones. There is no fifth milestone in this deck.

## Sources

All content comes from repository commit `8465f0bb54b50024315b01fd167ee8bc0139e24e` and the local explainers. The presentation date and Vismay spelling come from your instructions.

Slide 1: [results/demo/timeline.json](../../../results/demo/timeline.json), [results/separation_test_summary.csv](../../../results/separation_test_summary.csv), [results/eval_test_indicconformer.csv](../../../results/eval_test_indicconformer.csv), [docs/presentation/demo-video/make_speaker_clips.py](../demo-video/make_speaker_clips.py), [results/tuned_params.json](../../../results/tuned_params.json).

Slide 2: [results/demo/transcript.json](../../../results/demo/transcript.json), [results/asr_comparison.csv](../../../results/asr_comparison.csv), [results/eval_test_indicconformer.csv](../../../results/eval_test_indicconformer.csv), [docs/DATASETS.md](../../DATASETS.md).

Slide 3: [results/demo/report.md](../../../results/demo/report.md), [results/eval_postprocess.csv](../../../results/eval_postprocess.csv), [docs/RESULTS_TABLES.md](../../RESULTS_TABLES.md), [docs/presentation/demo-video/deck/slides/next.html](../demo-video/deck/slides/next.html).
