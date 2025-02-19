
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


Dynamic Routing:
```python
try:
    if API_KEYS["ANTHROPIC_API_KEY"]:
        return await self._query_claude(prompt, task_type)
    return await self._query_openai(prompt, task_type)
except Exception as e:
    console.print(f"[yellow]Fallback triggered: {e}[/]")
    return await self._query_openai(prompt, task_type)
```

Asynchronous Execution:
Ensures that AI calls do not block other operations using asyncio.gather.
Learning Content Generator Module (content_generator.py)

Class: LearningContentGenerator

Core Methods:
_create_study_guide(): Generates detailed Markdown guides.
_generate_quiz(): Produces adaptive quizzes in JSON format.
_create_flashcards(): Outputs Anki-compatible flashcards.
_create_exercises(): Designs programming exercises with sample solutions.

Integration:
Directly uses analysis insights and transcript excerpts to create educational content.

## Installation & Execution ##

Installation
Dependencies:
Ensure Python 3.8+ and FFmpeg are installed.
Install Required Packages:

```
pip install yt-dlp openai anthropic assemblyai pydub rich
```
#### Execution

Run the Script:
```
python yt_mastery.py "https://youtube.com/watch?v=YOUR_VIDEO_ID" --format markdown
```

Do not forget to change the URL to your video.

Pipeline Flow:


Download & Process Audio:
Uses yt_dlp for video download and FFmpeg for audio extraction.

Transcription:
Calls AssemblyAI, with a fallback to Whisper.


Analysis:
Executes multiple AI-powered analysis tasks concurrently.

Content Generation:
Produces study guides, quizzes, flashcards, and exercises.

Output Formatting:
Results are output in Markdown or JSON based on the --format parameter.


## Configuration & API Management ##

#### Centralized Configuration

CONFIG Dictionary:
Holds all critical settings (token limits, model IDs, audio settings).

```
CONFIG = {
    "MAX_TOKENS": {"summary": 1000, "quiz": 1500, "outline": 2000, "analysis": 3000},
    "MODELS": {"transcription": "whisper-1", "analysis": "claude-3-opus-20240229", "generation": "gpt-4-turbo-2024-04-09"},
    "AUDIO": {"bitrate": "320k", "format": "mp3", "sample_rate": 44100}
}
```

API Key Handling

#### Environment Variables:

API keys are loaded at runtime from environment variables:

```
API_KEYS = {
    "OPENAI_API_KEY": os.getenv("OPENAI_API_KEY"),
    "ANTHROPIC_API_KEY": os.getenv("ANTHROPIC_API_KEY"),
    "ASSEMBLYAI_API_KEY": os.getenv("ASSEMBLYAI_API_KEY")
}
```

