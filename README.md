# Single-Axis BLDC / FOC Driver PCB

![3D render](media/pcb_3d_top.png)

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

<p align="center">
  <img src="media/pcb_2d_top.png" width="48%" />
  <img src="media/pcb_2d_bottom.png" width="48%" />
</p>

## Files

| Folder | Content |
|---|---|
| `altium/` | Project schematic |
| `docs/` | Schematic PDF |
| `media/` | Screenshots |

## What I'd change in the next revision

- Check current-sense and thermal design with calculations before routing
- Write and test FOC firmware on an STM32G4 development board first
- Order a first prototype and test it at low current
