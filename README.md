# YouTube_Learning_Assistant.py

## ## YouTube Mastery Assistant ## ##

The **YouTube Mastery Assistant** is a production-grade, asynchronous tool designed to convert YouTube video content into structured, AI-powered educational materials. This repository is aimed at developers and engineers who need a highly technical solution with detailed error handling, multi-model integration, and robust audio processing.

## ## Table of Contents ## ##
- [Project Structure & File Layout](#-project-structure--file-layout-)
- [In-Depth Technical Details](#-in-depth-technical-details-)
- [Core Modules & Code Architecture](#-core-modules--code-architecture-)
- [Installation & Execution](#-installation--execution-)
- [Configuration & API Management](#-configuration--api-management-)

## ## Project Structure & File Layout ## ##
```bash
YouTube_Learning_Assistant.py/
├── YouTube_Learning_Assistant.py  # Main script orchestrating the pipeline
├── modules/
│   ├── audio_processor.py  # Audio optimization & segmentation (using pydub)
│   ├── video_analyzer.py   # Transcript analysis & dynamic AI routing
│   └── content_generator.py# Learning material generation (study guides, quizzes, etc.)
└── requirements.txt        # Required Python packages
```

## In-Depth Technical Details ##

#### Asynchronous Concurrency

Implementation:

Uses Python’s asyncio to run I/O-bound tasks concurrently.

Key Methods:

asyncio.gather: Aggregates multiple AI queries.
asyncio.run: Boots the main event loop.

Example:

```python
insights = await asyncio.gather(
    self._generate_summary(),
    self._extract_key_concepts(),
    self._identify_learning_objectives(),
    self._assess_complexity()
)
```

#### Multi-Model AI Integration

Transcription Engines:
Primary: AssemblyAI via aai.Transcriber()

Fallback: OpenAI’s Whisper using openai.Audio.atranscribe()

Content Analysis Models:
Primary: Anthropic’s Claude 3 (via client.messages.create())

Fallback: OpenAI’s GPT-4 Turbo (via openai.ChatCompletion.acreate())

Unified AI Query Interface:
Uses _query_ai(prompt, task_type) to wrap calls and dynamically route requests based on API key availability and error status.


#### High-Quality Audio Processing

Audio Optimization:
Converts audio to MP3 at 320k bitrate, 44100 Hz, mono-channel.
Utilizes pydub.AudioSegment for sample rate conversion and channel mixing.

```python
audio = AudioSegment.from_file(input_path)
audio = audio.set_frame_rate(CONFIG["AUDIO"]["sample_rate"])
audio = audio.set_channels(1)
audio.export(output_path, format=CONFIG["AUDIO"]["format"], bitrate=CONFIG["AUDIO"]["bitrate"])
```

Dynamic AI Model Routing:
If a primary AI call (e.g., Anthropic's Claude 3) fails, the script automatically routes to OpenAI’s GPT-4 Turbo.

## Core Modules & Code Architecture ##

Audio Module (audio_processor.py)

Functions:
optimize_audio(input_path, output_path):
Optimizes audio with consistent parameters.

split_audio(file_path, segment_length):
Segments long audio files into smaller chunks.
Key Libraries: pydub, ffmpeg (via FFmpeg postprocessors in yt-dlp).

Transcription & Video Analyzer Module (video_analyzer.py)
Class: VideoAnalyzer

Core Methods:
analyze_content(): Executes parallel tasks (summary, key concepts, learning objectives, complexity analysis).
_query_ai(prompt, task_type): Unified AI interface for model selection and fallback.
