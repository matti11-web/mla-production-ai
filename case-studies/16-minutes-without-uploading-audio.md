# Meeting Minutes Without Uploading the Audio

> **In business terms**
>
> - **Problem:** Meeting minutes went through a cloud route that uploaded the full recording for transcription. On one evening that upload stayed stuck for more than 15 minutes and produced nothing.
> - **What changed:** The recording is now transcribed on a local GPU in the office. Only the text goes to the cloud, where a language model writes the summary.
> - **Result:** A 30-minute recording takes about 4 minutes to transcribe, and the summary is ready about 30 seconds later. The audio never leaves the building.

Our internal Hub turns a meeting recording into a summary and files it in the document store. The first version sent the audio file to a cloud transcription service. I moved the transcription to a local machine and kept only the summary in the cloud. This is why, and what it changed.

**Status:** Cloud upload route stuck on 30 September 2026. Local route chosen the same night. Timings are approximate and were noted on 30 September 2026. GPU confirmed on 28 October 2026.

---

## The route that stalled

The original route was simple on paper. Upload the recording, wait for a cloud service to transcribe it, then summarise the text.

On 30 September 2026 the upload stayed in the "uploading" state for more than 15 minutes. The transcription service never returned a job number. Nothing failed loudly. The meeting tile just waited.

The audio file was the problem. A meeting recording is large. It has to travel over the office connection to a third party before any work starts. Every step in that chain is a place to wait or to break.

## Split the work where the data gets small

Transcription needs the audio. The summary needs only the text. The text of a 30-minute meeting is a small fraction of the size of the recording.

So the split is:

- **On a local machine:** transcribe the recording with Whisper (the open `large-v3` model, through the faster-whisper library) on the workstation GPU, an NVIDIA RTX 5090.
- **In the cloud:** send the finished transcript to the Hub. The Hub asks a Claude model for the summary and archives the result.

The Hub already had an import route that accepts a transcript instead of audio. The local script calls that route with a service key. No new cloud component was needed.

## What it took, as noted on 30 September 2026

| Step | Time |
|---|---|
| Local transcription of a 30-minute recording | about 4 minutes |
| Summary after the transcript arrives | about 30 seconds |
| The stalled cloud upload, same evening | more than 15 minutes, no result |

These are working notes, not a benchmark. I did not time many meetings or compare the two routes on the same file.

## Why the audio staying local matters

For a small company the speed is the visible gain. The quieter gain is that the recording stays in the building.

A recording holds more than the minutes do. It holds voices, side remarks, and everything said before and after the agenda. The transcript still goes to the cloud for the summary, so this is not a fully offline setup. But the raw audio, which is the most sensitive and the hardest to take back, never goes to a third party.

Some meetings also need a recorded consent flag before the Hub will process them, for example meetings about suppliers or about staff. The local route does not skip that check. The person who runs the import still sets it.

## Two small details that cost time

**The firewall rejected the script.** A plain Python HTTP request without a `User-Agent` header got a 403 from the edge firewall. Adding the header fixed it. The error did not say why.

**Consent is a question, not a default.** For meetings that need consent, the script does not assume it. The person who owns the recording confirms it each time.

## What generalises

- **Cut where the data gets small.** Audio is heavy, text is light. Do the heavy step close to the data and send only the light result.
- **A local GPU is now enough for transcription.** One workstation card handles a 30-minute meeting in a few minutes. For this job the cloud is not the faster option.
- **Keep the cloud for the part that needs a strong model.** The summary still uses a large language model. The transcription did not need one.
- **Less upload means less to leak.** The recording is the most sensitive artefact of a meeting. Keeping it local removes it from every third-party system.
- **Silent waits are failures.** The stuck upload showed no error. A route that can wait forever needs a time limit, or a different design.

---

## What's NOT in this case study

- Any meeting content, participant, customer or quote
- The name of the cloud transcription service, beyond that it was a third party
- Repository paths, endpoints, keys and code
- A timed comparison of both routes on the same recording
- Costs

---

**Cloud route stalled:** 30 September 2026. **Local route chosen:** 30 September 2026. **Timings noted:** 30 September 2026, approximate. **GPU confirmed:** 28 October 2026. **Last updated:** 28 October 2026.
