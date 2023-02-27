# Hardware

## Printer

### Chassis

[Tenlog TL-3D Pro](https://www.tenlog3d.com/tenlog-tl-d3-pro-independent-dual-extruder-dmp-3d-printer-with-touch-screen-support-pva-tpu-abs-pla-600w-version_p11.html)

| Machine Dimensions | Width | Depth | Height |
|--------------------|-------|-------|--------|
| Machine Size | 590mm | 520mm | 760mm |
| Build Volume | 300mm | 300mm | 350mm |

No mechanical modification on the printer except fitting of the screw domes for the new motherboard.

## Electronic

### Motherboard

[BIGTREETECH OCTOPUS Pro V1.0.1](https://github.com/bigtreetech/BIGTREETECH-OCTOPUS-Pro)

### CPU

**STM32F429ZG**

### Stepper driver

The [TMC2209](https://github.com/bigtreetech/BIGTREETECH-TMC2209-V1.2) has only Step/Dir mode and UART, **NO SPI**.  
Therefor the pins below the StepDriver are configured in UART mode:

```
oIoo
oIoo
oooo
```

The output voltage of the StepDriver should be the same as the supply voltage.  
Since the TCM2209 is not supporting other output voltages,
the jumper will use the 24V of the board:

```
o==
```

The TMC2209 supports StallGuard, which is connected to the diag pin.  
The StallGuard is overwriting the endstops, therefor they will not be used:

```
oooooooo
oooooooo
```

### Fans

The Fans are all 24V therefor the jumpers will bridge the last pins:

```
oo  5V
oo  12V
==  24V
```

### PT100 DIP Switch

The Tenlog 3D-Pro is using PT100 temperature sensors.  
Therefor all dip switches will be set to ON (which was default on my board).

### Power supply

To avoid powering the board via USB,  
the Power jumper in the center of the board needs to me removed.  
For installation and debugging I will keep it connected for now,  
But in production **I need to remove** the jumper!

### BLTouch

The BLTouch will be connected the flowing way:

```
ooooo
||||'-- white  [OUT] \ Sensor     > PB7
|||'--- black  [GND] /            x N/C
||'---- yellow [IN]  \            > PB6
|'----- red    [+5V]  | Controll  > 5VDC
'------ blue   [GND] /            > GND
```

### sub-D (DE-15)

#### E1/E2 Pinout

```
        Female               Male
   ,---------------.   ,---------------.
 5 \   o o o o o   / 1 \   * * * * *   / 5
10  \   o o o o o /  6  \   * * * * * / 10
15   \ o o o o o /  11   \ * * * * * /  15
      '---------'         '---------'
```

| 🟩 | Pin | Function |  | 🟩 | Pin | Function |
|----|-----|----------|--|----|-----|----------|
| ✅ | 1 | E 3 (C2) |  | ✅ | 3 | E 1 (C1) |
| ✅ | 6 | FZ - |  | ⬛ | 8 | LED + |
| ✅ | 2 | E 4 (C2) |  | ✅ | 14 | FC, FZ, H + |
| ✅ | 7 | FC - |  | ⬛ | ~~13~~ | *N/C* |
| ✅ | 9 | DJ |  | ✅ | 4 | E 2 (C1) |
| 🔀 | 10 | H - |  | ✅ | 5 | H - |
| ✅ | 11 | RH |  | ✅ | 15 | FC, FZ, H + |
| ✅ | 12 | DJ, RH, LED - |  |  |  |  |

| Function | Pins | Explanation |
|----------|------|-------------|
| FZ | 6, 14, 15 | Hot-End Cooling (red/black) |
| FC | 7, 14, 15 | Part Cooling (yellow/blue) |
| RH | 11, 12 | Temp Sensor |
| DJ | 9, 12 | Filament Run-Out Sensor |
| E | 1, 2, 3, 4 | Stepper |
| H | 5, 10, 14, 15 | Heater |
| LED | 8, 12 | Blue LED |

### Fans

Seems like all fans can be connected directly.

#### Housing Fans

Use the 2 extra fan plugs (Fan4 & Fan5) in Controller Fan Mode.  
This disables the fans if there is no driver or MOSFETs in use.

# [Firmware](https://github.com/abangtor/Marlin4TL3DProWithOctopusPro)

## Stepper

| Plug | Driver | Axis |
|------|--------|------|
| M0 | Driver 0 | X1 |
| M1 | Driver 1 | Y |
| M2.1/M2.2 | Driver 2 | Z1 |
| M3 | Driver 3 | Z2 |
| M4 | Driver 4 | E0 |
| M5 | Driver 5 | E1 |
| M6 | Driver 6 | X2 |
| M7 | Driver 7 | *NC* |

## Endstops/Filament

| Port | Plug | Axis | Mod |
|------|------|------|-----|
| PG6 | Stop 0 | X1 | Org:PSG > New:PGS |
| PG9 | Stop 1 | Y | Org:PSG > New:PGS |
| PG10 | Stop 2 | Z1 | Org:PSG > New:PGS |
| PG11 | Stop 3 | Z2 | Org:PSG > New:PGS |
| PG12 | Stop 4 | E0 | E1 sub-D 9 |
| PG13 | Stop 5 | E1 | E2 sub-D 9 |
| PG14 | Stop 6 | X2 | Org:PSG > New:PGS |
| PG15 | Stop 7 | *NC* |  |

## Fans

| Port | Plug | Fan |
|------|------|-----|
| PA8 | FAN0 | E0 Part Cooling Fan |
| PE5 | FAN1 | E1 Part Cooling Fan |
| PD12 | FAN2 | E0 Hot-End Cooling Fan |
| PD13 | FAN3 | E1 Hot-End Cooling Fan |
| PD14 | FAN4 | Housing Fan 1 |
| PD15 | FAN5 | Housing Fan 2 |

## Hot-Ends

| Port | Plug | Device |
|------|------|--------|
| PA1 | Heated Bed | Bed |
| PA2 | HE0 | E1 |
| PA3 | HE1 | E2 |
| PB10 | HE2 | *N/C* |
| PB11 | HE3 | Casing Light |

## Buffers

Increased all buffers to the max to reduce lags in WiFi data transfer.  
Enabled XON/XOFF control characters.

```
#define BLOCK_BUFFER_SIZE 32
#define MAX_CMD_SIZE 96
#define BUFSIZE 32
#define TX_BUFFER_SIZE 256
#define RX_BUFFER_SIZE 2048
#define SERIAL_XON_XOFF
```

## Calibration

### PID Auto Tune

#### Bed

I run a PID Auto Tune with 9 cycles on the bed:

```
M303 C9 S60 D1 U1 E-1
```

With the new values the configuration is:

```
#define DEFAULT_bedKp 136.38
#define DEFAULT_bedKi 25.83
#define DEFAULT_bedKd 480.06
```

### Hotends

I run a PID Auto Tune with 9 cycles on both hotends:

```
M303 C9 S210 D1 U1 E0
M303 C9 S210 D1 U1 E1
```

With the new values the configuration is:

```
#define DEFAULT_Kp_LIST { 26.65, 24.61 }
#define DEFAULT_Ki_LIST {  3.00,  2.65 }
#define DEFAULT_Kd_LIST { 59.16, 57.10 }
```

### Step Driver Current

**TMC2209**  

$I_{RMS}=\frac{325\textup{mV}}{R_{sense}+20\textup{m}\Omega}\cdot\frac{1}{\sqrt{2}}\cdot\frac{V_{ref}}{2.5\textup{V}}$

|  Model   | Length | Current | Resistance | Inductance | H.Torque | D.Torque |
|:--------:|:------:|:-------:|:----------:|:----------:|:--------:|:--------:|
| 17HS2408 | 28 mm  |  0.6 A  |    8 Ω     |   10 mH    | 12 N·cm  | 1.6 N·cm |
| 17HS3401 | 34 mm  |  1.3 A  |    2.4 Ω   |    2.8 mH  | 28 N·cm  | 1.6 N·cm |
| 17HS3410 | 34 mm  |  1.7 A  |    1.2 Ω   |    1.8 mH  | 28 N·cm  | 1.6 N·cm |
| 17HS3403 | 34 mm  |  0.4 A  |   30 Ω     |   35 mH    | 28 N·cm  | 1.6 N·cm |
| 17HS4401 | 40 mm  |  1.7 A  |    1.5 Ω   |    2.8 mH  | 40 N·cm  | 2.2 N·cm |
| 17HS4402 | 40 mm  |  1.3 A  |  **2.5 Ω** |    5 mH    | 40 N·cm  | 2.2 N·cm |
| 17HS8401 | 48 mm  |  1.7 A  |    1.8 Ω   |    3.2 mH  | 52 N·cm  | 2.6 N·cm |
| 17HS8402 | 48 mm  |  1.3 A  |  **3.2 Ω** |    5.5 mH  | 52 N·cm  | 2.6 N·cm |
| 17HS8403 | 48 mm  |  2.3 A  |    1.2 Ω   |    1.6 mH  | 46 N·cm  | 2.6 N·cm |

* X- & Z-Axis
  - 42-40 Motor (~3.2 Ω)
  - 1.3 A (- 10%) => 1170
* Y- & E-Axis
  - 42-48 Motor (~3.2 Ω)
  - 1.3 A (- 10%) => 1170

### UBL

1. Level bed manually on all 4 corners
2. Run  
   ```
   G0 Z10
   G30 X154 Y154
   ```
3. Read out result and calculate  
   $Z_{M851}=Z_{M851}-Z_{Probe}{\color{Gray}-Z{Paper}}$
4. Write probe offset with  
   ```
   M851 Z#.##
   ```
5. Run UBL
   ```
   M420 S0   ; disable bed leveling
   G28       ; home all axes
   M155 S30  ; reduce temperature reporting rate to reduce output pollution
   M190 S60  ; wait for the bed to get up to temperature
   G4 S60    ; wait another 1 min for the bed to reach temperature
   G29 P0    ; clear mesh
   G29 P1    ; automatically populate mesh with all reachable points
   G29 P3    ; infer the rest of the mesh values
   G29 P3    ; infer the rest of the mesh values again
   M420 S1 V ; enabled leveling and report the new mesh
   M500      ; save the new mesh to EEPROM
   M155 S3   ; reset temperature reporting
   M140 S0   ; cooling down the bed
   ```
6. Correct whole mesh with
   ```
   G29 P6 C#.##
   ```