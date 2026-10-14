# Presentations — team vismAI

## Current: 16 October 2026, 11:00 AM

**Start with the [current presentation package](2026-10-16/README.md).** It contains the three-slide output-focused deck, PDF backup, timed script and separate preparation guides.

| Material | Link |
|---|---|
| PowerPoint with two embedded separation clips | [vismAI.pptx](2026-10-16/vismAI.pptx) |
| Static PDF backup | [vismAI.pdf](2026-10-16/vismAI.pdf) |
| Presenter script | [SCRIPT.md](2026-10-16/SCRIPT.md) |
| Vismay: M1 and phone-audio fallback | [M1_PREP_VISMAY.md](2026-10-16/M1_PREP_VISMAY.md) |
| Darshan: M2 | [M2_PREP.md](2026-10-16/M2_PREP.md) |
| Abhinav: M3 | [M3_PREP_ABHINAV.md](2026-10-16/M3_PREP_ABHINAV.md) |
| Shrivaths: M4 | [M4_PREP_SHRIVATHS.md](2026-10-16/M4_PREP_SHRIVATHS.md) |

Each person prepares their own milestone. Vismay presents the opening and M1; Darshan takes over for M2 and narrates the remaining results. Abhinav and Shrivaths handle technical questions on their stages. Vismay owns the phone-audio fallback using Darshan's phone.

## Historical archive: 6 October 2026

The material below is retained at its original paths so existing links continue to work. Its date, script and playback instructions are historical. Use the current package above for the upcoming talk.

Tuesday 6 October 2026, 4:00 PM, D-LT10. 3 to 4 minutes, 3 slides, 10 % of the course grade. Vismay presents slide 1
and Darshan slides 2–3. Abhinav and Shrivaths stand with the team; all four must be there.

**Script to read on the day: [demo-video/SCRIPT.md](demo-video/SCRIPT.md).**

| | **Audio version (submitted)** | First version (not submitted) |
|---|---|---|
| File | [demo-video/vismAI.pptx](demo-video/vismAI.pptx) (and a [PDF](demo-video/vismAI.pdf)) | — |
| Slide 1 | Pipeline with owners and tools; an 18 s clip of a test conversation with whospoke's timeline and transcript in sync; Speaker A and Speaker B separated from the overlap, each playable | Pipeline, owners, one real output line |
| Script | [demo-video/SCRIPT.md](demo-video/SCRIPT.md), about 3:55 | [main/SCRIPT.md](main/SCRIPT.md), about 3:25 |
| What every line means | [demo-video/EXPLAINER.md](demo-video/EXPLAINER.md): every slide, number and clip in the submitted deck | [main/EXPLAINER.md](main/EXPLAINER.md) (older wording) |
| Slides' source | [demo-video/deck/](demo-video/deck/) | [main/deck/](main/deck/) |

Who built what (the contributions statement the FAQ asks for):

| Milestone | Owner |
|---|---|
| 1 · Separation | Vismay |
| 2 · Diarization | Darshan |
| 3 · Transcription | Abhinav |
| 4 · LLM report | Shrivaths |

## The submitted file

`vismAI.pptx` is built by [demo-video/build_pptx.py](demo-video/build_pptx.py): each slide is a picture of the
published slide (so it looks the same on any PC, whatever fonts it has) with the script in the speaker notes, and
slide 1 has the three clips embedded. A click on a clip plays it with sound; a second click pauses it. Clicking
anywhere else moves to the next slide.

On slide 1, click the big clip first, then "Speaker A", then "Speaker B". If the hall has no sound, the big clip still
works silently (the transcript appears in sync); skip the two voice clips and carry on.

The slides follow Anthropic's visual style: ivory `#FAF9F5`, near-black `#141413`, clay `#D97757` as the only strong
accent, Poppins for headings and Lora for body text.
