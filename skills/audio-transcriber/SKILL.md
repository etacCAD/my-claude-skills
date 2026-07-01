---
name: audio-transcriber
description: Enables the AI to review, process, and transcribe audio files using OpenAI Whisper. NOT for real-time speech recognition, live captioning, or transcribing audio directly from browser microphone.
---

# Audio Transcriber Skill

This skill allows the AI to "listen" to voice notes, meeting recordings, or other audio files by transcribing them locally.

## Workflow

When tasked to transcribe or review an audio file, follow these steps:

1. **Locate the File**: Ask the user for the absolute path if it is not provided.
2. **First-time Setup**: 
   - Check if `ffmpeg` is installed: `which ffmpeg`. If missing, install it: `brew install ffmpeg`.
   - Check if Python packages are installed: `python3 -c "import whisper"`. If missing, install: `pip install -U openai-whisper`.
3. **Run Transcription**:
   - Execute the helper script included in this skill's `scripts` directory.
   - Example Command: `python3 ~/.gemini/antigravity/skills/audio-transcriber/scripts/transcribe.py "/path/to/audio/file"`
4. **Review & Action**:
   - The script outputs the transcribed text to standard output. 
   - Provide the transcription to the user, or use it to act on their specific request (such as summarizing, extracting action items, or modifying a codebase based on dictated instructions).

## Common Anti-Patterns

### 1. Sending very large audio files without chunking
**Symptom**: Sending very large audio files without chunking
**Problem**: Whisper has a 25MB file limit. Large files sent without chunking will fail silently or error.
**Solution**: Split audio files exceeding 25MB into overlapping chunks before transcription to prevent data loss at boundaries.

### 2. Ignoring language specification
**Symptom**: Ignoring language specification
**Problem**: Omitting the language parameter causes Whisper to auto-detect, which adds latency and reduces accuracy for accented speech.
**Solution**: Always specify the target language explicitly when known to improve accuracy and speed.
