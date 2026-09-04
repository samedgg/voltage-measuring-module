# Voltage Measuring Module

This repository contains the hardware design files for a custom mixed-signal PCB module that functions as a digital multimeter interface. The primary purpose of this board is to safely measure external potential differences using control probes, process the scaled voltage via a microcontroller, and display the results on an external screen.

This is my very first complete PCB design project, implemented on a standard **2-layer board**. The design focuses heavily on proper grounding techniques, manufacturability, and analog/digital isolation.

## 1. System Architecture

The hardware is divided into three main domains: the Analog Front-End (AFE), the Power Delivery Network (PDN), and the Digital/Processing Core.

### Analog Front-End (AFE)
Bare microcontroller pins are sensitive and typically limited to 3.3V inputs. Connecting external probes directly to the MCU is unsafe and electrically unstable. To solve this:
*   **Voltage Scaling & Buffering:** An **LMV358** operational amplifier is used. The external voltage from the probe socket (`V_IN`) is first scaled down using a resistor divider network.
*   **Impedance Matching:** The op-amp acts as a high-impedance buffer. This ensures the module draws virtually no current from the circuit being measured, preventing the measurement process from altering the target circuit's behavior. The buffered, safe voltage is then routed to the ADC channel of the MCU.

### Power Delivery Network (PDN)
The board supports a dual-power architecture (Li-Ion Battery and USB Type-C):
*   **Charging:** A **TP4056** linear charger IC is implemented. If both the battery and USB are connected simultaneously, the system continues to operate while the TP4056 charges the battery.
*   **Regulation:** An **AP2112K-3.3** Ultra-Low Dropout (LDO) regulator steps down the battery/USB voltage to a stable 3.3V for the MCU and logic peripherals. 
*   **Status Indicators:** Dedicated LEDs (Red/Green) are tied to the TP4056 to indicate charging and standby statuses.

### Processing & Interfaces
*   **Core:** **STM32F411CEU6** (ARM Cortex-M4) handles the ADC conversions and logic.
*   **Programming:** A standard SWD header is exposed for flashing and debugging.
*   **Display:** An I2C header is provided to transmit the calculated voltage values to an external OLED display.

## 2. Board Renders & Schematics

<img width="885" height="733" alt="3d" src="https://github.com/user-attachments/assets/aa6a9191-47b6-4cde-9777-d965ea38815c" />

<img width="765" height="690" alt="3d_back" src="https://github.com/user-attachments/assets/3291196e-ac92-4987-aaae-fbd524505a55" />

<img width="3509" height="2481" alt="schematic" src="https://github.com/user-attachments/assets/aae74e79-95c6-40bb-a03b-cecd99c01741" />

## 3. PCB Layout & Design Considerations

Special attention was given to the mixed-signal nature of this board to prevent crosstalk and electromagnetic interference (EMI).

*   **Layer Stackup (2-Layer Design):** The board is designed as a standard 2-layer PCB. The top layer is primarily dedicated to signal routing and component placement, while the bottom layer is utilized to maintain maximum signal integrity.
*   **Component Placement & Analog Isolation:** The analog measuring circuitry (Op-Amp and scaling resistors) is strictly grouped in the bottom-left corner of the PCB. Physical distance is used as the primary isolator. This prevents high-frequency digital return currents (from the MCU, crystal oscillator, and USB) from crossing into the sensitive analog region.
*   **Solid Ground Plane (No Split-Ground):** Instead of splitting the ground into `AGND` and `DGND` (which can create loop antennas and EMI issues if traces cross the gap), a single, continuous ground plane is poured across the entire bottom layer. Because the components are properly partitioned by physical distance, the analog and digital return currents naturally follow their paths of least impedance without interfering with each other.
*   **Trace Clearances:** The board follows strict width and clearance constraints (minimum 0.254mm / 10 mils) to ensure manufacturability and prevent solder mask sliver issues.

### Layer Views

<img width="854" height="761" alt="layers" src="https://github.com/user-attachments/assets/e72f97a7-3e20-47be-8020-13ff69a20029" />

<img width="841" height="732" alt="layer1" src="https://github.com/user-attachments/assets/c7104abc-3fc8-4588-bea2-d08c1629bec3" />

<img width="840" height="762" alt="layer2" src="https://github.com/user-attachments/assets/3166f7c8-0c08-445c-99dd-ce0aac3c97ef" />

## 4. Bill of Materials (Key Components)

| Designator | Component | Description |
| :--- | :--- | :--- |
| **MCU1** | STM32F411CEU6 | 100MHz ARM Cortex-M4 Microcontroller |
| **U14A** | LMV358DR2G | Low-Voltage Operational Amplifier |
| **CHARGE1**| TP4056 | Standalone Linear Li-Ion Battery Charger |
| **REG1** | AP2112K-3.3 | 3.3V, 600mA Low Dropout Regulator |
| **HSE_OSC**| HC49/4HSMX | 8MHz External Crystal Oscillator |
| **USB-C1** | Type-C 16-pin | Power and Communication Interface |
| **V_IN** | Screw Terminal | External Probe Input |
