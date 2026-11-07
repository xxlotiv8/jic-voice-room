# Agent voice choice

The agent picks its own speaking voice. Daniel and Tingting are only examples that worked on one Mac, not defaults.

## Choose

1. List voices from the backend already selected in platform setup.
   - macOS `say`: run `say -v '?'` and read the name and language columns.
   - eSpeak NG: list installed voices from its own help, then test the chosen name.
   - Another local engine: use that engine's voice list. Do not invent names.
2. Prefer a voice whose language matches the reply. Keep one voice per language if the agent switches languages.
3. Save the choice in the room runtime, for example `voice-choice.json` with engine, voice name, language, and rate. Reuse it after reconnecting to the same room. Do not store it in the skill files or in git.
4. An owner instruction such as "use voice X" overrides the saved choice when X is installed. If X is missing, say so and keep the current voice.
5. The first spoken introduction names the voice and says it is synthetic. Later lines do not repeat that label.

## Speak

Pass the saved voice name to the synthesizer. On macOS the observed form is:

`say -v <chosen-voice> -r <rate> -o reply.wav --data-format=LEI16@24000 --file-format=WAVE -f reply.txt`

Rate stays with the voice choice. 175 was understandable for Daniel; another voice may need a different rate. Confirm the WAV is mono, 16-bit, 24000 Hz before publishing, then name the LiveKit track with the chosen voice, such as `synthetic <voice name>`.

If synthesis fails, list voices again and pick once more. Do not fall back to a hardcoded name that is not installed.
