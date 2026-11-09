# Bead-Chain Blind Motor

A DIY motor that opens and closes a **bead-chain roller blind** from **Home Assistant**.
An ESP32 running ESPHome drives a NEMA17 stepper through a TMC2209 driver, and a
3D-printed sprocket pulls the blind's existing chain, so nothing on the blind itself is modified.

[[https://github.com/user-attachments/assets/04f6f5f8-53a8-480c-ac67-adde5940ab2a](https://github.com/user-attachments/assets/5d26fc04-4414-43c1-ac54-96c5d099e476)](https://github.com/user-attachments/assets/5d26fc04-4414-43c1-ac54-96c5d099e476)

## Status

- [x] Motor moves from Home Assistant (step / position buttons)
- [x] 3D-printed chain sprocket works on a 6 mm bead chain
- [x] 3D-printed case (version 1 files included, **not yet test-printed**)
- [ ] Single Open / Close / position-slider entity (commented-out starter in the YAML)
- [ ] Single power cable (12 V brick plus a small 12 V to 5 V converter for the ESP32)

This is a hobby project and a first build. Treat the files as a starting point, not a finished product.

## How it works

1. Home Assistant sends a command to the ESP32 over Wi-Fi (ESPHome native API).
2. The ESP32 sends STEP and DIR pulses to the TMC2209 driver.
3. The motor turns a 12-tooth sprocket. Each turn pulls about **72 mm** of chain
   (12 beads x 6 mm). With the driver at its default microstepping that is **1,600 steps per turn**,
   or about 22 steps per millimeter of chain.

The TMC2209 is used in plain STEP/DIR mode (UART is not used), so motor current is set with
the small trimmer screw on the driver module instead of in software.

## Parts

| Part | Notes |
|---|---|
| ESP32 dev board (WROOM-32) | Runs ESPHome |
| BigTreeTech TMC2209 V1.3 driver | Used in STEP/DIR mode |
| Screw-terminal stepper driver expansion board | Takes the driver module; screw terminal for 12 V, 4-pin motor socket, STEP/DIR/EN pins |
| NEMA17 stepper, 17HS19-2004S1 (2 A, 48 mm body, 5 mm D-shaft) | Run at a low current |
| 12 V DC power supply (1.25 A) | Plug cut off and wired into the screw terminal |
| Female-to-female jumper wires | ESP32 to the expansion board |
| M3 x 6 mm screws (4) | Motor to the case end wall |
| M3 x 10 mm screws (5) | Case lid |
| M3 set screw | Holds the sprocket on the shaft |
| Wall screws and anchors | Case mounting |
| 3D prints (PETG) | Sprocket, case body, case lid |

## Wiring

| ESP32 | Driver expansion board |
|---|---|
| GPIO18 | STEP (yellow row) |
| GPIO19 | DIR (yellow row) |
| 3V3 | V (red row), logic power |
| GND | G (black row) |
| not connected to the ESP32 | EN is jumpered to G (always enabled) |

| Other connections | |
|---|---|
| 12 V supply + | VIN (green terminal) |
| 12 V supply - | GND (green terminal) |
| Motor | 4-pin white socket (keep each coil pair together: red/blue and green/black) |

**Do not connect 5 V to the red row.** The ESP32's pins are 3.3 V only.

<img width="3024" height="4032" alt="Image" src="https://github.com/user-attachments/assets/cfc21064-bbd4-4aaa-9f6a-5e28579ebc24" />

Wiring for the motor to TMC board and cut charger cable directly into Driver expansion board.

<img width="3024" height="4032" alt="Image" src="https://github.com/user-attachments/assets/894c82e1-c2b7-451e-b746-ebe13e60b20b" />

Better image of charger screwed into expansion board.

<img width="3024" height="4032" alt="Image" src="https://github.com/user-attachments/assets/52b3dc5d-b29a-4978-8ab2-0627561a560c" />

Better picture of wiring from stepper motor to expansion board.

<img width="3024" height="4032" alt="Image" src="https://github.com/user-attachments/assets/852681ab-17ab-4d09-a865-a2dd3ba4695e" />

ESP32 wiring. Green jumper cable 3v3, yellow jumper cable GND, brown jumper cable D18, red jumper cable D19

<img width="3024" height="4032" alt="Image" src="https://github.com/user-attachments/assets/990ed036-c269-418b-bfa5-24145b9fcb05" />
Expansion wiring. Green jumper cable V column, yellow jumper cable G column, brown jumper cable STEP, red jumper cable DIR, orange jumper cable EN to G column (Not sure if necessary).

## Software

1. Install [ESPHome](https://esphome.io) (Home Assistant add-on or command line).
2. Copy `esphome/secrets.yaml.example` to `esphome/secrets.yaml` and fill in your values.
   Generate a fresh API key. `secrets.yaml` is git-ignored.
3. Flash `esphome/roller-blind.yaml` over USB the first time. Later updates work over Wi-Fi.
4. In Home Assistant, add the discovered **Roller Blind** device.

<img width="524" height="399" alt="Image" src="https://github.com/user-attachments/assets/c4862456-3f4f-4bc6-b88e-d52ca76b0ad8" />

## Calibration

The motor counts steps from wherever it was when the ESP32 powered on, so start with the blind
**fully closed** each time you power it up (there are no limit switches yet).

1. Close the blind by hand and press **Set Position Zero**.
2. Press **Nudge Open +500** repeatedly until the blind is fully open (use **Stop** if needed).
3. Read **Blind Position (steps)**. That is your `open_steps` value.
4. If the blind moves the wrong way, invert the direction pin (see the comment in the YAML).

## 3D-printed parts (`cad/`)

| File | What it is |
|---|---|
| `cad/sprocket/bead_chain_sprocket_test_plate.stl` | 12-tooth sprocket for 6 mm-pitch ball chain, three pocket sizes (4.4, 4.8, 5.2 mm) so you can pick the best fit. **TODO: note which size fit best.** |
| `cad/case/case_body.stl` | Case v1: motor channel, board holders, vents, chain guard, wall ears |
| `cad/case/case_lid.stl` | Case v1 lid |

**Print settings:** PETG (PLA softens in a sunny window), 0.2 mm layers, 3+ walls, about 40% infill.
Sprocket: no supports. Case body: back plate on the bed, supports only under the chain guard.
Case lid: flat, pad facing up.

**Fit notes:** the sprocket bore is 5.1 mm with a D-flat, and an M3 set screw threads into the hub.
Measure your own chain first: 10 beads should span about 60 mm for these files.

<img width="423" height="458" alt="Image" src="https://github.com/user-attachments/assets/76ea3b55-0836-4bb3-8d3c-e0c43f6650b6" />

## Assembly overview

1. Fit the sprocket on the motor shaft, flat side under the set screw.
2. Screw the motor into the case end wall (four M3 screws), then set the board stack and wiring.
3. Close the lid and fasten the case to the wall so the chain has a slight tension around the sprocket.

(Case currently being printed, picture will be added later)

## Known limitations

- No UART, so no software current control and no stall detection.
- Position is forgotten at power-up and there are no limit switches.
- The motor is energized all the time (EN tied to ground), so it runs a little warm.
- The ESP32 currently needs its own USB power in addition to the 12 V supply.
- Case v1 is untested.

## Safety

Low-voltage hobby electronics (12 V), but: check polarity before powering on, never connect or disconnect
the motor while the driver is powered, and keep the driver and wiring out of cardboard and other flammable materials.
Use at your own risk.

## License

Code and configuration: [MIT](LICENSE).
3D models (`cad/`): CC BY 4.0 (https://creativecommons.org/licenses/by/4.0/).

## Credits

- [ESPHome](https://esphome.io) and [Home Assistant](https://www.home-assistant.io)
- TODO: thank the neighbor who printed the parts (if they agree to be named)
