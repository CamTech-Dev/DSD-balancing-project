# FPGA Ball-and-Beam Control System

A Verilog project for the Digilent Nexys A7 that controls a ball-and-beam system using ultrasonic distance feedback and a servo motor.

The design measures the ball position with an ultrasonic sensor, compares the measured position with a switch-selected setpoint, and adjusts the beam angle through a proportional controller. The measured distance is also shown on the onboard seven-segment display.

## Hardware

- Digilent Nexys A7
- Ultrasonic distance sensor
- Servo motor
- Ball-and-beam mechanical setup
- Jumper wires and external power as required

## Features

- Ultrasonic distance measurement
- Closed-loop proportional position control
- Manual servo control mode
- PWM servo output
- Seven-segment distance display
- Binary-to-BCD conversion
- Nexys A7 pin constraints included

## Control Method

The automatic mode uses a proportional controller:

```text
error = setpoint - measured_position
control = CENTER + (KP * error)
```

Current controller constants:

```text
CENTER = 150000
MINC   = 100000
MAXC   = 200000
KP     = 10
```

The controller output is limited between `MINC` and `MAXC` before being sent to the PWM module.

This is a proportional-only controller, not a full PID controller.

## Operating Modes

### Automatic Mode

`SW[15] = 1`

The FPGA:

1. Reads the desired position from `SW[7:0]`.
2. Measures the current ball position with the ultrasonic sensor.
3. Calculates the position error.
4. Applies proportional control.
5. Sends the resulting value to the PWM module.
6. Adjusts the servo position.

### Manual Mode

`SW[15] = 0`

The controller is bypassed and the switches directly determine the PWM compare value. This mode is useful for testing the servo and beam movement.

## Main Modules

### `BallBeamTop`

Top-level module that connects the sensor, controller, PWM generator, and display logic.

### `Measurer`

Generates the ultrasonic trigger pulse, counts the echo duration, converts the measured value into a distance result, and sends the result to both the controller and display logic.

### `Controller`

Calculates:

```text
setpoint - measured position
```

and applies the proportional gain `KP`.

### `PWM`

Generates the servo PWM signal from the controller or manual input.

### `PWM_Format`

Limits the PWM compare value to the valid range used by the servo.

### `division`

Implements binary division used to scale the ultrasonic timing result.

### `bin2bcd`

Converts the binary distance result into BCD digits.

### `SSEG`

Drives and multiplexes the Nexys A7 seven-segment display.

### `CLK100MHZ_divider`

Creates the slower clock used by the display multiplexing logic.

## Nexys A7 Connections

| Signal | Board Connection | Purpose |
|---|---|---|
| `CLK100MHZ` | E3 | 100 MHz system clock |
| `PWM_Pulse` | JA[1] / C17 | Servo PWM output |
| `Echo` | JB[1] / D14 | Ultrasonic echo input |
| `Trig` | JB[2] / F16 | Ultrasonic trigger output |
| `SW[15:0]` | Onboard switches | Mode and position input |
| `A-G`, `DP` | Onboard display pins | Seven-segment outputs |
| `Enable[7:0]` | Onboard display pins | Seven-segment digit enables |

## Build and Program

### Requirements

Install:

- Xilinx Vivado
- Nexys A7 board files if they are not already available in Vivado

### Build Steps

1. Open Vivado.
2. Create a new RTL project.
3. Select the correct Nexys A7 device or board.
4. Add the Verilog source file containing the project modules.
5. Add the provided `.xdc` constraints file.
6. Set `BallBeamTop` as the top-level module.
7. Run **Synthesis**.
8. Run **Implementation**.
9. Generate the **Bitstream**.

### Program the FPGA

1. Connect the Nexys A7 to the computer through USB.
2. Open **Hardware Manager** in Vivado.
3. Open the hardware target.
4. Select the generated bitstream.
5. Click **Program Device**.

## Running the Project

1. Connect the ultrasonic sensor to the configured `Trig` and `Echo` pins.
2. Connect the servo signal line to `PWM_Pulse`.
3. Power the sensor and servo correctly.
4. Program the FPGA.
5. Use `SW[15]` to select automatic or manual mode.
6. In automatic mode, use `SW[7:0]` to select the desired ball position.
7. Observe the measured distance on the seven-segment display.

## Repository Structure

A simple repository layout is:

```text
Ball-Beam-FPGA/
├── BallBeamTop.v
├── BallBeam.xdc
└── README.md
```

## Results

The completed system was able to keep the ball near the center of the beam using ultrasonic distance feedback and proportional control.

As the controller was tuned for finer corrections, the servo made smaller adjustments and gradually moved the ball toward the center instead of making large sudden movements.

This demonstrated that the feedback loop was functioning correctly and that the proportional controller could continuously correct the ball position based on the measured distance.

## My Contributions

- Implemented and integrated the Verilog control logic on the Nexys A7.
- Worked with ultrasonic distance feedback for ball-position measurement.
- Integrated the seven-segment display and FPGA I/O constraints.
- Tested and tuned the controller so the servo made progressively smaller corrections as the ball approached the center.

## Notes

- The project uses a 100 MHz system clock.
- The ultrasonic measurement result is scaled in hardware before being displayed.
- Servo output is constrained to a fixed minimum and maximum compare value.
- The current feedback controller uses only proportional control.

## Possible Improvements

- Add integral and derivative terms for full PID control
- Add filtering or averaging to ultrasonic measurements
- Make `KP` adjustable with switches or buttons
- Add UART output for debugging and data logging
- Add simulation testbenches for the controller, PWM, and measurement logic
