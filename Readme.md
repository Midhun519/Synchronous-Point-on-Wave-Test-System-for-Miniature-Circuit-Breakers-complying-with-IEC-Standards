# Synchronous Point-on-Wave Test System for Miniature Circuit Breakers Complying with IEC Standards


An advanced cyber-physical **Hardware-in-the-Loop (HIL) Point-on-Wave (POW) Test System** designed to synchronize switching angles, measure voltage telemetry, and simulate continuous thermal energy accumulation (I^2t) for Miniature Circuit Breakers (MCBs) in strict compliance with **IEC standard protection curves**, eliminating physical high-current testing hazards.

---

## 📌 1. Project Overview & Description

Testing mechanical circuit breakers under arbitrary point-on-wave conditions typically requires high fault-current generators, which are expensive, dangerous, and physically destructive to test specimens. 

This project implements a low-voltage **Digital Twin Test Bed** using an **ESP32 Microcontroller**. The system:
* Synchronizes phase-angle triggering directly with the AC power grid via zero-crossing detection.
* Dynamically adjusts conduction/firing angles ($\gamma \in [0^\circ, 360^\circ]$) using a rotary encoder interface.
* Models real-time bimetallic heating and natural dissipation dynamics using a discretized continuous LTI thermal model ($I^2t$).
* Provides real-time dual feedback through a local **0.96" SSD1306 OLED graphical waveform display** and a remote, dark-themed **IoT Web Dashboard** featuring a 2D interactive MCB UI.

---

## 🔬 2. Core Mathematical & Control Concepts

### 2.1 First-Order Continuous LTI Thermal Model
The internal heat accumulation and cooling dissipation of an MCB's bimetallic element are governed by a continuous first-order Linear Time-Invariant (LTI) transfer function:

$$G(s) = \frac{K}{\tau s + 1}$$

Where:
* $K$: Thermal gain constant (steady-state temperature rise per unit power).
* $\tau$: Thermal time constant (seconds) representing physical thermal inertia.

### 2.2 Discrete-Time Model Formulation (Backward Euler Method)
To execute continuous differential thermal equations on an ESP32 sampled at discrete intervals $\Delta t = 20\text{ ms}$ (50 Hz grid cycle):

$$\frac{x[k] - x[k-1]}{\Delta t} = -\frac{1}{\tau} x[k] + \frac{K}{\tau} u[k]$$

Rearranging into a discrete-time state update equation yields:

$$x[k] = \left(\frac{\tau}{\tau + \Delta t}\right) x[k-1] + \left(\frac{K \cdot \Delta t}{\tau + \Delta t}\right) u[k]$$

Where the thermal stress input $u[k]$ is proportional to the square of the conduction-angle-modulated current:

$$u[k] \propto \sin^2(\theta)$$

---

## 🏗️ 3. System Block Diagram

```text
 ┌──────────────────────┐      ┌─────────────────────────┐
 │   AC Mains Supply    │─────▶│  Step-Down Transformer  │
 └──────────────────────┘      └────────────┬────────────┘
                                            │
                     ┌──────────────────────┴──────────────────────┐
                     ▼                                             ▼
       ┌───────────────────────────┐                 ┌───────────────────────────┐
       │   ZMPT101B Voltage Sensor │                 │  H11AA1 Zero-Cross Circuit │
       └─────────────┬─────────────┘                 └─────────────┬─────────────┘
                     │                                             │
                     └──────────────────────┬──────────────────────┘
                                            ▼
 ┌──────────────────────┐      ┌─────────────────────────────────────────────────┐
 │ KY-040 Rotary Encoder│─────▶│           ESP32 Microcontroller Core            │
 │ (Fault Severity /    │      │                                                 │
 │  Conduction Angle)   │      │   ┌─────────────────────────────────────────┐   │
 └──────────────────────┘      │   │          MCB DIGITAL TWIN CORE          │   │
                               │   │  • Zero-Cross Synchronization (ISR)     │   │
                               │   │  • Discretized LTI Thermal Model        │   │
                               │   │    (Backward Euler I²t Accumulation)    │   │
                               │   │  • IEC Inverse-Time Trip Logic          │   │
                               │   └─────────────────────────────────────────┘   │
                               └────────────┬─────────────────────────┬──────────┘
                                            │                         │
                                            ▼                         ▼
                               ┌─────────────────────────┐ ┌─────────────────────────┐
                               │  0.96" OLED Display     │ │ Indicator LEDs &        │
                               │  (Phase Angle & Status) │ │ IoT Web Server Interface│
                               └─────────────────────────┘ └─────────────────────────┘
```

> **Architecture Overview:** Low-voltage AC telemetry and zero-cross synchronization feed directly into an ESP32 discrete thermal accumulator ($I^2t$), allowing real-time inverse-time trip curve validation without high-current hardware destruction.

---

## 🔌 4. Hardware Component Connections

| Component / Subsystem | Hardware Module | ESP32 Pin Assignment | Functional Description |
| :--- | :--- | :--- | :--- |
| **Zero-Cross Detector** | H11AA1 Optocoupler | `GPIO 34` (Input) | External hardware interrupt line for AC zero-voltage crossing |
| **Voltage Telemetry** | ZMPT101B Module | `GPIO 35` (ADC Input) | Samples live AC line voltage amplitude |
| **Encoder Clock Signal**| KY-040 (CLK) | `GPIO 18` (Input Pull-up) | Detects phase angle adjustment steps |
| **Encoder Data Signal** | KY-040 (DT) | `GPIO 19` (Input Pull-up) | Determines rotation direction (increment/decrement) |
| **Encoder Push-Button**| KY-040 (SW) | `GPIO 23` (Input Pull-up) | Resets the virtual thermal trip state |
| **OLED Data Line** | SSD1306 (SDA) | `GPIO 21` (I²C SDA) | Serial data line for U8g2 graphic frame buffer |
| **OLED Clock Line** | SSD1306 (SCL) | `GPIO 22` (I²C SCL) | Serial clock line for OLED synchronization |
| **Normal State LED** | Green LED | `GPIO 26` (Digital Output)| Signals normal, healthy operation |
| **Trip / Fault LED** | Red LED | `GPIO 27` (Digital Output)| Signals thermal overload breach or trip state |

---

## 🖼️ 5. Hardware Setup & Visualization

### System Block Diagram Image
![System Block Diagram](docs/block_diagram.png)
*Figure 5.1: Functional block diagram showing hardware telemetry, processing engine, and dual output interfaces.*

### Physical Hardware Setup
![Hardware Prototype Setup](docs/hardware_setup.jpg)
*Figure 5.2: Complete experimental setup displaying the ESP32 microcontroller, ZMPT101B voltage sensor, step-down transformer, and rotary encoder on breadboard.*

### Real-Time Waveform Output on OLED
![OLED Sine Wave Output](docs/oled_waveform.jpg)
*Figure 5.3: 0.96" SSD1306 OLED rendering live sinusoidal wave graph with active phase cursor tracking.*

### IOT-Dashboard
![Iot dashboard images](docs/iot_dashboard.jpg)

---

## 💻 6. Getting Started

### 6.1 Prerequisites
* **Arduino IDE** (v2.0 or higher recommended)
* **ESP32 Board Package** installed (`esp32` by Espressif)
* **Required Libraries:**
  * `U8g2` by Oliver
  * `WiFi.h` & `WebServer.h` (Built-in ESP32 libraries)
  * `Wire.h`

### 6.2 Installation & Execution
1. Clone this repository:
   ```bash
   git clone https://github.com/YOUR_USERNAME/Synchronous-Point-On-Wave-MCB-Test-System.git
   ```
2. Navigate to `firmware/point_on_wave_mcb_test.ino` and open it in Arduino IDE.
3. Connect your ESP32 board, select **ESP32 Dev Module**, and set the appropriate COM Port.
4. Flash the code to your ESP32 board.
5. Connect your phone or PC to the ESP32 Wi-Fi Access Point:
   * **SSID:** `POW_DigitalTwin`
   * **Password:** `password123`
6. Open your web browser and go to `http://192.168.4.1` to access the interactive 2D MCB Web Dashboard.

---

## 🎯 7. Conclusion

The **Synchronous Point-on-Wave Test System** successfully demonstrates a safe, highly accurate cyber-physical alternative for evaluating Miniature Circuit Breakers against IEC standards. By substituting destructive physical fault currents with an ESP32-based hardware-in-the-loop thermal digital twin ($I^2t$), the system provides:
1. **Safe Overload Emulation:** Accurate inverse-time trip response without component destruction or arc fault hazards.
2. **High-Precision Point-on-Wave Control:** Full $360^\circ$ conduction angle modulation synchronized to grid zero-crossing points.
3. **Multi-Platform Telemetry:** Simultaneous real-time local display rendering (OLED) and wireless web monitoring (IoT Dashboard).

This project highlights the efficiency of embedded control systems and digital twin methodology in modern electrical power system testing and protection verification.

---


