[Package home](README.md) · [Presentation index](../README.md)

# Presentation preparation

1. Open [vismAI.pptx](vismAI.pptx). There are exactly three slides. Use PowerPoint Slide Show for the two embedded audio clips on slide 1.
2. Read [SCRIPT.md](SCRIPT.md), then rehearse together with a timer. Vismay handles the opening and separation. Darshan takes over on slide 1.
3. Darshan: read [M2_PREP.md](M2_PREP.md) first. It starts from zero and covers the exact selected configuration, a board explanation and examiner questions.
4. Study your own milestone: Darshan studies [M2_PREP.md](M2_PREP.md), Vismay [M1_PREP_VISMAY.md](M1_PREP_VISMAY.md), Abhinav [M3_PREP_ABHINAV.md](M3_PREP_ABHINAV.md), and Shrivaths [M4_PREP_SHRIVATHS.md](M4_PREP_SHRIVATHS.md). Each guide follows the same ELI5, implementation, results, board explanation and Q&A format. Darshan does not need to study all stages for Q&A. [TEAM_QA.md](TEAM_QA.md) remains a shared reference.
5. Vismay owns the audio fallback using Darshan's phone. Download and test the two separation clips from the [linked audio files](README.md#phone-audio-fallback-vismay-owns-it), keep them ready in A-then-B order, and follow section 10 of [M1_PREP_VISMAY.md](M1_PREP_VISMAY.md).
6. Keep [vismAI.pdf](vismAI.pdf) as a static backup. The PDF has no playable audio; Vismay can play the clips from Darshan's phone alongside it.

The slides show saved repository outputs, not a newly executed inference run. No new model training or benchmark result is claimed. The original presentation is unchanged.

## Important corrections to older material

- “All four milestones complete” is too broad if it implies transcript correction and action-item extraction succeeded. All four stages produce outputs, but Stage 4's requested improvements remain unachieved.
- “Both clusterers from scratch” is inaccurate at the numerical implementation level. The repository builds custom clustering workflows using SciPy and scikit-learn, including library KMeans, PCA and GaussianMixture.
- Selected B-spectral settings: `min_cluster_frac=0.06`, `overlap_aware=false`. Old explainers mention generic defaults of 0.03 and overlap assignment enabled. Overlap detection still runs for separation.
- Pyannote's DER is numerically better. Overlapping uncertainty intervals do not establish equal performance.
- The 9.6 dB separation result is over overlap regions; the 4.5 dB result in old material uses whole recordings. Do not describe their ratio as “twice as good.”
- Stage 4 uses 63 conversations, while the main pipeline uses 72. The 0.9-point change is the paired mean; pooled cpWER changes from 51.8% to 52.8%. These are different summaries of the same subset.

These corrections are reflected in the new script and preparation notes. Read those before the older explainers.
