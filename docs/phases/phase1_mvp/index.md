# 🔥 Phase 1: Minimum Viable Product (MVP)

The MVP phase focuses on validating core dubbing and localization capabilities by integrating state-of-the-art models with minimal orchestration. This phase is hands-on and experimentation-heavy, enabling us to evaluate models individually before building full automation in future phases.

---

## 🎯 Objectives

- Explore and evaluate pretrained models for:
  - Speech-to-text (Whisper, Google ASR)
  - Translation (mT5, MarianMT)
  - Emotion recognition (BiLSTM, AudioSet-based CNN)
  - Voice synthesis (Tacotron2, VITS, ElevenLabs,Fastspeech)
  - Lip syncing (Wav2Lip, SyncNet)
- Understand compatibility of intermediate outputs across stages
- Simulate the full dubbing flow by manually stitching outputs
- Identify model strengths, limitations, and domain tuning needs

---

## 🛠️ Key Activities

- Collect diverse multimedia inputs (video/audio/text samples)
- Manually run each model and store outputs
- Document sample inputs, outputs, formats, and transformations
- Track latency, accuracy, and subjective quality
- Log known challenges or friction between model handovers
- Create reusable scripts for preprocessing and evaluation

---

## 🧪 MVP Evaluation Metrics

- Transcription accuracy (WER)
- Translation fidelity (BLEU, subjective review)
- Emotion match rate (human annotated)
- Voice clarity & naturalness (MOS score or rating scale)
- Lip sync believability (frame match ratio or user score)

---

## 📦 Deliverables

- Evaluation reports for each model
- Test dataset and results repository
- Manual flow reference implementation
- Recommendations for Phase 2 orchestration
