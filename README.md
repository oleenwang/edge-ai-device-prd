# Edge AI Device — Product Requirements Document

> **Product:** NeoVoice V1 — Offline AI Assistant for Emerging Markets
> **Target Markets:** Sub-Saharan Africa, South Asia, Southeast Asia
> **Target Price:** $149 MSRP (retail)
> **Target Users:** First-time smartphone users, feature phone upgraders, users in low-connectivity environments
> **Version:** 1.0 | **Status:** Draft

---

## 1. Product Overview

### 1.1 What This Product Is

NeoVoice V1 is a $149 smartphone that delivers real AI capabilities **without requiring internet connectivity**. Unlike competitors that require cloud APIs for AI features, NeoVoice V1 runs a complete voice assistant, translation, and camera AI stack entirely on-device.

The target user is a first-time smartphone buyer in an emerging market who:
- Speaks a local language (Hausa, Swahili, Yoruba, Amharic, Bengali, Urdu, Bahasa, etc.)
- Has unreliable or expensive mobile data ($0.50-2/MB)
- Uses WhatsApp voice messages as their primary communication tool
- Takes photos primarily indoors and in low-light conditions

### 1.2 What This Product Is NOT

- Not a flagship AI phone competing with Samsung S-series
- Not a developer device or prototyping platform
- Not dependent on cloud connectivity for any core feature

### 1.3 Product Vision Statement

> "Every user, regardless of where they live or what language they speak, deserves an AI that works when they need it — not when they have data."

---

## 2. Market Context

### 2.1 Problem Statement

```
Current state for target users:
- WhatsApp voice messages require typing to reply (can't speak in local language)
- Translation requires $0.10-0.50 per message via data
- Camera AI is optimized for fair skin tones and good lighting
- AI assistants (Google Assistant, Siri) don't support local languages
- 70%+ of target users will never use cloud AI due to data cost

Result: AI is a premium feature for the connected few, not a utility for the many.
```

### 2.2 Competitive Landscape

| Competitor | AI On-Device | Local Language | Price | Cloud Required |
|-----------|-------------|----------------|-------|--------------|
| Transsion (our parent) | Partial (camera AI only) | Limited | $80-180 | Yes for most AI |
| Samsung A-series | Partial | Limited | $200-350 | Partial |
| Xiaomi | Partial | Limited | $150-300 | Partial |
| Nokia G-series | ❌ | ❌ | $150-200 | Yes |
| **NeoVoice V1** | **Full** | **20+ languages** | **$149** | **❌ No** |

### 2.3 Target Market Size

```
Total addressable market: ~800M users in emerging markets who are
                           first-time smartphone upgraders from feature phones

Serviceable addressable market (Year 1): ~50M users across
  - Nigeria + Ghana + Kenya + Tanzania: 25M
  - Bangladesh + Pakistan: 15M  
  - Indonesia + Philippines + Vietnam: 10M

Year 1 units: 3M devices
Year 3 units: 12M devices
```

---

## 3. Product Requirements

### 3.1 Hardware Requirements

#### 3.1.1 Processor — MediaTek Dimensity 6080 (or equivalent)

| Specification | Requirement | Rationale |
|---------------|-------------|-----------|
| NPU | ≥ 10 TOPS sustained | Required for simultaneous AI features |
| CPU | 2x A76 @ 2.2GHz + 6x A55 @ 2.0GHz | Balanced power/performance |
| Modem | 4G LTE Cat.7 | Adequate for rare connectivity moments |
| Power | < 3W at sustained AI inference | Battery life constraint |
| Cost | BOM ≤ $38 | Required for $149 retail at 35% margin |

**Selection rationale:** The Dimensity 6080 family is the sweet spot between Helio G99 (underpowered for full AI stack) and Dimensity 7200 (too expensive). It delivers 10+ TOPS at a cost point that fits the $149 price target.

#### 3.1.2 Memory & Storage

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| RAM | 6GB LPDDR4X | 8GB |
| Storage | 128GB UFS 2.2 | 256GB |
| Expandable | microSD up to 1TB | Same |

**Rationale:** 6GB is minimum for running on-device LLM (3B model) alongside OS. 128GB is required because target users store locally (cloud backup is unreliable/expensive).

#### 3.1.3 Display

| Specification | Requirement |
|---------------|-------------|
| Size | 6.6" - 6.8" |
| Resolution | 720p+ (HD+, 1612x720 or similar) |
| Refresh | 90Hz |
| Brightness | 600 nits minimum (outdoor readability in tropical markets) |

**Rationale:** 720p+ instead of 1080p to conserve power. 90Hz is now baseline expectation even in budget segment.

#### 3.1.4 Battery

| Specification | Requirement |
|---------------|-------------|
| Capacity | 5000mAh minimum |
| Charging | 18W fast charge |
| Design | Removable preferred for emerging market serviceability |

**Rationale:** 5000mAh + 18W is now standard even in budget tier. Users in markets with unreliable electricity benefit from removable battery option.

#### 3.1.5 Camera

| Camera | Specification | AI Feature |
|--------|-------------|-----------|
| Rear Main | 50MP, f/1.8, PDAF | Low-light enhancement, local skin tone optimization |
| Ultra-wide | 8MP, f/2.2 | AI scene detection |
| Macro | 2MP | Basic |
| Front | 13MP, f/2.0 | Portrait AI, optimized for local skin tones |

**Camera AI requirements:**
- Must work without cloud connectivity
- Low-light enhancement: < 200ms processing time
- Skin tone optimization: adapted for African, South Asian, Southeast Asian skin tone diversity

### 3.2 AI Feature Requirements (Software)

#### 3.2.1 Voice AI — Core Feature

```
Feature: Offline Voice Assistant
Model: Whisper-large-v3-turbo (or equivalent, 1.5B params)
Languages: 20+ emerging market languages (see 3.2.3)
Latency: < 300ms from end of speech to response start
RAM usage: < 1.2GB

Feature: Voice-to-Voice Translation
Capability: Bi-directional speech translation between any two supported languages
Offline model: 500MB per language pair
Accuracy target: > 85% intent recognition (not verbatim transcription accuracy)

Feature: WhatsApp Voice Reply
Integration: Intercept voice note, transcribe to local language, generate text reply
         OR: generate voice reply in same language as sender
Context awareness: Understands WhatsApp conversation context (NOT just standalone)
```

#### 3.2.2 On-Device Language Model

```
Model: Llama 3.2 3B Instruct (quantized to 4-bit Q4_K_M)
Context: 8K tokens
Capabilities:
  - General Q&A in local languages
  - Draft generation (SMS, WhatsApp)
  - Agricultural/market information query (weather, crop prices)
  - Health information (symptom checker, not diagnosis)
  
NOT included (out of scope):
  - Real-time web search (requires connectivity)
  - Voice synthesis (TTS handled separately)
  - Code generation / complex reasoning

Performance: 30+ tokens/second on Dimensity 6080
Memory: ~2.1GB for model + KV cache
```

#### 3.2.3 Language Support Priority

**Tier 1 — Day 1 launch:**
```
Africa: Hausa (80M speakers), Swahili (100M), Yoruba (45M), Igbo (45M), Amharic (32M)
South Asia: Bengali (260M), Urdu (230M), Hindi (600M already supported by many)
Southeast Asia: Bahasa Indonesia (200M), Filipino/Tagalog (90M), Vietnamese (90M)
```

**Tier 2 — 6 months post-launch:**
```
Additional African: Somali, Shona, Lingala, Kinyarwanda
Additional South Asia: Tamil, Telugu, Marathi, Gujarati
Additional SEA: Thai, Lao, Khmer, Malay
```

#### 3.2.4 Camera AI

```
Feature: Low-Light Photography Enhancement
Capability: Multi-frame RAW processing optimized for dark skin tones and indoor lighting
Processing time: < 200ms
Output: 12MP enhanced output
Power impact: < 5% battery per 100 shots

Feature: AI Scene Detection  
Classes: Food, text, document, face, night, auto
Accuracy: > 90% top-1 on common scenes
Power: < 2% battery per 1000 detections

Feature: Document Scanner
Capability: Auto-crop, perspective correction, text enhancement
Offline: Fully offline
Languages: OCR support for all Tier 1 languages
```

### 3.3 Connectivity Requirements

```
WiFi: 802.11ac (2.4GHz + 5GHz)
Bluetooth: 5.0
USB: USB-C 2.0
OTG: Supported (for external storage)
FM Radio: Supported (important for emerging market users)
4G LTE: Band support optimized per target market
  - Africa: Band 1/3/7/8/20/28
  - South Asia: Band 1/3/5/8/40/41
  - SEA: Band 1/3/5/7/8
```

**Critical constraint:** All AI features MUST work completely offline. Connectivity is optional, not required.

### 3.4 Durability Requirements

| Test | Standard |
|------|----------|
| Drop | 1.2m on concrete (6 faces) |
| Water | IP54 (splash resistant) |
| Temperature | -10°C to 45°C operating range |
| Humidity | 95% RH non-condensing |

**Rationale:** Emerging market users have active outdoor lifestyles and less controlled environments. IP54 is achievable at target cost; IP67 is not.

---

## 4. User Experience Requirements

### 4.1 First-Time User Experience (FTUE)

```
Critical requirement: Must be operable WITHOUT any internet connectivity at first boot.

FTUE flow:
1. Power on → immediate language selection (no connectivity check)
2. Basic setup (WiFi optional, can be skipped)
3. "Try the voice assistant" onboarding prompt
4. First voice command demo (pre-loaded local language model)
5. Camera intro with sample shot

Total offline-capable setup: < 5 minutes
```

### 4.2 Voice Interaction Design

```
Wake word: Local language wake word (e.g., "Modupe" for Yoruba, "Sema" for Swahili)
Alternative: Hardware button for voice assistant (volume-down long-press)

Interaction model:
- Single command: immediate response
- Multi-turn: up to 5 turns before context reset
- Error: graceful degradation with local language error messages

Power behavior:
- Wake word detection: always-on, < 50mW
- Active voice: < 1.5W during processing
- Idle: same as standard Android idle
```

### 4.3 Privacy Requirements

```
Data collection:
- No user voice data leaves device (fully on-device)
- Anonymous usage statistics: opt-in only
- No advertising ID tied to AI features

User control:
- Per-feature AI data toggle
- Voice data auto-delete after processing (configurable)
- No cloud backup of AI model outputs without explicit consent
```

---

## 5. Success Metrics

### 5.1 Launch Criteria (Must Meet to Ship)

| Metric | Target | Measurement |
|--------|--------|-------------|
| On-device LLM inference speed | ≥ 25 tokens/sec | Internal benchmark |
| Voice recognition accuracy | ≥ 85% intent | User testing, 1000+ samples per language |
| Battery life | ≥ 18 hours mixed use | TUV/DxOMark certification |
| Offline AI feature completion | 100% | All listed features work offline |
| Camera AI processing time | ≤ 200ms per shot | Internal benchmark |

### 5.2 Post-Launch Metrics (6-Month Targets)

| Metric | Target | Rationale |
|--------|--------|-----------|
| NPS | ≥ 45 | Indicator of user satisfaction |
| Daily voice assistant usage | ≥ 3 interactions/day | Engagement with core feature |
| Feature adoption rate | ≥ 40% try voice assistant | First-time user activation |
| Retention (6-month) | ≥ 60% | Sufficient product value to upgrade cycle |
| Voice data opt-in | ≥ 25% | Shows trust in privacy proposition |

---

## 6. Technical Architecture Overview

### 6.1 AI Software Stack

```
Application Layer:
  ├── Voice Assistant (native app)
  ├── Camera AI (integrated with camera app)
  ├── Translation (integrated with messaging)
  └── On-device LLM (service layer)

Model Runtime:
  ├── llama.cpp (LLM inference)
  ├── whisper.cpp (speech recognition)
  ├── SNPE / QNN / NPU runtime (chip-specific acceleration)
  └── MediaPipe (auxiliary ML tasks)

Hardware Abstraction:
  ├── MediaTek NPU API
  ├── Android Neural Networks API (NNAPI)
  └── Direct NPU access for performance-critical paths

OS: Android 14 (Go edition compatible if BOM pressure requires)
```

### 6.2 Storage Architecture

```
Total storage: 128GB default

Partition:
├── System: 15GB
├── AI Models: 8GB (pre-installed models + expandability)
│   ├── whisper: ~1.5GB (compressed)
│   ├── llama: ~2.1GB (4-bit quantized)
│   └── supporting models: ~4GB
├── User Data: 95GB
└── SD card (external): up to 1TB
```

---

## 7. Project Timeline

```
Phase 1 — Feasibility (Month 1-2):
  [x] Tech feasibility analysis (current state)
  [ ] Engineering validation of Dimensity 6080 AI stack
  [ ] Language model performance benchmarking
  [ ] Camera AI algorithm evaluation

Phase 2 — Design (Month 3-4):
  [ ] Complete product spec lock
  [ ] Industrial design direction
  [ ] User research validation
  [ ] Pricing and margin model

Phase 3 — Development (Month 5-10):
  [ ] Hardware development (EVT/DVT)
  [ ] Software development (AI stack + Android)
  [ ] Localization for Tier 1 languages
  [ ] Field testing in target markets

Phase 4 — Launch Preparation (Month 11-12):
  [ ] PVT (Production Validation Test)
  [ ] Regulatory certification (per market)
  [ ] Go-to-market planning
  [ ] Soft launch (pilot markets)

Target Launch: Month 12 (pilot), Month 14 (broad rollout)
```

---

## 8. Appendix

### 8.1 Glossary

| Term | Definition |
|------|-----------|
| NPU | Neural Processing Unit — dedicated AI accelerator chip |
| TOPS | Trillion Operations Per Second — NPU performance metric |
| BOM | Bill of Materials — total component cost |
| LLML | Large Language Model |
| Whisper | OpenAI's speech recognition model (open-source) |
| llama.cpp | George's open-source LLM inference engine |
| NNAPI | Android Neural Networks API — OS-level AI acceleration |

### 8.2 Reference Documents

- [MediaTek Dimensity 6080 Product Brief](https://www.mediatek.com/products/smartphones/dimensity-600) (public)
- [Whisper Model Card](https://github.com/openai/whisper) (open-source)
- [llama.cpp Documentation](https://github.com/ggerganov/llama.cpp) (open-source)
- Android NNAPI Documentation (Google public docs)

---

*PRD version 1.0 — Not a final specification. For portfolio/portfolio demonstration purposes.*
