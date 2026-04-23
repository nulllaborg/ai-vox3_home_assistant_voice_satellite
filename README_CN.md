# Home Assistant 语音助手终端：AI-VOX3 配置指南

[English](README.md)

## 概述

AI-VOX3 是一款专为语音交互场景设计的智能硬件开发板。其硬件集成了语音输入（麦克风）、语音输出（扬声器接口）、状态显示屏（LCD）及交互按键，构成了一个功能完备的语音终端原型。将其接入 Home Assistant，即可打造一个**响应迅速、隐私得以保护**的本地智能家居语音控制终端。

**核心硬件概览**：

- **主控芯片**：采用**ESP32-S3-R8**芯片，并板载**16MB Flash**存储器
- **音频系统**：高品质 ES8311 音频编解码器，负责声音的采集与播放。
- **人机交互**：1.54英寸、240x240分辨率的LCD屏幕；三个功能按键及一个 WS2812 RGB LED，用于提供丰富的状态反馈与交互。
- **设计初衷**：本硬件专为语音识别与交互应用设计，充当 Home Assistant 的语音终端是其核心应用场景之一。

> 提示：本文档旨在提供清晰的软件配置指引。关于更详细的硬件规格、引脚定义等硬件信息，请参阅 AI-VOX3 的官方硬件文档。

---

## 工作原理

AI-VOX3 需要与您的 Home Assistant 服务器协同工作。理解双方的角色与分工，是后续使用和排查问题的基础。

### 设备端：专注的“耳朵”与“嘴巴”

AI-VOX3 的硬件专为语音交互设计，它在系统中扮演一个**高保真的音频网关**角色：

- **输入（听）**：通过麦克风采集环境声音，在设备端完成**唤醒词检测**（如“Okay Nabu”），并在唤醒后将您的语音指令编码、传输。
- **输出（说）**：接收来自 Home Assistant 的音频流，通过扬声器清晰播放。

**关键限制**：AI-VOX3 **不执行**任何语义理解或逻辑决策。它仅负责声音与数字信号之间的可靠转换与传输。

### 服务器端：智慧的“大脑”

所有的智能处理都发生在 **Home Assistant 服务器端**（或您配置的云端服务）。AI-VOX3 上传的语音数据，将进入一个名为 **“语音助手管道 (Voice Pipeline)”** 的流程：

1. **语音识别 (STT)**：将语音转换为文本。
2. **意图识别 (NLU)**：理解文本指令的意图（如“打开客厅灯”）。
3. **执行**：Home Assistant 调用相关服务执行指令（如控制灯光、查询传感器）。
4. **语音合成 (TTS)**：将执行结果生成语音回复，下发给设备播放。

### 您需要确保什么？

要使整个系统运转，您必须确保两端都已就绪：

1. **硬件与网络**：AI-VOX3 设备正常工作，并稳定接入家庭网络。（本指南确保此项）
2. **服务与配置**：您的 Home Assistant 中已正确配置**语音助手管道**。（您需自行完成）

> 核心配置指南：我们强烈建议您阅读官方文档，以理解语音助手的基础配置：[Home Assistant Voice Control](https://www.home-assistant.io/voice_control/)

**排查问题提示**：当出现“唤醒无反应”或“答非所问”时，此分工原理是您的排查地图：首先检查设备连接（“耳朵嘴巴”是否在线），然后检查 Home Assistant 的语音助手日志与配置（“大脑”是否工作正常）。

---

## 快速开始

本章将指导您完成 AI-VOX3 的基础固件部署。其核心流程是：**使用 ESPHome 工具，根据我们提供的配置文件，为您的 AI-VOX3 编译并烧录一个专用的固件**。

### 认识 ESPHome

[ESPHome](https://esphome.io/) 是一个用于配置 ESP32/ESP8266 系列微控制器的强大工具。它允许您通过简单的 YAML 配置文件定义设备功能，并自动编译、生成可与 Home Assistant 无缝集成的固件。

### 配置文件

您需要准备两个核心的配置文件：

1. [config.yaml](config.yaml)：这是设备功能的主配置文件。我们已经为您编写好，您可以直接使用。

2. `secrets.yaml`：这是您的私人配置文件，用于存放 Wi-Fi 名称、密码等敏感信息。**您需要根据自身网络情况创建并填写此文件**。

工作原理：ESPHome 在编译时，会先读取 `config.yaml`，然后自动将其中的 `!secret` 变量替换为 `secrets.yaml` 中定义的对应值，从而生成包含您个人信息的完整固件。

### 部署

#### 选择您的部署方式

ESPHome 工具为您提供了两种使用方式，其本质都是处理上述两个 YAML 文件：

| 方式 | 描述 | 适用场景 |
| :--- | :--- | :--- |
| 图形化工具 | 通过 Home Assistant 的 **ESPHome Device Builder** 插件在网页端操作。 | 适合所有用户，可视化操作，无需记忆命令。 |
| 命令行工具 | 在电脑终端中使用 `esphome` 命令。 | 适合开发者、高级用户或在无法使用图形插件时选用。 |

无论选择哪种方式，操作的本质都是让 ESPHome 读取并编译我们提供的 `config.yaml`​ 配置文件。图形化界面是将这些步骤转化为点击操作，而命令行则通过指令执行。

#### 方式一：通过`esphome`命令行部署

以下是使用 ESPHome 命令行工具进行部署的完整流程。

##### 安装 ESPHome 命令行工具

您需要在您的电脑上安装 ESPHome 命令行工具。

1. 请按照 [ESPHome 官方安装指南](https://esphome.io/guides/installing_esphome) 完成安装。

2. 安装完成后，请在终端中运行以下命令验证版本：

    ```shell
    esphome version
    ```

    **请确保输出的版本号大于等于 `2026.4.0`**。如果版本过低，请使用 `pip install --upgrade esphome` 命令进行升级。

##### 获取配置文件

1. 从本项目的 Release 页面下载最新的配置文件压缩包。
2. 将压缩包解压到您电脑上的任意目录（例如 ~/ai-vox3_home_assistant_voice_assistant）。
3. 打开终端，使用 cd命令进入解压后的目录。例如：

    ```shell
    cd ~/ai-vox3_home_assistant_voice_assistant
    ```

4. 确认当前目录下存在名为 `config.yaml` 的文件。

##### 配置 Wi-Fi 信息

AI-VOX3 仅支持连接 **2.4GHz** 频段的 Wi-Fi 网络。

1. 在 `config.yaml` 文件所在的目录中，创建一个名为 `secrets.yaml` 的新文件。
2. 使用文本编辑器打开 `secrets.yaml` 文件，并填入您的 Wi-Fi 信息，格式如下：

    ```yaml
    ---
    # 将引号内的内容替换为您的实际信息
    wifi_ssid: "Your wifi ssid"
    wifi_password: "Your wifi password"
    ```

*注意：请确保您的 Wi-Fi 是 2.4GHz 频段*。

##### 连接硬件

1. 使用 USB Type-C 数据线，将 AI-VOX3 设备连接到您的电脑。
2. 单击 AI-VOX3 板上的开机按键，启动设备。

##### 编译与上传固件

在终端中，确保位于包含配置文件的目录，然后执行：

```shell
esphome run config.yaml
```

此命令将：

1. **编译**：首次编译可能需要几分钟。
2. **选择端口**：编译成功后，按提示选择 AI-VOX3 对应的串行端口（如 `/dev/cu.usbmodemXXXX` 或 `COMX`）。
3. **上传**：程序将自动烧录固件。请保持设备连接稳定。

##### 确认部署成功

固件上传后，设备将自动重启。请观察：

1. **屏幕**：会亮起并显示启动动画。

2. **LED 指示灯**：闪烁红色，表示正在尝试连接 Wi-Fi。

3. **终端输出**：连接成功后，终端将显示设备获取到的 IP 地址，并会自动尝试接入同一网络下的 Home Assistant 实例。

至此，AI-VOX3 的设备端固件部署已完成。接下来，请前往 Home Assistant 中完成语音助手的配置。

---

#### 方式二：通过**ESPHome Device Builder**部署

此方式通过 Home Assistant 官方插件提供图形化界面。请参考 [官方指南](https://esphome.io/guides/getting_started_hassio/) 安装插件，随后在插件界面中导入我们提供的 [config.yaml](config.yaml) 配置文件（可通过上传文件或复制粘贴内容实现），并**新建 `secrets.yaml` 文件以填写您的 Wi-Fi 信息**，后续编译与部署步骤请遵循 ESPHome 插件的引导完成。本指南旨在提供适配 AI-VOX3 的配置文件，图形化界面的详细操作不再赘述，请以官方文档为准。

`secrets.yaml` **文件内容示例**：

```yaml
---
# 将引号内的内容替换为您的实际信息
wifi_ssid: "Your wifi ssid"
wifi_password: "Your wifi password"
```

---

### 在 Home Assistant 中添加并连接设备

完成固件部署后，您的 AI-VOX3 设备将尝试连接至 Wi-Fi 网络并与 Home Assistant 建立通信。本部分将指导您在 Home Assistant 界面中完成设备的最终添加与配对。

#### 连接状态指示

您可以通过设备上的 WS2812 LED 指示灯颜色来判断其当前的连接状态：

- **红灯闪烁**：设备正在尝试连接您在 `secrets.yaml` 中配置的 Wi-Fi 网络。
- **绿灯闪烁**：Wi-Fi 连接成功，设备已接入您的家庭网络，并正在等待与 Home Assistant 服务器建立连接。此时，您需要在 Home Assistant 中添加此设备。
- **白灯闪烁**：设备已被成功添加到 Home Assistant，但尚未连接到语音助手功能。
- **白灯常亮**：设备已完全就绪，成功连接到 Home Assistant 及其语音助手功能，可随时接受指令。

#### 在 Home Assistant 中添加设备

请按照以下步骤将 AI-VOX3 设备集成到您的 Home Assistant 中：

1. 可以在[Home Assistant的“设备与服务”集成页面](http://homeassistant.local:8123/config/integrations/dashboard)找到AI-VOX3设备

2. 点击“添加”， 然后按照屏幕提示完成添加。

    - 通常，您只需在出现的确认窗口中点击“提交”
    - 接下来，系统可能会提示您为此设备选择所属的“区域”。您可以选择一个合适的区域，或直接点击“跳过并完成”按钮

3. 连接语音助手

    成功添加设备后，其指示灯应进入 **白灯闪烁** 状态。这表示设备已与 Home Assistant 的核心系统连接，但尚未绑定到语音助手。

4. 完成连接

如果您已经在 Home Assistant 中完成了语音助手（Voice Assistant）的管道配置，AI-VOX3 将自动连接至语音助手功能。一旦连接成功，设备指示灯将变为 **白灯常亮**，这意味着您的 AI-VOX3 语音终端已经完全就绪，可以开始工作。

### 在 Home Assistant 中配置语音助手

要使用 AI-VOX3 的语音功能，您需要在 Home Assistant 中完成语音助手（Assist）的配置。其核心是建立一个处理语音的“管道”（Pipeline），该管道负责唤醒、识别、理解和回复。

对于想最快验证设备是否工作的用户，我们推荐遵循官方最简化的指引，使用 Home Assistant Cloud 服务来构建管道。这省去了自行搭建语音识别与合成服务的复杂步骤。

**官方配置指南**：Home Assistant 提供了非常全面的配置说明，建议您以官方文档为主进行配置：[配置云端语音助手 \| Home Assistant](https://www.home-assistant.io/voice_control/voice_remote_cloud_assistant/)

以下是在官方指南基础上的关键步骤和注意事项：

1. 启用 Home Assistant Cloud

    如果您尚未启用，请在 Home Assistant 的“配置”中设置并连接 Home Assistant Cloud。

2. 创建或编辑语音助手

    - 进入 “设置” → “语音助手”，创建新助手或编辑现有助手。

    - 在管道配置中，请进行如下设置：

        - 对话代理：选择 Home Assistant Cloud

        - 文字转语音：选择 Home Assistant Cloud

    - 关于唤醒词的重要说明：

    - 无需配置“流媒体唤醒词”。因为 AI-VOX3 支持在设备端进行唤醒词检测，您在此处无需进行任何设置。

    - 请确保 AI-VOX3 的设备配置中，`Wake word engine location` 选项为 `On device`（此为默认值）。这意味着唤醒词（如“Okay Nabu”）的检测完全在设备本地完成，响应迅速且保护隐私。

3. 保存并连接

    - 完成配置后，点击“更新”。

    - 保存后，您的 AI-VOX3 将自动连接到新配置的语音助手管道。连接成功后，设备上的 WS2812 LED 指示灯会变为**常亮白色**，表示设备已就绪，正在等待您的语音唤醒。

### 测试与验证

完成所有配置后，您可以通过一个简单的测试流程来验证 AI-VOX3 语音助手是否已完全正常工作。

#### 预期测试流程

1. 唤醒设备

    - 在安静的环境中，对着 AI-VOX3 说出唤醒词：“Okay Nabu”。

2. 下达指令

    - 唤醒成功后，设备板载的 WS2812 LED 指示灯将变为紫色，表示设备正处于“监听”状态，正在接收您的语音指令。
    - 此时，您可以清晰地提出一个问题，例如：“现在几点？” 或 “时间？”。

3. 接收回复与状态循环

    - Home Assistant 接收到指令后，会进行处理并生成语音回复。
    - 当设备开始播放语音回复时，LED 指示灯将变为黄色，表示处于“播报”状态。
    - 播报完成后，设备将自动恢复到待机状态（LED 变为白色常亮）。您可以再次说出唤醒词，进行新一轮的交互。

**测试成功标志**：

如果整个流程——**唤醒（亮紫灯）→ 监听指令 → 接收并播报回复（亮黄灯） → 恢复待机（白灯常亮）**——能够顺畅执行，并且您能听到准确的时间播报，则证明您的 AI-VOX3 已成功接入语音助手，所有功能运行正常。

## 流媒体唤醒词配置

AI-VOX3 支持两种语音唤醒模式，您可以根据对隐私、响应速度或自定义程度的不同偏好进行选择。

1. 设备端唤醒（推荐，默认模式）

    在此模式下，唤醒词（例如“Okay Nabu”）的检测完全在 **AI-VOX3 设备本地** 完成。

    **优点**：
    - **响应迅速**：检测无需网络往返，唤醒延迟极低。
    - **保护隐私**：您的日常对话语音仅在唤醒后才会被发送至服务器处理。
    - **节省资源**：减轻了Home Assistant服务器的计算负担。

    **配置方法**：
    - 确保在AI-VOX3的配置中，`Wake word engine location` 选项设置为 `On device`。此为该选项的默认值，通常无需更改。

2. 流媒体唤醒（Home Assistant端唤醒）

    在此模式下，AI-VOX3会将麦克风的音频流持续发送至Home Assistant服务器，由服务器端进行唤醒词检测。

    - **适用场景**：当您需要使用设备端不支持的自定义唤醒词时，需使用此模式。

    **配置方法**：
    - 在Home Assistant的 “设置” → “语音助手” 中，为您使用的语音助手管道添加并配置“流媒体唤醒词”。
    - 在AI-VOX3的设备配置中，将 `Wake word engine location` 选项修改为 `In Home Assistant` 。

### 重要提示与已知问题

- **关于模式选择**：两种模式各有侧重，请根据您的核心需求（隐私/速度 vs. 自定义唤醒词）选择一种模式使用，不建议频繁切换。

- **已知问题**： 在 `In Home Assistant` 模式下，设备在唤醒时可能会在日志中报告以下错误，导致唤醒失败：

    ```text
    [E][voice_assistant:806]: Error: duplicate_wake_up_detected - Duplicate wake-up detected for Okay Nabu
    ```

    此问题目前尚未定位根本原因。临时解决方案是：在Home Assistant中完成流媒体唤醒词配置并切换模式后，重启一次您的AI-VOX3设备，这通常可使流媒体唤醒功能恢复正常工作。

## 高级配置

验证成功后，您便可以开始探索更丰富的语音助手功能，例如控制智能设备、查询天气、设置提醒等。更高级的语音助手设置（如自定义语音、复杂自动化）请查阅 Home Assistant 官方语音助手文档。同时，您也可以通过修改 `config.yaml` 配置文件，来自定义设备的唤醒词、显示界面和 LED 行为，实现个性化 DIY。

**相关文档链接**：

- [Home Assistant Voice Assistant](https://www.home-assistant.io/voice_control)
- [ESPHome Voice Assistant Component](https://esphome.io/components/voice_assistant)
