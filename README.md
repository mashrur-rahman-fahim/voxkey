# VoxKey

Global push-to-talk speech-to-text for Linux. Hold a key, speak, release — your words are typed wherever your cursor is.

Powered by [Groq](https://groq.com/) Whisper Large v3 Turbo for near-instant transcription.

## How It Works

1. **Hold F8** — starts recording from your microphone
2. **Release F8** — audio is sent to Groq Whisper API
3. **Text is typed** at your current cursor position (any app — terminal, browser, editor)

## Requirements

- Linux with Wayland or X11
- Python 3.10+
- `arecord` (ALSA utils)
- `ydotool` (for typing text)
- `evdev` Python package
- A [Groq API key](https://console.groq.com/keys) (free tier available)

## Installation

```bash
# Install system dependencies
sudo apt install -y alsa-utils ydotool

# Install Python dependencies
pip install groq evdev

# Clone the repo
git clone https://github.com/mashrur-rahman-fahim/voxkey.git
cd voxkey

# Set up your API key
cp .env.example .env
# Edit .env and add your Groq API key

# Add the API key to your shell
echo 'export GROQ_API_KEY="your_key_here"' >> ~/.bashrc
source ~/.bashrc

# Install the script
cp stt ~/.local/bin/stt
chmod +x ~/.local/bin/stt
```

## Usage

The daemon needs root access to read keyboard input devices via evdev.

```bash
# Start the daemon
sudo -E stt

# Or run in background
sudo -E nohup stt > /tmp/stt.log 2>&1 &
```

Then hold **F8** anywhere, speak, and release. The transcribed text appears at your cursor.

### Stop the daemon

```bash
sudo pkill -f "python3.*stt"
```

### Only one instance

VoxKey uses a PID lock file (`/tmp/stt-daemon.pid`). If you try to start a second instance, it will tell you to stop the first one.

## Autostart on Login

To start VoxKey automatically when you log in:

```bash
# Allow passwordless sudo for the stt script
sudo bash -c 'echo "YOUR_USERNAME ALL=(root) NOPASSWD:SETENV: /usr/bin/python3 /home/YOUR_USERNAME/.local/bin/stt" > /etc/sudoers.d/99-stt && chmod 440 /etc/sudoers.d/99-stt'

# Create autostart entry
mkdir -p ~/.config/autostart
cat > ~/.config/autostart/stt.desktop << 'EOF'
[Desktop Entry]
Type=Application
Name=STT Daemon
Comment=Push-to-talk speech-to-text (F8)
Exec=sudo -E /usr/bin/python3 /home/YOUR_USERNAME/.local/bin/stt
Terminal=false
X-GNOME-Autostart-enabled=true
EOF
```

Replace `YOUR_USERNAME` with your actual username.

## Configuration

Edit the variables at the top of the `stt` script:

| Variable | Default | Description |
|----------|---------|-------------|
| `F8_CODE` | `66` | evdev keycode for the hotkey |
| `AUDIO_DEVICE` | `plughw:1,0` | ALSA capture device |
| `AUDIO_RATE` | `48000` | Sample rate in Hz |
| `MODEL` | `whisper-large-v3-turbo` | Groq Whisper model |

To find your audio device: `arecord -l`

## License

MIT
