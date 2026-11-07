# Verified macOS audio pipeline

This reference records the Apple Silicon implementation. For other devices, select adapters using [platform setup](platform-setup.md); retain the PCM contracts and verification gates.

## Local components

Check `python3` for `mlx_whisper` and the Hugging Face model cache before downloading anything. On Apple Silicon, the cached model `mlx-community/whisper-large-v3-turbo` worked.

Install the observed RTC package in a dedicated runtime folder when absent:
`npm install --prefix <runtime-dir> @livekit/rtc-node@1.1.0 --no-audit --no-fund`.

## Receive

Register `RoomEvent.TrackSubscribed` before connecting. Its arguments are track, publication, participant. In the observed SDK, audio track kind is 1; check the installed types when upgrading.

Construct `new AudioStream(track, {sampleRate:16000,numChannels:1})`. Iterate frames; copy the underlying bytes before buffering because frame memory may be reused. Eight-second chunks worked for automatic replies; fifteen-second chunks worked for initial listening but add latency.

Send PCM to one persistent Python worker with base64, timestamp and speaker label. Decode to signed int16, convert to float32, divide by 32768, and call:
`mlx_whisper.transcribe(samples, path_or_hf_repo='mlx-community/whisper-large-v3-turbo', condition_on_previous_text=False, verbose=False)`.

Whisper may print language-detection lines even with verbose=False. Parse only valid JSON transcript lines, using a buffer across stdout chunks. Local audio need not be retained. Use a bounded transcript log for debugging; do not place peer speech in durable personal memory by default.

## Synthesize

Use the voice selected in [voice choice](voice-choice.md). The observed macOS form is:
`say -v <chosen-voice> -r <chosen-rate> -o reply.wav --data-format=LEI16@24000 --file-format=WAVE -f reply.txt`.

Daniel at rate 175 and Tingting are examples from one Mac, not required voices. Write WAV to a real file, not stdout. Read it with Python's wave module and assert mono, 16-bit, 24000 Hz before streaming.

## Publish

Import AudioSource, AudioFrame, LocalAudioTrack, TrackPublishOptions and TrackSource from the installed RTC SDK.

1. Create `new AudioSource(24000,1)`.
2. Create `LocalAudioTrack.createAudioTrack('synthetic <chosen voice>',source)`.
3. Set `new TrackPublishOptions().source = TrackSource.SOURCE_MICROPHONE`.
4. Publish using `room.localParticipant.publishTrack(track,options)`.
5. Split signed PCM into 480-sample frames (20 ms). Use `new AudioFrame(int16Array,24000,1,int16Array.length)` and await `source.captureFrame(frame)`.
6. Await `source.waitForPlayout()`.
7. For one-shot speech, call `unpublishTrack(publication.sid)` and close the source. For continuous synthetic voice mode, retain them for subsequent replies.

An open synthetic track is not an automatic conversation loop. Implement and verify STT → model → TTS → publication separately.

## Process lifetime

Keep one room connection and one transcription worker. Reconnecting with the same identity can displace the previous connection; explicitly check and clean up the previous process and child worker instead of relying on displacement. Keep credentials in memory, never in status output. Use ESM imports with top-level await. Preserve stop/leave control through the stored process handle.
