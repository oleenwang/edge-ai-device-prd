# NeoVoice V1 — AI Feature Specification

> Detailed specification for each AI feature in the NeoVoice V1 product. This document defines the engineering requirements, success criteria, and implementation notes for each AI capability.

---

## Feature Overview

| Feature | Priority | Offline? | Complexity |
|---------|----------|----------|------------|
| Offline Voice Assistant | P0 | ✅ | High |
| WhatsApp Voice Reply | P0 | ✅ | High |
| Voice-to-Voice Translation | P1 | ✅ | Very High |
| Camera AI — Low Light | P0 | ✅ | Medium |
| Camera AI — Skin Tone Optimization | P0 | ✅ | High |
| Document Scanner + OCR | P1 | ✅ | Medium |
| On-Device LLM (3B) | P0 | ✅ | Very High |
| Predictive Text (Local Scripts) | P2 | ⚠️ | Low |

---

## P0: Offline Voice Assistant

### Description

A voice assistant that responds to commands and queries entirely on-device, without internet connectivity. Supports 20+ emerging market languages.

### User Story

```
User: "Jeka wo ni owu" (What time is it in Yoruba)
Assistant: "Ojo merinla, 3:45 l'Ojo Ajua" (It is 3:45 PM on Tuesday)

No internet required. Response in < 500ms from end of speech.
```

### Technical Specification

```
Model: Whisper-large-v3-turbo (1.5B params)
Quantization: 4-bit integer (Q4_K_M via llama.cpp)
Model size: ~1.2GB (compressed), ~1.5GB (loaded)
RAM requirement: 1.2GB peak during inference
Storage: Pre-installed in 8GB AI model partition

Language support:
- Tier 1 (launch): Hausa, Swahili, Yoruba, Igbo, Amharic, Bengali, Urdu, Bahasa Indonesia
- Tier 2 (6 months): Somali, Shona, Tamil, Telugu, Thai, Vietnamese

Latency target:
- Wake word detection: < 100ms
- Speech-to-text: < 300ms (short phrase)
- LLM response: < 500ms (local LLM)
- Total E2E: < 900ms

Power consumption:
- Idle (always-on wake word): < 50mW
- Active processing: < 1.5W peak, < 1.0W sustained
```

### Implementation Notes

```
1. Wake word: Custom trained for each language market
   - Yoruba: "Modupe"
   - Hausa: "Ya!"
   - Swahili: "Sema"
   Not universal wake word — local language wake words increase engagement

2. VAD (Voice Activity Detection):
   - Use Silero VAD (open source, ONNX)
   - Optimized for emerging market accents

3. Fallback behavior:
   - If model fails to recognize language: prompt in English
   - If user speaks unsupported language: explain in local language what is supported
```

---

## P0: WhatsApp Voice Reply

### Description

Intercept incoming WhatsApp voice notes, transcribe in sender's language, generate contextual reply options.

### User Story

```
Scenario:
- Contact sends voice note in Hausa
- User receives notification
- NeoVoice shows: "Reply to Aminu in Hausa" with 3 suggested responses
- User taps one, voice note sent back in Hausa
- All processing happens on device, no internet required
```

### Technical Specification

```
Architecture:
- WhatsApp notification intercepted (accessibility service on Android)
- Audio streamed to whisper.cpp
- Transcription in sender's detected language
- Context injected into local LLM with prompt template
- 3 reply options generated
- User selects, records response, sends

Languages: Same as voice assistant (20+ languages)
Latency:
- Transcription: < 300ms per voice note
- Reply generation: < 500ms
- Total: < 1 second from notification

Privacy:
- No audio stored beyond processing
- No data leaves device
- No logging of conversation content
```

### Implementation Notes

```
1. Context window management:
   - Last 10 messages of conversation injected into LLM prompt
   - Older messages summarized and injected as context

2. Reply quality criteria:
   - Contextually appropriate (matches conversation topic)
   - In same language as sender
   - Short enough for voice note (max 30 seconds)
   - Varied in tone (friendly, formal, casual)

3. Permission handling:
   - Accessibility service permission required (user opt-in)
   - Clear UX explaining what data is accessed and how
```

---

## P0: Camera AI — Low Light Enhancement

### Description

Multi-frame RAW processing optimized for African/South Asian lighting conditions: indoor, mixed artificial/ambient, low lux.

### Technical Specification

```
Input: 12MP Bayer RAW from 50MP sensor (4-in-1 binning)
Processing: 
  - 8-frame RAW burst
  - Multi-frame HDR merge
  - Local tone mapping
  - Skin-preserving noise reduction
  - Sharpening

Output: 12MP enhanced JPEG
Processing time: < 200ms (ISP pipeline)
Power impact: < 5% battery per 100 shots

Skin tone handling:
- Separate processing for dark skin (Type V-VI on Fitzpatrick scale)
- Warm tone preservation (not desaturated as in most "brightening" filters)
- Detail in shadows without overexposing highlights

Benchmark target: Comparable to Samsung A54 night mode
  - Subjective quality score > 3.5/5 in user testing
  - Detail preservation > 80% vs. ground truth
```

### Implementation Notes

```
1. Dataset requirements:
   - 10,000+ images from African and South Asian markets
   - Ground truth: professional photography of same scenes
   - Skin tone diversity across Type III to Type VI

2. Key differentiator vs. competitors:
   - Most night mode algorithms optimized for East Asian skin tones
   - Our processing preserves warm undertones and detail in dark skin
   - This is a real product differentiator, not a marketing claim
```

---

## P0: Camera AI — Skin Tone Optimization

### Description

Portrait and selfie mode AI optimized for the full spectrum of emerging market skin tones.

### Technical Specification

```
Features:
- Real-time portrait segmentation (foreground/background)
- Skin tone detection and separate processing
- Local tone mapping per skin region
- Color temperature adjustment per skin tone
- Red-eye correction (off-angle flash common in African lighting)

Performance:
- Processing: < 50ms per frame (real-time in viewfinder)
- Memory: < 200MB
- Power: < 2% battery per 100 portrait shots

Supported skin tones: Fitzpatrick Type III through Type VI
  - Covers South Asian, Southeast Asian, and African skin tone ranges
```

### Implementation Notes

```
1. The industry standard for "beauty mode" AI is optimized for:
   - East Asian skin tones (fair, light)
   - Type I-II on Fitzpatrick scale
   - Most "whitening" and "smoothing" filters assume light skin

2. Our approach:
   - Detect skin tone from scene analysis
   - Apply tone-appropriate processing (not just "brighten")
   - Preserve natural warmth and texture
   - Avoid "ashy" or "grey" artifacts common in dark skin processing

3. Testing protocol:
   - Must be tested with 100+ models from target markets
   - User research: "Does this photo look like me?"
   - Qualitative validation, not just technical metrics
```

---

## P1: Voice-to-Voice Translation

### Description

Bi-directional speech-to-speech translation between any two Tier 1 languages.

### Technical Specification

```
Supported pairs: Any two of 20+ Tier 1 languages
  - Example: Yoruba ↔ Hausa, Swahili ↔ English, Bengali ↔ Urdu

Pipeline:
  1. ASR (Whisper) → source text
  2. Translation (local LLM fine-tuned for translation) → target text
  3. TTS (Coqui TTS, open source) → target audio

Model sizes:
  - ASR: ~1.5GB (shared with voice assistant)
  - Translation: ~2GB (fine-tuned 3B LLM)
  - TTS per language: ~200MB
  - Total per additional language pair: ~2.2GB

Latency:
  - Total E2E: < 2 seconds for 10-second source speech
  - TTS generation: < 500ms after translation

Accuracy target:
  - Intent preservation: > 85% (not verbatim transcription accuracy)
  - Naturalness: > 3.5/5 (user study rating)
```

### Implementation Notes

```
1. Why not cloud translation?
   - Even 10-second voice note requires data
   - At $0.50-2/MB, users can't afford cloud translation
   - Offline capability is the core differentiator

2. TTS voice quality challenge:
   - Most open-source TTS sounds robotic in non-English languages
   - Coqui TTS quality varies significantly by language
   - Investment needed: voice recording sessions for Tier 1 languages

3. Language pair priority:
   - English ↔ Local language: highest demand
   - Cross-African: Hausa ↔ Yoruba, Swahili ↔ Hausa: business/trade use case
   - South Asia: Urdu ↔ Hindi (though politically sensitive)
```

---

## P1: Document Scanner + OCR

### Description

Auto-crop, perspective correction, and OCR for documents in local scripts.

### Technical Specification

```
Capabilities:
  - Auto-detect document boundaries
  - Perspective correction
  - Binarization / contrast enhancement
  - OCR in 20+ local scripts

OCR engines:
  - English + Latin scripts: Tesseract 5 (open source)
  - Arabic script (Urdu, Pashto): Tesseract with Arabic training data
  - Bengali, Devanagari: Tesseract with Indic training data
  - Amharic (Ge'ez script): Custom training required (sparse data)

Accuracy targets:
  - Clean document: > 95% character accuracy
  - Noisy document: > 85% character accuracy
  - Handwritten: > 70% (best effort)

Languages supported (Tier 1 launch):
  - English, Hausa (Latin), Swahili (Latin), Yoruba (Latin), Igbo (Latin)
  - Urdu (Arabic script), Bengali, Amharic, Bahasa Indonesia

Processing: Fully offline, < 500ms per page
```

---

## P0: On-Device LLM (3B Parameter Model)

### Description

Local large language model for general Q&A, draft generation, and information queries.

### Technical Specification

```
Base model: Llama 3.2 3B Instruct
Quantization: 4-bit Q4_K_M (via llama.cpp)
Context: 8K tokens
Model size: ~2.1GB (4-bit quantized)
RAM usage: ~3GB (model + KV cache + app)

Capabilities (allowed):
  - General Q&A in local languages
  - Draft generation (SMS, WhatsApp)
  - Agricultural information (weather, crop prices via pre-loaded data)
  - Health information (symptom checker, NOT diagnosis)
  - Calculator and unit conversion
  - Local language learning

Capabilities (explicitly NOT included):
  - Real-time web search
  - Location-based services
  - Social media integration
  - Code generation or complex reasoning tasks
  - Any form of diagnosis or medical advice

Performance:
  - Throughput: 30+ tokens/second on Dimensity 6080
  - First token latency: < 300ms
  - Power: < 2W sustained

Fine-tuning:
  - Domain-adapted for emerging market use cases
  - Local language instruction tuning
  - Safety alignment for medical/health queries
```

### Implementation Notes

```
1. Model selection rationale:
   - 3B is the minimum viable size for coherent instruction following
   - 7B requires too much RAM for 6GB device variant
   - 1B insufficient for multi-turn conversation quality

2. Fine-tuning priorities:
   - Local language instruction following (Hausa, Swahili, Yoruba, etc.)
   - Agricultural information retrieval (pre-loaded dataset)
   - Medical information safety alignment
   - Concise response style (mobile-optimized)

3. Memory management:
   - Model loaded on-demand, unloaded after 5 minutes idle
   - KV cache limited to 2K tokens to reduce memory footprint
   - Graceful degradation on 6GB device variant
```

---

## Success Criteria Summary

| Feature | Launch Criteria |
|---------|----------------|
| Offline Voice Assistant | 85%+ intent recognition, < 900ms E2E latency |
| WhatsApp Voice Reply | Works on WhatsApp 2.24+, privacy audit passed |
| Camera AI — Low Light | > 3.5/5 user rating, < 200ms per shot |
| Camera AI — Skin Tone | > 80% user preference vs. current leading phone |
| Voice-to-Voice Translation | > 85% intent preservation, < 2s E2E latency |
| Document Scanner + OCR | > 95% accuracy clean docs, > 85% noisy |
| On-Device LLM | > 30 tokens/sec, > 3.5/5 helpfulness rating |
