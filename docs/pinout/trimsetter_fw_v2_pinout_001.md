# Trimsetter firmware v2 pinout authority 001

Status: draft from legacy firmware extraction
Board target: STM32F103C8T6 / STM32F103C8Tx black pill class board
Project: Trimsetter_FW_V2

## Purpose

This file is the pin authority for firmware v2.

The robot is already wired. Firmware v2 must match the existing wiring before any GPIO, timer, PWM, motor, switch, encoder, or sensor behavior is enabled.

## Bring-up rule

USB CDC first.

All motor, PWM, and actuator outputs remain disabled until this pinout is verified against the physical robot wiring and existing schematics.

## Known fixed pins

| Function | MCU pin | CubeMX signal | Direction | Firmware v2 phase | Notes |
|---|---|---|---|---|---|
| USB DM | PA11 | USB_DM | USB | Phase 0 | Required for USB CDC |
| USB DP | PA12 | USB_DP | USB | Phase 0 | Required for USB CDC |
| SWDIO | PA13 | SYS_JTMS-SWDIO | debug | Phase 0 | ST-Link debug |
| SWCLK | PA14 | SYS_JTCK-SWCLK | debug | Phase 0 | ST-Link debug |

## Legacy extracted robot wiring

Source evidence:

    trimsetter-firmware/Trimsetter_Source/hardware/hardware.h

| Subsystem | Legacy signal | MCU pin | Electrical direction | Active level | Timer/channel if any | Safe boot state | Source evidence |
|---|---|---|---|---|---|---|---|
| Fault LED | LED_FAULT | PB10 | output | TBD | GPIO | off | hardware.h |
| Process LED | LED_PROCESS | PB11 | output | TBD | GPIO | off | hardware.h |
| Load cell data | LOADCELL_DT | PB8 | input | TBD | GPIO | input | hardware.h |
| Load cell clock | LOADCELL_CLK | PB9 | output | TBD | GPIO | low/disabled | hardware.h |
| Winder 1 encoder A | ENC1_A | PB6 | input | quadrature | timer TBD | input | hardware.h |
| Winder 1 encoder B | ENC1_B | PB7 | input | quadrature | timer TBD | input | hardware.h |
| Winder 2 encoder A | ENC2_A | PB4 | input | quadrature | timer TBD | input | hardware.h |
| Winder 2 encoder B | ENC2_B | PB5 | input | quadrature | timer TBD | input | hardware.h |
| Material encoder A | ENCM_A | PA9 | input | quadrature | timer TBD | input | hardware.h |
| Material encoder B | ENCM_B | PA8 | input | quadrature | timer TBD | input | hardware.h |
| Motor 1 direction | M1_DIR | PA7 | output | TBD | GPIO | disabled/safe | hardware.h |
| Motor 2 direction | M2_DIR | PA6 | output | TBD | GPIO | disabled/safe | hardware.h |
| Motor 3 direction | M3_DIR | PA5 | output | TBD | GPIO | disabled/safe | hardware.h |
| Motor 4 direction | M4_DIR | PA4 | output | TBD | GPIO | disabled/safe | hardware.h |
| Motor 1 PWM | PWM_OUT_1 | PA3 | output | PWM | timer TBD | disabled/0 duty | hardware.h |
| Motor 2 PWM | PWM_OUT_2 | PA2 | output | PWM | timer TBD | disabled/0 duty | hardware.h |
| Motor 3 PWM | PWM_OUT_3 | PA1 | output | PWM | timer TBD | disabled/0 duty | hardware.h |
| Motor 4 PWM | PWM_OUT_4 | PA0 | output | PWM | timer TBD | disabled/0 duty | hardware.h |
| Solenoid | SOLENOID_OUT | PB0 | output | TBD | GPIO | disabled/off | hardware.h |
| Button matrix input 1 | BTN_IN_GRID_1 | PB12 | input | active-low in legacy scan | GPIO | input pullup TBD | hardware.h, switch_panel.c |
| Button matrix input 2 | BTN_IN_GRID_2 | PB13 | input | active-low in legacy scan | GPIO | input pullup TBD | hardware.h, switch_panel.c |
| Button matrix input 3 | BTN_IN_GRID_3 | PB14 | input | active-low in legacy scan | GPIO | input pullup TBD | hardware.h, switch_panel.c |
| Button matrix output 1 | BTN_OUT_GRID_1 | PA10 | output | active-low row drive | GPIO | high/inactive | hardware.h, switch_panel.c |
| Button matrix output 2 | BTN_OUT_GRID_2 | PA15 | output | active-low row drive | GPIO | high/inactive | hardware.h, switch_panel.c |
| Button matrix output 3 | BTN_OUT_GRID_3 | PB3 | output | active-low row drive | GPIO | high/inactive | hardware.h, switch_panel.c |

## Legacy button logical map

Source evidence:

    trimsetter-firmware/Trimsetter_Source/hardware/switch_panel.h

| Logical button | Legacy matrix slot | Notes |
|---|---|---|
| BUTTON_RESET | BUTTON_1_1 | reset |
| BUTTON_UNLOAD_B | BUTTON_1_2 | right side row 2 |
| BUTTON_LOAD_B | BUTTON_1_3 | far right row 3 |
| BUTTON_MARK | BUTTON_2_1 | mark |
| BUTTON_UNLOAD_A | BUTTON_2_2 | left side row 2 |
| BUTTON_LOAD_A | BUTTON_2_3 | far left row 3 |
| BUTTON_SHIFT | BUTTON_3_1 | center row 2 in legacy comment |
| BUTTON_3_2 | BUTTON_3_2 | currently unassigned |
| BUTTON_3_3 | BUTTON_3_3 | currently unassigned |

## Pin collision warnings

### USART1 collision

Do not enable USART1 casually in firmware v2.

| MCU pin | Legacy use | Common alternate |
|---|---|---|
| PA9 | ENCM_A | USART1_TX |
| PA10 | BTN_OUT_GRID_1 | USART1_RX |

USB CDC is the console path for firmware v2.

### JTAG/SWD remap warning

Legacy wiring uses:

| MCU pin | Legacy use | Debug/JTAG concern |
|---|---|---|
| PA15 | BTN_OUT_GRID_2 | JTDI when full JTAG enabled |
| PB3 | BTN_OUT_GRID_3 | JTDO/TRACESWO when full JTAG enabled |
| PB4 | ENC2_A | NJTRST when full JTAG enabled |

Firmware v2 must keep SWD enabled but disable full JTAG if these pins are used for robot IO.

### USB fixed pins

Do not assign robot IO to PA11 or PA12.

## Firmware v2 phase gates

### Phase 0: USB witness only

Allowed pins:

- PA11 USB_DM
- PA12 USB_DP
- PA13 SWDIO
- PA14 SWCLK

All other robot pins must remain unclaimed or configured into safe non-driving states.

### Phase 1: input-only witness

Allowed additions:

- E-stop input if identified
- clear/reset input if identified
- switch-panel inputs
- sensor/ADC inputs
- encoder inputs

No motor outputs.

### Phase 2: outputs declared but disabled

Allowed additions:

- named motor and solenoid pins in code
- no PWM start
- no GPIO output transitions except explicit safe disabled level
- no motion side effects

### Phase 3: hardware actuation

Deferred.

Requires reviewed pinout, verified safe boot states, and explicit commit/receipt boundary.

## Open questions

- Exact timer/channel mapping for PWM_OUT_1 through PWM_OUT_4.
- Exact timer/channel mapping for ENC1, ENC2, and ENCM.
- Active polarity for direction pins.
- Safe disabled level for each motor driver and solenoid output.
- Whether load cell pins are still wired to PB8/PB9 on the current robot.
- Whether the physical button matrix matches the legacy comments.
- Whether PA15/PB3/PB4 require CubeMX remap settings to disable full JTAG while preserving SWD.
