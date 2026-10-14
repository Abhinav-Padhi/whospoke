[Package home](README.md) · [Presentation index](../README.md)

# Team preparation and likely questions

The repo identifies Vismay with M1, Darshan with M2, Abhinav with M3 and Shrivaths with M4. Vismay and Darshan deliver the talk. The other two should be ready for questions. These answers describe the saved implementation and evaluations, not new experiments.

## Vismay: Milestone 1, separation

### Your job: phone-audio backup

**Vismay, you own audio playback and the fallback using Darshan's phone.** Arrange this with Darshan before the talk; do not leave it for him to troubleshoot while presenting.

Before the presentation:

1. Save [Speaker A clip](../demo-video/speaker_a.mp4) and [Speaker B clip](../demo-video/speaker_b.mp4) locally on Darshan's phone. Confirm both play with sound without internet.
2. Keep the files ready in playback order: Speaker A, then Speaker B. Each lasts about 7.5 seconds.
3. Check media volume, disconnect unintended Bluetooth audio, and enable Do Not Disturb. Agree with Darshan how you will access the phone during your opening.
4. Rehearse switching to the phone once. If a microphone is available, test holding the phone speaker near it at a suitable volume.

During slide 1, try the embedded clips first. If the PC has no sound or playback fails, say **“We'll play the same clips from the phone.”** Keep slide 1 visible and play Speaker A, then Speaker B, from Darshan's phone. Do not replay a clip that already played successfully. Then continue your script and hand over to Darshan.

If the phone also fails, skip further troubleshooting and continue using the visible outputs. The older `whospoke_demo.mp4` is optional backup material; it is not part of the timed script.

### ELI5

“Two voices are mixed into one recording. The separator estimates two audio tracks so we can listen to each voice more clearly.”

Separation produces **audio**, not words or speaker names. Diarization assigns speakers to time intervals. ASR turns speech into words.

### Explain the implementation

The selected model is a **pretrained Conv-TasNet checkpoint trained on noisy Libri2Mix at 16 kHz**. The repo uses windowing, gain fitting and speaker-consistent stitching. The project compares checkpoints on dev data; it does not train a new separator.

The standalone separation command processes a recording and saves one WAV per output voice. The two existing video clips on slide 1 were made from that standalone full-recording separation, cropped to **28.0–35.5 seconds**, and matched to predicted speaker embeddings. The clip generator uses reference voices only for a diagnostic printout, not for naming the outputs.

The selected **Order B pipeline** does something more targeted: detect speakers and overlap first, separate overlap regions with context, select the appropriate voice using speaker centroids, and splice the separated region into the audio used for ASR. Outside those regions it retains the mixture. Therefore, the clips demonstrate real separation output, but are not recordings of the exact overlap-only processing used for the headline evaluation.

Sources: [src/whospoke/separation.py](../../../src/whospoke/separation.py), `pipeline.py:_targeted_separation`, [docs/presentation/demo-video/make_speaker_clips.py](../demo-video/make_speaker_clips.py).

### The number on your slide

**+9.62 dB SI-SDR improvement**, rounded to +9.6, over **245 test overlap regions**. SI-SDR compares the estimated waveform with a reference voice after accounting for scale; improvement means the output score minus the mixture score. Higher is better. It is not a word accuracy, a volume increase or “96% separation.”

The whole-recording result is +4.45 dB across 24 recordings. Those are different scopes and counts. Do not divide the two decibel values and claim “twice as good.” The repo provides a matched 23-conversation analysis: overlap-targeted separation improves by +5.62 dB relative to full-recording separation, with interval +3.81 to +7.55, winning 22 of 23.

### Questions to practise

**Can it separate three simultaneous speakers?** This separator outputs two voices. The simulated conversations may contain three people, but at most two speak simultaneously. Three-way simultaneous overlap is outside the tested setup.

**How do you stop speaker swaps?** The separation code aligns/stitches outputs. The pipeline also uses voice embeddings to match separated audio to speaker centroids. Output channel index alone is not a persistent identity.

**Did you train on Hindi?** The selected separator is pretrained on English Libri2Mix mixtures. Indian voices and noise are a proposed future training direction, not completed work.

**Does better SI-SDR always improve text?** No. Waveform quality and recognition error are distinct. The repo measures downstream cpWER as well. With the full diarizer, targeted separation changes pooled cpWER from 50.0% to 49.5%; with the true timeline, heavy-overlap cpWER changes from 35.5% to 27.6%. The latter is an oracle experiment, not the actual full pipeline.

**Why no clean input waveform on the slide?** The presentation uses saved separation outputs to keep the three-slide talk focused. The original noisy demo clip remains in the media folder if needed outside the timed talk.

## Abhinav: Milestone 3, transcription

### ELI5 and output

“For each speaker turn, ASR predicts the words in Devanagari. The romaniser then renders that text in readable Latin-script Hinglish. We retain the speaker and timestamps from the preceding pipeline.”

Slide 2 uses exact lines from [results/demo/transcript.json](../../../results/demo/transcript.json). Files include Devanagari text, Hinglish text, JSON and SRT subtitles. Hinglish is romanisation, not translation into English. The LLM's English translation belongs to M4.

### Model and comparison

The selected ASR is AI4Bharat IndicConformer with RNNT decoding. The alternative is IndicWav2Vec. On **400 Vaani test clips, about 0.747 hours**, the saved WERs are **20.984% and 38.004%**, shown as 21.0% and 38.0%. These are single-speaker ASR results, not end-to-end results on the 72 mixed conversations.

WER is substitutions + deletions + insertions, divided by the number of reference words. Lower is better. Normalisation handles punctuation/case, but spelling choices still affect scoring. The repo's spelling-variant check preserves the model ranking.

The main pipeline conversations use IndicVoices speech, and IndicConformer was pretrained on IndicVoices. That can flatter ASR performance. Vaani is the repository's separate model-selection comparison. Do not claim that the pretrained model has never seen the source collection used for the pipeline test.

### Questions to practise

**Why retain English words?** Hindi speech contains English loanwords. The romaniser uses rules plus a lexicon mined from Vaani train transcripts and a hand-made list. Unknown words can receive approximate phonetic spellings.

**What do 78% and 82% mean?** 78.2% is English-word recall for 775 tagged English words in the 400 audio clips. 82.0% is correct romaniser spelling on 8,630 tagged words from the text benchmark. They use different denominators and test different things. Neither is the end-to-end English accuracy of the pipeline.

**Are the displayed words corrected?** Slide 2 copies Stage-3 Hinglish output, including its imperfect wording. It is not a manually polished transcript.

**Is WER the same as cpWER?** WER scores words. cpWER compares whole speaker-specific word streams after the best speaker matching, so it also penalises misattribution. Consistent relabelling does not count as an error.

**Why is 49.5% still high?** The full system combines noise/overlap, transcription errors and speaker-assignment errors. An output demonstration shows that the system runs; it does not establish reliable transcription in every case.

Sources: `asr_backends.py`, `hinglish.py`, [results/asr_comparison.csv](../../../results/asr_comparison.csv), [docs/DATASETS.md](../../DATASETS.md), [docs/RESULTS_TABLES.md](../../RESULTS_TABLES.md).

## Shrivaths: Milestone 4, report

### ELI5 and output

“A local language model reads the transcript and prepares a report. It does not listen to the original audio and cannot recover words the recogniser never captured.”

The model is **Airavata 7B at 4-bit precision, served through llama.cpp**. It produces a summary, topic, key points, keywords and translated dialogue. The code retains the Stage-3 transcript separately.

The slide quotes [results/demo/report.md](../../../results/demo/report.md) exactly: “Speaker A lost his ID card” and “The office is open until what time?” These are selected report excerpts, not evidence that the entire report is correct. The demo report also contains incorrect “e-pass” and “base card” interpretations, which its README explicitly discusses.

### Guardrails and limits

The model's response format does not let it rewrite speaker identities or timestamps. The processing code copies those through. It checks/reverts over-edited lines, preserves line alignment, and filters unsupported keywords/actions. These checks reduce some failure modes but do not guarantee semantic truth. A keyword may reuse words in the transcript with the wrong meaning.

The evaluation scores **63 conversations**, seven speaker groups. Group 7 was used for prompt development and excluded. Four input conditions per conversation give **252 reports**. Full-pipeline repair raises cpWER by **0.9399 points as a paired mean**, rounded to +0.9. Pooled error on that subset changes from **51.8% to 52.8%**. Do not add 0.9 to the separate 72-conversation score of 49.5%.

No accepted action items remain. The repo records four proposed actions rejected as unsupported. Thus “all 252 reports have no accepted actions” is accurate; “the model never proposed anything” is not.

### Questions to practise

**Why keep Stage 4 if repair makes things worse?** The proposal asks for a report, and this implementation produces one. Evaluation shows its limits. Keep Stage 3 as the record and treat the report as a reading aid requiring checking.

**Can the model hallucinate?** Yes. Fixed speaker/time fields do not ensure a correct translation or summary. The demo documents mistranslations. Do not say the guardrails eliminate hallucination.

**Why quantise?** The repo's hardware is a 6 GB laptop GPU. Quantisation makes the 7B model fit for a separate Stage-4 pass. It is not evidence of a measured accuracy improvement. Stages 1–3 and the LLM do not all fit together there.

**What does “local” mean?** Inference runs via a local llama.cpp server. Initial model download is separate. The supplied report is saved output; this talk does not rely on a running server.

**Will a larger LLM solve it?** That is a proposed experiment. No larger-model improvement has been measured in this repo.

Sources: `llm_postprocess.py`, [docs/MILESTONE4.md](../../MILESTONE4.md), [results/demo/README.md](../../../results/demo/README.md), [results/eval_postprocess.csv](../../../results/eval_postprocess.csv).

## Everyone: experiment design and contribution questions

**What data did you evaluate?** Simulated conversations constructed from real IndicVoices Hindi speech and real DEMAND/ESC-50 noise. Test has eight groups, each under three overlap levels and three noise conditions, giving 72 conversations and 82.9 minutes. Dev has 36 conversations from different speakers. Market uses 5 dB SNR; village uses 10 dB. This does not model all real-room acoustics, codecs or natural turn-taking.

**What is novel or yours?** The integration, custom diarization/clustering workflow, overlap-targeted ordering, comparison of both orders, evaluation and reporting. The neural models are pretrained. Do not claim to have invented Conv-TasNet, spectral clustering, GMM or IndicConformer.

**What is actually completed?** Saved separation clips, speaker timelines, speaker-attributed transcripts/subtitles and LLM reports, with evaluation results. Improved LLM correction and action extraction are not achieved. Hindi-specific fine-tuning and a larger LLM are future work.

**What changed from the proposal?** Order B runs M2 before M1 and separates detected overlaps, while Order A implements the proposal's separation-first ordering. Both are measured on the same test cases. See `DEVIATIONS.md` for dataset and model substitutions. There are four milestones in the proposal, not five.

**How should we answer an unknown question?** State the boundary: “I have not measured that. In our saved test, the result was X; to establish your claim we would need Y.” Do not substitute a plausible explanation for evidence.
