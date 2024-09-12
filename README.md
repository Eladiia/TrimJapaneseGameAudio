# Sentence Mining Game Audio Trim Helper

This project automates the recording of game sentence audio to help with Anki Card Creation. 

You can trigger the entire process with a single hotkey that cuts out the before and after voice, gets a screenshot, and sends both to Anki.


If you run into issues find me on discord @Beangate, or make an issue here. I've used this process to generate ~100 cards from Dragon Quest XI so far and it's worked quite well.


## Features:
- **Vosk Speech Recognition**: Automatically cuts the end of the clip to the exact moment the voice ended.
- **OBS Replay Buffer**: Constantly records the last X seconds of gameplay.
- **ShareX**: Takes screenshots of the game at the moment of the replay.
- **Clipboard Interaction**: Automatically monitors the clipboard for dialogue events.
- **Hotkey Automation**: Single hotkey to trigger video recording, screenshot, and transcription.

## Prerequisites

- [Python 3.7+](https://www.python.org/downloads/)
- [OBS Studio](https://obsproject.com/)

---

## 1. Setting Up OBS 60-Second Replay Buffer

1. **Install OBS Studio**: Download and install OBS from [here](https://obsproject.com/).
2. **Enable Replay Buffer**:
   1. Open OBS and navigate to **Settings → Output → Replay Buffer**.
   2. Enable the **Replay Buffer** and set the duration to **60 seconds**, this can be lower, or higher, but 60 works for a very simple setup.
3. **Set a Hotkey for the Replay Buffer**:
   1. Go to **Settings → Hotkeys** and find **Save Replay Buffer**.
   2. Assign a hotkey for saving the replay.
4. Set Scene/Source. I recommend using "Game Capture" with "Capture Audio" Enabled. And then mute Desktop/microphone
   1. If "Game Capture" Does not work, use "screen capture" with a second source "Application Audio Capture"
5. In Output Settings, set "Recording Format" to mkv, and "Audio Encoder" to Opus. Alternative Settings may be supported at a later date.
6. **Set up obs websocket** (Super Optional)
    1. Can allow my script to automatically start (and stop) the replay buffer.

---

## 2. Configuring `config.toml`

I redid the config parsing cause `config.py` is not ideal, especially when distributing a script via git.

Your `config.toml` file allows you to configure key settings for the automation process, file paths, and other behavior. Here are the configurable options:

Duplicate/rename config_EXAMPLE.toml to get started

```toml
# Path configurations
[paths]
folder_to_watch = "~/Videos/OBS"
audio_destination = "~/Videos/OBS/Audio/"
screenshot_destination = "~/Videos/OBS/SS/"

# Anki Fields
[anki]
url = 'http://127.0.0.1:8765'
sentence_audio_field = "SentenceAudio"
picture_field = "Picture"
current_game = "Japanese Game"
custom_tags = ['JapaneseGameMiner', "Test Another Tag"] # leave Empty if you dont want to add tags
add_game_tag = true

# Feature Flags
[features]
do_vosk_postprocessing = true
remove_video = true
update_anki = true

# Vosk Model
[vosk]
url = "https://alphacephei.com/vosk/models/vosk-model-small-ja-0.22.zip"
# If you have a high-performance PC, with 16GB+ of RAM, you can uncomment and use this model:
# url = "https://alphacephei.com/vosk/models/vosk-model-ja-0.22.zip"
log-level = -1

[screenshot]
width = 0 # Desired Width of Screenshot, 0 to disable scaling (Default 0)
quality = 85 # Quality of image, 100 is lossless (Default 85)
extension = "webp" # Codec of screenshot, Recommend Keeping this as webp (Default webp)

[audio]
extension = "opus" # Desired Extension/codec of Trimmed Audio, (Default opus)

[obs]
enabled = true
start_buffer = true
full_auto_mode = false # Automatically Create Cards when you Create in Yomi. REQUIRED for multi-card-per-voiceline
host = "localhost"
port = 4455
password = "your_password_here"
```


---

## 3. Install Requirements

If you know what you are doing, do this in a venv, but I'm not going to explain that here.

`pip install -r requirements.txt`

## 4. Installing FFmpeg

To run this script, you will need to have **FFmpeg** installed. If you don't have FFmpeg installed on your system, you can easily install it via **Chocolatey** (Preferred), or install it yourself and ensure it's in the PATH.

#### Step-by-Step Instructions:

1. First, ensure you have **Chocolatey** installed. If you don't have it installed, follow the instructions on the [Chocolatey installation page](https://chocolatey.org/install) or run the following command in an **elevated** PowerShell window (run as Administrator):
   
   ```bash
   Set-ExecutionPolicy Bypass -Scope Process -Force; `
   [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; `
   iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
   ```

2. Once Chocolatey is installed, open a **new** PowerShell or Command Prompt window (with administrator rights).

3. Run the following command to install FFmpeg:

   ```bash
   choco install ffmpeg
   ```

4. After the installation is complete, verify that FFmpeg is correctly installed by running the following command:

   ```bash
   ffmpeg -version
   ```

   If the installation was successful, you should see the version information for FFmpeg.

Now you're ready to use FFmpeg in the script!


---

## 5. One Click Card Creation

With the Latest Update it is now possible to do full 1-click card creation with this tool + Yomitan. This is configured in the `obs` section in your `config.toml`

Demo: https://www.youtube.com/watch?v=9dmmXO2CGNw

Screenshots to help with setup:

![image](https://github.com/user-attachments/assets/7de031e9-ce28-42eb-a8fd-0e60ef70dc3d)

![image](https://github.com/user-attachments/assets/b0c70a1a-65b5-4fe7-a7e4-ccb0b9a5b249)

## 6. Example Process

1. Start game
2. Hook Game with Agent (or textractor) with clipboard enabled
3. start script: `python main.py`
   1. Create Anki Card with target word (through a texthooker page/Yomitan)
   2. (If full-auto-mode not on) Trigger Hotkey to record replay buffer
4. When finished gaming, end script

Once the hotkey is triggered:
1. **OBS** will save the last X seconds of gameplay.
2. The Python script will trim the audio based on last clipboard event, and the end of voiceline detected in Vosk if enabled.
3. Will attempt to update the LAST anki card created.

---

## How to Update the Script

To ensure you always have the latest version of this script, you can use `git pull` to update your local repository with the latest changes from the remote repository.

### Step-by-Step Instructions

1. Open your terminal and navigate to the directory where you cloned the repository:
    ```bash
    cd path/to/script
    ```

2. Run the following command to fetch and integrate the latest changes:
    ```bash
    git pull origin main
    ```

    - **`origin`** refers to the remote repository from which you cloned the code.
    - **`main`** refers to the main branch of the repository. If your default branch has a different name (e.g., `master` or `dev`), replace `main` with that branch name.

3. The `git pull` command will download and apply any updates from the remote repository to your local version.

### Example:

```bash
$ cd path/to/script
$ git pull origin master
```

---

## Conclusion

This setup allows you to record key moments in your game automatically, capture screenshots, and transcribe dialogue, all through a simple hotkey. Enjoy automating your gaming content creation!
