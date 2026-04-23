# Setting Up AI-VOX3 as a Home Assistant Voice Satellite

[中文](README_CN.md)

## Overview

The AI-VOX3 is an intelligent hardware development board designed specifically for voice interaction scenarios. Its hardware integrates voice input (microphone), voice output (speaker interface), a status display screen (LCD), and interactive buttons, forming a ready-to-use voice satellite. By connecting it to Home Assistant, you can create a **fast-responding, privacy-preserving** local smart home voice control assistant.

**Core Hardware Overview**:

- **Main Control Chip**: Features the **ESP32-S3-R8** chip with onboard **16MB Flash** memory.
- **Audio System**: High-quality ES8311 audio codec, responsible for sound capture and playback.
- **Human-Machine Interaction**: A 1.54-inch, 240x240 resolution LCD screen; three functional buttons and one WS2812 RGB LED, providing rich status feedback and interaction.
- **Design Purpose**: This hardware is designed for voice recognition and interaction applications. Acting as a **Home Assistant voice satellite** is one of its core application scenarios.

> Note: This document aims to provide clear software configuration guidance. For more detailed hardware specifications, pin definitions, and other hardware information, please refer to the official AI-VOX3 hardware documentation.

---

## System Architecture

The AI-VOX3 needs to work in conjunction with your Home Assistant server. Understanding the roles and division of labor between the two is fundamental for subsequent use and troubleshooting.

### Device Role: The Satellite

The AI-VOX3 hardware is designed for voice interaction, acting as a **high-fidelity audio gateway** in the system:

- **Input (Hear)**: Captures ambient sound via the microphone, performs **wake word detection** locally on the device (e.g., "Okay Nabu"), and upon waking, encodes and transmits your voice command.
- **Output (Speak)**: Receives the audio stream from Home Assistant and plays it clearly through the speaker.

**Key Limitation**: The AI-VOX3 **does not perform** any semantic understanding or logical decision-making. It is solely responsible for the reliable conversion and transmission of sound to/from digital signals.

### Server Side: Home Assistant Assist​

All intelligent processing (speech recognition, intent understanding) occurs within your **Home Assistant instance** (which can be on a local server or a cloud service). The voice data uploaded by AI-VOX3 enters a process called the **"Voice Assistant Pipeline"**:

1. **Speech-to-Text (STT)**: Converts speech to text.
2. **Intent Recognition (NLU)**: Understands the intent of the text command (e.g., "turn on the living room light").
3. **Execution**: Home Assistant calls the relevant services to execute the command (e.g., controlling lights, querying sensors).
4. **Text-to-Speech (TTS)**: Generates a voice reply from the execution result and sends it down to the device for playback.

### Requirements

Your AI-VOX3 voice satellite requires two components to work in tandem:

1. **A Functional AI-VOX3 Device**: The device must be powered on and connected to your local Wi-Fi network. *(This guide covers this setup.)*
2. **A Configured Home Assistant with Assist**: Your Home Assistant server must have **Assist** (the voice assistant) configured with a functional pipeline. *(This is a separate setup. See the [official Assist documentation](https://www.home-assistant.io/voice_control/) to get started.)*

**Troubleshooting Tip**: This division of labor is your debugging map. For issues like “no wake-up” or “incorrect replies”:

- First, verify the device and network connection (are the “ears and mouth” online?).
- Then, check the logs and configuration of Assist in Home Assistant (is the “brain” processing correctly?).

---

## Understanding ESPHome

[ESPHome](https://esphome.io/) is a powerful tool for configuring ESP32/ESP8266 series microcontrollers. It allows you to define device functions through simple YAML configuration files and automatically compiles and generates firmware that integrates seamlessly with Home Assistant.

## Configuration Files

You need to prepare two core configuration files:

1. [config.yaml](config.yaml): This is the main configuration file for device functionality. We have prepared it for you, and you can use it directly.
2. `secrets.yaml`: This is your private configuration file for storing sensitive information like Wi-Fi name and password. **You need to create and fill in this file according to your network situation**.

**How it works**: During compilation, ESPHome first reads `config.yaml`, then automatically replaces the `!secret` variables within it with the corresponding values defined in `secrets.yaml`, thereby generating the complete firmware containing your personal information.

## Firmware Deployment

### Choosing Your Deployment Method

The ESPHome tool offers you two ways to use it, both essentially processing the two YAML files mentioned above:

- Graphical Tool: Use the ESPHome add-on for a web-based, visual workflow.
- Command Line Tool: Use the esphome CLI for direct control and scripting.

Regardless of the method chosen, the essence of the operation is to have ESPHome read and compile the `config.yaml` configuration file we provide. The graphical interface converts these steps into click operations, while the command line executes them via instructions.

### Command Line Deployment​

The following is the complete process for deployment using the ESPHome command-line tool.

#### Install the ESPHome Command Line Tool

1. Please follow the [Install ESPHome](https://esphome.io/guides/installing_esphome) to complete the installation.
2. After installation, run the following command in the terminal to verify the version:

    ```shell
    esphome version
    ```

**Please ensure the output version number is greater than or equal to `2026.4.0`**. If the version is too low, use the `pip install --upgrade esphome` command to upgrade.

#### Obtain the Configuration Files

1. Download the latest configuration file archive from this project's Release page.
2. Extract the archive to any directory on your computer (e.g., `~/ai-vox3_home_assistant_voice_assistant`).
3. Open a terminal and use the `cd` command to enter the extracted directory. For example:

    ```shell
    cd ~/ai-vox3_home_assistant_voice_assistant
    ```

4. Confirm that a file named `config.yaml` exists in the current directory.

#### Configure Wi-Fi Information

AI-VOX3 requires a **2.4GHz** Wi-Fi network. (Its Wi-Fi chip does not support 5GHz bands.)

1. In the directory containing the `config.yaml` file, create a new file named `secrets.yaml`.
2. Open the `secrets.yaml` file with a text editor and fill in your Wi-Fi information in the following format:

    ```yaml
    ---
    # Replace the content inside quotes with your actual information
    wifi_ssid: "Your wifi ssid"
    wifi_password: "Your wifi password"
    ```

*Note: Please ensure your Wi-Fi is on the 2.4GHz band.*

#### Connect the Hardware

1. Connect the AI-VOX3 device to your computer using a USB Type-C cable.
2. Click the power button on the AI-VOX3 board to start the device.

#### Compile and Upload the Firmware

In the terminal, ensure you are in the directory containing the configuration files, then execute:

```shell
esphome run config.yaml
```

This command will:

1. **Compile**: The first compilation may take a few minutes.
2. **Select Port**: After successful compilation, you will be prompted to select the serial port corresponding to AI-VOX3 (e.g., `/dev/cu.usbmodemXXXX` or `COMX`).
3. **Upload**: The program will automatically flash the firmware. Please keep the device connection stable.

#### Confirm Successful Deployment

After the firmware is uploaded, the device will automatically restart. Please observe:

1. **Screen**: Will light up and display the boot animation.
2. **LED Indicator**: Will flash **red**, indicating it is attempting to connect to Wi-Fi.
3. **Terminal Output**: Upon successful connection, the terminal will display the IP address obtained by the device, and it will automatically attempt to connect to the Home Assistant instance on the same network.

At this point, the device-side firmware deployment for AI-VOX3 is complete. Next, proceed to Home Assistant to complete the voice assistant configuration.

---

### Graphical Interface Deployment

This method provides a graphical interface through the official Home Assistant add-on. Please refer to the [Official Guide](https://esphome.io/guides/getting_started_hassio/) to install the add-on, then import our provided [config.yaml](config.yaml) configuration file in the add-on interface (either by uploading the file or copying and pasting the content), and **create a new `secrets.yaml` file to fill in your Wi-Fi information**. Subsequent compilation and deployment steps should be followed as guided by the ESPHome add-on. This guide aims to provide configuration files adapted for AI-VOX3; detailed operations for the graphical interface are not covered here, please refer to the official documentation.

`secrets.yaml` **File Content Example**:

```yaml
---
# Replace the content inside quotes with your actual information
wifi_ssid: "Your wifi ssid"
wifi_password: "Your wifi password"
```

---

## Integrate with Home Assistant

After completing the firmware deployment, your AI-VOX3 device will attempt to connect to the Wi-Fi network and establish communication with Home Assistant. This section will guide you through the final addition and pairing of the device in the Home Assistant interface.

### Connection Status Indication

You can judge the current connection status of the device by the color of the onboard WS2812 LED indicator:

- **Red Blinking**: The device is attempting to connect to the Wi-Fi network configured in your `secrets.yaml`.
- **Green Blinking**: Wi-Fi connected. The device is on the network and discoverable by​ Home Assistant.
- **White Blinking**: The device has been successfully added to Home Assistant but is not yet connected to the voice assistant function.
- **White Solid**: The device is fully ready, successfully connected to Home Assistant and ready for voice commands.

### Adding the Device in Home Assistant

Follow these steps to integrate the AI-VOX3 device into your Home Assistant:

1. You can find the AI-VOX3 device on the [Home Assistant "Devices & Services" integrations page](http://homeassistant.local:8123/config/integrations/dashboard).
2. Click "Add", then follow the on-screen prompts to complete the addition.
    - Usually, you just need to click "Submit" in the confirmation window that appears.
    - Next, the system may prompt you to select a "Zone" for this device. You can choose a suitable zone or simply click the "Skip and finish" button.
3. Connecting to Voice Assistant
    After successfully adding the device, its indicator should enter the **White Blinking** state. This means the device is connected to Home Assistant's core system but is not yet bound to the voice assistant.
4. Completing the Connection
    If you have already configured the Voice Assistant pipeline in Home Assistant, AI-VOX3 will automatically connect to the voice assistant function. Once the connection is successful, the device indicator will change to **White Solid**, meaning your AI-VOX3 voice satellite is fully ready to start working.

## Configure Assist​

### Basic Cloud Setup

To use the voice function of AI-VOX3, you need to complete the configuration of the voice assistant (Assist) in Home Assistant. The core is to establish a voice processing "Pipeline" responsible for wake-up, recognition, understanding, and reply.

For users who want to verify device functionality as quickly as possible, we recommend following the official simplified guidance to build the pipeline using the Home Assistant Cloud service. This saves the complex steps of setting up your own speech recognition and synthesis services.

**Official Configuration Guide**: Home Assistant provides comprehensive configuration instructions. It is recommended to use the official documentation as the primary resource: [Configure Cloud Voice Assistant \| Home Assistant](https://www.home-assistant.io/voice_control/voice_remote_cloud_assistant/)

Here are the key steps and considerations based on the official guide:

1. Enable Home Assistant Cloud
    If you haven't already, set up and connect Home Assistant Cloud in the Home Assistant "Configuration".
2. Create or Edit a Voice Assistant
    - Go to "Settings" → "Voice Assistant", create a new assistant, or edit an existing one.
    - In the pipeline configuration, make the following settings:
        - Conversation Agent: Select **Home Assistant Cloud**.
        - Text-to-Speech: Select **Home Assistant Cloud**.
    - Important Note on Wake Words:
        - **No need to configure a "Streaming Wake Word"**. Because AI-VOX3 supports wake word detection on the device side, you do not need to configure anything here.
        - Ensure the `Wake word engine location` option in the AI-VOX3 device configuration is set to **`On device`** (this is the default). This means wake word detection (e.g., for "Okay Nabu") is performed entirely locally on the device, ensuring fast response and privacy protection.
3. Save and Connect
    - After completing the configuration, click "Update".
    - After saving, your AI-VOX3 will automatically connect to the newly configured voice assistant pipeline. Upon successful connection, the WS2812 LED indicator on the device will become **Solid White**, indicating the device is ready and waiting for your voice wake-up.

### Select Wake Word Mode

AI-VOX3 supports two voice wake-up modes. You can choose based on your preferences for privacy, response speed, or customizability.

1. On-Device Wake-Up (Recommended, Default Mode)

    In this mode, wake word detection (e.g., "Okay Nabu") is performed entirely **locally on the AI-VOX3 device**.

    - **Advantages**:
        - **Fast Response**: Detection requires no network round-trip, resulting in very low wake-up latency.
        - **Privacy Protection**: Your everyday conversation audio is only sent to the server after wake-up.
        - **Saves Resources**: Reduces the computational load on the Home Assistant server.
    - **Configuration Method**:
        - Ensure the `Wake word engine location` option in the AI-VOX3 configuration is set to **`On device`**. This is the default value for this option and usually requires no change.

2. Streaming Wake-Up (Home Assistant Side Wake-Up)

    In this mode, AI-VOX3 continuously sends the microphone audio stream to the Home Assistant server, where wake word detection is performed.

    - **Use Case**: Use this mode when you need to use a custom wake word not supported on the device side.
    - **Configuration Method**:
        - In Home Assistant's "Settings" → "Voice Assistant", add and configure a "Streaming Wake Word" for the voice assistant pipeline you are using.
        - In the AI-VOX3 device configuration, change the `Wake word engine location` option to **`In Home Assistant`**.

#### Important Notes and Known Issues

- **Choosing a Mode:**: The two modes have different emphases. Please choose one mode to use based on your core needs (privacy/speed vs. custom wake word). Frequent switching is not recommended.
- **Known Issue**: In **`In Home Assistant`** mode, the device may report the following error in the logs upon wake-up, causing wake-up failure:

    ```text
    [E][voice_assistant:806]: Error: duplicate_wake_up_detected - Duplicate wake-up detected for Okay Nabu
    ```

This is a known issue with the current integration. The workaround is to restart the AI-VOX3 device after changing the wake word mode. This usually restores normal streaming wake-up functionality.

## Test the Setup

After completing all configurations, you can verify that the AI-VOX3 voice assistant is fully functional through a simple test process.

### Expected Test Process

1. Wake Up the Device
    - In a quiet environment, speak the wake word to AI-VOX3: "Okay Nabu".
2. Issue a Command
    - Upon successful wake-up, the onboard WS2812 LED indicator will turn **purple**, indicating the device is in the "Listening" state, receiving your voice command.
    - At this point, you can clearly ask a question, for example: "What time is it?" or "Time?".
3. Receive Reply and State Cycle
    - Home Assistant will receive the command, process it, and generate a voice reply.
    - When the device starts playing the voice reply, the LED indicator will turn **yellow**, indicating the "Speaking" state.
    - After playback finishes, the device will automatically return to the standby state (LED changes to solid white). You can say the wake word again for a new round of interaction.

**Test Success Criteria**:

If the entire process—**Wake word detected (Purple) → Listening for command → Playing reply (Yellow) → Standby (Solid White)**—executes smoothly, and you can hear the correct time announcement, it proves your AI-VOX3 has been successfully set up as a voice satellite and is working with Assist.

## Next Steps & Advanced Configuration​

Next Steps: Once successfully verified, you can start exploring richer voice assistant features, such as controlling smart devices, querying weather, setting reminders, etc. For more advanced voice assistant settings (like custom voices, complex automations), please refer to the official Home Assistant voice assistant documentation. You can also customize the device's wake word, display interface, and LED behavior by modifying the `config.yaml` configuration file for personalized DIY.

**Related Documentation Links**:

- [Home Assistant Voice Assistant](https://www.home-assistant.io/voice_control)
- [ESPHome Voice Assistant Component](https://esphome.io/components/voice_assistant)
