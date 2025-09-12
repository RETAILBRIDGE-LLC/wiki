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

---

# AWS Setup
#### 1. Launch EC2 Instance
#### 🚀 1.1 Choose Instance

Before deploying **VocaSync** on AWS, choose an EC2 instance that meets the following requirements:

| **Requirement** | **Recommended** |
|------------------|----------------|
| **Instance Type** | `g4dn.xlarge` (GPU-enabled for faster Whisper + XTTS) <br> or `t3.xlarge` (CPU-only, slower) |
| **Operating System** | **Ubuntu 22.04 LTS** |
| **vCPUs / RAM** | 4 vCPUs / 16GB RAM minimum |
| **Storage** | 30 GB EBS (gp3 recommended) |

#### 🔐 1.2 Security Group

Configure the **Security Group** for your EC2 instance to allow the following inbound rules:

| **Port** | **Protocol** | **Purpose** |
|---------|-------------|-------------|
| **22** | TCP | SSH access to the instance |
| **7860** | TCP | Access to the Gradio Web UI |
| **80** | TCP | *(Optional)* Allow HTTP traffic (for Nginx or a reverse proxy) |
| **443** | TCP | *(Optional)* Allow HTTPS traffic (for secure Nginx reverse proxy) |

---

#### ⚙️ 2. Instance Setup & Auto-Start

This step covers setting up your EC2 instance so that your Gradio app (`app.py`) runs automatically on every boot.

---

#### 🖥️ 2.1 Prepare the Instance

 **SSH into your EC2 Instance:**
 
   ```
   bash
   ssh -i your-key.pem ec2-user@<your-ec2-public-ip>
   ```

---

#### 🖥️ 2.2 Ensure Your Environment is Ready

Before proceeding, make sure the following are already set up on your EC2 instance:

- ✅ `app.py` and all required dependencies are installed in `/home/ec2-user/`
- ✅ A Python virtual environment exists at `/home/ec2-user/gradio_env/`

---

#### 🏃 2.3 Create Start Script

Create a file called `start_gradio.sh` in `/home/ec2-user/`:

```
bash
nano /home/ec2-user/start_gradio.sh
```

---

```
cd /home/ec2-user/
source /home/ec2-user/gradio_env/bin/activate
python3 app.py
```

---

##### Create and Enable Systemd Service

To ensure that your Gradio app automatically starts when the instance boots, create a **systemd service**.

---

##### Create the Service File

Run the following command to create a new service file:

```
bash
sudo nano /etc/systemd/system/gradio.service
```

---

```
[Unit]
Description=Gradio App
After=network.target

[Service]
User=ec2-user
WorkingDirectory=/home/ec2-user
ExecStart=/home/ec2-user/start_gradio.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

---

#### 🖥 3. Modify for Public Access

To make the Gradio UI accessible from your EC2 instance’s public IP, edit `vocasync.py` and modify the `ui.launch()` section as follows:

```
python
if __name__ == "__main__":
    if os.path.exists(TEMP_DIR):
        shutil.rmtree(TEMP_DIR)
    os.makedirs(TEMP_DIR, exist_ok=True)
    
    # Launch Gradio publicly
    ui.launch(
        server_name="0.0.0.0",  # Listen on all network interfaces
        server_port=7860,       # Port exposed in Security Group
        share=False,            # Disable Gradio's share link (optional)
        debug=True
    )
```

---

#### 🌐 4. Optional – Domain + HTTPS (Production)

For production deployments, you can expose your Gradio app securely via **Nginx** and **Let's Encrypt (Certbot)**.

---

#### 📦 4.1 Install Nginx + Certbot

Run the following commands:  

```
bash
sudo apt-get install -y nginx certbot python3-certbot-nginx
```

---

#### ⚙️ 4.2 Configure Reverse Proxy

Create a new Nginx configuration file:  

```
bash
sudo nano /etc/nginx/sites-available/vocasync
```

---

```
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:7860;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

#### 🔗 4.3 Enable Site and Restart Nginx **  

Run the following commands:  

```bash
sudo ln -s /etc/nginx/sites-available/vocasync /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

### ▶️ 5. Running the Application  

Run your application manually with:  

```
bash
python app.py
```

Running on public URL: https://1234abcd.gradio.live

---
You can access the VocaSync UI in two ways:

- Via the Gradio link shown in the terminal

- Directly from your EC2 Public IPv4

```
ui.launch(server_name="0.0.0.0", server_port=7860)
```

then simply open:

```
http://<your-ec2-public-ip>:7860
```
Or click on the Public IPv4 address shown in your AWS EC2 console, append :7860 at the end, and the VocaSync UI will open in your browser

