# Single-Axis BLDC / FOC Driver PCB

<p align="center">
  <img src="media/PCB_top.png" width="48%" />
  <img src="media/PCB_below.png" width="48%" />
</p>
<p align="center"><i>Top and bottom of the board.</i></p>

A single-axis BLDC motor driver board designed in **Altium Designer** for the joint actuators of a future version of the KOUDRUPED quadruped. It is sized to mount on the back of a **T-Motor U8II Pro** actuator.

> **Status:** design study, made to learn Altium Designer. The board has not been manufactured or tested, and no firmware has been written for it yet.

## Architecture

```mermaid
flowchart LR
    HOST["Raspberry Pi<br/>(host)"] <-->|"CAN<br/>setpoints / telemetry"| MCU["STM32G4<br/>FOC"]
    ENC["AS5047P<br/>magnetic encoder"] -->|"rotor angle"| MCU
    MCU <-->|"SPI<br/>config / faults"| DRV["DRV8323<br/>gate driver"]
    MCU -->|"PWM"| DRV
    DRV -->|"gate drive"| PWR["3× N-MOSFET<br/>half-bridges"]
    VIN["24 V DC"] --> PWR
    PWR -->|"3-phase"| MOT["T-Motor<br/>U8II Pro"]
    MOT -. "shaft magnet" .-> ENC
```


| Block | Part | Role |
|---|---|---|
| MCU | STM32G4 | Runs field-oriented control (FOC) |
| Gate driver | DRV8323 | Configured over SPI, reports faults |
| Power stage | N-channel MOSFET half-bridges | Drives the three motor phases |
| Position sensor | AS5047P magnetic encoder | Rotor angle for FOC |
| Communication | CAN | Setpoints from and telemetry to a Raspberry Pi host |

- **Input:** 24 V DC, targeting the 31 A peak current of the U8II Pro
- **Planned telemetry over CAN:** motor current, velocity, position and fault status
- **Stack-up:** 4 layers, with an internal ground reference plane
- **Heat:** thermal vias under the power stage

## Layout

| Layer 1 – Top (signal) | Layer 2 – GND plane |
|:---:|:---:|
| <img src="media/signal.png" width="100%" /> | <img src="media/gnd.png" width="100%" /> |
| **Layer 3 – Power plane** | **Layer 4 – Bottom (encoder side)** |
| <img src="media/powerr.png" width="100%" /> | <img src="media/bottom-encoder.png" width="100%" /> |

- **Top:** components and signal routing
- **GND plane:** solid ground reference under the signal layers
- **Power plane:** shown as a negative, as Altium displays plane layers. Rings are clearances, X marks are vias connected to the plane, and the outer ring is the edge pullback.
- **Bottom:** the AS5047P encoder faces the motor's shaft magnet from this side

## Files

| Folder | Content |
|---|---|
| [`altium/`](altium/) | Altium project, schematic and PCB files |
| [`media/`](media/) | Renders and layer screenshots |

## What I'd change in the next revision

- Check current-sense and thermal design with calculations before routing
- Write and test FOC firmware on an STM32G4 development board first
- Order a first prototype and test it at low current
