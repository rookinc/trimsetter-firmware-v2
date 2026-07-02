# Trimsetter Firmware v2

Clean STM32 firmware project for the Trimsetter black-box machine witness.

This project is not the legacy TM5 firmware project.

## Bring-up rule

USB CDC and status first.

No motor outputs, no PWM, no motion, and no machine action until the firmware can expose a truthful machine witness status contract.

## Initial target

Board:

    STM32F103C8Tx black pill / blue pill class board

Transport:

    USB CDC virtual COM port

First commands:

    help
    status
    clear alarms
    reset

First status shape:

    mode: real
    contract_version: trimsetter.mcu.status.v0.1
    authority_boundary: mcu
    connected: true
    alarm_state: FAULT or OK
    system_state: READY
    motion_admission: closed
    machine_action: none

## Boundary language

The app requests.

The firmware admits, rejects, receipts, and reports.

Motion is closed by default.
