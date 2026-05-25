# IBM Bob Setup Guide for /watch Skill

This guide will help you set up the `/watch` skill for IBM Bob, enabling Bob to analyze video content from URLs or local files.

## Prerequisites

Before using the `/watch` skill, you need:

1. **Python 3** (macOS/Linux) or **Python** (Windows)
2. **ffmpeg** and **ffprobe** - for video frame extraction
3. **yt-dlp** - for downloading videos from URLs
4. **Whisper API key** (optional but recommended) - for transcription when videos lack captions

## Installation Steps

### 1. Install the Skill

Clone this repository into your Bob skills directory:

```bash
git clone https://github.com/YOUR_USERNAME/bob-watch-video.git ~/.bob/skills/watch-video
```

### 2. Install Dependencies

The skill will attempt to auto-install dependencies on first use, but you can also install them manually:

#### macOS (with Homebrew)

```bash
brew install ffmpeg yt-dlp
```

If you don't have Homebrew installed:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### Linux (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install ffmpeg
python3 -m pip install --user yt-dlp
```

#### Linux (Fedora/RHEL)

```bash
sudo dnf install ffmpeg
python3 -m pip install --user yt-dlp
```

#### Windows

```bash
# Using winget
winget install ffmpeg
winget install yt-dlp

# Or using pip
pip install yt-dlp
```

### 3. Configure Whisper API (Optional but Recommended)

Whisper transcription is only needed when videos don't have native captions. The skill checks for API keys in two locations (in order):
1. `~/.config/watch/.env` (global config)
2. `.env` in your current project directory (project-specific)

**Quick Start:** Copy the provided `.env.example` to `.env` and add your API key:
```bash
cp .env.example .env
# Then edit .env and uncomment/add your API key
```

You have three options:

#### Option A: Groq (Recommended - Faster & Cheaper)

1. Get a free API key at [console.groq.com/keys](https://console.groq.com/keys)
2. Add it to your config:

**Global config (recommended):**
```bash
mkdir -p ~/.config/watch
echo "GROQ_API_KEY=your-key-here" >> ~/.config/watch/.env
chmod 600 ~/.config/watch/.env
```

**OR project-specific config:**
```bash
# In your project directory
echo "GROQ_API_KEY=your-key-here" >> .env
chmod 600 .env
```

#### Option B: OpenAI

1. Get an API key at [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
2. Add it to your config:

**Global config (recommended):**
```bash
mkdir -p ~/.config/watch
echo "OPENAI_API_KEY=your-key-here" >> ~/.config/watch/.env
chmod 600 ~/.config/watch/.env
```

**OR project-specific config:**
```bash
# In your project directory
echo "OPENAI_API_KEY=your-key-here" >> .env
chmod 600 .env
```

#### Option C: Skip Whisper Entirely

If you don't want to set up transcription, you can use `--no-whisper` flag. Videos without native captions will be analyzed using frames only (no audio/speech).

**Note:** If you already have API keys in your project's `.env` file, the skill will automatically pick them up! No additional configuration needed.

## First Use

The first time you use `/watch`, Bob will run a setup check. If anything is missing, it will guide you through the installation process.

Simply try:
```
/watch https://youtu.be/dQw4w9WgXcQ
```

Bob will either:
- Process the video (if everything is set up)
- Guide you through installing missing dependencies
- Ask for your Whisper API key if needed

## Usage Examples

### Basic Usage

```
/watch https://youtu.be/VIDEO_ID
/watch https://youtu.be/VIDEO_ID what happens at 2:30?
/watch ~/Videos/recording.mp4 when does the error occur?
```

### Focused Analysis (Specific Time Range)

For better accuracy and lower token cost, focus on specific sections:

```
/watch https://youtu.be/VIDEO_ID --start 2:15 --end 2:45
/watch video.mp4 --start 30 --end 60
/watch https://youtu.be/VIDEO_ID --start 1:12:00  # from 1h12m to end
```

### Advanced Options

```
# Lower frame count for tighter token budget
/watch VIDEO_URL --max-frames 40

# Higher resolution for reading on-screen text
/watch VIDEO_URL --resolution 1024

# Force specific Whisper backend
/watch VIDEO_URL --whisper groq
/watch VIDEO_URL --whisper openai

# Disable transcription (frames only)
/watch VIDEO_URL --no-whisper

# Custom output directory
/watch VIDEO_URL --out-dir /path/to/keep/files
```

## Supported Video Sources

- **YouTube** - most common use case
- **Vimeo, TikTok, X (Twitter), Instagram** - via yt-dlp
- **Local files** - `.mp4`, `.mov`, `.mkv`, `.webm`, etc.
- **Any yt-dlp-supported site** - hundreds of platforms

## Troubleshooting

### "Missing binaries" error

Run the setup script manually:
```bash
python3 ~/.bob/skills/watch-video/scripts/setup.py
```

### "No Whisper API key" warning

Either:
1. Add a Groq or OpenAI API key (see step 3 above)
2. Use `--no-whisper` flag to skip transcription

### Video download fails

- Check if the video is public (no login required)
- Try updating yt-dlp: `pip install --upgrade yt-dlp`
- Some platforms may be region-locked

### Whisper transcription fails

- Check your API key is correct in `~/.config/watch/.env`
- Verify you have API credits/quota remaining
- Audio files over 25 MB may fail (use `--start`/`--end` for long videos)

### Python command not found (Windows)

On Windows, use `python` instead of `python3`:
```bash
python ~/.bob/skills/watch-video/scripts/setup.py
```

## Best Practices

1. **Videos under 10 minutes** work best for full analysis
2. **Use `--start`/`--end`** for longer videos to focus on relevant sections
3. **Native captions are free** - most YouTube videos have them
4. **Groq is faster and cheaper** than OpenAI for Whisper transcription
5. **Clean up working directories** after analysis (Bob does this automatically)

## Token Efficiency

- 80 frames at 512px ≈ 50-80k image tokens
- Transcripts are cheap (few thousand tokens for 10-min video)
- `--resolution 1024` quadruples image tokens per frame
- Use focused mode (`--start`/`--end`) to reduce token cost

## Privacy & Security

- Videos are downloaded locally to temp directories
- Only extracted audio is sent to Whisper API (when needed)
- No video content is uploaded to any service
- API keys are stored locally at `~/.config/watch/.env` with restricted permissions (0600)
- Working directories are cleaned up after use

## Getting Help

If you encounter issues:

1. Check this setup guide
2. Review the main [README.md](README.md)
3. Check the [original project](https://github.com/bradautomates/claude-video) for additional context
4. Open an issue on GitHub

## Credits

This skill is adapted from [claude-video](https://github.com/bradautomates/claude-video) by Bradley Bonanno ([@bradautomates](https://github.com/bradautomates)). All credit for the core concept and implementation goes to the original author.