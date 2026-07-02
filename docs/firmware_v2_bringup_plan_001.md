# Firmware v2 bring-up plan 001

## Purpose

Start a clean STM32 firmware project that implements the same machine witness law now represented by the Trimsetter emulator.

The old firmware project proved the board can enumerate as USB CDC. Firmware v2 starts from that proof but moves the logic toward the emulator contract.

## Phase 0: USB-only witness

Acceptance:

- board enumerates as STM32 Virtual ComPort
- host can open serial at 115200
- `help` returns a command list
- `status` returns a stable status packet
- all motor/PWM outputs remain inactive

No motion commands are allowed in Phase 0.

## Phase 1: alarm and reset law

Acceptance:

- boot state is safe
- alarm/fault state is explicit
- clear alarms is separate from reset
- reset does not unlock motion
- status reports alarm_state, system_state, motion_admission, and machine_action

## Phase 2: command file admission

Acceptance:

- firmware receives a bounded command file envelope
- firmware rejects malformed or unknown contract files
- firmware admits only recognized command file shape
- firmware returns a receipt
- motion remains closed after command file admission unless a separate unlock path is admitted

## Phase 3: dry-run motion grammar

Acceptance:

- motion commands update simulated internal state only
- no PWM output
- no GPIO motor actuation
- all actions are receipted

## Phase 4: hardware outputs

Deferred until the witness, receipt, alarm, and command-file law are stable.
