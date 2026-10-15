# Where the data, slide outputs and numbers come from

[Package home](README.md) · [Full dataset reference](../../DATASETS.md) · [Repository index](../../../INDEX.md)

**The short answer:** the team uses pretrained models, builds controlled conversations from real Hindi speech and real background sounds, and evaluates against known labels. The PPT shows saved outputs and recorded results, not a new inference run.

This document summarises repository evidence inspected at commit `8465f0bb54b50024315b01fd167ee8bc0139e24e`. It does not independently audit the providers' entire training datasets.

## 1. The exact example in the PPT

The clips, timeline and transcript/report excerpts refer to **`test_g02_2spk_ovl-high_market5`**: test group 2, two speakers, high overlap and market noise at 5 dB signal-to-noise ratio. This is an assembled test conversation, not an actual meeting recording between those speakers.

| What you see or hear | Where it comes from |
|---|---|
| Mixed recording | The example's `mixture.wav`, cropped to **26.4–44.8 s**. The saved [video](../demo-video/whospoke_demo.mp4) also displays predicted timeline/transcript output. [Generator](../demo-video/make_demo_video.py) |
| Speaker A and Speaker B | Existing Conv-TasNet outputs from standalone full-recording separation, cropped to **28.0–35.5 s**. Predicted embeddings name the tracks; clean references are used only for a diagnostic comparison. Tracks are levelled for playback. [Generator](../demo-video/make_speaker_clips.py), [A](../demo-video/speaker_a.mp4), [B](../demo-video/speaker_b.mp4) |
| Slide 1 timeline | Actual predicted intervals from [timeline.json](../../../results/demo/timeline.json) |
| Slide 2 Hinglish | Exact saved lines from [transcript.json](../../../results/demo/transcript.json) and [Hinglish text](../../../results/demo/transcript_hinglish.txt) |
| Slide 3 report | Excerpts from [report.md](../../../results/demo/report.md). Its [README](../../../results/demo/README.md) also explains errors in the report |

The mixed clip is about 18 seconds. Each split clip is about 7.5 seconds from an overlapping stretch inside that excerpt. They are not equal-duration versions. The split clips demonstrate standalone separation; the chosen full pipeline separates only detected overlaps.

The demo README records that this example was selected near the test-set median cpWER. Its sensible report excerpts do not prove the entire report is correct.

## 2. The project's data sources

| Source | What the project uses it for |
|---|---|
| **AI4Bharat IndicVoices** | Hindi conversation speech from the valid split and train shard 0, used to construct dev/test conversations; additional ASR comparison |
| **IISc / ARTPARK Project Vaani** | Hindi test audio/transcripts for ASR comparison; train transcripts for loanword mappings |
| **DEMAND** | Channel 1 of eight environmental recordings at 16 kHz, providing continuous background sound |
| **ESC-50** | Seventeen categories of sound events such as animals, horns, engines and bells, added to the background |
| **AI4Bharat Nirantar Hindi sample** | Teammate's Colab extraction; 406 clips after excluding test speakers, used as an additional ASR cross-check |
| **AIR-RS-DB** | Not used. The repository records that it could not find a public source under the proposal's stated name |

Provider locations recorded in the repo: [IndicVoices](https://huggingface.co/datasets/ai4bharat/IndicVoices), [Vaani](https://huggingface.co/datasets/ARTPARK-IISc/Vaani-transcription-part), [DEMAND](https://zenodo.org/records/1227121), [ESC-50](https://github.com/karolpiczak/ESC-50). IndicVoices and Vaani require provider access/terms acceptance. [DATASETS.md](../../DATASETS.md) records exact subsets, licences and Nirantar provenance.

## 3. How the conversations and reference labels are made

The code selects two or three speakers, arranges utterances with controlled gaps/overlap, and adds noise. It knows which source was placed where, so it can save reference speakers, times, words and clean voice tracks.

| Set or condition | Details |
|---|---|
| Dev, for choosing settings | 4 groups × 9 conditions = **36 conversations**, 38.9 minutes |
| Main test | 8 groups × 9 conditions = **72 conversations**, 82.9 minutes |
| Nine conditions | Three overlap levels × clean, village at 10 dB SNR, market at 5 dB SNR |
| Stage 4 | **63 conversations**, groups 0–6. Group 7 was used for prompt development and excluded from scoring |

The 72 cases reuse eight speaker groups. They are not 72 independent groups of people. At most two speakers overlap simultaneously. The simulation does not comprehensively model room reverberation, phone codecs or natural turn-taking.

Dev/test speakers are separated. Noise uses different halves of DEMAND recordings and different ESC-50 folds. Decisions D25/D26 document fixes after an initial test run; its results remain saved. Do not claim the team never inspected test output.

Sources: [builder](../../../scripts/build_dataset.py), [simulation](../../../src/whospoke/synth.py), [noise](../../../src/whospoke/noise.py), [decisions](../../../DECISIONS.md).

## 4. Every headline number in the PPT

| Claim | Scope and saved evidence |
|---|---|
| **+9.6 dB** separation improvement | +9.62 dB across **245 test overlap regions**, noisy Conv-TasNet checkpoint. [Summary](../../../results/separation_test_summary.csv) |
| Spectral **20.3% DER**, GMM **20.3%**, pyannote **18.4%** | Main 72-conversation evaluation; lower is better. [CSV](../../../results/eval_test_indicconformer.csv), [tables](../../RESULTS_TABLES.md) |
| Timeline intervals | Predictions for one saved demo, not reference labels. [Timeline](../../../results/demo/timeline.json) |
| **21.0% vs 38.0% WER** | IndicConformer versus IndicWav2Vec on **400 Vaani clips**, 0.747 hours. Separate single-speaker ASR comparison/selection set. [CSV](../../../results/asr_comparison.csv) |
| **55.8% vs 49.5% cpWER**, B wins **50/72** | Pooled word/speaker-attribution errors, A-spectral versus B-spectral on the same 72 cases. [CSV](../../../results/eval_test_indicconformer.csv) |
| Repair adds **0.9 points** | Paired mean increase, about 0.94 unrounded, on **63** full-pipeline transcripts. Pooled rates are 51.8% and 52.8%, a different aggregation. [CSV](../../../results/eval_postprocess.csv) |
| No accepted actions in **252 reports** | Four input conditions × 63 conversations. Four unsupported proposals were rejected; zero accepted does not mean zero proposed. [CSV](../../../results/eval_postprocess.csv) |
| Next steps | Hindi speaker embeddings, a larger LLM and separation retraining were already proposed in the [previous deck](../demo-video/deck/slides/next.html). They are not completed improvements |

The new PPT uses editable text/tables for results. Other repo figures come from [make_report.py](../../../scripts/make_report.py) and live in `results/figures/`.

## 5. Where the models and Hinglish dictionary come from

| Component | Provenance recorded in the repo |
|---|---|
| Conv-TasNet | Pretrained `JorisCos/ConvTasNet_Libri2Mix_sepnoisy_16k`, noisy two-speaker English Libri2Mix |
| Speech/overlap detection | Pretrained `pyannote/segmentation-3.0`, through pyannote pipelines |
| Speaker embeddings | Pretrained `pyannote/wespeaker-voxceleb-resnet34-LM`, associated with VoxCeleb |
| Clustering | Custom workflows using NumPy, SciPy and scikit-learn, including library KMeans, PCA and GaussianMixture |
| IndicConformer | AI4Bharat pretrained ASR; the repo notes that IndicVoices is included in its training sources |
| IndicWav2Vec Hindi | AI4Bharat pretrained model, fine-tuned for Hindi by the provider |
| Airavata | AI4Bharat pretrained instruction model, locally quantised and served using llama.cpp |
| Hinglish lexicon | Hand-made entries plus mappings mined from Vaani **train** transcripts, used with romanisation rules |

The current repo trains no neural network. The team integrates and evaluates models and tunes settings. IndicConformer's connection to IndicVoices limits the independence of its IndicVoices-based results; Vaani is used for the ASR comparison.

Two prep-guide figures use different denominators: recognition of English words is measured on **775 words in 400 audio clips**, while romaniser spelling is measured on **8,630 tagged words**. They are not one end-to-end accuracy measure.

Sources: [pipeline reference](../../PIPELINE.md), [datasets](../../DATASETS.md), [LLM reference](../../MILESTONE4.md), [loanword mining](../../../scripts/mine_loanwords.py), [lexicon](../../../src/whospoke/resources/loanwords.tsv).

## 6. What is downloadable from GitHub

The deck, PDF, prep guides, code, saved metrics, demo transcripts/reports and presentation clips are tracked. All three clips are embedded inside the new PPT, so desktop PowerPoint does not need neighbouring media files. Download the PPT and use **PowerPoint Slide Show**, not GitHub's file preview. Audio output and media permissions still depend on the presentation computer.

Raw corpora, full synthetic WAV collections and model weights are not committed. Data location comes from `WHOSPOKE_DATA`, `data_location.txt` or the default `data` directory. [fetch_data.py](../../../scripts/fetch_data.py) and [build_dataset.py](../../../scripts/build_dataset.py) fetch/build the main data after access is configured. The Nirantar sample came from a teammate's extraction rather than those scripts.

For this presentation, download the PPT and phone clips. Nobody needs to download raw datasets or run models to present the saved results.
