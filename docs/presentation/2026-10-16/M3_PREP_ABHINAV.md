[Package home](README.md) · [Presentation index](../README.md)

# Abhinav's Milestone 3 preparation

Study your own stage. Read sections 1–5 first, practise the board explanation, then rehearse the questions. Vismay narrates the slide-2 transcription results during the flash talk; you answer technical questions about M3. You do not need to study every other milestone.

## 1. Your job in one sentence

**“My stage recognises the words in each speaker turn and saves them in Devanagari and readable Hinglish, retaining the supplied speaker labels and times.”**

This is **automatic speech recognition (ASR)** followed by **romanisation**. ASR converts sound into text. Romanisation changes the writing system; it does not translate Hindi into English. English translation belongs to M4.

**Input:** audio segments associated with speaker turns. **Output:** text with speaker labels and times, saved as Devanagari text, Hinglish text, JSON and SRT subtitles.

An actual saved Hinglish line is:

`[00:32 - 00:33] Speaker_B: haan aur baki sun`

Source: [results/demo/transcript_hinglish.txt](../../../results/demo/transcript_hinglish.txt). It is recognised output, not a manually corrected reference.

## 2. ELI5: listen, write, then change the alphabet

Imagine listening to one person's turn and writing down what you hear. The speech recogniser does that in Devanagari. Then another component writes the same words using Latin letters so the output is readable as Hinglish.

The second component cannot recover a word the recogniser missed. It can restore a known English spelling from a dictionary or use phonetic rules for an unfamiliar Hindi word. Errors in speaker assignment also remain: the recogniser inherits the speaker label instead of identifying the person itself.

## 3. The actual path through your code

### A. Receive the appropriate audio

In Order B, non-overlap audio comes from the mixture. In overlapping portions the pipeline can splice in the separated voice matched to that speaker. Order A instead transcribes the separated track attached to the turn.

The pipeline splits long turns into pieces with a default maximum of 25 seconds, choosing a quiet region near the cut. This is separate from the ASR backend's own long-input handling. You only need the interface to M1/M2, not their clustering or separation mathematics.

### B. Run the selected recogniser

`asr_backends.py` provides wrappers for **AI4Bharat IndicConformer** and **IndicWav2Vec Hindi**. `pipeline.py:make_asr` selects IndicConformer with **RNNT decoding** by default.

The repository's explainer describes the Conformer encoder as combining convolution and self-attention, with CTC and RNNT decoding options. RNNT uses previous output tokens as part of prediction. The deployed wrapper uses a TorchScript preprocessing stage and ONNX inference components. These models are pretrained; the project does not train their neural weights.

### C. Handle input length and batching

The shared backend resamples to 16 kHz if needed, ignores clips below 0.1 seconds, splits long audio at low-energy points, and batches compatible inputs. The documented backend chunk limits are 30 seconds for IndicConformer and 20 seconds for IndicWav2Vec. Cutting at a quiet point reduces the chance of cutting through a word, but does not guarantee a perfect linguistic boundary.

For implementation questions, the repo includes ONNX/CUDA configuration choices and a targeted TorchScript-fusion workaround. These are inference engineering details, not evidence that the team trained a new recogniser.

### D. Romanise the Devanagari

`hinglish.py` combines a loanword lexicon, common-word overrides and Hindi phonetic rules. The lexicon maps known Devanagari forms of English loanwords back to English spellings. Remaining Hindi words go through character parsing, vowel/nasal handling and schwa-deletion rules.

**Schwa deletion** means omitting an inherent short “a” that is not pronounced in a word. The old explainer uses `कमरा → kamra` as an explanation. This is a linguistic example, not a measured accuracy claim.

The resulting Hinglish is a practical spelling convention, not a universally standard romanisation. Unknown loanwords may receive approximate phonetic spellings.

### E. Save the result

`pipeline.py:Result.save` writes `timeline.json`, `transcript.txt`, `transcript_hinglish.txt`, `transcript.srt` and `transcript.json`. `Line` retains the speaker, start/end, Devanagari text, Hinglish and audio-source tag.

Speaker timestamps describe segments supplied by the pipeline. They are not a separate word-by-word forced alignment performed by this stage.

## 4. The exact settings and evaluation scopes

| Item | Saved implementation or evaluation |
|---|---|
| Default recogniser | IndicConformer |
| Default decoding | RNNT |
| Alternative | IndicWav2Vec Hindi, CTC |
| Audio rate | 16 kHz |
| Main model comparison | 400 Vaani clips, 0.747 hours |
| Romanisation | Rules, overrides and roughly 2,600 loanword mappings |
| Lexicon data | Hand-made list plus Vaani train transcripts |
| Full-pipeline comparison | Separate 72-conversation simulated test |

The dataset is called “Vaani test” in the repo, but the team uses it to choose between the ASR candidates. Describe it as the **ASR model-comparison/selection set**, not an untouched final evaluation after model selection.

## 5. What the results mean

**Word error rate (WER)** = substitutions + deletions + insertions, divided by reference-word count. Lower is better. Insertions mean WER can exceed 100%; it is not ordinary classification accuracy.

| Model | Vaani WER | Vaani CER | English-word recall |
|---|---:|---:|---:|
| IndicConformer | 21.0% | 10.0% | 78.2% |
| IndicWav2Vec | 38.0% | 16.8% | 48.3% |

These are the saved results on 400 Vaani clips. CER counts character edits rather than word edits. The 17-percentage-point WER difference is not a 17% relative change.

**Keep two similar-looking figures separate:**

- **78.2%:** recognition recall of 775 tagged English words in the 400 Vaani audio clips.
- **82.0%:** romaniser spelling correctness on 8,630 tagged English words in the text benchmark.

Different denominators, different stages. The 82% is not the end-to-end probability of recognising and spelling an English word correctly.

The 49.5% cpWER on slide 2 belongs to the **full pipeline**, not this single-speaker ASR benchmark. cpWER compares speaker-specific word streams after finding the best speaker matching; incorrect attribution can therefore hurt it even when a word was heard correctly.

**Data limitation:** IndicConformer was pretrained on IndicVoices, which also supplies the synthetic-conversation speech. The repo therefore chooses the ASR on Vaani and explicitly warns that IndicVoices results can flatter it. Nirantar is an additional cross-check, not a fully independent collection from IndicVoices.

## 6. Your board explanation, in about 90 seconds

Write **speaker-labelled audio → IndicConformer → Devanagari → romaniser → Hinglish**.

Say: “The recogniser predicts words from audio. The romaniser changes their written form. Speaker labels and times come from the earlier pipeline.”

Under romaniser write **loanword dictionary + Hindi rules**. Use the saved example from section 1 to show the final format. Do not invent an input waveform or claim a spelling example is a benchmark result.

Write **WER = (S + D + I) / N** and explain each symbol. Add **21.0% vs 38.0%, 400 Vaani clips**. Finish: “This is ASR alone. The full pipeline also has noise and speaker-attribution errors.”

## 7. Likely questions and short answers

**Did you fine-tune IndicConformer?** No. The repo uses pretrained weights and implements the wrappers, processing, comparison and romanisation workflow.

**Why this model?** It has lower saved WER than IndicWav2Vec on the Vaani comparison. The selected decoder is RNNT. Do not claim all models or all languages were benchmarked.

**Why not just use the larger model's training-set results?** IndicVoices overlaps the pretrained model's source collection. The repo uses Vaani for selection to reduce that concern.

**Does Hinglish mean English translation?** No. It is Latin-script rendering of the Hindi/English words. M4 produces the English translation.

**Can romanisation fix ASR mistakes?** It can map a recognised loanword to a familiar spelling. It cannot know a missing or misheard word from sound because it only receives text.

**Where did the dictionary come from?** A hand-made list and mappings mined from Vaani train transcripts. The evaluation's test words were not used to construct those mined mappings.

**Why is the transcript not grammatically polished?** It is recogniser output, preserved as evidence. The displayed lines were not manually rewritten. Spontaneous speech and recognition errors can both make it awkward.

**Does it know who said the words?** It inherits the speaker label attached to the segment. It does not independently verify real identity.

**Do spelling variants affect WER?** Yes. The repo also checks merged spelling variants, and the model ranking stays the same. That does not make every alternative spelling automatically accepted by the main scorer.

**How fast is it?** The saved Vaani IndicConformer ASR RTF is about 0.054, meaning ASR processing time divided by audio duration. This is a measured backend benchmark, not the speed of all four stages or a guarantee on another machine.

**Does it support every Indian language?** The project's implemented/evaluated scope is Hindi and English. Do not infer project-level multilingual validation from the pretrained model's broader capabilities.

## 8. Read the code in this order

1. [src/whospoke/pipeline.py](../../../src/whospoke/pipeline.py): `Line`, `make_asr`, `_transcribe`, `Result.save`.
2. [src/whospoke/asr_backends.py](../../../src/whospoke/asr_backends.py): shared backend and both model wrappers.
3. [src/whospoke/hinglish.py](../../../src/whospoke/hinglish.py): lexicon lookup, word parsing and romanisation.
4. [scripts/mine_loanwords.py](../../../scripts/mine_loanwords.py) and [src/whospoke/resources/loanwords.tsv](../../../src/whospoke/resources/loanwords.tsv).
5. [results/asr_comparison.csv](../../../results/asr_comparison.csv), [docs/DATASETS.md](../../DATASETS.md), [docs/RESULTS_TABLES.md](../../RESULTS_TABLES.md).
6. [results/demo/transcript.json](../../../results/demo/transcript.json) and `transcript_hinglish.txt`.

The older `docs/explainers/stage3_transcription.md` (local-only background) is useful for architecture details. Keep its recognition and romanisation percentages separate, and do not describe quiet-point splitting as guaranteed word-boundary detection.

## 9. Final self-test

Explain ASR versus romanisation versus translation, the selected model/decoder, where speaker labels come from, WER, the 400-clip scope, the 775 versus 8,630 denominators, and the pretrained-data caveat. Practise answering each in two sentences. Leave detailed M1, M2 and M4 questions to their owners.
