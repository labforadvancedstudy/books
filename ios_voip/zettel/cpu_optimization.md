# CPU Optimization

## Core Insight
CPU optimization in VoIP is about doing less, less often - every saved cycle extends battery life and improves call quality on constrained devices.

Audio processing dominates CPU usage. Encoding, decoding, echo cancellation, noise suppression - all running 50 times per second. On modern iPhones, this might use 5-10% CPU. On older devices or with video, it can hit 30-40%. Optimization isn't premature - it directly affects user experience.

SIMD instructions are the biggest win. ARM NEON instructions process multiple audio samples simultaneously. Hand-optimized assembly for critical loops (FFT, FIR filters) can be 4-8x faster than naive C code. Most codecs include NEON optimizations, but verify they're enabled in your build configuration.

Block processing reduces overhead. Instead of processing each 20ms frame individually, batch multiple frames. This improves cache locality and reduces function call overhead. But don't batch too much - latency increases with block size. The sweet spot is usually 40-60ms blocks.

Algorithmic optimization often beats micro-optimization. Use faster algorithms even if they're slightly worse. Replace complex echo cancellation with simpler algorithms on low-end devices. Reduce FFT sizes. Skip processing stages when possible - why run noise suppression if the noise level is already low?

Thread architecture matters. Separate audio capture, processing, and network threads. Use real-time priority for audio threads but normal priority for networking. Avoid locks in audio callbacks - use lock-free queues for inter-thread communication. One blocked thread can cause audio glitches.

Profile on real devices, not simulators. Xcode Instruments shows exactly where CPU time goes. Look for unexpected hotspots - string formatting in audio callbacks, memory allocations in tight loops, excessive logging. Sometimes a single badly placed NSLog can double CPU usage.

## Connections
→ [[simd_optimization]] - Vectorization techniques
→ [[thread_architecture]] - Concurrent processing
→ [[profiling_tools]] - Performance measurement
← [[battery_optimization]] - CPU affects battery
← [[audio_processing]] - Main CPU consumer
← [[device_capabilities]] - Optimization targets

---
Level: L7
Date: 2025-08-15
Tags: #cpu #optimization #performance #simd