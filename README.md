
# YouTube_Learning_Assistant.py

## ## YouTube Mastery Assistant ## ##

The **YouTube Mastery Assistant** is a asynchronous tool designed to convert YouTube video content into structured, AI-powered educational materials. This repository is aimed at developers and engineers who need a highly technical solution with detailed error handling, multi-model integration, and robust audio processing.

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



## Introduction and Purpose

This project automates the conversion of a YouTube video into educational materials. The system handles:

- Audio extraction from a YouTube link
- Audio normalization (bitrate, sample rate)
- Transcription through multiple engines (primary + fallback)
- Parallel analysis of the transcription
- Generation of study guides, quizzes, and related resources

The code is designed for maintainability, testing, and future expansion.

## Overall Architecture

The pipeline is broken down into these stages:

1. **Download**: Retrieves the audio from a given YouTube link using `yt_dlp`.
2. **Process Audio**: Normalizes and segments audio files using `pydub`.
3. **Transcription**: Sends audio segments to AssemblyAI, with OpenAI’s Whisper as a fallback.
4. **Analysis**: Runs parallel AI tasks (summary, key concepts, learning objectives, complexity rating) via Anthropic or OpenAI models.
5. **Content Generation**: Creates learning materials (study guide, quiz, flashcards) based on the analysis.

Each stage is built as a separate module or class, making it easier to swap out functionality or add new capabilities.

# Text-Based Diagrams

Below are ASCII diagrams that illustrate the internal architecture and data flow of the YouTube Mastery Assistant.

---

## Pipeline Flow Diagram
      +-------------------+
      |   YouTube URL     |
      +-------------------+
               |
               v
      +-------------------+
      |    yt_dlp        |  <-- Download video & extract audio
      +-------------------+
               |
               v
      +-------------------+
      |   Raw Audio File  |
      +-------------------+
               |
               v
      +-----------------------------+
      |       AudioProcessor        |  
      |-----------------------------|
      | - optimize_audio()          |  <-- Normalize (frame rate, channels, bitrate)
      | - split_audio()             |  <-- Segment into chunks
      +-----------------------------+
               |
               v
      +-----------------------------+
      | Processed Audio Segments    |
      +-----------------------------+
               |
               v
      +-----------------------------+
      |       Transcription         |  
      |-----------------------------|
      | Primary: AssemblyAI         |  <-- Fast transcription
      | Fallback: OpenAI Whisper    |  <-- For error recovery
      +-----------------------------+
               |
               v
      +-----------------------------+
      |       Transcript            |
      +-----------------------------+
               |
               v
      +-----------------------------+
      |       VideoAnalyzer         |
      |-----------------------------|
      | - _generate_summary()       |  
      | - _extract_key_concepts()   |  
      | - _identify_learning_objs() |  
      | - _assess_complexity()      |  
      +-----------------------------+
               |
               v
      +-----------------------------+
      |    Analysis Insights        |
      +-----------------------------+
               |
               v
      +-----------------------------+
      | LearningContentGenerator    |
      |-----------------------------|
      | - _create_study_guide()     |  
      | - _generate_quiz()          |  
      | - _create_flashcards()      |  
      | - _create_exercises()       |  
      +-----------------------------+
               |
               v
      +-----------------------------+
      |   Formatted Output          |
      | (Markdown / JSON Format)    |
      +-----------------------------+

## In-Depth Technical Details ##

### Asynchronous Concurrency

Implementation:

Uses Python’s asyncio to run I/O-bound tasks concurrently.

Example:

```python
insights = await asyncio.gather(
    self._generate_summary(),
    self._extract_key_concepts(),
    self._identify_learning_objectives(),
    self._assess_complexity()
)
```

### Multi-Model AI Integration

Transcription Engines:
Primary: AssemblyAI via aai.Transcriber()

Fallback: OpenAI’s Whisper using openai.Audio.atranscribe()

Content Analysis Models:
Primary: Anthropic’s Claude 3 (via client.messages.create())

Fallback: OpenAI’s GPT-4 Turbo (via openai.ChatCompletion.acreate())

Unified AI Query Interface:
Uses _query_ai(prompt, task_type) to wrap calls and dynamically route requests based on API key availability and error status.


### High-Quality Audio Processing

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

### 1. Audio Module (audio_processor.py)

- **Implementation Details**:

  - Convert downloaded audio to a uniform format (MP3, 320k, 44100 Hz).
  - Split the audio into fixed-length chunks (default 10 minutes).
  - Uses [pydub](https://github.com/jiaaro/pydub) for file handling.
  - Ensures consistent frame rate, mono channel, and bitrate.
  - Allows for easy extension if additional audio processing steps (noise reduction, volume leveling) are required.

### 2. `VideoAnalyzer`

- **Purpose**:  
  - Perform multiple analysis tasks on the transcript.
  
- **Key Methods**:
  - `_generate_summary()`
  - `_extract_key_concepts()`
  - `_identify_learning_objectives()`
  - `_assess_complexity()`
  
- **Implementation Details**:
  - Uses asynchronous calls to AI models (Anthropic Claude 3 or OpenAI GPT-4).
  - Gathers responses in parallel using `asyncio.gather`.

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

### 3. `LearningContentGenerator`

- **Purpose**:  
  - Produces educational content (study guides, quizzes, flashcards, coding exercises) from the analyzer’s insights.
  
- **Implementation Details**:
  - Reuses the `_query_ai` logic from `VideoAnalyzer` for content creation.
  - Uses structured prompts adapted to each content type (e.g., multiple choice questions, code samples).

---

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

## Concurrency and Workflow

1. **Asynchronous Entry Point**  
   - The `main` function is `async`, invoked via `asyncio.run(main(...))`.
   - Each time an I/O-bound or network-bound task is requested, a separate coroutine is used.

2. **Concurrent Analysis**  
   - The analyzer runs four tasks (`_generate_summary`, `_extract_key_concepts`, `_identify_learning_objectives`, `_assess_complexity`) at the same time.
   - This design reduces total wait time when multiple requests to AI models are needed.

3. **Example Code Snippet**  
   ```python
   tasks = [
       self._generate_summary(),
       self._extract_key_concepts(),
       self._identify_learning_objectives(),
       self._assess_complexity()
   ]
   results = await asyncio.gather(*tasks)
   ```
   
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


#### Error Handling and Logging

Local Exception Blocks:

Each stage (download, transcription, analysis, generation) has its own exception blocks to catch and handle errors specifically in that context.

Fallback Mechanisms:
--If Anthropic’s API call fails, the script switches to OpenAI GPT-4.
--If AssemblyAI transcription fails, the script uses Whisper.

Logging:
--The script uses rich.console.Console to provide on-screen logs and formatted messages.
--All warnings and errors are displayed in different colors to alert the user.
