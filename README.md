# CAN-BUS — Vehicle Speed Warning System

Simulated a 3-node automotive CAN bus network using **Vector CANoe 10**
and **CAPL scripting**. No physical hardware required.

## Project Overview

|  Item    |  Details                                            |
|----------|-----------------------------------------------------|
| Tool     | Vector CANoe 10 (Evolution)                         |
| Language | CAPL (Communication Access Programming Language)    |
| Bus      | Virtual CAN Channel 1 — 500 kbps                    |
| Nodes    | 3 (Engine_ECU, Dashboard, Buzzer)                   |
| Message  | ID 0x100, DLC 1, cyclic 100ms                       |
| Hardware | Software simulation only — no physical ECU required |

## Architecture
---------------
Engine_ECU.can ──[0x100 every 100ms]──► CAN Bus ──► Dashboard.can
                                             └──► Buzzer.can


One transmitter → two independent receivers.
Demonstrates the CAN **broadcast model** — both Dashboard and Buzzer
receive the same frame without the Engine ECU knowing or caring.

## Node Behavior

### Engine_ECU.can — Transmitter
- Sends vehicle speed as byte 0 of CAN message 0x100 every 100ms
- Speed ramps: **0 → 140 → 0 km/h** in 5 km/h steps

### Dashboard.can — Receiver (3-state warning engine)

|  State    |   Speed       |   Action       |
|-----------|---------------|----------------|
| NORMAL    | < 80 km/h     | Silent log     |
| OVERSPEED | 80 – 120 km/h | Warning alert  |
| CRITICAL  | > 120 km/h    | Critical alert |

- Uses 'previousState' tracking — alerts fire **only on transitions**

### Buzzer.can — Independent Receiver (alert patterns)

| Zone       | Speed          | Beep Pattern |
|------------|----------------|--------------|
| Silent     | ≤ 120 km/h     | No sound     |
| Slow beep  | 120 – 130 km/h | Every 500ms  |
| Rapid beep | > 130 km/h     | Every 200ms  |

- Uses an **independent timer** decoupled from the 100ms CAN cycle

## CAPL Concepts Used

| Concept                  | Usage                                            |
|--------------------------|--------------------------------------------------|
| 'msTimer' + 'setTimer()' | Cyclic 100ms Engine ECU transmission             |
| 'on message 0x100'       | Event-driven reception in Dashboard and Buzzer   |
| 'output()'               | Transmit CAN frame onto virtual bus              |
| 'this.byte(0)'           | Read speed value from received frame             |
| 'cancelTimer()'          | Stop beep timer when speed drops below threshold |
| 'write()'                | Log output to CANoe Write Window                 |

## How to Run

1. Open **Vector CANoe 10**
2. File → New → save as 'SpeedWarning.cfg'
3. In Simulation Setup → Insert 3 Network Nodes on CAN Channel 1:
   - 'Engine_ECU' → assign 'Engine_ECU.can'
   - 'Dashboard' → assign 'Dashboard.can'
   - 'Buzzer' → assign 'Buzzer.can'
4. Press **F9** to start measurement
5. Watch output in **Trace Window** and **Write Window**

## Sample Output
-------------------------------------------------------------------
Engine ECU: Tx 0x100  | speed = 80 km/h
Dashboard:  Frame #16 | Speed = 80 km/h | State = OVERSPEED
>>> Dashboard: State changed -> OVERSPEED WARNING! (80 km/h)

Engine ECU: Tx 0x100  | speed = 125 km/h
Dashboard:  Frame #25 | Speed = 125 km/h | State = CRITICAL
>>> Dashboard: State changed -> CRITICAL ALERT! (125 km/h) <
Buzzer:     SLOW BEEP activated -- speed = 125 km/h (120-130 zone)
Buzzer:     *** BEEP *** (speed = 125 km/h | beep #1)

Engine ECU: Tx 0x100 | speed = 135 km/h
Buzzer:     RAPID BEEP activated -- speed = 135 km/h (DANGER > 130)
Buzzer:     *** BEEP *** (speed = 135 km/h | beep #3)
----------------------------------------------------------------------

## Project Files

| File             | Description                            |
|------------------|----------------------------------------|
| 'Engine_ECU.can' | CAPL node — cyclic speed transmission  |
| 'Dashboard.can'  | CAPL node — 3-state warning engine     |
| 'Buzzer.can'     | CAPL node — independent beep alert     |

## Full Project on Google Drive
[View CAPL files + simulation output + screenshots](https://drive.google.com/drive/folders/1uZgVX7XytuwMeWptTm-WEh2wQQbBc1e2)

## Skills Demonstrated
- CAN Bus protocol (ISO 11898) — frame structure, DLC, broadcast model
- CAPL scripting — timers, message handlers, state machines
- Vector CANoe — simulation setup, Trace Window, Write Window
- Embedded software patterns — cyclic Tx, event-driven Rx, state transitions
