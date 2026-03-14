<div align="center">
    <p>
        <strong>أبصرني · Abserny</strong><br />
        AI-powered vision assistant for the visually impaired
    </p>

![Platform](https://img.shields.io/badge/platform-Android-3DDC84?style=flat-square&logo=android&logoColor=white)
![Expo](https://img.shields.io/badge/built%20with-Expo%20SDK%2054-000020?style=flat-square&logo=expo&logoColor=white)
![AI](https://img.shields.io/badge/AI-Gemini%202.0%20Flash-4285F4?style=flat-square&logo=google&logoColor=white)
![Languages](https://img.shields.io/badge/language-Arabic%20%2B%20English-00BFFF?style=flat-square)

[Website](https://abserny.github.io/abserny.com/) · [Download APK](https://github.com/abserny/Abserny-Mobile/releases/latest) · [Issues](https://github.com/abserny/Abserny-Mobile/issues)

</div>

---

## Table of Contents

- [1. Project Overview](#1-project-overview)
- [2. The Core Idea](#2-the-core-idea)
- [3. Design Philosophy](#3-design-philosophy)
- [4. Shared Conceptual Architecture](#4-shared-conceptual-architecture)
- [5. Execution Flow (Conceptual)](#5-execution-flow-conceptual)
- [6. Abserny Variants](#6-abserny-variants)
- [7. Variant Architectures](#7-variant-architectures)
- [8. Execution Flow per Variant](#8-execution-flow-per-variant)
- [9. Subsystem Breakdown](#9-subsystem-breakdown)
- [10. Repository Structures](#10-repository-structures)
- [11. App State Machine](#11-app-state-machine-mobile)
- [12. Contribution Guide](#12-contribution-guide)
- [13. Development Philosophy](#13-development-philosophy)
- [14. Why Variants Are Not Forks](#14-why-variants-are-not-forks)
- [15. Roadmap](#15-roadmap)
- [16. Final Notes](#16-final-notes)

---

## 1. Project Overview

**Abserny** (أبصرني) is an assistive perception system designed to help visually impaired users understand their surroundings through AI-powered camera analysis and calm spoken feedback.

Abserny is not a single application. It is a **perception philosophy** implemented across multiple platforms (Desktop, Mobile, and future embedded systems), each respecting its own physical and system constraints.

### Core Promise
- **Gesture-first** — no screen interaction required
- **Deterministic** and explainable flow
- **Human-centered** spoken feedback
- **Offline-capable** with on-device ML fallback

| | |
|---|---|
| App Name | Abserny (أبصرني) |
| Bundle ID | `com.abserny.app` |
| Platform | Android — Standalone APK via EAS |
| Languages | Arabic (ar-SA) + English (en-US) |
| Framework | React Native via Expo SDK 54 |
| Build | EAS — Expo Application Services |

[↑ Back to Top](#table-of-contents)

---

## 2. The Core Idea

```
World  →  Camera  →  Signal  →  AI  →  Speech
```

Abserny transforms raw visual input into contextual meaning and communicates that meaning back to the user through speech and haptics — in the least cognitively demanding way possible.

Everything else in the codebase exists to serve this pipeline. No feature should make this pipeline harder to understand or predict.

[↑ Back to Top](#table-of-contents)

---

## 3. Design Philosophy

### 3.1 Gesture-First
Every interaction is designed assuming the user cannot see the screen. Navigation, mode switching, settings — all controlled through simple touch gestures learned in under two minutes via a spoken tutorial.

### 3.2 Clarity Over Cleverness
The system prioritizes understandable pipelines over opaque abstractions. Descriptions are concise, spatial, and actionable — never verbose.

### 3.3 Assistive-Centered Design
Latency, predictability, and calm output matter more than raw accuracy. Haptic confirmation always precedes speech response so the user knows the system heard them.

### 3.4 Learnable System
Abserny is meant to be read, understood, and extended. The onboarding tutorial teaches every gesture through practice, not documentation.

[↑ Back to Top](#table-of-contents)

---

## 4. Shared Conceptual Architecture

This architecture represents the **idea**, not the repository layout.

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  Camera Input → Signal → Perception → Reasoning    │
│                                  ↓                 │
│                           User Feedback            │
│                        (Speech + Haptics)          │
│                                                     │
└─────────────────────────────────────────────────────┘
```

[↑ Back to Top](#table-of-contents)

---

## 5. Execution Flow (Conceptual)

```
User
│
├─ Gesture Trigger
│        │
│   System captures frame
│        │
│   JPEG Base64 → AI Model
│        │
│   Contextual Description
│        │
└─ Speech + Haptic Response
```

[↑ Back to Top](#table-of-contents)

---

## 6. Abserny Variants

Abserny exists as multiple **variants**, each adapting the same conceptual pipeline to different constraints.

### 6.1 Variant Overview

| Variant | Platform | Constraints | Interaction |
|---|---|---|---|
| Desktop | Linux / Windows | Power-rich, static | Keyboard / Voice |
| Mobile | Android | Battery, thermal | Gesture / Speech |
| Future | Wearables | Ultra-low power | Audio / Haptics |

[↑ Back to Top](#table-of-contents)

---

## 7. Variant Architectures

### 7.1 Desktop Architecture

```
USB Camera
│
OpenCV  (frame capture)
│
YOLO    (object detection)
│
Reasoner  (natural language)
┌──┴──┐
TTS   CLI
```

**Characteristics:**
- Continuous capture
- Long-running processes
- Higher memory budget

---

### 7.2 Mobile Architecture

```
Android Camera
│
GestureHandler  (PanResponder)
│
FrameCapture    (takePictureAsync — quality 0.35)
│
DetectionRouter
┌───┴──────────────┐
│ Online           │ Offline
│                  │
Gemini 2.0         ML Kit
Flash Lite         On-Device
└───┬──────────────┘
│
SpeechQueue     (never overlaps)
┌───┴──────┐
expo-speech  expo-haptics
```

**Characteristics:**
- On-demand execution — triggered by gesture only
- Automatic online/offline routing — no configuration needed
- Battery-aware: single frame per scan, camera released immediately after
- Aggressive resource cleanup between scans

[↑ Back to Top](#table-of-contents)

---

## 8. Execution Flow per Variant

### 8.1 Desktop Flow

```
App Starts
│
Load Config (Camera ID, YOLO Model)
│
Create VisionSupervisor
│
Initialize Services
┌───┼──────────┬──────────────┐
│   │          │              │
Cam YOLO     Audio          Vosk SR
└───┴──────────┴──────────────┘
│
Warm Up → TTS: "Ready, say start"
│
┌─ Listening Loop ──────────────────────┐
│                                       │
│  Mic → Vosk → Trigger word?           │
│                    │ Yes              │
│             Cooldown active? ──Yes──► │
│                    │ No               │
│             Capture Frame             │
│                    │                  │
│             YOLO → Detect Objects     │
│                    │                  │
│             Generate Description      │
│                    │                  │
│             TTS speaks result         │
│                    │                  │
└────────────────────┴──────────────────┘
```

---

### 8.2 Mobile Flow

```
App Launches
│
First launch? ──Yes──► Language Picker (swipe R/L)
│                          │
│                  Gesture Tutorial (9 steps)
│                          │
◄───┴──────────────────────────┘
│
Main Camera Screen
│
Pre-warm TTS (silent) → Speak: "Abserny ready."
│
┌─ READY STATE ──────────────────────────────────┐
│                                                │
│  Double Tap ──────► Scan                      │
│  Long Press ──────► Repeat Last Result        │
│  Swipe Right ─────► Next Mode                 │
│  Swipe Left ──────► Previous Mode             │
│  Triple Tap ──────► Settings Sheet            │
│                                                │
└────────────────────────────────────────────────┘
│
┌─ SCAN FLOW ────────────────────────────────────┐
│                                                │
│  Haptic: Warning                              │
│      │                                        │
│  takePictureAsync (quality 0.35, base64)      │
│      │                                        │
│  Internet?                                    │
│  ┌───┴───────────┐                            │
│  │ Yes           │ No                         │
│  │               │                            │
│ Gemini 2.0    ML Kit                          │
│ Flash Lite    On-Device                       │
│  └───┬───────────┘                            │
│      │                                        │
│  Haptic: Heavy → Fade-in text                 │
│      │                                        │
│  Queued TTS speaks result                     │
│      │                                        │
└──────► Back to READY STATE ───────────────────┘
```

---

### 8.3 Settings Flow (Mobile)

```
Triple Tap → Settings sheet slides up
│
Three options (swipe to navigate):
┌────────────────────────────────┐
│  1. Repeat Tutorial            │
│  2. Change Language            │
│  3. Close                      │
└────────────────────────────────┘
│
Double tap to select
│
┌───┴────────────────────────────────────┐
│ Repeat Tutorial → re-runs 9-step flow  │
│ Change Language → Language Picker      │
│ Close           → back to Ready State  │
└────────────────────────────────────────┘
```

[↑ Back to Top](#table-of-contents)

---

## 9. Subsystem Breakdown

### 9.1 Signal Layer

```
Gesture  →  PanResponder  →  GestureClassifier  →  Intent
```

Handles touch input classification — distinguishing double tap, long press, triple tap, and swipe from raw touch events using timing windows and displacement thresholds.

| Gesture | Detection Rule |
|---|---|
| Double Tap | 2 taps within 320ms, displacement < 20px |
| Long Press | Touch held > 700ms without movement |
| Triple Tap | 3 taps within 320ms |
| Swipe | Displacement ≥ 55px, dx/dy ratio > 1.2 |

---

### 9.2 Perception Layer

```
CameraFrame  →  Base64 JPEG  →  DetectionRouter  →  Inference  →  Output
```

Routes each frame to either Gemini 2.0 Flash Lite (online, natural language) or ML Kit (offline, label-based). Mode determines the prompt and output format.

| Mode | Online Prompt Strategy | Offline Fallback |
|---|---|---|
| Scene | Describe surroundings, mention obstacles and layout first | ML Kit labels + spatial |
| Object | Identify the main object. Be specific. | ML Kit image labeling |
| Read | Read all visible text exactly as written. | ML Kit text recognition (OCR) |
| People | How many people? Brief description of each. | ML Kit face detection |

---

### 9.3 Reasoning Layer

```
Raw AI Output
│
Valid? ──No──► Fallback message (language-aware)
│ Yes
Trim to spoken length (~15 words max)
│
Queue for TTS
```

Validates model output, applies language-specific fallback messages, and manages the speech queue to prevent overlapping utterances.

---

### 9.4 Feedback Layer

```
Intent  →  expo-haptics     (immediate — fires before speech)
Result  →  useVoice queue   →  expo-speech
Result  →  Animated UI         (fade-in text — supplemental only)
```

Converts intent confirmations into immediate haptic pulses and model results into queued speech. UI animations are supplemental — the app functions completely without them.

[↑ Back to Top](#table-of-contents)

---

## 10. Repository Structures

### 10.1 Desktop — Core

```
abserny-desktop/
├─ app/
│  └─ main.py                    # KivyMD application entry
├─ orchestrator/
│  └─ supervisor.py              # Orchestrates all services
├─ domain/
│  ├─ entities.py                # Data models
│  └─ usecases.py                # Use cases
├─ services/
│  ├─ camera.py                  # Camera interface
│  ├─ vision.py                  # YOLO wrapper
│  ├─ audio.py                   # Text-to-speech
│  └─ speech_recognition.py      # Vosk wrapper
├─ infra/
│  └─ config.py                  # Configuration
└─ requirements.txt
```

---

### 10.2 Mobile

```
Abserny-Mobile/
├── App.js                       # Root — main camera screen, state machine
├── OnboardingScreen.js          # First-launch 9-step gesture tutorial
├── LanguagePicker.js            # Standalone language selection screen
├── SettingsOverlay.js           # Settings bottom sheet (slide-up)
├── AbsernyIcons.js              # Custom vector icons — no SVG dependency
├── index.js                     # Expo entry point
├── app.json                     # Expo config (bundle: com.abserny.app)
├── eas.json                     # EAS build profiles
├── assets/
│   └── logo.png
└── hooks/
├── useDetection.js          # Gemini + ML Kit routing, mode prompts
├── useGestures.js           # PanResponder gesture classifier
├── useLanguage.js           # All strings (ar + en), t(), AsyncStorage
├── useModes.js              # Scene / Object / Read / People
└── useVoice.js              # Queued TTS — never overlaps utterances
```

> Structural differences between Desktop and Mobile reflect platform constraints, not conceptual divergence. Both follow the same Signal → Perception → Reasoning → Feedback pipeline.

[↑ Back to Top](#table-of-contents)

---

## 11. App State Machine (Mobile)

The main screen operates as an explicit state machine. No implicit transitions.

| State | Visual | Gestures Active |
|---|---|---|
| `LOADING` | Black screen | None |
| `READY` | Pulse ring + mode color | All gestures |
| `SCANNING` | Spinning ring + label | None (blocked) |
| `SPEAKING` | Waveform bars | Long press only |
| `ERROR` | Red X ring | Double tap to retry |
| `AUTO_SCAN` | READY + AUTO badge | Double tap to stop |

[↑ Back to Top](#table-of-contents)

---

## 12. Contribution Guide

### 12.1 Who Should Contribute
- Curious engineers
- Accessibility advocates
- Arabic / English NLP specialists
- System thinkers

### 12.2 How to Start

1. Read this document fully
2. Understand the conceptual flow (Sections 4–5)
3. Choose a variant — Desktop or Mobile
4. Improve one layer: Signal, Perception, Reasoning, or Feedback

### 12.3 Contribution Rules
- One concern per PR
- Respect architectural boundaries — no Signal logic in the Feedback layer
- Document before optimizing
- Never break the offline fallback

[↑ Back to Top](#table-of-contents)

---

## 13. Development Philosophy

### Preferred
- Explicit pipelines over implicit magic
- Clear state transitions — name your states
- Deterministic gesture behavior — same gesture, same result, always
- Refs over stale closures in async callbacks (`useRef`, not `useCallback` for recursion)
- Haptics before speech — user gets instant confirmation before TTS finishes

### Avoid
- Hidden state — never let state live in a closure
- Magic abstractions that hide the pipeline
- Platform leakage — no Android-specific code inside logic hooks
- Overlapping speech utterances — always use the speech queue

### Critical Implementation Notes

```js
// expo-file-system — always use legacy import
import * as FileSystem from 'expo-file-system/legacy';

// ML Kit label() — single argument only
const result = await label(imageUri);  // NOT label(uri, options)
```

```bash
# Install packages
npx expo install package -- --legacy-peer-deps
npm install package --legacy-peer-deps --ignore-scripts
```

> ⚠️ Never add `@react-native-community/netinfo` (ReferenceError crash) or `expo-keep-awake` (build failure).

[↑ Back to Top](#table-of-contents)

---

## 14. Why Variants Are Not Forks

Abserny variants are **not forks**. They are contextual embodiments of the same perception philosophy.

| What Changes | What Never Changes |
|---|---|
| Constraints | Signal → Perception → Reasoning → Feedback |
| Execution rhythm | User safety first |
| Input interfaces (voice vs gesture) | Offline-capable commitment |
| | Spoken output as primary feedback |

[↑ Back to Top](#table-of-contents)

---

## 15. Roadmap

```
v1.0 ──────────────────► v1.1 ──────────────────► v2.0
Mobile Android            Lottie gesture icons      iOS build
Gemini + ML Kit           Waveform visualization    AbsernyVision TFLite
Bilingual AR/EN           Scan history              Custom model >80% acc
9-step tutorial           Confidence indicator      Wearable integration
Settings sheet            Battery-aware quality
```

Planned features in priority order:

1. Lottie animated gesture icons replacing current custom vector icons
2. Waveform visualization during speech (real audio amplitude via expo-av)
3. Scan history — swipe up to review last 5 results with timestamps
4. AbsernyVision custom TFLite model (target >80% accuracy, currently 37% in training)
5. iOS build
6. Confidence indicator — arc around the ready ring based on Gemini certainty
7. Battery-aware quality — reduced processing when battery < 20%
8. Shake to re-scan (accelerometer alternative trigger)
9. Result sharing — long press result → share sheet

[↑ Back to Top](#table-of-contents)

---

## 16. Final Notes

> Abserny is not built to impress machines.
> It is built to serve humans who deserve technology that works for them.

If you understand this document, you are ready to contribute.

---

**Links**
- [Website](https://abserny.github.io/abserny.com/)
- [Download APK](https://github.com/abserny/Abserny-Mobile/releases/latest)
- [GitHub Issues](https://github.com/abserny/Abserny-Mobile/issues)

**License:** This project is intended for educational purposes as a graduation project.

[↑ Back to Top](#table-of-contents)
