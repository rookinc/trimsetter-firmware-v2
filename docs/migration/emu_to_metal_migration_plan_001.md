# Emulator to metal migration plan 001

Status: draft
Target repo: trimsetter-firmware-v2
Reference repo: trimsetter-emu

## Current metal status

Firmware v2 currently proves the USB witness layer:

- USB CDC enumerates on the Mac.
- `help` responds.
- `status` responds.
- unknown commands reject.
- `motion_admission` is `closed`.
- `machine_action` is `none`.

No robot outputs are enabled.

## Reference emulator laws to migrate

The emulator is the reference law surface for the first metal controller.

Core state:

- `alarm_state`
- `system_state`
- `motion_admitted`
- `loaded_command_file_id`
- `command_file_admission`
- `last_command_file_receipt`
- `machine_action`

Core law:

- Motion is closed unless alarm state is OK and motion has been explicitly admitted.
- Command file admission is separate from motion unlock.
- Unlock motion requires alarm OK.
- Unlock motion requires an admitted command file.
- Reset closes motion.
- ESTOP closes motion and latches a hard stop state.
- Unknown commands reject.
- Every admission boundary should return a reason or receipt.

## Phase 0: USB witness

Status: complete on metal.

Commands:

- `help`
- `status`
- unknown command rejection

Rules:

- no motor pins
- no PWM
- no motion side effects
- `motion_admission=closed`
- `machine_action=none`

## Phase 1: state model parity

Replace hardcoded status strings with MCU state variables.

Add firmware state fields:

- `alarm_state`
- `system_state`
- `motion_admitted`
- `loaded_command_file_id`
- `machine_action`

Keep all hardware outputs disabled.

Acceptance:

- `status` is generated from state, not hardcoded text.
- `motion_admission` is computed from state.
- motion remains closed by default.

## Phase 2: alarm and reset law

Add commands:

- `alarm`
- `reset`
- `clear alarms`
- `estop`

Rules:

- any active alarm closes motion
- ESTOP latches ESTOP
- reset closes motion
- reset does not unlock motion
- clear alarms returns to OK/READY only when allowed

Acceptance:

- status reflects alarm transitions
- ESTOP blocks unlock
- reset preserves safety boundary

## Phase 3: command file placeholder admission

Add commands:

- `load test command file`
- `unload command file`
- `unlock motion`

Rules:

- loading a command file does not unlock motion
- unlock motion denied if alarm is not OK
- unlock motion denied if no command file is loaded
- unloading command file closes motion
- reset closes motion

No real command file parsing yet.

Acceptance:

- placeholder command file creates a loaded ID
- unlock opens motion only after explicit allowed path
- `machine_action` remains `none`

## Phase 4: command file envelope

Add bounded parser for a small command file envelope:

- `contract`
- `request_id`
- `seq`
- `command_file_id`
- minimal geometry/job fields

Return receipt-shaped output.

Acceptance:

- malformed payload rejected
- unknown contract rejected
- valid minimal envelope admitted
- admission still does not start motion

## Phase 5: input-only robot witness

Use `docs/pinout/trimsetter_fw_v2_pinout_001.md`.

Enable input-only witness pins first:

- button matrix inputs
- encoder inputs if safe
- load cell input if safe
- estop/clear input if identified

No motor outputs.
No PWM.
No solenoid output.

Acceptance:

- inputs are readable
- status can report input witness state
- no actuator pin changes occur

## Phase 6: dry-run motion grammar

Implement internal simulated motion state only.

Rules:

- no PWM start
- no motor GPIO output transitions
- every motion request is admitted or rejected
- every dry-run action is receipted

Acceptance:

- dry-run can prove grammar and receipts
- hardware remains quiet

## Phase 7: physical outputs

Deferred.

Requirements before enabling:

- reviewed pinout
- safe boot state for every output
- active polarity known
- motor power safety confirmed
- explicit commit/receipt boundary
- no output enabled by accident during boot or reset

## Keeper

The app requests.

The MCU admits, rejects, receipts, and reports.

The metal is master.
