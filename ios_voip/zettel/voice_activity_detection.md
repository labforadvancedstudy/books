# Voice Activity Detection

## Core Insight
Voice Activity Detection is binary classification at 50Hz - deciding every 20ms whether the current audio contains speech or just background noise.

VAD saves bandwidth and battery by not transmitting silence. In typical conversation, people speak only 40% of the time. Without VAD, you waste 60% of bandwidth sending background noise. With VAD, you send nothing during silence, reducing data usage and extending battery life. But aggressive VAD clips speech, making conversation feel unnatural.

The classification challenge is harder than it seems. Speech isn't just loud sounds - whispers are quieter than background noise. Fricatives ('f', 's', 'th') look like noise spectrally. Background speech from TV or others shouldn't trigger VAD. The algorithm must distinguish intentional speech from everything else in milliseconds.

Classical VAD uses energy and zero-crossing rate. If energy exceeds a threshold and zero-crossings suggest speech frequencies, classify as voice. But this fails in noisy environments. Modern VAD adds spectral features: pitch detection, formant analysis, spectral entropy. Machine learning approaches use neural networks trained on thousands of hours of speech and noise.

WebRTC's VAD is configurable for aggressiveness. Mode 0 (least aggressive) rarely clips speech but transmits more noise. Mode 3 (most aggressive) saves maximum bandwidth but might clip soft speech. Most VoIP apps use mode 1 or 2, balancing quality and efficiency.

Comfort noise is VAD's companion. When VAD stops transmission, the receiver hears absolute silence - unnatural and jarring. Comfort Noise Generation (CNG) synthesizes low-level background noise matching the sender's environment. The transition from real to synthetic noise should be imperceptible.

Hangover time prevents choppy audio. After speech ends, VAD continues transmitting for 200-300ms. This captures word endings and natural speech decay. Without hangover, words get cut off mid-syllable. But too much hangover defeats the purpose of VAD.

## Connections
→ [[comfort_noise]] - Fills silent periods
→ [[bandwidth_optimization]] - Primary purpose
→ [[speech_classification]] - Core algorithm
← [[opus_codec]] - Includes VAD/DTX
← [[battery_optimization]] - Reduces transmission
← [[audio_quality]] - Must balance accuracy

---
Level: L4
Date: 2025-08-15
Tags: #vad #speech #detection #optimization