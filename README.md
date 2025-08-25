ZPTTLink — Bridge Zello (BlueStacks) to Radio Hardware Fast 🚀

[![Releases](https://img.shields.io/badge/Releases-download-brightgreen)](https://github.com/MrZamasu/ZPTTLink/releases)

![ZPTTLink banner](https://upload.wikimedia.org/wikipedia/commons/6/6f/Radio_antenna_icon.png)

ZPTTLink connects Zello (running inside BlueStacks) to radio gateway hardware such as the AIOC (All-In-One Cable). It routes audio and controls Push-to-Talk (PTT) from your PC to USB radio interfaces. Use it to join digital voice workflows with RF gear.

Topics: aioc, bluestacks, cross-platform, gmrs, ham-radio, open-source, ptt, pttoverip, push-to-talk, python, radio-gateway, usb-radio, virtual-audio, zello, zpttlink

Table of contents
- Features
- How it works
- Supported hardware and platforms
- Quick start
- Install from source
- Configuration examples
- Audio routing and PTT wiring
- Troubleshooting
- FAQ
- Contributing
- License
- Releases

Features
- Bridge App-to-Radio audio between Zello and USB radio gateways.
- Control PTT over serial toggles (DTR/RTS) or GPIO on supported devices.
- Cross-platform: Windows, macOS, Linux.
- Python-based daemon and CLI for scripting and service mode.
- Virtual audio device support: route mic/speaker to BlueStacks and radio simultaneously.
- Low-latency mode for near real-time PTT handoff.
- Logs and debug levels for reproducible diagnosis.

How it works
- ZPTTLink listens for Zello PTT events or synthetic push events.
- It captures audio from a chosen input (virtual mic) and forwards it to the radio's audio channel.
- It asserts PTT via USB serial control lines or via vendor protocol for supported gateways (AIOC).
- It releases PTT when transmission ends and restores receive path to your speaker device.
- It can run as a foreground process, systemd/launchd service, or Windows service.

Supported hardware and platforms
- Radio gateway: AIOC (All-In-One Cable) and similar USB-radio interfaces.
- BlueStacks: runs Zello inside the emulator; ZPTTLink pairs with BlueStacks audio devices.
- OS: Windows 10/11, macOS 10.14+, Linux (Ubuntu/Debian flavors tested).
- Python: 3.8+ runtime.
- Virtual audio: VB-Audio, Soundflower, PulseAudio loopbacks, JACK.

Quick start
1. Install BlueStacks and Zello. Configure Zello to use a virtual microphone and virtual speaker that the host OS recognizes.
2. Prepare your radio gateway (AIOC). Connect it to the host via USB.
3. Download the release package and run the installer or binary:
   - Visit the releases page and download the correct asset for your OS.
   - The release file must be downloaded and executed.
4. Configure ZPTTLink for your audio devices and serial port.
5. Start ZPTTLink and press PTT inside Zello to transmit over RF.

Install from source
- Clone the repo and install dependencies:
  - git clone https://github.com/MrZamasu/ZPTTLink.git
  - cd ZPTTLink
  - python -m venv venv
  - venv\Scripts\activate (Windows) or source venv/bin/activate (macOS/Linux)
  - pip install -r requirements.txt
- Run the main program:
  - python -m zpttlink --config config.yaml
- Use system tools to run as a service for persistent usage.

Configuration examples
- Basic YAML config (config.yaml):
  device:
    audio_input: "Virtual-Audio-Cable 1"
    audio_output: "Speakers (High Definition Audio)"
    serial_port: "COM3"          # or /dev/ttyUSB0 on Linux
    ptt_control: "dtr"           # options: dtr, rts, vendor
    sample_rate: 48000
    buffer_size: 256
- Windows example:
  serial_port: "COM5"
  audio_input: "VB-Audio Virtual Cable"
- Linux example:
  serial_port: "/dev/ttyUSB0"
  audio_input: "alsa_output.pci-0000_00_1b.0.analog-stereo.monitor"
- CLI examples:
  - Start in foreground: python -m zpttlink --config config.yaml
  - Test audio loop: python -m zpttlink --test-audio --device "Virtual-Audio-Cable 1"

Audio routing and PTT wiring
- Audio:
  - Create a virtual mic device that BlueStacks uses for Zello.
  - Set ZPTTLink audio_input to that virtual mic.
  - Set audio_output to your speaker or monitor device.
  - On Windows use VB-Audio or VoiceMeeter. On macOS use Soundflower or BlackHole. On Linux use PulseAudio loopback or JACK.
- PTT control:
  - For AIOC and similar, ZPTTLink toggles DTR/RTS via the USB serial driver to assert PTT.
  - Alternate modes use vendor API or USB HID commands.
  - Example wiring: set serial DTR high to key transmit, clear to unkey.
  - Use the config option ptt_control to match your hardware mode.
- Latency:
  - Use 48 kHz and small buffer sizes for minimal delay.
  - Watch CPU when using tiny buffers; raise buffer_size if you get underruns.

Best practices
- Lock BlueStacks audio device to the virtual cable to avoid device switching.
- Run ZPTTLink before launching Zello for reliable device mapping.
- Test PTT with a local monitor and an attenuator on your radio if you plan on transmitting on-air.
- Use a neutral TX audio profile. Avoid clipping.

Troubleshooting
- No audio to radio:
  - Check that audio_input matches the virtual mic BlueStacks uses.
  - Confirm the radio input gains and levels.
  - Run the built-in audio test: python -m zpttlink --test-audio
- PTT does not key:
  - Confirm serial_port path and permissions.
  - Use a serial terminal to toggle DTR/RTS manually to confirm hardware behavior.
  - If using AIOC, confirm vendor mode in config.
- High latency:
  - Reduce buffer_size and use 48 kHz.
  - Use a dedicated virtual audio cable. Avoid mixing across many apps.
- Permission errors on Linux:
  - Add your user to dialout or plugdev group or add a udev rule for the device.
  - Example udev rule:
    SUBSYSTEM=="tty", ATTRS{idVendor}=="XXXX", ATTRS{idProduct}=="YYYY", MODE="0666", GROUP="dialout"
- BlueStacks audio grabs device:
  - Open BlueStacks audio settings and set the mic to the virtual cable explicitly.
  - Restart BlueStacks after changing virtual audio devices.

FAQ
Q: Does this work with hardware other than AIOC?
A: Yes. ZPTTLink supports any USB-radio interface that accepts serial control lines or vendor commands. If the device exposes a serial endpoint, configure serial_port and ptt_control.

Q: Can I run ZPTTLink on macOS?
A: Yes. Use BlackHole or Soundflower for virtual audio. Python 3.8+ runs the daemon.

Q: Do I need root or admin rights?
A: You need rights to open audio devices and serial ports. On Linux add proper udev rules or use sudo for initial setup.

Q: Will this rebroadcast digital voice streams over RF?
A: It will route the live audio you send via Zello to the radio. Respect regulations and frequencies before transmitting.

Contributing
- Fork the repo.
- Open an issue for any bug or feature request.
- Create a branch per change: feature/ or bugfix/.
- Write tests for new behavior.
- Submit pull requests with clear summaries.
- Keep changes small and focused.

Automated tests
- Use the test suite in tests/ and run:
  - pytest -q

Coding style
- Follow PEP8.
- Use type hints where helpful.
- Keep CLI flags stable for backward compatibility.

Assets, logos and images
- BlueStacks: use the emulator audio device in configs.
- Zello: set app mic to virtual cable.
- Radio: AIOC hardware expects serial control lines.

License
- Licensed under MIT. See LICENSE file.

Releases
[![Download Releases](https://img.shields.io/badge/Get%20Latest%20Release-Click%20Here-blue)](https://github.com/MrZamasu/ZPTTLink/releases)

Visit the releases page linked above. Download the release asset for your platform (example: zpttlink-windows-x64.exe, zpttlink-macos.tar.gz, zpttlink-linux.tar.gz) and execute the file or follow the packaged install instructions. The release file must be downloaded and executed for the packaged installer or binary to run.

Changelog and release notes live on the same page. Check the release you download for platform-specific instructions and bundled dependencies.