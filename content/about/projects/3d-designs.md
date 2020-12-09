---
title: "3D Printing Designs"
date: 2020-12-09 09:35:58.515346
draft: false
---

### 3D Designs

#### Some of my designs for 3D printing

### [Ikea Lamp Desk Mount Bracket](https://cults3d.com/en/3d-model/home/ikea-lamp-desk-mount-bracket)

![Ikea Lamp Desk Mount Bracket](IMG_0276_large.JPG)

Replacement bracket for the desk mount piece, works with the existing hardware.

### [Simple 40mm Fan Bracket](https://cults3d.com/en/3d-model/tool/simple-40mm-fan-bracket)

![Simple 40mm Fan Bracket](IMG_4640_large.jpg)

A simple fan L bracket for 40mm fans.  Mounts easily to 2020 aluminium extrusion with M3 bolts.

### [Bed Level Knob M3 Nut](https://cults3d.com/en/3d-model/tool/bed-level-knob-m3-nut)

![Bed Level Knob M3 Nut](P_20170719_100716_vHDR_Auto_cropped_large.jpg)

This is a bed level knob for M3 size knurled nuts.

Printed at 0.2mm layer height with 15% infill.

### [2020 Extrusion 90 Degree Bracket](https://cults3d.com/en/3d-model/tool/2020-extrusion-90-degree-bracket)

![2020 Extrusion 90 Degree Bracket](P_20170804_201850_cropped_large.jpeg)

A 90 degree bracket for mounting with 2020 extrusions.  Has a channel that accepts T-Slot nuts for mounting things at right angles to the frame.

### [Titan Extruder Bowden Lock Clip](https://cults3d.com/en/3d-model/tool/titan-extruder-bowden-lock-clip)

![Titan Extruder Bowden Lock Clip](titan_bowden_clip_large.jpg)

Clip to ensure bowden tube is snug in the coupler.

Printed at 0.2mm layer height in PETG.

### [RC Transmitter Stand](https://cults3d.com/en/3d-model/game/rc-transmitter-stand)

![RC Transmitter Stand](P_20180126_143849_vHDR_Auto_large.jpg)

A stand for holding up an RC transmitter.  Fits on the metal bar at the back of your radio, tested on my Taranis.

Use 2mm carbon fibre rods for the cross supports.  Glue the rods in the holes of the leg piece.

### [Briefcase Bike Pannier Hook](https://cults3d.com/en/3d-model/various/briefcase-bike-pannier-hook)

![Briefcase Bike Pannier Hook](P_20180523_200239_vHDR_Auto_large.jpg)

Turn a briefcase with a luggage strap into a pannier for your bicycle.  Mounts to a rack with zipties and allows you to attach or remove your bag from the bike.

Printed in PETG with 40% infill and it's strong enough to hold a loaded briefcase.

### [Ender 3 Fysetc F6 Case](https://cults3d.com/en/3d-model/tool/ender-3-fysetc-f6-case)

![Ender 3 Fysetc F6 Case](IMG_0031_large.JPG)

Case for the Fysetc F6 controller board with the Ender-3. Works with original cover and screws and has lots of space for wiring.

Printed with 2 shells and 10% infill in PLA.

### [Ender 3 Side Spool Mount](https://cults3d.com/en/3d-model/tool/ender-3-side-spool-mount)

![Ender 3 Side Spool Mount](IMG_0158_large.JPG)

Side spool mount for the Ender-3 filament holder. Moves the filament spool holder to the side of the printer and moves the roll to be parallel with the Y axis. You may need to use some PTFE tubing and route the filament path with clips to keep the filament moving smoothly.

Works with the original Ender-3 spool mount arm.

### [Ender-3 Filament Waste Squeegee](https://cults3d.com/en/3d-model/tool/ender-3-filament-waste-squeegee)

![Ender-3 Filament Waste Squeegee](ezgif.com-optimize2_large.gif)

After modifying my Ender-3 to be direct drive I found the X axis had a bit of extra range to the side of the bed. Using this space I designed a waste tray and squeegee to make priming the nozzle a quick and clean process. No longer will you have to remove the extra plastic lines on the print bed from priming the nozzle. 

In order to use this you will need at least 248mm of range in your X axis. You will also need a piece of silicone for the squeegee part. I used a piece from an automotive detailing squeegee but any piece around 2.75mm thick will work. 

Detailing squeegee I used (affiliate link): https://amzn.to/2tiy0A4

You will also need to move the display over in order to leave space for the tray. I designed a simple mount to move the display over [here.](https://www.thingiverse.com/thing:4048717)

After installing you can update your print starting gcode to clean the nozzle before printing. I'm using this gcode at the moment:

```
M80                               ; turn power on
M73 P0                            ; Reset print progress bar
G21                               ; metric values
M117 Homing...
G28                               ; home
M117 Cleaning nozzle...
G92 E0                            ; Reset Extruder
G1 Z5.0 F3000                     ; Move Z Axis up little to prevent scratching of Heat Bed
G1 X248 Y10 F3000.0               ; Move to start position
G1 E20.8 F300.0                   ; Extrude 20mm
G1 E18.8 Z0 F500.0                ; Retract and lower
G1 X230 Y10 F2000.0               ; Wipe
G1 X248 Y10 F2000.0               ; Wipe
G1 Z0 X200 Y10 F1500.0            ; 
M117 Leveling bed...
M280 P0 S160                      ; BLTouch alarm release
G4 P100                           ; delay for BLTouch
G29                               ; auto bed leveling
M117 Cleaning nozzle...
G92 E0                            ; Reset Extruder
G1 Z5.0 F3000                     ; Move Z Axis up little to prevent scratching of Heat Bed
G1 X248 Y10 F3000.0               ; Move to start position
G1 E20.8 F300.0                   ; Extrude 20mm
G1 E18.8 Z0 F500.0                ; Retract and lower
G1 X230 Y10 F2000.0               ; Wipe
G1 X248 Y10 F2000.0               ; Wipe
G1 Z0 X200 Y10 F1500.0            ; 
M117 Printing...
```

### [Ender-3 Display Offset Mount](https://cults3d.com/en/3d-model/tool/ender-3-display-offset-mount)

![Ender-3 Display Offset Mount](IMG_0181_large.JPG)

Offsets the display from the frame of the Ender-3 in order to accommodate a waste filament tray.

