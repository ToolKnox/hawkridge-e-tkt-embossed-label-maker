# E-TKT ESP32 Daisy Wheel Embossed Label Maker - Build Guide

https://github.com/ToolKnox/hawkridge-e-tkt-embossed-label-maker

Remix of E-TKT by Andrei Speridião (https://github.com/andreisperid/E-TKT, project site https://andreisperid.github.io/E-TKT/), updated/refurbished for modern use. This guide is a rewritten adaptation of the E-TKT documentation and is shared under CC BY-SA 4.0, like the original documentation; the print files, firmware and PCB are the unchanged MIT-licensed originals. Changes: new purchase list, reordered build steps, added checks and troubleshooting.

## What you are building

A desktop label maker that embosses raised letters into standard 9 mm plastic embossing tape, the way old hand-held embossers do, but driven by motors and controlled from a phone or computer over Wi-Fi.

- A NEMA 17 stepper turns a MOTEX daisy wheel (43 positions: A-Z, digits and a few symbols) to the right character. A hall sensor and a small magnet in the wheel hub give it a home position.
- An MG996R servo swings the printed press arm and stamps the character into the tape.
- A 12 V 28BYJ-48 geared stepper with an MK8 gear feeds the tape forward.
- An ESP32 on a custom 4-layer PCB runs everything. A 0.96 inch OLED and two LEDs in the top cover show status. The device hosts its own web app: type the label, press print.

Parts to buy are in the Bill of Material PDF. You also need a 3D printer (PETG recommended), a soldering iron, a crimp tool for 2.54 mm Dupont pins, hex keys, small pliers and a computer with VS Code + PlatformIO.

Plan for a few hours of assembly and expect to reprint the press arm or adjust the hall sensor during calibration. This is a precision build.

## 1. Order and assemble the PCB

The controller is a custom board, not a shop module, so it is a separate order:

1. Download the KiCad 6 project, schematic PDF and Gerber files from the `_electronic/` folder of the upstream repository at the pinned commit: https://github.com/andreisperid/E-TKT/tree/d63a636e16f25aaf3568ad580cc7558c940871c9/_electronic (the same files are in our GitHub mirror).
2. Order the board from any PCB service with these settings: 4 layers, FR-4, 1.6 mm, 99.6 x 76.2 mm, HASL. Order a stencil for the front (top) layer only. The two inner layers are a ground plane and a 6 V plane.
3. The SMD components (ESP32-WROOM-32D, FT231XS USB-serial chip, TPS54302 buck converter for 6 V, AZ1117 3.3 V regulator, ULN2003 for the feed motor, BSS138 level shifters, USB-C connector, barrel jack and passives) are listed with manufacturer part numbers in the upstream bill of materials (`docs/diy/bom.md`, section C). Either place them yourself (stencil + paste + reflow; the back side has only through-hole headers and one electrolytic capacitor) or let the PCB service assemble the board (turnkey assembly).
4. Leave the alternative L7806 linear 6 V regulator footprint on the back empty unless you deliberately skip the buck converter; upstream says that option was not tested on this board revision.

Before powering anything, check with a multimeter that 12 V, 6 V, 3.3 V and GND are not shorted to each other.

## 2. Flash the firmware and the web app first

Do this on the bare PCB, before it goes into the case. Some assembly steps need the firmware running (the servo must be in its rest position before you fit the press arm).

1. Install VS Code and the PlatformIO extension.
2. Download the upstream project at the pinned commit (https://github.com/andreisperid/E-TKT/tree/d63a636e16f25aaf3568ad580cc7558c940871c9, Code > Download ZIP) and open the folder in PlatformIO. Use the upstream download for flashing: its `data/` folder contains the font file the web app needs, which is not part of our mirror.
3. The project pins `espressif32@3.3.2`, board `nodemcu-32s`, and pulls its libraries (AccelStepper, ESP32Servo, ESPAsyncWiFiManager, U8g2, QRCode, ArduinoJson, ESPAsyncWebServer, AsyncElegantOTA) automatically. Use the `serial-upload` environment.
4. Connect the PCB to the computer with a USB-A to USB-C data cable. Hold the **FN** button on the board while the upload starts to put the ESP32 in download mode.
5. Upload the firmware: PlatformIO > serial-upload > Upload (`pio run -e serial-upload -t upload`).
6. Upload the web app files (the `data/` folder) to the ESP32 flash: PlatformIO > serial-upload > Upload Filesystem Image (`pio run -e serial-upload -t uploadfs`), again holding **FN**. Do not skip this step; without it the device runs but serves no app.

If a current PlatformIO install has trouble with the old platform version, let PlatformIO install exactly `espressif32@3.3.2` as pinned; do not raise the version without testing.

## 3. Print the parts

16 parts, about 220 g of PETG. The STL files are already oriented for printing.

| Part | Support |
|---|---|
| A_bottom | yes |
| B_wall | yes |
| C_wall_track | yes |
| D_pillar, E_pillar, F_pillar, G_pillar | no |
| H_nema_cover | yes |
| I_nema_wheel_hub | no |
| J_top | yes |
| K_top_screenholder, L_top_tapefeeder, M_top_wheel_cover | no |
| N_wheel_coupling_bottom, O_wheel_coupling_top | no |
| P_press (P_press_1mmleft is an alternative version shifted 1 mm to the left) | no |

Settings from the designer: 0.20 mm layers, 20 % infill, 3 walls (plus an alternating extra wall if your slicer supports it), 5 top/bottom layers. The fits are tight; print a test part first and tune your flow if parts do not slide together. The STEP files of every part are included if you want to adjust a fit.

## 4. Prepare the harnesses

- **Hall sensor (KY-003):** remove the male header, solder a 3-pin female 2.54 mm header to the back, and bend the sensor legs 90 degrees about 7 mm from the board so the sensor sits perpendicular to the breakout.
- **OLED:** remove the header and solder a 4-wire piece of flat ribbon cable directly to the pads (there is no room for a header). Crimp female Dupont pins and fit a 4-pin housing. The PCB connector order is SDA, SCL, VCC, GND. Check the pin labels on your display: on some modules VCC and GND are in the opposite order and must be crossed in the harness.
- **NEMA 17 cable:** cut the motor cable, keep the motor end, crimp female Dupont pins and fit a 4-pin housing. Upstream swaps the two inner wires; confirm the coil pairs with a multimeter (each pair reads a few ohms) so each coil ends up on adjacent pins.
- **LEDs:** shorten the legs of the two white 2 x 5 x 7 mm LEDs, solder each to a 2-wire piece of ribbon cable and insulate each leg with heat shrink. Do not crimp the plugs yet; the wires are threaded through the top part first.

## 5. Assemble

Upstream has photos for every step on the project site (https://andreisperid.github.io/E-TKT/diy/assembly/assembly.html). The order is:

1. **NEMA module.** Screw H_nema_cover onto the motor (4 x M3x6). Find the magnet polarity: plug the hall breakout into the PCB, power it, bring each end of the magnet near the sensor, and mark the end that triggers it. Press the magnet into I_nema_wheel_hub with the marked end out, flush with the surface. Slide 2 M3 nuts into the hub slot, start 2 x M3x8 screws, slide the hub onto the motor shaft matching the flat, and leave the screws loose.
2. **PCB.** Fit the A4988 driver (match the VMOT and GND markings), plug in the hall sensor, fit F_pillar with 2 nuts and 2 x M3x10 + washers, plug the NEMA harness into the motor and the PCB, and fit G_pillar on the other side.
3. **Chassis.** Set the PCB and motor into A_bottom, fix the motor with 4 x M3x20 from below, the pillars with 2 x M3x16 from below, and the PCB with 3 x M3x20 from the side.
4. **Press (servo).** Plug the servo into the PCB (brown GND, red VCC, orange signal). Plug in the display and the 12 V supply and wait for the splash screen: the servo moves to its rest angle. Put an M6 washer in P_press, line the press arm up with the servo body using the line on its side, and push it straight onto the servo spline (do not twist; the plastic forms itself to the spline). Fix it with M3x6 + washer. There must be no side play. Unplug power and the display, then mount the servo to A_bottom with 2 x M3x20 + washers into 2 nuts, pushed inward as far as it goes. Check: turn the press a little by hand, power up, and it must return to exactly vertical. If not, reprint P_press and repeat.
5. **Feed.** Fix the MK8 gear on the 28BYJ-48 shaft (one grub screw on the flat). Mount the motor in B_wall with 2 x M3x6 into nuts, plug it into the PCB (red wire to the left when facing the PCB), set B_wall into A_bottom (2 x M3x20 from below), then fit C_wall_track and D_pillar (1 x M3x20).
6. **Daisy wheel.** With the hub magnet facing the hall sensor (hub screws towards you), set the wheel so the letter **J** faces the press arm; the single large cutting tooth then faces you. Click the wheel onto the hub, fit the wheel top and N_wheel_coupling_bottom by their marks, press the 608 bearing into O_wheel_coupling_top and fix it with 1 x M3x16 until there is no play.
7. **Top.** Seat the OLED in J_top (wires to the outside), cover with K_top_screenholder and tighten its 2 x M3x6 evenly so the screen stays flat. Thread the LED cables through their channel, click the LEDs into their slots, keep the cables clear of the wheel, then crimp the 2-pin plugs (the arrow on the housing goes to ground). Plug the LEDs and the display into the PCB, close the top, fix the servo top with 1 x M3x20 + washer into a nut, then J_top to the pillars and wall with 4 x M3x20 and 2 x M3x16 on the side.
8. **Tape feeder.** Put a washer each side of the two 623 bearings, slide them into L_top_tapefeeder on 1 x M3x16 with a nut. Fit the feeder into J_top with the M3x40 through to B_wall, and start the M4x10 tension screw without tightening it.
9. **Finish.** Slide in M_top_wheel_cover (2 x M3x10, only lightly touching the bearing), fix D_pillar with 2 x M3x20, lift the wheel hub with a screwdriver against H_nema_cover and now tighten both hub screws, then the wheel cover screws. The wheel must sit firm with no play. Stick the 4 silicone feet under the base.

## 6. Wiring reference (from the firmware)

| Function | ESP32 GPIO |
|---|---|
| Hall sensor | 34 |
| Wi-Fi reset button | 13 |
| Press servo | 14 |
| Character window LED | 17 |
| Finish LED | 5 |
| NEMA 17 (A4988) STEP / DIR / ENABLE | 32 / 33 / 25 |
| Feed motor 28BYJ-48 via ULN2003 | 15, 2, 16, 4 |
| Buzzer (on the PCB) | 26 |
| OLED SSD1306 128 x 64 | hardware I2C |

Power: a 12 V supply of at least 3 A (a real 3 A; the servo draws peaks while pressing) feeds both motor drivers; the on-board buck converter makes 6 V for the servo and hall sensor, and a regulator makes 3.3 V for the ESP32. Use a 5.5 x 2.1 mm centre-positive plug; check it seats firmly in the board's barrel jack. Never run the MG996R from 12 V.

## 7. Connect to Wi-Fi

The device needs a 2.4 GHz network.

1. Power it on. The display asks you to connect.
2. On your phone, join the Wi-Fi network named **E-TKT**. A setup page opens (if not, open it from the network settings).
3. Choose Configure WiFi, pick your network, enter the password and save.
4. The display then shows a QR code and an address: open it to reach the label app.

To forget the saved network: hold the WI-FI button, power-cycle or press reset, wait for the confirmation on the screen, then release.

## 8. Load the tape

1. Push 9 mm embossing tape into the slit at the back, between J_top and C_wall_track.
2. Make sure it touches the feed gear and the two bearings.
3. In the app open SETUP and press +REEL, confirm and wait while it feeds through.

If the tape slips, tighten the M4 screw on the feeder a little.

## 9. Calibrate

1. Reboot. After the splash screen the wheel spins until the magnet passes the hall sensor (homing). The letter **J** should now sit centred in the top window.
2. If J is off-centre, **switch the device off** and nudge the hall sensor along the wheel's tangent: push it inward if J sits too far right, pull it outward if J sits too far left, keeping the same distance to the hub. Repeat until J is centred. If the wheel moves a little and stops at the wrong position every time, your sensor needs the opposite logic: set `INVERT_HALL_SENSOR_LOGIC` to `true` in `src/Configuration.h` and flash again. If the tape runs backwards, set `REVERSE_FEED_STEPPER_DIRECTION` to `true`.
3. At rest the press tip should be about 2 mm from I_nema_wheel_hub. If not, revisit step 5.4.
4. In the app press SETUP. **Align** (1-9, default 5): press TEST and watch the press tap the character slowly; adjust until the press meets the character centre in the window.
5. **Force** (1-9, default 1): raise it one step at a time and print a test label each time. Stop at the lowest value that gives a clean, readable emboss; higher values do not improve the result.
6. Press save; the machine reboots with the new values.

Note: the wheel has no separate 0 and 1; the firmware prints them with the O and I characters.

## Troubleshooting

- **App does not load:** the filesystem image was not uploaded. Repeat step 2.6.
- **Upload fails:** use a data cable and hold FN while the upload starts.
- **Homing never ends or stops in the wrong place:** magnet polarity, sensor distance, or `INVERT_HALL_SENSOR_LOGIC` (step 9.2).
- **Letters faint or doubled:** recheck Align first, then Force; check the press arm has no side play.
- **Brown-outs or resets while pressing:** the power supply cannot deliver 3 A.
- **Display blank or garbled:** VCC/GND order in the OLED harness.

## Safety

- The servo and steppers have pinch points; keep fingers away from the wheel and press while it runs.
- Switch off before moving the hall sensor; its pins sit close to other contacts.
- Use a certified 12 V adapter.

## Credits

E-TKT was designed by Andrei Speridião (2022). Software and hardware: MIT licence. Documentation and photos: CC BY-SA 4.0. Full notices are in the GitHub repository.
