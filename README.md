# Audio-Based Emotion Detection Using STM32 Embedded Edge AI

[cite_start]A low-latency, fully on-device Edge AI pipeline that classifies human emotional states directly from raw audio data[cite: 199, 202]. [cite_start]Utilizing **NanoEdge AI Studio** to generate an optimized Machine Learning knowledge base, the firmware captures microphone input, handles double-buffered signal normalization, and executes real-time micro-inference without cloud dependencies[cite: 199, 200, 202].

## 🛠️ Hardware Subsystem
* [cite_start]**Processor Unit:** STM32F410RBTx (ARM Cortex-M4 @ 84 MHz with Floating Point Unit) [cite: 234, 323, 324]
* [cite_start]**Sensor Core:** Analog MEMS Microphone Module [cite: 200, 308]
* [cite_start]**Host Link:** Onboard ST-LINK Virtual COM Port (UART2 @ 115200 baud, 8-N-1 configuration) [cite: 203, 338]

## 📊 Core Application Pipeline
1. [cite_start]**Isochronous Sampling:** A 32-bit hardware timer (`TIM5` Channel 1) triggers the successive-approximation `ADC1` at a precise sample rate ($f_{sample} \approx 16.13 \text{ kHz}$) to capture human voice bandwidth[cite: 263, 327, 328, 370, 374].
2. [cite_start]**DMA Ping-Pong Buffering:** `DMA2 Stream 0` continuously updates a circular 1024-word destination buffer[cite: 329, 360, 365]. [cite_start]Half-Transfer (`HalfCplt`) and Transfer-Complete (`Cplt`) interrupts isolate alternate 256-sample window frames[cite: 361, 366].
3. [cite_start]**Signal Normalization:** Digital audio frames are centered around zero by subtracting a specific DC-bias offset ($1310 \text{ counts}$) and scaled proportionally to match the expected input envelope of the machine learning engine[cite: 334, 367].
4. **Energy Gating & Inference:** The application monitors signal energy dynamically. [cite_start]If signal power sits beneath a specific background threshold, processing is bypassed; otherwise, it calls the `neai_classification()` routine to determine the active class index[cite: 376].

## 🧠 Model & Taxonomy
[cite_start]The statically linked classification engine maps the extracted time/frequency features into four trained emotional signatures[cite: 201, 314]:
* [cite_start]`0` ➡️ **Background / Neutral** [cite: 201]
* [cite_start]`1` ➡️ **Happy** [cite: 201]
* [cite_start]`2` ➡️ **Sad** [cite: 201]
* [cite_start]`3` ➡️ **Angry** [cite: 201]

## 🚀 Getting Started

### 1. Inter-Chip Mapping
| MEMS Microphone Pin | STM32 MCU Pin | Function |
|:---:|:---:|:---:|
| **VCC** | 3.3V | Target Power |
| **GND** | GND | Common Reference |
| **OUT** | PA0 | [cite_start]ADC1 Channel 0 Input [cite: 279]

### 2. Workspace Setup
* [cite_start]Clone the workspace and open it within **STM32CubeIDE**[cite: 212].
* [cite_start]Ensure the target specific library archive `libneai_audioemotions_2.a` is linked under `Project Properties -> C/C++ Build -> Settings -> MCU GCC Linker -> Libraries`[cite: 315, 355, 358].
* Flash the binaries over the integrated ST-LINK debugging interface.

### 3. Verification & Execution
[cite_start]Establish a serial connection to your board's active terminal device (e.g., `/dev/ttyACM0` or `COMx`) using your preferred client application at **115200-8-N-1**[cite: 203, 339]. [cite_start]Upon power-on reset, confirm the status logging reports a successful initialization handshake[cite: 420]:

```text
AI Knowledge Loaded Successfully
Detected: Normal
Detected: Happy
Detected: Happy
