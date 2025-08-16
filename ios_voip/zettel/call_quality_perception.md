# Call Quality Perception

## Core Insight
Call quality perception is psychoacoustic - users don't hear objective metrics but subjective experience shaped by expectations, context, and comparison.

The brain is remarkably adaptive. Users quickly adjust to consistent quality, even if poor. But quality variations are immediately noticeable and annoying. Constant 3/5 quality feels better than alternating 5/5 and 1/5. This is why adaptive algorithms must change gradually, not abruptly.

Latency is the invisible killer. Below 150ms mouth-to-ear delay, conversation feels natural. At 300ms, people start talking over each other. Above 500ms, conversation becomes turn-taking like walkie-talkies. Users blame "call quality" without understanding that latency, not audio quality, is the problem.

Frequency range shapes perception more than bitrate. Narrow-band (300-3400 Hz) sounds like old phones. Wide-band (50-7000 Hz) sounds natural. Full-band (20-20000 Hz) sounds present. Users perceive wide-band at 16 kbps as higher quality than narrow-band at 64 kbps. The ear wants those missing frequencies.

Echo is disproportionately annoying. Even slight echo makes calls feel broken. Users might tolerate noise, distortion, even occasional drops. But echo triggers immediate complaints. This is why echo cancellation gets priority over other enhancements - it's better to have noisy audio than echo.

First impressions anchor perception. The first 5 seconds of a call set expectations. Start with good quality that degrades? Users notice every glitch. Start with poor quality that improves? Users are pleased. This is why many apps add artificial "connecting" delay to ensure good initial quality.

Comparison shapes satisfaction. Users don't judge VoIP calls in isolation but against cellular calls, other VoIP apps, and their best past experience. WhatsApp calls feel good not because they're perfect, but because they're consistently better than traditional international calls they replace.

## Connections
→ [[psychoacoustics]] - Perception science
→ [[latency_perception]] - Delay sensitivity
→ [[frequency_response]] - Hearing range
← [[audio_quality]] - Objective metrics
← [[user_experience]] - Subjective satisfaction
← [[echo_cancellation]] - Critical for perception

---
Level: L0
Date: 2025-08-15
Tags: #perception #quality #psychoacoustics #ux