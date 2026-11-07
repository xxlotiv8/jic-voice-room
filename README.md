# JIC Voice Room

A portable **Codex / Claude Code skill**, not a standalone voice application. Join JIC rooms quietly, transcribe remote audio, speak once, or enable automatic spoken replies when requested.

## Install

Download `jic-voice-room.zip`, extract it, and copy its `jic-voice-room` folder into your agent's skills directory:

- Codex: `~/.codex/skills/` (or your configured Codex skills directory).
- Claude Code: `~/.claude/skills/`, or `.claude/skills/` inside a project.
- On Windows, `~` means your user home directory, not a literal folder name.

Keep all Markdown files together. Ask your agent to use `jic-voice-room` with the room link and desired mode. The agent checks dependencies and identity before joining. No credentials, models, room transcripts or binaries are included.

## Device compatibility

Includes setup-selection guidance for Apple Silicon / Intel macOS, Windows, Linux, CPU-only and constrained ARM hosts. Read [platform setup](platform-setup.md) for alternatives and verification requirements. Phones and unsupported native hosts can control a compatible remote host; this is not a mobile app.

**Tested end-to-end:** Apple Silicon macOS with local MLX Whisper and macOS speech synthesis. **Not yet tested end-to-end:** Windows, Linux, Intel Mac and ARM alternatives. The Markdown skill is portable; actual audio support depends on compatible installed engines and the RTC SDK.

## Files

- [SKILL.md](SKILL.md): agent procedure and verification gates.
- [platform-setup.md](platform-setup.md): platform selection, fallbacks and smoke tests.
- [voice-choice.md](voice-choice.md): how an agent selects, saves, and overrides its own voice.
- [audio-pipeline.md](audio-pipeline.md): observed macOS PCM and LiveKit implementation.

OpenClaw is optional. The skill does not capture your physical microphone by default. Automatic responses require explicit activation; leaving must stop the listener and workers.
