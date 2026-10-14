[Package home](README.md) · [Presentation index](../README.md)

# Darshan's Milestone 2 preparation

Read sections 1–5 first. Practise section 6 aloud and on paper. Use the remaining sections for questions. This guide follows the code and saved configuration, rather than repeating every claim in the older explainer.

## 1. Your job in one sentence

**“My stage takes a recording and produces a timeline saying which anonymous speaker is active during each interval.”**

This is **speaker diarization**. It answers **who spoke when**. It does not recognise the words, separate the waveform, or identify someone's real name. Those are different tasks. Speaker_A and Speaker_B are names assigned to groups of similar voices within this recording.

**Input:** a 16 kHz mono audio array. **Main output:** turns containing start, end and speaker. **Other information:** speaker embedding centroids and detected overlap regions, which the next stage uses.

Actual saved output from [results/demo/timeline.json](../../../results/demo/timeline.json):

| Start in seconds | End in seconds | Predicted speaker |
|---:|---:|---|
| 26.85 | 31.75 | Speaker_A |
| 31.75 | 33.20 | Speaker_B |
| 33.20 | 40.20 | Speaker_A |

These are predicted labels, not ground truth. The JSON stores decimal seconds as well as rounded display strings. Use the decimal values to explain boundaries precisely. The first interval lasts 4.90 seconds, calculated as 31.75 minus 26.85.

## 2. ELI5: sorting voice snippets

Imagine listening to a conversation without seeing the people. First, mark where anyone is talking. Then listen to short snippets and group snippets that sound like the same person. Give each group a colour, then colour the recording's timeline.

The computer uses a pretrained network to turn each snippet into a list of numbers, called an **embedding**. Think of this as a voice description. Similar descriptions tend to belong together. The grouping step is **clustering**.

The analogy has limits: these fingerprints are imperfect, especially with noise, overlapping voices or very short speech. They are not unique IDs. A single person can split into several groups, or two people can merge into one.

## 3. The actual path through your code

### A. Locate speech and overlap

[src/whospoke/vad.py](../../../src/whospoke/vad.py) loads pretrained `pyannote/segmentation-3.0` through two pyannote pipelines: VoiceActivityDetection and OverlappedSpeechDetection.

- VAD marks intervals containing speech.
- Overlap detection marks intervals where multiple voices occur together.
- These decisions locate sound activity; they do not yet assign persistent speaker identities.

### B. Make speaker embeddings

`diarization.py:sliding_windows` covers speech with **1.5-second windows**, starting every **0.75 seconds**. Speech regions shorter than 0.4 seconds do not get an embedding window. Shorter eligible regions get one window; the final window covers the region's end.

`embed` uses pretrained **WeSpeaker ResNet-34** to produce voice embeddings (256 numbers per window, as documented in the repo). It zero-pads snippets to a common batch length and provides masks. It batches up to 32 snippets.

Why overlapping windows? They provide repeated local evidence as the voice changes over time. The 1.5/0.75-second settings are implementation choices, not a guarantee that every turn is that long.

### C. Group the embeddings

Both methods are in `clustering.py`. They estimate a speaker count unless one is supplied explicitly.

**Spectral workflow, the selected default:**

1. Normalise each embedding to unit length. Compare directions using cosine similarity.
2. Form an N-by-N similarity matrix for N windows.
3. Keep the strongest fraction of links in each row, clip negative links, symmetrise, then zero the diagonal.
4. Build the unnormalised graph Laplacian **L = D − A**. A is the graph's weighted adjacency matrix; D holds each row's total weight on its diagonal.
5. Sort the eigenvalues and choose the largest gap within the allowed count range. This proposes the number of speakers, k. A preliminary similarity test can instead decide that the recording appears to have one speaker.
6. Take the first k eigenvectors, normalise their rows, and run scikit-learn KMeans with 10 initialisations.

Intuition: windows from the same voice should form a strongly connected group. The eigenvectors describe that graph structure. The eigengap is a heuristic for the number of groups, not proof of the true number of speakers.

**GMM alternative:**

1. Normalise embeddings and reduce dimensions using scikit-learn PCA.
2. Fit scikit-learn GaussianMixture models with diagonal covariance for candidate speaker counts.
3. Select the model with the lowest BIC, which trades fit against complexity.
4. Assign each window to its predicted mixture component.

The selected B-GMM configuration uses **4 PCA dimensions**, not the class default of 8. Candidate k runs from 1 to min(8, floor(N/3)). If a count is supplied, the code uses that count.

**What you implemented:** the spectral graph construction, speaker-count selection workflow, configuration selection, cleanup, and assembly into a diarization pipeline. Numerical pieces come from NumPy/SciPy/scikit-learn; the speech and embedding networks are pretrained. Avoid “I trained the model” or “I wrote every algorithm from scratch.” The repository documents ownership, but only describe personal work you can honestly explain.

### D. Clean up groups

`consolidate` can absorb tiny clusters into larger ones and merge clusters with similar centroids. The selected B-spectral configuration uses **6%** for the small-cluster threshold and **0.6** for centroid cosine similarity. These are heuristics tuned on dev data. A small cluster can be a genuine brief speaker, so cleanup can also remove useful distinctions.

Clusters are relabelled by first appearance. Speaker_A refers to the first cluster encountered, not a known person's identity.

### E. Convert window labels into a timeline

Each **50 ms** frame gets votes from windows that cover it. A window's centre gets more weight than its edges. The highest-vote speaker wins on speech frames. The code smooths very short speaker runs, nominally below **0.3 seconds**, into a longer adjacent run, for up to three passes. Consecutive frames with the same label become a turn.

This is why the output has start/end times even though the embeddings used longer windows. A 50 ms output grid does not imply 50 ms measurement accuracy.

### F. The overlap distinction you must know

The class supports adding a second speaker during detected overlap by choosing another speaker nearby in time. **The tuned B-spectral configuration sets `overlap_aware=false`.** So that extra-label heuristic is disabled for the displayed headline result.

**Overlap detection still runs.** It stores regions in the diarization information. Order B uses those regions to decide where to separate audio, even without a second speaker label in the timeline. Separation cannot automatically fix every missing speaker turn. This is a real limitation.

Source: [results/tuned_params.json](../../../results/tuned_params.json), `diarization.py:Diarizer.__call__`, `assign`, and `pipeline.py:_overlap_regions`.

## 4. The exact selected settings

| Setting | B-spectral | B-GMM |
|---|---:|---:|
| Retained graph-link fraction | 0.2 | not applicable |
| PCA dimensions | not applicable | 4 |
| Merge similarity | 0.6 | 0.5 |
| Minimum cluster fraction | 0.06 | 0 |
| Add second-speaker labels | false | true |
| Dev DER | 16.28% | 18.44% |
| Test DER | 20.3% | 20.3% |

Shared windowing is 1.5 seconds with a 0.75-second hop. Do not confuse dev scores with test scores. The command-line path loads tuned settings; a bare `Diarizer()` constructor has generic defaults that differ.

## 5. What the results mean

**DER, diarization error rate:** missed speaker-time + false-alarm speaker-time + speaker-confusion time, divided by reference speaker-time. The implementation uses pyannote.metrics, a **0.25-second collar**, and **scores overlap**. The collar excludes boundary regions from scoring; it does not correct the predicted boundaries.

For B-spectral, the reported average components are 6.7% missed speech, 1.6% false alarm and 11.9% confusion. They sum to 20.2 after rounding; the unrounded aggregate rounds to **20.3%**. Do not force rounded parts to agree exactly.

| System | DER | Reported conversation-bootstrap 95% interval |
|---|---:|---|
| B-spectral | 20.3% | 17.3–23.6% |
| B-GMM | 20.3% | 17.2–23.4% |
| Full pyannote 3.1 | 18.4% | 15.5–21.4% |
| A-spectral | 26.2% | 23.4–29.1% |

Pyannote has the lower observed error. The repo's paired analysis does not establish a clear difference between ours and pyannote at this sample size. Say that, rather than “we match the industry standard.”

**cpWER:** concatenate each speaker's words in time order, find the best matching between predicted and reference speakers, then count word edits. A consistent A/B renaming is harmless because the matching resolves it. Mixing one person's words into another person's stream still causes errors. Misattribution can incur both a deletion and an insertion. It is not a simple percentage of correct timestamps or a classification accuracy.

Order A gives **55.8% pooled cpWER**. Order B gives **49.5%**. Their rounded pooled difference is 6.3 percentage points, approximately 11.4% relative reduction. The repo also reports **6.7 points** as the mean of paired per-conversation differences. That uses different weighting. If asked, explain the distinction instead of treating the numbers as inconsistent.

The main test consists of **72 condition variants from eight speaker groups**, not 72 independent groups of people. The repository also bootstraps whole groups. The A-minus-B mean cpWER difference remains positive: **+6.7 points, interval +3.2 to +10.4** with group resampling.

## 6. Your board explanation, in about 90 seconds

Write the following in five boxes, connecting them as you talk:

**Audio → speech regions → voice embeddings → clusters → speaker timeline**

Say: “We first find speech. We describe each short speech window using a pretrained embedding model. Similar voice embeddings are grouped. Then window labels vote on a time grid, giving speaker intervals.”

Under embeddings write **1.5 s window / 0.75 s hop**. Under clusters write **cosine similarity**, then **L = D − A**, **eigengap chooses k**, **KMeans labels windows**. Add: “These steps estimate speaker groups; they do not identify people by name.”

At the right, copy the real three-row table from section 1. Say: “This is the saved output, not a drawing of the reference labels.”

Below the chain add **overlap detector → targeted separation**. Say: “Our selected spectral setting disables the extra overlap-label heuristic, but overlap detection still routes audio to separation.”

Finish with **DER 20.3%, versus 26.2% when separating first**. If you draw a cluster sketch, label it **conceptual illustration**, not measured embeddings. You do not have a saved embedding scatter plot to present as evidence.

## 7. Likely questions and short answers

**Why diarize first?** In the saved comparison, the original mixture gives lower DER than the separated tracks: 20.3% versus 26.2%. Order B also has lower cpWER. The project attributes the difference to separation artifacts and speaker consistency issues. Present that as an interpretation supported by this experiment, not a universal rule.

**Why use spectral rather than GMM?** Both give 20.3% test DER after rounding. Spectral had lower dev DER, 16.28% versus 18.44%, and is the selected default. The test does not establish a decisive spectral advantage.

**How do you know k?** We estimate it: a single-speaker check and eigengap for spectral, or BIC for GMM. The optional speaker count overrides estimation. Speaker counting is a weakness: the repo reports collapse to one predicted speaker in 36% of the two-speaker cases.

**Why not just use pyannote?** We use pyannote's segmentation, but compare our own clustering/timeline pipeline with the complete pyannote 3.1 baseline. The custom workflow fulfils the clustering investigation in the proposal. The full baseline is numerically better; we do not hide it.

**Does this recognise Darshan or Vismay?** No. It assigns anonymous speaker groups inside the recording. It has no enrolled identities.

**Does a separated track always mean one named speaker?** No. Separation output channels can swap speakers. Order A jointly clusters embeddings from both separated tracks. Order B matches separated snippets to speaker centroids before using them for ASR.

**If two people overlap, do both appear in the selected timeline?** Not necessarily. The selected B-spectral extra-label heuristic is off. Detection still triggers separation, but missed turns can remain.

**Did you train a neural network?** No neural model is trained in this repo. We use pretrained models, implement/integrate the processing workflow, tune settings on dev data and evaluate the saved systems. Fine-tuning is future work.

**Is 20.3% DER the same as 79.7% transcription accuracy?** No. DER is an error over speaker-time. Transcription is measured with WER/cpWER, based on word edits.

**Why does the oracle timeline have 8.7% DER?** The repo distinguishes oracle utterance intervals from the speech-activity reference used for DER. Internal silence inside an utterance can count as false alarm. “Oracle” here supplies the turn boundaries/speakers for ASR; it does not make every DER frame perfect. See [docs/RESULTS.md](../../RESULTS.md) and the evaluation construction.

**Is the test fully independent?** Dev speakers are excluded from test, but condition variants reuse the same eight groups. IndicConformer was pretrained on IndicVoices, which is also the source of these conversations. Vaani is the separate ASR comparison set. Do not claim the full pipeline has been validated on arbitrary real meetings or broadcasts.

**Did you tune only once and never inspect test?** The repo keeps an initial run and documents post-test bug fixes in decisions D25/D26. Say that transparently if asked. Do not make a stronger “never looked at test” claim than the history supports.

**Where should we improve first?** Speaker embeddings/counting and confusion are priorities already proposed in the repo. The actual effect of any fine-tuning has not yet been measured.

## 8. Read the code in this order

1. [src/whospoke/diarization.py](../../../src/whospoke/diarization.py): `Turn`, `Diarization.timeline_json`, `Diarizer.analyse`, `assign`.
2. [src/whospoke/vad.py](../../../src/whospoke/vad.py): `_pipelines`, `speech_regions`, `overlap_regions`.
3. [src/whospoke/clustering.py](../../../src/whospoke/clustering.py): `affinity`, spectral `__call__`, `GMMClustering`, `consolidate`.
4. [results/tuned_params.json](../../../results/tuned_params.json): selected B-spectral and B-GMM settings.
5. [src/whospoke/pipeline.py](../../../src/whospoke/pipeline.py): `_overlap_regions`, `_targeted_separation`.
6. [src/whospoke/metrics.py](../../../src/whospoke/metrics.py): `der`, `cp_error`.

The older [docs/explainers/stage2_diarization.md](../../explainers/stage2_diarization.md) is useful background, but use this guide's corrections for library reuse, tuned thresholds and overlap handling.

## 9. Final self-test

Without looking, explain the input/output, embedding, cosine similarity, why k is estimated, the timeline vote, the overlap-setting distinction, DER versus cpWER, and why pyannote is a baseline rather than a component identical to the full custom system. Then deliver your slide-1 section in under 45 seconds.
