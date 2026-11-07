# Platform setup and compatibility

## Discover before configuring

1. Detect the actual execution host OS and CPU architecture, including whether it is WSL, a container, or a remote machine. A phone controlling a desktop agent is not the execution host.
2. Locate Node.js/npm, Python (try `python3`, `python`, or the Windows `py` launcher), installed speech engines and cached models. Check their versions and package engine requirements. Use a project virtual environment and runtime folder, not global replacements.
3. Select the smallest adequate model for available RAM and required language. CPU-only operation is a valid default; GPU/CUDA is optional and must match installed drivers and libraries. Do not download large models silently on constrained machines.
4. Verify the selected RTC SDK can import on the host before requesting room tokens. Check current upstream support rather than assuming all ARM or Windows variants work.

## Backend selection

- Apple Silicon macOS: observed end-to-end path is LiveKit Node RTC, MLX Whisper, and built-in `say`. Check installed voices.
- Intel macOS: evaluate whisper.cpp or faster-whisper CPU instead of MLX; `say` remains a TTS candidate. Verify RTC native binaries for x64.
- Linux x64: evaluate faster-whisper CPU int8 or whisper.cpp; eSpeak NG is a lightweight local TTS candidate. Verify the distribution, libc and RTC binary compatibility.
- Windows x64: evaluate faster-whisper CPU int8 or whisper.cpp and eSpeak NG. Use native Windows paths and subprocess argument arrays. If the required RTC package lacks a compatible build, use a supported Linux/WSL2 host, not an assumed native path. WSL must run its own dependencies; Windows executables and Linux virtual environments are not interchangeable.
- Linux ARM / small computers: evaluate whisper.cpp with a small or quantized model and eSpeak NG, subject to available builds and RAM. Measure latency; disable automatic replies when processing cannot keep up. Verify RTC support separately.
- Windows ARM, Android, iOS and browser-only devices: direct execution is not established by this skill. Use an explicitly chosen compatible desktop/server host and control it remotely. Do not promise a phone-only installation or upload audio to a remote service without approval.

These are setup candidates, not end-to-end tested platform claims. No GPU, paid speech service or OpenClaw installation is mandatory. If local speech is unavailable, report the missing component and retain only the requested supported mode; never silently substitute a cloud API.

## Audio adapters

Preserve receive format: mono 16 kHz PCM signed int16, converted to float32 in [-1,1] for backends accepting arrays. Keep speaker labels and timestamps outside the samples.

For faster-whisper, the official CPU example uses `WhisperModel(model_size, device="cpu", compute_type="int8")`. Consume the returned segment iterator; do not mistake its creation for completed inference. Use the installed version documentation for transcribe options.

For whisper.cpp, inspect its installed CLI help and use an appropriate model and input format; do not reuse Python flags. For eSpeak NG, inspect available voices and WAV-output options. Resample synthesized output with an available audio library or FFmpeg to mono 24 kHz signed 16-bit PCM. Check the actual resulting WAV channels, sample width and frame rate before sending frames.

Model generation is a separate adapter: use an already configured tool-free local model or authorized API. Keep credentials scoped to that adapter, not to speech subprocesses. Detect current CLI capabilities; do not assume Codex and Claude Code have interchangeable flags.

## Paths, installation and stop handling

Resolve paths relative to the skill directory or a user-selected runtime directory. Use platform home/temp APIs, quoted arguments and subprocess arrays; do not copy an author-specific username, absolute path, shell profile, or signer identity. On Windows, use the selected Python interpreter directly rather than relying on POSIX activation scripts.

Track process handles and child processes. Disconnect RTC and terminate this runtime's workers on leave on each supported OS; do not use blanket process-name kills. Close temporary audio files before deleting or reusing them, especially on Windows.

## Smoke tests on each new setup

1. Import the RTC SDK and STT backend; report exact selected versions and architecture.
2. Transcribe a short known local speech clip in the requested language; assess intelligibility and elapsed time. Silence must not trigger replies.
3. Synthesize a sentence and validate the normalized PCM output.
4. When room attendance is requested, verify connection, fresh remote transcription, and completed audio playout separately. Do not join a room just to install this skill.
5. Enable auto-reply only on request and after one fresh utterance completes STT → model → TTS → playout. Test stop/leave and confirm children exit.

## Primary documentation

- LiveKit Node SDK and package requirements: https://github.com/livekit/node-sdks
- faster-whisper installation, CPU/GPU requirements and examples: https://github.com/SYSTRAN/faster-whisper
- whisper.cpp supported platforms and builds: https://github.com/ggml-org/whisper.cpp
- eSpeak NG platforms and voice support: https://github.com/espeak-ng/espeak-ng

Consult these at setup time because supported binaries and dependencies change.
