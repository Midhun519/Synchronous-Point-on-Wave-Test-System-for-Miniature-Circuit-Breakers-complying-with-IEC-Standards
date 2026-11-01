# Synchronous-Point-on-Wave-Test-System-for-Miniature-Circuit-Breakers-complying-with-IEC-Standards

An embedded **Hardware-in-the-Loop (HIL) Digital Twin** designed to safely emulate the thermal dynamics ($I^2t$), bimetallic heat accumulation, and inverse-time trip characteristics of an IEC-compliant Miniature Circuit Breaker (MCB) at low voltage without physical hardware destruction.

---

## 📌 Project Overview

Testing physical MCBs under high fault currents is hazardous, expensive, and destructive. This project replaces physical thermal degradation with a real-time discretized continuous thermal model running on an **ESP32 microcontroller**. 

The system samples live AC mains voltage via telemetry, synchronizes timing with zero-crossing interrupts, dynamically modulates phase firing angles ($\gamma$) via a rotary encoder, and displays continuous $I^2t$ thermal status over both a local **0.96" SSD1306 OLED** and an **embedded IoT Web Dashboard**.

---

## ⚙️ Key Technical Features

* **Hardware-in-the-Loop Telemetry:** AC voltage sampling via ZMPT101B and phase synchronization using an H11AA1 zero-crossing optocoupler.
* **Discrete-Time Thermal Engine:** Continuous-to-discrete implementation of a first-order LTI bimetallic model using **Backward Euler approximation**.
* **Interactive Fault Modulation:** Real-time conduction angle ($\gamma$) adjustment using a KY-040 rotary encoder to simulate dynamic overload severity.
* **Dual Monitoring Interfaces:** Local OLED visualization and a standalone, dark-themed ESP32 Wi-Fi Access Point IoT web dashboard (`http://192.168.4.1`).

---

## 📐 Control Systems Architecture

The bimetallic heating and natural dissipation dynamics are modeled as a Continuous-Time First-Order Linear Time-Invariant (LTI) transfer function:

$$G(s) = \frac{K}{\tau s + 1}$$

Where:
* $K$ = Thermal gain constant (steady-state temperature rise per unit power)
* $\tau$ = Thermal time constant (seconds) representing thermal inertia

### Discrete Model Formulation (Backward Euler)
To execute continuous differential equations inside firmware sampled at time step $\Delta t = 20\text{ ms}$ (50 Hz grid):

$$\frac{x[k] - x[k-1]}{\Delta t} = -\frac{1}{\tau} x[k] + \frac{K}{\tau} u[k]$$

Yielding the discrete state update equation:

$$x[k] = \left(\frac{\tau}{\tau + \Delta t}\right) x[k-1] + \left(\frac{K \cdot \Delta t}{\tau + \Delta t}\right) u[k]$$

---

## 🏗️ System Block Diagram

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
                               │  0.96" OLED Display     │ │ Indicator LED / Relay   │
                               │  (Phase Angle & Status) │ │ (Safe Trip Output)      │
                               └─────────────────────────┘ └─────────────────────────┘
