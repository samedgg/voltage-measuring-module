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

<img width="679" height="637" alt="3d" src="https://github.com/user-attachments/assets/612135b3-76c4-4576-a170-acf2c8664b93" />

<img width="679" height="637" alt="3d_back" src="https://github.com/user-attachments/assets/d0887b14-e3bc-4fa5-a1b6-eb770bb5daa7" />

<img width="3509" height="2481" alt="schematic" src="https://github.com/user-attachments/assets/6566c517-1350-4349-b34f-9b56dd708481" />

## 3. PCB Layout & Design Considerations

Special attention was given to the mixed-signal nature of this board to prevent crosstalk and electromagnetic interference (EMI).

*   **Layer Stackup (2-Layer Design):** The board is designed as a standard 2-layer PCB. The top layer is primarily dedicated to signal routing and component placement, while the bottom layer is utilized to maintain maximum signal integrity.
*   **Component Placement & Analog Isolation:** The analog measuring circuitry (Op-Amp and scaling resistors) is strictly grouped in the bottom-left corner of the PCB. Physical distance is used as the primary isolator. This prevents high-frequency digital return currents (from the MCU, crystal oscillator, and USB) from crossing into the sensitive analog region.
*   **Solid Ground Plane (No Split-Ground):** Instead of splitting the ground into `AGND` and `DGND` (which can create loop antennas and EMI issues if traces cross the gap), a single, continuous ground plane is poured across the entire bottom layer. Because the components are properly partitioned by physical distance, the analog and digital return currents naturally follow their paths of least impedance without interfering with each other.
*   **Via Stitching & EMI Shielding:** To further improve signal integrity, the empty areas on the top layer are poured with a GND polygon and aggressively stitched to the bottom ground plane using an extensive via network. This eliminates floating "dead copper" islands that could act as antennas, minimizes overall ground impedance, and creates a Faraday cage effect to shield the internal traces from external noise.   
*   **Trace Clearances:** The board follows strict width and clearance constraints (minimum 0.254mm / 10 mils) to ensure manufacturability and prevent solder mask sliver issues.

### Layer Views

<img width="818" height="733" alt="layers" src="https://github.com/user-attachments/assets/46d34397-652d-4f26-bcac-815d5b3623f9" />

<img width="818" height="733" alt="layer1" src="https://github.com/user-attachments/assets/f0e75d22-85c7-4b5b-9e75-5d1ed80bc372" />

<img width="818" height="733" alt="layer2" src="https://github.com/user-attachments/assets/25325813-9003-4551-b9d0-4cbdd97b90c7" />

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
