# Overview of Vocasync UI

To implement an AI-powered video dubbing and voice cloning system, built using Gradio for the user interface. The pipeline takes an input video, extracts its audio, segments it into smaller chunks based on transcription timestamps, translates each segment into the target language, and then generates dubbed speech for each segment using advanced text-to-speech (TTS) models. It supports two modes of synthesis: original speaker cloning to preserve the speaker’s identity, and emotion cloning where a separate reference audio is used to transfer a specific emotional tone.The final result is a properly clubbed cloned audio tracks of both original audio clone and emotion reference audio clone.

---

### VocaSync Emotion Pipeline

#### 1. Video Input

**Inputs (Gradio UI)**

- **Video Input (MP4 file)** → Source video from which audio is extracted.
- **Reference Emotion Audio** → Used if specific emotional style is needed in dubbing.
- **Source Language** → Language of the original audio (e.g., hi, en, te, ta, ur).
- **Target Language** → Language into which speech will be translated and dubbed (e.g., en).
- **Whisper Model Size** → Choice of transcription model (tiny, base, small, medium) depending on accuracy vs. speed.
- **TTS Model Type** → Choice of cloning model (YourTTS, XTTS) for generating cloned voice output.

---
  
#### 2. Audio Extraction

**Library:** MoviePy

- Extracts audio from the uploaded MP4 video using MoviePy library and saves it as .wav for further processing.

---
  
#### 3. Audio Transcription & Segmentation

**Model:** Whisper

- The extracted audio is transcribed into text using Whisper.
- Whisper is a transcription model developed by OpenAI, designed to transcribe audio in multiple languages, including Indian regional languages.
- It supports various model sizes such as tiny, small, medium, and large, where larger models provide higher accuracy and better multilingual support.
- For Indian languages, it is recommended to use Whisper Medium or above, as these can effectively detect the spoken language, generate accurate transcriptions, and produce timestamps for each spoken segment.
- Since not all modules efficiently handle long audios or lengthy text sequences, the extracted text is further segmented based on Whisper’s timestamps.
- This timestamp-based segmentation ensures that each portion of audio is transcribed, translated, and processed in manageable units.


---

  
#### 4. Text Translation

**Model:** GoogleTranslator (deep-translator)

- Translates each transcribed segment into the target language.
- Works segment-wise to preserve timing and flow.
- Prevents errors and context loss in long passages.

---

#### 5. Voice Cloning

**Model:** Coqui TTS (XTTS v2 / YourTTS)

- The translated text is cloned using audio references.
- Two audio inputs can be used:
  
    - Original video audio → to preserve the speaker’s identity.
    - Emotion reference audio → to add a specific emotion.
      
- XTTS v2 provides better performance than YourTTS, producing highly similar cloned audio while retaining emotional tone from the original.
- YourTTS can be used when explicit emotion transfer is required.
- Since TTS models support only about 300 words per generation, the segment-based method is used to process large audios efficiently.
- Each segment is cloned separately, ensuring both speaker identity and emotion consistency are maintained in the dubbed output.

---
  
#### 6.Segment Stitching

**Library:** MoviePy + NumPy

- All cloned segments (original clone and emotion clone) are stitched back together.
- Silence padding is added where needed so the audio aligns perfectly with the original timestamps (should optimize this).

---
  
#### 7. Audio Similarity Analysis

**Library:** Resemblyzer

- The speaker embeddings of original audio and cloned audio are compared.
- A similarity score is calculated to measure how closely the cloned voice matches the original speaker.
- Similarity score is calculated for both original audio and emotion reference audio.

---
  
#### 8. Final Output Generation

**Library:** Gradio

- The user receives two outputs:
- Original Speaker Clone Audio (translated speech in the speaker’s voice).
- Emotion-reference clone Audio (translated speech with emotion reference).
- Results (audio files, transcription (both original and translated text) , similarity scores) are displayed in the Gradio interface.
