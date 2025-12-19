# Gemini Audio Transcriber - Development Specification

## Project Overview

A standalone desktop application for file-based audio transcription using Gemini Flash 2.5 via OpenRouter. This utility extracts and focuses on the file-based transcription functionality from the Voice Notepad application, providing intelligent audio pre-processing and flexible prompt-based transcript formatting.

**Reference Implementation**: Voice-Notepad (/home/daniel/repos/github/Voice-Notepad)

## Core Features

### 1. File-Based Audio Transcription
- Upload single audio files (no direct recording functionality)
- Process audio through OpenRouter API (using Gemini Flash 2.5 model)
- Display transcribed text for user review and editing
- Export options: download as file or copy to clipboard

### 2. Audio Pre-Processing Pipeline
Before sending to the API, optimize audio files to meet API constraints:

- **Input**: Accept all common audio formats (WAV, MP3, M4A, FLAC, OGG, etc.)
- **Output Format**:
  - Default: MP3
  - Optional: Opus (user-configurable setting)
- **File Size Limit**: OpenRouter has a 20MB maximum upload size
  - Pre-processing must ensure compressed audio stays under 20MB
  - Display warning if unable to compress below limit
- **VAD (Voice Activity Detection)**: Remove silence segments
- **Compression**: Aggressive compression to meet 20MB limit
- **Channel Conversion**: Convert stereo to mono
- **Bitrate Optimization**: Reduce bitrate while maintaining clarity
- **Goal**: Minimize file size while preserving transcription quality

### 3. System Prompt Management

#### Default Prompt (Always Active)
- General cleanup and formatting
- Remove filler words (um, uh, like, etc.)
- Add proper punctuation
- Add paragraph breaks for readability

#### Template Prompts (User-Selectable)
Pre-built templates for common use cases:
- **Tech Documentation**: Format as technical documentation
- **Emails**: Format as professional email
- **Additional templates**: (to be defined based on Voice Notepad)

User can select multiple templates that layer on top of the default prompt.

#### Custom Prompts
- Allow users to write and save custom system prompts
- Store prompts locally
- Apply custom prompts in addition to or instead of templates

#### Prompt Concatenation Logic
Backend combines:
1. Base cleanup prompt (default)
2. Selected template prompt(s)
3. Custom user prompt (if provided)
4. Personalization data (if provided)

### 4. Personalization
- User profile settings:
  - Name
  - Email/signature
- Automatically injected into email template prompt
- Enables personalized output (e.g., "Best regards, [User Name]")

### 5. Structured Output
API returns structured data:
- **Transcript**: Cleaned and formatted text
- **Title**: Auto-generated title for use as filename

### 6. User Interface

#### Main Workflow
1. User uploads audio file
2. App processes audio (pre-processing pipeline)
   - Display progress indicator during audio processing
   - Show file size before/after compression
3. App sends to OpenRouter API with selected prompts
   - Display progress indicator during API call
4. App displays structured output:
   - Title (editable)
   - Transcript (editable text box)
5. User can:
   - Make light edits to transcript
   - Download as file (using title as filename)
   - Copy to clipboard
6. Starting new upload job clears previous transcription

#### Progress Indicators
- Audio processing: Show current step (VAD, conversion, compression)
- File size: Display original size and compressed size
- API call: Show "Transcribing..." with loading indicator
- Warnings: Alert if file cannot be compressed below 20MB

#### Settings/Configuration
- **API Key Management**:
  - OpenRouter API key input
  - Persistent storage (local file)
  - Secure storage on user's computer
- **Audio Processing**:
  - Output format selection: MP3 (default) or Opus
- **Personalization**:
  - Name field
  - Email/signature field
- **Prompt Management**:
  - View/edit custom prompts
  - Enable/disable template prompts
  - Manage default prompt settings

## Technical Architecture

### Technology Stack
(To be determined based on Voice Notepad implementation)
- **Language**: (Python/JavaScript/etc.)
- **UI Framework**: (Electron/PyQt/etc.)
- **Audio Processing**: (FFmpeg/pydub/etc.)
- **API Client**: OpenRouter SDK

### Key Components

#### 1. Audio Processing Module
- File upload handler (supports WAV, MP3, M4A, FLAC, OGG, etc.)
- VAD silence detection and removal
- Audio format conversion (to MP3 or Opus)
- Stereo to mono conversion
- Compression/bitrate optimization
- File size validation (must be <20MB)
- Warning system for oversized files

#### 2. API Integration Module
- OpenRouter client configuration
- API calls to Gemini Flash 2.5 via OpenRouter
- Structured output parsing
- Error handling and retries

#### 3. Prompt Management Module
- Prompt storage (local files or database)
- Prompt concatenation logic
- Template library
- Custom prompt CRUD operations

#### 4. Settings Module
- API key storage and retrieval
- User profile management
- Persistent configuration

#### 5. UI Components
- File upload interface
- Progress indicators (audio processing, API calls)
- File size display (original and compressed)
- Transcript display/editor
- Settings panel (API key, audio format, personalization, prompts)
- Export buttons (download/copy)
- Warning dialogs (oversized files, errors)

### Data Flow

```
Audio File Upload
    ↓
Audio Pre-Processing Pipeline
    ↓
Prompt Construction (Default + Templates + Custom + Personalization)
    ↓
API Call (OpenRouter → Gemini Flash 2.5)
    ↓
Structured Output Parsing
    ↓
UI Display (Title + Transcript)
    ↓
User Editing (Optional)
    ↓
Export (Download/Clipboard)
```

### File Storage

#### Local Configuration Files
- `config.json`: API key, user preferences (audio format: MP3/Opus)
- `prompts/`: Directory for custom prompts
  - `templates/`: Built-in template prompts
  - `custom/`: User-created prompts
- `profiles/`: User profile data (name, email)

#### Temporary Files
- Processed audio files (cleaned up after transcription)

## Development Phases

### Phase 1: Core Infrastructure
- [ ] Project setup and dependencies
- [ ] Audio processing pipeline implementation
- [ ] OpenRouter API integration
- [ ] Basic UI skeleton

### Phase 2: Prompt System
- [ ] Default cleanup prompt implementation
- [ ] Template prompt library
- [ ] Custom prompt management
- [ ] Prompt concatenation logic
- [ ] Personalization injection

### Phase 3: UI/UX
- [ ] File upload interface
- [ ] Transcript display and editing
- [ ] Settings panel
- [ ] Export functionality (download/clipboard)

### Phase 4: Configuration & Persistence
- [ ] API key storage
- [ ] User profile management
- [ ] Prompt storage
- [ ] Application settings

### Phase 5: Testing & Refinement
- [ ] Audio processing testing with various formats
- [ ] API integration testing
- [ ] Prompt system validation
- [ ] UI/UX refinement
- [ ] Error handling and edge cases

## Success Criteria

- Successfully processes various audio formats (WAV, MP3, M4A, FLAC, OGG, etc.)
- Consistently produces file sizes under 20MB limit
- Warns user when compression cannot achieve 20MB limit
- Generates accurate, well-formatted transcripts
- Provides flexible prompt customization
- Maintains secure, persistent configuration
- Offers intuitive user experience with clear progress indicators
- Supports both MP3 and Opus output formats

## Design Decisions

1. **Batch Processing**: No - single file at a time to keep the app simple
2. **Audio Formats**:
   - Input: Accept all common formats (WAV, MP3, M4A, FLAC, OGG, etc.)
   - Output: MP3 (default) or Opus (user-configurable)
3. **Transcript History**: No - keeping the app simple and focused
4. **File Size Limits**:
   - No duration limit
   - 20MB maximum file size (OpenRouter API limit)
   - Must compress audio to stay under limit
   - Warn user if compression cannot achieve <20MB
5. **Progress Indicators**: Yes - show processing steps and API call status

## References

- Voice Notepad codebase: `/home/daniel/repos/github/Voice-Notepad`
- OpenRouter API documentation
- Gemini Flash 2.5 model documentation (via OpenRouter)
