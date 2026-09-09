# BOLT

A sumo robot with 8 driven wheels, built around a custom STM32G431 board. This is the second revision. The first one didn't work, and most of what's here exists because of what broke last time.

It's registered in the 3 kg class, but I'm aiming for about 1.5 kg. 8 Pololu HP 50:1 micro metal gearmotors, each with its own DRV8251 driver and current sensing. Two VL53L1X time of flight sensors searchin for the opponent, four KTIR0711S for the ring edge dedection, and an LSM6DSOX handles orientation. Steel base, CNC cut steel plough, printed PETG on top.

## Why I built it

I built v1, put it together, and it couldn't run at all. I could have patched it, but by the time I listed everything wrong with it, patching made no sense.

The eight motor layout is the part I care about most. Most minisumo robots run two motors. Eight individually driven wheels means every wheel gets its own driver and its own current reading, so the firmware can tell which side is actually gripping and which one is spinning. Whether that's worth the complexity is something I'll find out.

## What went wrong in v1

These are the main reasons the board looks the way it does now.

Everything on one I2C bus. The OLED, both VL53 sensors and the accelerometer shared the same bus. Address conflicts and a bus that was too busy to be useful.

Not enough room for wiring. I laid the board out without thinking about where cables would physically go. They ended up pressed against things they shouldn't touch and shorted.

Blind distance sensors. I mounted the VL53s without leaving clear space in front of them. Each sensor saw the chassis in its field of view and reported a wall a few centimetres away, constantly.

Unfinished PCB. Beyond the specific faults, a lot of the layout was just rushed. The separate DRV modules where bringing more problems than profits, they are one of the main reasons why the robot didn't work btw.

All of the problems above are fixed in this version

## Pictures

*(to be added — 3D model, PCB render, schematic)*

![3D model](docs/img/cad.png)
![PCB](docs/img/pcb.png)
![Schematic](docs/img/schematic.png)

## Hardware

 MCU - STM32G431 
 Motor drivers - 8x DRV8251/DRV8231, one per motor 
 Current sensing - INA180 per channel -
 IMU - LSM6DSOX 
 Distance - 2x VL53L1X 
 Line detection - 4x KTIR0711S  
 Battery - LiPo, carried over from v1 
 Board - 70x70 mm, 2 layer 

## Assembly

1. Order the PCB as an assembled run, with the stencil.
2. Hand solder anything the assembler skipped: paste through the stencil, place, reflow on a hot plate.
3. Go easy on paste under the DRV8251s. HSOP-8 has a thermal pad, and too much paste floats the part so it cures tilted with open pins.
4. Inspect the board along a low angle before powering anything. Look for tombstoned parts, skewed drivers and bridges on the fine pitch pins.
5. Put the board on a bench supply with the current limit set low, a couple hundred mA.
6. Confirm the 3.3 V rail is actually at 3.3 V, not sagging and not high.
7. Confirm the current draw is small and stable. If it runs away the moment you apply power, stop and find the short.
8. Touch the regulators and drivers. Nothing should be warm at idle.
9. Do not skip to a battery at this stage. A LiPo will deliver enough current to destroy the board before you notice anything is wrong.
10. Connect SWD and flash a minimal build.
11. Confirm the debugger sees the chip and the clock comes up.
12. Toggle something you can observe: a LED, a UART print, anything.
13. Scan the I2C bus and confirm the IMU and both VL53s each answer on their own address. This is the exact failure from v1, so it gets checked here, not after assembly.
14. Connect one motor to one driver, still on the bench supply, still no chassis.
15. Command it at low duty in both directions and confirm it spins the way you expect.
16. Load the shaft with your fingers and confirm the current reading on that channel moves.
17. Confirm the driver does not get hot.
18. Repeat for all eight channels.
19. Write down which channel maps to which physical wheel position. You will need this for the firmware, and guessing later wastes an afternoon.
20. Mix and cast the silicone tires. Get the ratio right and use mould release.
21. Cast all eight from one batch if you can. Different batches cure to slightly different hardness, and uneven grip left to right makes the robot pull.
22. Let them cure fully before fitting. Not "mostly".
23. Fit the tires and spin each wheel by hand to check it runs true, without wobble.
24. Fit the motors into the printed mounts, four per side.
25. Before tightening, sit the chassis on a flat surface and check all eight wheels touch it. A wheel in the air contributes nothing and its current reading will lie to the firmware.
26. Check the gearbox output shafts are parallel. A crooked motor fights the others and burns current for no thrust.
27. Bolt the plough to the steel base plate.
28. Mount the base plate to the chassis.
29. Check the plough height along the whole edge against a flat surface, not just in the middle. A corner sitting high is a corner an opponent gets under.
30. Mount the board.
31. Route the motor wiring away from the sensor lines.
32. Keep the power wiring short and anchored. Silicone wire is flexible, which also means it moves under acceleration, so tie it down.
33. Check the clearance in front of both VL53 sensors. Nothing in the field of view: no chassis edge, no wire, no bolt head.
34. Power the sensors and confirm they read the real distance to a wall a metre away. A fixed short number means something is still in front of them.
35. Put the robot on a stand with all wheels free and run the full state machine.
36. Confirm it searches, reacts to a target, and triggers on the line sensors when you hold something reflective underneath.
37. Only then put it on the ring, and keep a hand near the kill switch for the first few runs.


 
## Flashing

The board exposes SWD. Flash with ST-Link:

```
st-flash write build/bolt.bin 0x8000000
```

Or open the project in STM32CubeIDE and flash from there.

Test each motor channel on its own first. With eight drivers, a single reversed motor is easy to miss once everything is spinning.

## BOM


 PCBA 70x70 mm, 2 layer — board, stencil, assembly, components, shipping - 5 - $135 
 Pololu micro metal gearmotor HP 50:1 6V - 4 - $85 
 Steel base plate, cut to size - 1 - $14 
 Steel plough, CNC machined - 1 - $14–41 
 VL53L1X ToF sensor - 2 - $11 
 KTIR0711S reflective sensor - 4 - $5 
 Interface PCB 50x30 mm, 2 layer - 5 - $5–11 
 Silicone casting compound A+B - 1 kg - $20–40 
 PETG filament - 2 kg - $16–22 
 Solder paste - 1 - $8–32 
 Hardened 3D printer nozzle - 1 - $8–32 
 Silicone wire - 1 set-  $5–14 
 Electric screwdriver - 1 - $25–65 

Four of the eight motors, the LiPo packs and most of the mechanical hardware is taken from he 1 version.

## Known issues
-

## Credits

- Pololu, for the VL53L1X driver the v1 firmware was ported from.
- ST, for the HAL and the VL53 API.
