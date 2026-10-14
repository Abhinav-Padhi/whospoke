[Package home](README.md) · [Presentation index](../README.md)

# Vismay's Milestone 1 preparation

Study your own stage. Read sections 1–5 first, practise the board explanation, then use the questions for rehearsal. You present the opening and M1 on slide 1. Darshan takes over for M2. You also own the phone-audio fallback in section 10.

## 1. Your job in one sentence

**“My stage takes mixed speech and estimates two separate audio tracks, so overlapping voices can be heard and transcribed more clearly.”**

This is **speech separation**. Its output is audio, not words, timestamps or real identities. M2 handles speaker timing; M3 recognises words. You only need those boundaries to explain how your stage connects.

**Input:** mono audio sampled at 16 kHz. **Output:** two waveforms of the same length, represented as an array with shape `(2, T)`. The standalone command writes `separated_track_1.wav` and `separated_track_2.wav`.

The clips on slide 1 are existing saved outputs, each about 7.5 seconds long, from 28.0–35.5 seconds of the demo conversation.

## 2. ELI5: pulling apart two voices

Imagine two people speaking into one microphone. Their sounds are added together. The separator estimates which parts belong to each voice and produces two recordings.

It cannot simply choose different microphones: there is only one mixed channel. It uses patterns learned by a pretrained neural network. The result can still contain leakage from the other speaker, noise and distortion. Say “estimated separated voices,” not “perfectly clean voices.”

Two engineering problems matter: the network can give outputs inconsistent volume, and the order of its two output channels can swap between chunks. The repository corrects scale and aligns consecutive chunks.

## 3. The actual path through your code

### A. Load a pretrained separator

[src/whospoke/separation.py](../../../src/whospoke/separation.py) loads the Asteroid checkpoint `JorisCos/ConvTasNet_Libri2Mix_sepnoisy_16k`. It was trained on noisy two-speaker Libri2Mix mixtures. The project does not train Conv-TasNet from scratch.

The repository's architecture explainer describes a learned waveform encoder, a temporal convolutional mask estimator and a decoder. The masks operate on learned features, not directly as a simple frequency filter. The decoder reconstructs a waveform for each estimated voice.

### B. Process manageable windows

The wrapper defaults to **8-second windows** with a **6-second hop**, so adjacent windows share 2 seconds. Shorter audio is processed in one window. Eight seconds is this wrapper's processing choice, not a claim that the architecture cannot accept anything longer.

### C. Fit output gains

The raw output can have arbitrary scale and sign. `_separate_window` solves for two gains so that:

`g0 × estimated_voice_0 + g1 × estimated_voice_1 ≈ input_mixture`

This is a regularised least-squares calculation. The small diagonal regulariser helps prevent extreme amplification of a nearly silent output. It does not prove either voice is correct or completely remove background noise.

### D. Align channels and blend windows

`separate` compares both possible channel orderings against the previous output over the shared region. It chooses the ordering with greater summed cosine similarity, then blends the windows with linear fades and normalises by accumulated weights.

This handles the **permutation problem**: output channel 0 is not automatically the same person in every window. Channel alignment helps maintain continuity, but it is not real-name identification.

### E. Use separation selectively in Order B

The full pipeline first obtains diarization and overlap regions. `_targeted_separation` separates the overlap regions with **0.5 seconds of context**, matches the estimated voice to the turn's speaker centroid, and uses that voice in the overlap portion. Outside overlap it retains the original mixture. The default policy is `splice`, rather than replacing the whole speaker turn.

You do not need to explain the eigengap or M2 internals. Your interface is: “M2 supplies the speaker information and overlap regions; we use these to choose where and which voice to separate.”

## 4. The exact settings and demo distinction

| Item | Repository implementation |
|---|---|
| Selected checkpoint | Conv-TasNet Libri2Mix `sepnoisy`, 16 kHz |
| Output count | Two audio streams |
| Wrapper window / hop | 8 seconds / 6 seconds |
| Shared region for stitching | 2 seconds |
| Pipeline order | B: diarize first, separate detected overlaps |
| Pipeline replacement policy | Splice separated overlap audio into the mixture |
| Extra context around overlap | 0.5 seconds |
| Slide clips | Existing full-recording separation outputs cropped to 28.0–35.5 s |

**Do not conflate the clips with the headline evaluation.** The clip generator runs standalone separation over the whole demo recording, then crops and names the tracks by similarity to predicted speaker embeddings. Those clips demonstrate real separation output. The headline overlap-region metric and selected pipeline use a different processing scope.

The generator uses clean reference voices only to print a diagnostic comparison, not to choose the speaker labels. Source: [docs/presentation/demo-video/make_speaker_clips.py](../demo-video/make_speaker_clips.py).

## 5. What the results mean

**SI-SDR** compares an estimated voice with a clean reference after accounting for scale. **SI-SDR improvement** subtracts the score of the original mixture. Higher is better. Decibels are logarithmic; the score is not a percentage or a volume increase.

| Separator | Dev overlap SI-SDR improvement | Test overlap SI-SDR improvement |
|---|---:|---:|
| Conv-TasNet noisy checkpoint | +6.43 dB | +9.62 dB |
| Conv-TasNet clean checkpoint | +5.42 dB | +7.27 dB |
| SepFormer | +0.33 dB | +2.23 dB |

The **+9.6 dB** slide figure rounds +9.62 across **245 test overlap regions**. Model selection used dev results. The whole-recording figure, +4.45 dB, is across 24 recordings and has a different scope. Do not claim “twice as good” by dividing these decibel figures.

The repo also has a matched comparison over 23 two-speaker conversations: targeting overlap beats whole-recording separation by +5.62 dB, with a 95% interval of +3.81 to +7.55, winning 22 of 23.

For downstream words, with the **true speaker timeline**, heavy-overlap cpWER falls from 35.5% on the mixture to 27.6% with separated overlaps. That is an oracle experiment. With the actual diarizer, pooled cpWER is 50.0% without separation and 49.5% with targeted separation. Use the correct scope when quoting either.

## 6. Your board explanation, in about 90 seconds

Write **mixed waveform → pretrained Conv-TasNet → two estimated waveforms**.

Say: “The input contains both voices. The pretrained network predicts two source estimates. We adjust their gains so their sum approximates the input.” Write the two-gain equation from section 3.

Below it draw two overlapping windows labelled **0–8 s** and **6–14 s**. Label this a conceptual illustration of the wrapper, not measured demo output. Point at the shared two seconds and say: “We compare normal and swapped output order, choose the more consistent match, and crossfade.”

Finish with: “In Order B we apply this only around detected overlap. The measured separation gain is +9.6 dB on 245 test overlap regions.” Do not draw a fabricated waveform and call it an actual result; use the saved clips for evidence.

## 7. Likely questions and short answers

**Did you train this network?** No. We use pretrained checkpoints, engineer the wrapper and integration, compare choices on dev data, and evaluate the results.

**Why Conv-TasNet rather than SepFormer?** Its saved dev overlap score was higher. Test results support the same ranking. A proposed explanation involving training-domain mismatch is a hypothesis, not a separately proven cause.

**Why not Demucs?** The repo's decision D8 notes that the public checkpoints considered were aimed at music stems rather than separating individual speaking voices.

**Does it handle three people?** The conversation can contain three people taking turns, but the separator has two outputs and the test setup allows at most two simultaneous speakers. Three-way simultaneous overlap is not validated here.

**How do you know which output is Speaker A?** Channel index is insufficient. The pipeline matches separated snippets to speaker embedding centroids. The demo clip generator also uses predicted speaker embeddings for naming.

**Why separate only overlaps?** It avoids unnecessarily modifying non-overlapped speech. The saved comparisons favour the targeted policy in this setup. It is not a universal proof for every audio domain.

**Does a better waveform score guarantee better ASR?** No. SI-SDR and word errors measure different things; the repo evaluates both.

**What if overlap detection misses a region?** Separation may never run there. That is an upstream limitation of the integrated pipeline.

**Are the demo clips ground-truth clean sources?** No. They are saved separator outputs. They are also individually levelled for playback by the generator, so listening volume is not the separation metric.

**What is your next step?** The existing presentation proposes retraining separation on Indian voices and noise. No measured improvement from that retraining exists yet.

## 8. Read the code in this order

1. [src/whospoke/separation.py](../../../src/whospoke/separation.py): checkpoint map, `_separate_window`, `separate`.
2. [src/whospoke/pipeline.py](../../../src/whospoke/pipeline.py): `_overlap_regions`, `_targeted_separation`.
3. [docs/presentation/demo-video/make_speaker_clips.py](../demo-video/make_speaker_clips.py): `tracks_by_speaker`, crop bounds and playback normalisation.
4. [results/separation_dev_summary.csv](../../../results/separation_dev_summary.csv), [results/separation_test_summary.csv](../../../results/separation_test_summary.csv).
5. [docs/RESULTS_TABLES.md](../../RESULTS_TABLES.md): matched comparison and oracle transcript experiments.

The older [docs/explainers/stage1_separation.md](../../explainers/stage1_separation.md) is useful background. Correct its loose wording: the output is not guaranteed clean, eight-second processing is a wrapper setting, and separation is not universally proven to worsen all non-overlap audio.

## 9. Final self-test

Explain audio output versus text output, pretrained versus trained by the team, gain fitting, channel swapping, overlap-only processing, and what +9.6 dB means. Then practise your script and the handoff to Darshan. You do not need to study the other members' technical sections.

## 10. Your job: phone-audio backup

**You own playback and the fallback using Darshan's phone.** Arrange access before the talk.

1. Download [Speaker A clip](../demo-video/speaker_a.mp4) and [Speaker B clip](../demo-video/speaker_b.mp4) onto Darshan's phone and test them without internet.
2. Keep them ready in A-then-B order. Each is about 7.5 seconds.
3. Check media volume, disconnect unintended Bluetooth audio and enable Do Not Disturb.
4. Rehearse switching from the PC to the phone. If a microphone is available, test a suitable phone-speaker distance and volume.

On slide 1, try the embedded clips first. If playback fails, say **“We'll play the same clips from the phone.”** Keep the slide visible, play A then B, and continue. Do not replay a clip that already succeeded. If the phone also fails, stop troubleshooting and continue with the visible outputs.

The older `whospoke_demo.mp4` is optional backup material, not part of the timed script. Your standard opening, including the two clips, is in `SCRIPT.md`.
