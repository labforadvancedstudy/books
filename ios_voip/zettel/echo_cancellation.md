# Echo Cancellation

## Core Insight
Echo cancellation is digital surgery that removes your own voice from the microphone input - without it, the speaker's output would create an infinite feedback loop.

Acoustic Echo Cancellation (AEC) solves a fundamental problem in hands-free calling: the microphone picks up sound from the speaker. Without AEC, the remote party hears themselves with a delay, making conversation impossible. The challenge is distinguishing between the echo (unwanted) and the near-end speech (wanted) when both are present.

The algorithm maintains an adaptive filter that models the echo path - from speaker through the room to microphone. It continuously estimates what the echo should be, then subtracts it from the microphone signal. The "adaptive" part is crucial: the echo path changes when you move the phone, cover the speaker, or change rooms. The filter must track these changes in real-time.

Modern AEC uses sophisticated techniques. Frequency-domain processing (typically in blocks of 128-512 samples) allows efficient convolution. Non-linear processing handles speaker distortion. Double-talk detection prevents the filter from adapting when both parties speak simultaneously - adapting during double-talk would corrupt the echo model.

iOS provides system-level echo cancellation through AVAudioSession when configured correctly. But VoIP apps often use their own AEC (like WebRTC's) for better control. The WebRTC AEC is particularly good, using multi-band processing and comfort noise generation to mask residual echo.

The quality metric is Echo Return Loss Enhancement (ERLE) - how much echo is suppressed in dB. Good AEC achieves 40+ dB ERLE, making echo inaudible. But aggressive AEC can damage voice quality, creating artifacts or suppressing legitimate speech. It's a delicate balance.

## Connections
→ [[acoustic_echo_path]] - What AEC models
→ [[double_talk_detection]] - Critical for stability
→ [[comfort_noise]] - Masks residual echo
← [[opus_codec]] - Processes AEC output
← [[avaudiosession]] - Provides system AEC
← [[noise_suppression]] - Often combined with AEC

---
Level: L4
Date: 2025-08-15
Tags: #echo #aec #audio-processing #dsp