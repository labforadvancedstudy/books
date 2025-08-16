# Noise Suppression

## Core Insight
Noise suppression is statistical separation of signal from noise - it learns what noise sounds like, then removes it while preserving speech.

Noise Suppression (NS) distinguishes between speech (wanted) and background noise (unwanted) using spectral characteristics. Speech has harmonic structure, rapid changes, and specific frequency patterns. Noise is often stationary - consistent frequency content over time. The algorithm exploits these differences to enhance speech while attenuating noise.

The classic approach uses spectral subtraction. Transform audio to frequency domain (FFT), estimate the noise spectrum during non-speech periods, subtract it from the input spectrum, transform back to time domain. But simple subtraction creates "musical noise" - random tones that sound worse than the original noise.

Modern algorithms use statistical models. Wiener filtering finds the optimal gain for each frequency bin based on signal-to-noise ratio (SNR) estimates. Minimum Mean Square Error (MMSE) estimation considers the probability that speech is present. Machine learning approaches train neural networks to mask noise while preserving speech.

The challenge is preserving speech quality. Aggressive noise suppression makes voice sound robotic or underwater. It can remove speech harmonics that overlap with noise frequencies. The sweet spot removes enough noise to improve intelligibility without noticeable artifacts. This is why many apps offer adjustable noise suppression levels.

iOS/WebRTC provides multiple noise suppression options. The WebRTC noise suppressor works well for stationary noise (fans, air conditioners) but struggles with dynamic noise (typing, dogs barking). Apple's Voice Processing IO unit includes noise suppression optimized for iOS hardware, often performing better than software solutions.

## Connections
→ [[voice_activity_detection]] - Identifies speech periods
→ [[spectral_analysis]] - Foundation of NS algorithms
→ [[comfort_noise]] - Replaces suppressed noise
← [[echo_cancellation]] - Often processed together
← [[opus_codec]] - Encodes cleaned audio
← [[audio_quality]] - NS affects perceived quality

---
Level: L4
Date: 2025-08-15
Tags: #noise #suppression #audio-processing #dsp