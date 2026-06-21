# 🏠 Local Android Emulator Setup Guide

Want to run the Android emulator locally? Follow this guide!

## Prerequisites

### System Requirements

- **OS**: Linux, macOS, or Windows (with WSL2)
- **CPU**: Intel/AMD with virtualization support (VT-x/AMD-V)
- **RAM**: At least 4GB (8GB recommended)
- **Disk Space**: 5GB free space minimum
- **Java**: JDK 8 or higher

### Windows Users

Install WSL2 and Ubuntu:
```bash
wsl --install
```

## Installation Steps

### 1. Install Java Development Kit (JDK)

**macOS:**
```bash
brew install openjdk@11
export JAVA_HOME=$(/usr/libexec/java_home -v 11)
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt-get update
sudo apt-get install openjdk-11-jdk
```

**Windows (WSL2):**
```bash
sudo apt-get update
sudo apt-get install openjdk-11-jdk
```

### 2. Install Android SDK

Download from: https://developer.android.com/studio

**Or use command line:**

```bash
# Linux/macOS
mkdir -p ~/android-sdk
cd ~/android-sdk
wget https://dl.google.com/android/repository/commandlinetools-linux-9123335_latest.zip
unzip commandlinetools-linux-9123335_latest.zip

# Add to PATH
export ANDROID_SDK_ROOT=~/android-sdk
export PATH=$ANDROID_SDK_ROOT/cmdline-tools/latest/bin:$PATH
```

### 3. Install SDK Components

```bash
# Accept licenses
yes | sdkmanager --licenses

# Install necessary components
sdkmanager "platforms;android-30"
sdkmanager "platforms;android-28"
sdkmanager "platforms;android-24"
sdkmanager "build-tools;30.0.3"
sdkmanager "emulator"
sdkmanager "system-images;android-30;default;x86_64"
sdkmanager "system-images;android-28;default;x86_64"
sdkmanager "system-images;android-24;default;x86_64"
```

### 4. Create Virtual Devices (AVD)

```bash
# Android 11 (API 30)
avdmanager create avd \
  -n "Android_30" \
  -k "system-images;android-30;default;x86_64" \
  -d "Nexus 5"

# Android 9 (API 28)
avdmanager create avd \
  -n "Android_28" \
  -k "system-images;android-28;default;x86_64" \
  -d "Nexus 5"

# Android 7 (API 24)
avdmanager create avd \
  -n "Android_24" \
  -k "system-images;android-24;default;x86_64" \
  -d "Nexus 5"
```

### 5. Enable KVM (Linux only)

```bash
# Check if KVM is available
grep -c '^flags.*\bvmx\b' /proc/cpuinfo  # Intel
grep -c '^flags.*\bsvm\b' /proc/cpuinfo  # AMD

# Install KVM
sudo apt-get install qemu-kvm libvirt-bin libvirt-clients

# Add user to kvm group
sudo usermod -a -G kvm $USER
newgrp kvm
```

## Running the Emulator

### Start Emulator

```bash
# Android 30
emulator -avd Android_30 &

# Android 28
emulator -avd Android_28 &

# Android 24
emulator -avd Android_24 &
```

### Verify Emulator is Running

```bash
adb devices

# Output should show:
# List of attached devices
# emulator-5554          device
```

### ADB Commands

```bash
# List running devices
adb devices

# Shell access
adb shell

# Install APK
adb install myapp.apk

# Launch app
adb shell am start -n com.example.app/.MainActivity

# Take screenshot
adb shell screencap -p /sdcard/screenshot.png
adb pull /sdcard/screenshot.png

# View logs
adb logcat

# Check system properties
adb shell getprop ro.build.version.release
adb shell getprop ro.product.model
```

## Performance Optimization

### Increase Resources

Edit `~/.android/avd/Android_30/config.ini`:

```ini
hw.ramSize=2048
hw.diskSize=2048
hw.gpu=yes
```

### Enable GPU Acceleration

```bash
# Check GPU support
glxinfo | grep "direct rendering"

# Set in AVD config
hw.gpu=yes
```

### Headless Mode (No GUI)

```bash
emulator -avd Android_30 -no-window -no-audio &
```

## Troubleshooting

### Emulator won't start

1. Check virtualization is enabled in BIOS
2. Verify KVM is working: `kvm-ok` (Linux)
3. Try different API level
4. Increase RAM allocation

### KVM not available

```bash
# Linux only - enable KVM
sudo modprobe kvm
sudo modprobe kvm_intel  # or kvm_amd for AMD
```

### Performance issues

- Use smaller emulator profile (Nexus 5 vs Nexus 10)
- Reduce RAM/disk allocation in config
- Enable GPU acceleration
- Close other applications

### ADB can't find device

```bash
# Restart ADB server
adb kill-server
adb start-server
adb devices
```

## Quick Script

Save as `start-emulator.sh`:

```bash
#!/bin/bash

API_LEVEL=${1:-30}
AVD_NAME="Android_${API_LEVEL}"

echo "Starting Android Emulator - API $API_LEVEL..."
emulator -avd $AVD_NAME -no-snapshot-load &

echo "Waiting for emulator to boot..."
sleep 15

echo "Verifying connection..."
adb wait-for-device
adb devices

echo "✅ Emulator ready!"
```

Run:
```bash
chmod +x start-emulator.sh
./start-emulator.sh 30  # Start Android 11 emulator
```

## Advanced Usage

### Test Multiple Versions

```bash
#!/bin/bash
for API in 24 28 30; do
    echo "Testing on API $API..."
    emulator -avd Android_$API &
    sleep 20
    adb devices
    adb shell getprop ro.build.version.release
    adb emu kill
    sleep 5
done
```

### Network Testing

```bash
# Access host machine from emulator
adb shell ping 10.0.2.2

# Port forwarding
adb forward tcp:8080 tcp:8080
```

## Resources

- [Android Studio Emulator Docs](https://developer.android.com/studio/run/emulator)
- [Android SDK Tools](https://developer.android.com/tools)
- [KVM Setup Guide](https://help.ubuntu.com/community/KVM)

---

**Happy Emulating!** 🚀📱
