# Asset Enrichment Agent

The Asset Enrichment Agent automatically enhances files and media with metadata, descriptions, and searchable content.

## Status: 📋 Planned

## Overview

When users upload files, images, or other media to Onelist, the Asset Enrichment Agent will automatically:
- Extract text from images (OCR)
- Generate descriptions for photos
- Parse PDF content
- Extract audio transcripts
- Add relevant tags and metadata

## Planned Architecture

```
File Upload → Type Detection → Enrichment Pipeline → Metadata Storage
                   ↓                    ↓                   ↓
            Route to handler    Process content      Update entry
            (image/pdf/audio)   Extract metadata     Make searchable
```

## Planned Components

### 1. Type Router
- MIME type detection
- Route to appropriate processor
- Handle unknown types gracefully

### 2. Image Processor
- OCR for text in images (receipts, screenshots, whiteboards)
- Scene description via vision models
- Face detection (optional, privacy-aware)
- EXIF metadata extraction

### 3. Document Processor
- PDF text extraction
- Office document parsing (docx, xlsx)
- Table structure recognition
- Form field extraction

### 4. Audio/Video Processor
- Speech-to-text transcription
- Speaker diarization
- Chapter/segment detection
- Key moment extraction

### 5. Metadata Generator
- Auto-tagging based on content
- Date/location extraction
- Related entry suggestions
- Searchable summary generation

## Planned Configuration

```elixir
# config/config.exs
config :onelist, Onelist.AssetEnrichment,
  enabled: true,
  ocr_provider: :google_vision,
  transcription_provider: :whisper,
  vision_model: "claude-sonnet-4-20250514",
  max_file_size_mb: 50,
  async_processing: true
```

## Planned Usage

```elixir
# Auto-triggered on upload
{:ok, enriched} = Onelist.AssetEnrichment.process(asset)

# Manual re-processing
{:ok, enriched} = Onelist.AssetEnrichment.reprocess(asset_id)

# Batch processing
{:ok, results} = Onelist.AssetEnrichment.process_pending()
```

## Provider Options

| Provider | Use Case | Status |
|----------|----------|--------|
| Google Vision | OCR, image labels | 📋 Planned |
| OpenAI Whisper | Audio transcription | 📋 Planned |
| Claude Vision | Image understanding | 📋 Planned |
| Tesseract | Self-hosted OCR | 📋 Planned |

## Privacy Considerations

- All processing on user's own data
- No training data contribution
- Optional local processing
- Clear enrichment provenance

## Implementation Phases

### Phase 1: Image OCR
- [ ] Integrate Google Vision or Tesseract
- [ ] Extract text from screenshots
- [ ] Handle receipts and documents

### Phase 2: Image Understanding
- [ ] Scene descriptions via Claude Vision
- [ ] Auto-tagging for photos
- [ ] EXIF metadata extraction

### Phase 3: Documents
- [ ] PDF text extraction
- [ ] Office document parsing
- [ ] Table recognition

### Phase 4: Audio/Video
- [ ] Whisper integration
- [ ] Transcription storage
- [ ] Timestamp-based search

## Future Enhancements

- [ ] Real-time processing during upload
- [ ] Quality-based processing (skip low-res)
- [ ] User feedback loop for accuracy
- [ ] Custom enrichment rules per user
