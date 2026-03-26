# OpenClaw Android Companion Node - Build & Installation Guide

**Turn your Android phone into a powerful node** for OpenClaw.  
This allows all your AI agents to access the phone's **camera**, **microphone**, **SMS**, **notifications**, **contacts**, **calendar**, sensors, and more — natively over WebSocket.

> **Note**: The official Android Companion App is **not yet published** on Google Play. You must build it from source.

## Prerequisites

### On Ubuntu (headless server)
- Ubuntu 22.04 / 24.04 (or similar)
- Node.js 22+
- Git
- OpenJDK 17
- Android SDK Command Line Tools

### On Android Phone
- Android 10 or higher
- Allow installation from unknown sources

## Step 1: Install OpenClaw Gateway on Ubuntu

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install -y curl git build-essential ca-certificates

# Install Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Install OpenClaw
npm install -g openclaw@latest

# Onboarding (configure models, API keys, etc.)
openclaw onboard --install-daemon

# Start the gateway
openclaw gateway install
openclaw gateway start
```

### Check status:
```bash
openclaw gateway status
openclaw nodes status
```

Recommended: Install Tailscale on both Ubuntu and your Android phone for easy remote access.

## Step 2: Build the Android Companion Node APK
```bash
# Clone the repository (if not already done)
git clone https://github.com/openclaw/openclaw.git
cd openclaw/apps/android

# Install Java 17 (if not installed)
sudo apt install openjdk-17-jdk-headless -y

# Set JAVA_HOME
echo 'export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))' >> ~/.bashrc
source ~/.bashrc

# Set up Android SDK
mkdir -p ~/Android/Sdk/cmdline-tools
cd ~/Android/Sdk/cmdline-tools

# Download Command Line Tools (current version as of March 2026)
wget https://dl.google.com/android/repository/commandlinetools-linux-14742923_latest.zip
unzip commandlinetools-linux-14742923_latest.zip
mv cmdline-tools latest
rm commandlinetools-linux-14742923_latest.zip

# Add to PATH permanently
cat << 'EOF' >> ~/.bashrc
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
EOF

source ~/.bashrc

# Accept licenses and install SDK components
yes | sdkmanager --licenses
sdkmanager --update
sdkmanager "platform-tools" "platforms;android-34" "build-tools;34.0.0"

# Create local.properties
cd /opt/compaginion/openclaw/apps/android   # or your project path
echo "sdk.dir=$HOME/Android/Sdk" > local.properties

# Build the APK
./gradlew :app:assemblePlayDebug --stacktrace -Dorg.gradle.jvmargs="-Xmx2048m"
```

The APK will be located at:

```TEXT
app/build/outputs/apk/play/debug/app-play-debug.apk
```

## Step 3: Install & Connect on Android

1. Transfer the APK to your phone (scp, Tailscale, USB, etc.).
2. Install the APK (allow "Unknown sources").
3. Open OpenClaw Companion Node.
4. Grant all requested permissions (Camera, Microphone, SMS, Notifications, Contacts, etc.).
5. Go to Connect → Enter your Ubuntu Gateway address:
   - Local: ws://<ubuntu-ip>:18789
   - Remote: ws://<tailscale-ip>:18789 (recommended)
6. The app will request pairing.

## Step 4: Approve the Node on Ubuntu
```bash
# List pending devices
openclaw devices list

# Approve the Android node
openclaw devices approve <requestId>

# Check status
openclaw nodes status
openclaw nodes describe --node <node-name-or-id>
```

## Usage Examples
```bash
# Take a photo
openclaw nodes invoke --node android-phone --command camera.snap

# Send SMS
openclaw nodes invoke --node android-phone --command sms.send --params '{"to":"+41XXXXXXXX","message":"Hello from OpenClaw"}'

# List notifications
openclaw nodes invoke --node android-phone --command notifications.list
```

## Troubleshooting
- No connection: Use Tailscale or check firewall/port 18789.
- Commands fail in background: Keep the app in foreground (notification must stay active).
- Build errors: Ensure JAVA_HOME and ANDROID_HOME are set correctly.
- Permissions: All capabilities require explicit user approval on the phone.

## Useful Commands
```bash
openclaw gateway status
openclaw nodes status
openclaw devices list
openclaw logs
```
