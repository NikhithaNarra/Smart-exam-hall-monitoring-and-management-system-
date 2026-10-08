Smart Exam Hall Monitoring and Management System

Bare-metal embedded firmware for ARM7TDMI-S

An embedded examination-hall management system built around the NXP LPC2148. The firmware uses direct register-level peripheral control to automate examination timing, temperature monitoring, countdown display, configuration, and status alerts without an RTOS or hardware abstraction layer.

Core approach: Register-level drivers · Bare-metal Embedded C · Timer/VIC interrupts · Modular peripheral architecture

Project Overview

The system is designed to automate the timing and monitoring of an examination hall.

Instead of depending on the invigilator to manually start, track, pause, and end an examination, the firmware uses the RTC and a configured examination schedule to control the complete process.

During normal operation, the system displays the current time, date, and room temperature. When the RTC reaches the configured examination start time, the firmware automatically enters examination mode and begins the countdown.

During the examination:

Remaining time is shown on dual multiplexed 7-segment displays.

Temperature is continuously monitored using an LM35.

The LCD provides system and examination information.

Green, Yellow, and Red LEDs indicate the remaining-time status.

EINT1 provides pause/resume control.

The buzzer indicates examination completion.

EINT0 provides access to the protected configuration menu.

Key Characteristics

Category

Implementation

Microcontroller

NXP LPC2148

CPU

ARM7TDMI-S

Clock

60 MHz

Programming

Embedded C

Architecture

Bare-metal / register-level

HAL

Not used

RTOS

Not used

Interrupt Controller

VIC

IDE

Keil MDK / µVision

Programming Tool

Flash Magic

Simulation

Proteus ISIS

Temperature Sensor

LM35

Display

20×4 LCD + dual 7-segment

User Input

4×4 matrix keypad

Timing

RTC + Timer0

System Architecture

                         ┌─────────────────────────┐
                         │       INPUTS             │
                         │                         │
                         │  4×4 Keypad             │
                         │  EINT0 Configuration SW │
                         │  EINT1 Pause/Resume SW  │
                         │  LM35 Temperature      │
                         │  RTC                    │
                         └────────────┬────────────┘
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │        LPC2148          │
                         │      ARM7TDMI-S         │
                         │                         │
                         │  ┌───────────────────┐  │
                         │  │ Application FSM   │  │
                         │  │                   │  │
                         │  │ IDLE              │  │
                         │  │ CONFIG            │  │
                         │  │ EXAM_RUNNING      │  │
                         │  │ EXAM_END          │  │
                         │  └───────────────────┘  │
                         │                         │
                         │  Timer0 → VIC → ISR    │
                         │  EINT0  → VIC → ISR    │
                         │  EINT1  → VIC → ISR    │
                         └────────────┬────────────┘
                                      │
              ┌───────────────────────┼──────────────────────┐
              ▼                       ▼                      ▼
       ┌────────────┐          ┌────────────┐         ┌────────────┐
       │    LCD     │          │  7-Segment │         │ LED +      │
       │ Time/Date  │          │ Countdown  │         │ Buzzer     │
       │ Temperature│          │ 00–99 min  │         │ Alerts     │
       └────────────┘          └────────────┘         └────────────┘

Operating Flow

                         POWER ON
                            │
                            ▼
                    ┌───────────────┐
                    │   INITIALIZE  │
                    │ Peripherals   │
                    │ VIC / Timer   │
                    │ RTC / LCD etc.│
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │   IDLE MODE   │
                    │               │
                    │ TIME / DATE   │
                    │ TEMPERATURE   │
                    │               │
                    │ Wait for RTC  │
                    │ start time    │
                    └───────┬───────┘
                            │
                ┌───────────┴────────────┐
                │                        │
             EINT0                  RTC matches
                │                   exam start
                ▼                        │
        ┌───────────────┐                ▼
        │ CONFIGURATION │       ┌────────────────┐
        │     MODE      │       │ EXAM RUNNING   │
        │               │       │                │
        │ Password      │       │ Countdown      │
        │ RTC edit      │       │ Temperature    │
        │ Exam time     │       │ LED warnings   │
        │ Duration      │       │ 7-segment      │
        │ Password      │       │                │
        └───────┬───────┘       └───────┬────────┘
                │                       │
                │                       │ EINT1
                │                       ▼
                │               ┌────────────────┐
                │               │ PAUSED /       │
                │               │ RESUMED        │
                │               └───────┬────────┘
                │                       │
                └───────────────┐       │
                                │       │
                                ▼       │
                         ┌──────────────┘
                         │ Countdown = 0
                         ▼
                  ┌─────────────────┐
                  │   EXAM ENDED   │
                  │                 │
                  │ Buzzer          │
                  │ Red LED         │
                  │ EXAM OVER       │
                  └────────┬────────┘
                           │
                       EINT0
                           │
                           ▼
                       IDLE MODE

Application State Machine

The main application is organized around distinct operating states.

             ┌──────────────┐
             │     IDLE     │
             └──────┬───────┘
                    │ EINT0
                    ▼
             ┌──────────────┐
             │    CONFIG    │
             └──────┬───────┘
                    │ EXIT
                    ▼
             ┌──────────────┐
             │     IDLE     │
             └──────┬───────┘
                    │ RTC == configured start
                    ▼
             ┌──────────────┐
             │ EXAM_RUNNING │
             └──────┬───────┘
                    │ countdown == 0
                    ▼
             ┌──────────────┐
             │   EXAM_END   │
             └──────┬───────┘
                    │ EINT0
                    ▼
             ┌──────────────┐
             │     IDLE     │
             └──────────────┘

The state machine keeps configuration, examination execution, and examination completion logically separated.

Main Features

1. RTC-Based Automatic Exam Start

The examination start hour and minute can be configured through the keypad.

The main application continuously checks the RTC. When the current RTC time matches the configured start time, the system automatically transitions from IDLE to EXAM_RUNNING.

No manual start operation is required once the schedule is configured.

2. Examination Countdown

The examination duration is configured in minutes.

The remaining time is displayed on the dual multiplexed 7-segment display in the range:

00 – 99 minutes

3. Temperature Monitoring

The LM35 provides the room-temperature input through the LPC2148 ADC.

The measured temperature is displayed on the LCD during normal operation and throughout the examination.

4. Three-Level Time Warning

The LEDs provide an immediate visual indication of the remaining examination time.

Remaining Time

Status

More than 3 minutes

Green LED

1–3 minutes

Yellow LED

Less than 1 minute

Red LED

Examination finished

Red LED + buzzer

5. Automatic Examination End

When the countdown reaches zero:

Examination state changes to EXAM_END.

The buzzer operates for 5 seconds.

The Red LED remains active.

The LCD displays the examination-over status.

EINT0 can be used to acknowledge the completed examination and return to IDLE.

6. Pause / Resume

EINT1 provides an emergency pause/resume mechanism.

First EINT1 press  → PAUSE
Second EINT1 press → RESUME

The countdown is frozen while paused and continues from the previous value after resuming.

7. Password-Protected Configuration

EINT0 opens the configuration system.

A four-digit password is requested through the keypad and the entered digits are masked.

Incorrect authentication returns the system to the normal operating mode.

8. Password Change

The administrator can change the existing password after successful authentication.

The current password must be entered before a new password can be configured.

Three incorrect attempts result in the configuration being rejected.

9. Input Validation

Time and date entries are checked before being accepted.

Examples:

Hour    → 0–23
Minute  → 0–59
Day     → 1–31
Month   → 1–12
Duration→ 0–99 minutes

Configuration Menu

EINT0
  │
  ▼
ENTER PASSWORD
  │
  ├── Wrong password
  │       │
  │       └── Return to IDLE
  │
  └── Correct password
          │
          ▼
     CONFIGURATION
          │
          ├── 1. RTC EDIT
          │      ├── Edit Time
          │      ├── Edit Date
          │      └── Exit
          │
          ├── 2. SET EXAM TIME
          │      ├── Exam Start Time
          │      ├── Exam Duration
          │      └── Exit
          │
          ├── 3. EDIT PASSWORD
          │      ├── Current Password
          │      ├── New Password
          │      └── Confirmation
          │
          └── 4. EXIT
                 │
                 ▼
                IDLE

Keypad Interface

┌─────┬─────┬─────┬─────┐
│  1  │  2  │  3  │  A  │
├─────┼─────┼─────┼─────┤
│  4  │  5  │  6  │  B  │
├─────┼─────┼─────┼─────┤
│  7  │  8  │  9  │  C  │
├─────┼─────┼─────┼─────┤
│  *  │  0  │  #  │  D  │
└─────┴─────┴─────┴─────┘

Key

Function

0–9

Numeric input

#

Confirm input

D

Backspace

*

Cancel / exit current input

1–4

Configuration-menu selection

Password input is displayed as masked characters.

Hardware Interface

LPC2148 Pin Mapping

Peripheral / Signal

LPC2148 Pin

Configuration

LCD D0–D7

P0.8–P0.15

8-bit data

LCD RS

P0.16

Output

LCD EN

P0.17

Output

Keypad Rows

P1.16–P1.19

Output

Keypad Columns

P1.20–P1.23

Input + pull-ups

7-Segment A–G + DP

P0.18–P0.25

Segment output

7-Segment DSEL1

P1.24

Active high

7-Segment DSEL2

P1.25

Active high

LM35

P0.28 / AD0.1

ADC input

Green LED

P1.26

Active low

Yellow LED

P1.27

Active low

Red LED

P1.28

Active low

Buzzer

P1.29

Active high via BC109

EINT0

Board-dependent

Configuration switch

EINT1

Board-dependent

Pause/resume switch

Simulation Target

The Proteus implementation uses the LPC2129/LPC2124-family simulation setup with the corresponding external-interrupt pin configuration.

The firmware can use conditional CPU definitions to select the appropriate platform configuration.

Interrupt Architecture

The project uses the Vectored Interrupt Controller (VIC) rather than polling every event in the main application.

VIC Slot

Channel

Source

ISR

Purpose

0

4

Timer0

timer0_isr()

1 ms system tick / display refresh

1

14

EINT0

eint0_isr()

Configuration control

2

15

EINT1

eint1_isr()

Pause/resume

The Timer0 interrupt provides the periodic timing base used by the firmware.

The external interrupt inputs are handled using falling-edge interrupt configuration and software debounce with pin re-verification.

Timer0 and 7-Segment Multiplexing

The 7-segment display is multiplexed rather than driven as two continuously active digits.

Timer0 generates a periodic interrupt:

Timer0
   │
   ▼
VIC
   │
   ▼
timer0_isr()
   │
   ├── Update timing tick
   │
   └── Refresh 7-segment digit

The display refresh is handled from the interrupt context so that the main application does not need to block execution with long delays.

This is important for maintaining a stable multiplexed display while the application is handling RTC, keypad, temperature, and examination logic.

Driver Architecture

The firmware is separated into individual peripheral drivers.

                    APPLICATION
                         │
                         ▼
                mini_project_main.c
                         │
       ┌─────────────────┼─────────────────┐
       │                 │                 │
       ▼                 ▼                 ▼
     INPUT            TIMING            OUTPUT
       │                 │                 │
       ├── Keypad        ├── RTC           ├── LCD
       ├── EINT0         ├── Timer0        ├── 7-Segment
       ├── EINT1        └── Delay          ├── LEDs
       └── LM35 / ADC                     └── Buzzer

The application layer handles the examination state machine and menu logic, while the drivers handle the hardware-specific operations.

Source Tree

The repository is organized into driver source files, headers, peripheral definitions, and the main application.

Smart-Exam-Hall-Monitoring-and-Management-System/
│
├── README.md
│
├── adc.c
├── adc.h
├── adc_defines.h
│
├── delay.c
├── delay.h
│
├── interrupt.c
├── interrupt.h
├── interrupt_defines.h
│
├── kpm.c
├── kpm.h
├── kpm_defines.h
│
├── lcd.c
├── lcd.h
├── lcd_defines.h
│
├── lm35.c
├── lm35.h
│
├── rtc.c
├── rtc.h
├── rtc_defines.h
│
├── seg.c
├── seg.h
├── seg_defines.h
│
├── timer.h
├── timer_dly_seg.c
│
├── types.h
├── defines.h
│
└── mini_project_main.c

Module Responsibilities

File / Module

Responsibility

mini_project_main.c

Main application and examination state machine

adc.c / adc.h

ADC configuration and conversion

lm35.c / lm35.h

LM35 temperature processing

rtc.c / rtc.h

RTC initialization and time/date operations

lcd.c / lcd.h

LCD initialization and data/command handling

kpm.c / kpm.h

4×4 matrix keypad scanning and input

seg.c / seg.h

7-segment display handling

interrupt.c / interrupt.h

VIC and external interrupt configuration

delay.c / delay.h

Software delay functions

timer_dly_seg.c

Timer-related timing and display-refresh support

*_defines.h

Peripheral-specific definitions

types.h

Common data types

defines.h

Common project definitions

Software Flow

The application follows this high-level execution model:

Initialize hardware
       │
       ├── GPIO
       ├── LCD
       ├── RTC
       ├── ADC
       ├── Keypad
       ├── Timer0
       └── VIC / EINT
       │
       ▼
Enter IDLE
       │
       ├── Update LCD
       ├── Read temperature
       ├── Monitor RTC
       └── Check configuration request
       │
       ▼
RTC reaches configured exam time
       │
       ▼
Start examination
       │
       ├── Start countdown
       ├── Update remaining time
       ├── Update temperature
       ├── Update LEDs
       └── Refresh 7-segment
       │
       ├── EINT1 → pause/resume
       │
       ▼
Countdown reaches zero
       │
       ▼
End examination
       │
       ├── Buzzer
       ├── Red LED
       └── EXAM OVER display
       │
       ▼
Return to IDLE

Development Environment

IDE

Keil MDK / µVision

The firmware is developed as a bare-metal Embedded C project targeting the ARM7 LPC2148 family.

Programming / Flashing

Flash Magic

The generated HEX image can be programmed into the LPC2148 through the board's ISP/UART interface.

Simulation

Proteus ISIS

The project can be tested in simulation using the corresponding LPC2129/LPC2148-family configuration.

Build and Flash

1. Clone

git clone <your-github-repository-url>
cd Smart-Exam-Hall-Monitoring-and-Management-System

2. Open the Keil Project

Open the project in Keil µVision and verify that the application and driver source files are included in the target.

3. Select the Target

Select the appropriate NXP LPC2148 device and configure the project for the board's oscillator settings.

For simulation, select the corresponding CPU configuration used by the Proteus project.

4. Build

Use:

Project → Build Target

or press:

F7

The expected result is a successful build with no compilation or linker errors.

5. Generate the HEX File

Configure the Keil target to generate the HEX output required by Flash Magic.

6. Flash the LPC2148

Use Flash Magic with the appropriate:

COM port

LPC2148 device

ISP interface

Oscillator configuration

Generated HEX file

After programming, return the board to normal execution mode and reset the controller.

Running the System

Initial State

After reset, the system initializes the peripherals and enters IDLE.

The LCD provides the current monitoring information, including:

TIME
DATE
TEMPERATURE

The configured RTC schedule is continuously monitored.

Configure an Examination

Trigger EINT0.

Enter the administrator password.

Select the examination configuration menu.

Set the examination start hour and minute.

Set the examination duration.

Exit the configuration menu.

The system then waits in IDLE until the RTC reaches the configured start time.

During Examination

The system automatically enters EXAM_RUNNING.

The LCD shows current examination information, while the 7-segment display provides the remaining-minute countdown.

The LEDs indicate the remaining duration.

Pause / Resume

Press the EINT1 switch once to pause the countdown.

Press it again to resume.

Examination Completion

When the countdown reaches zero:

EXAMINATION COMPLETE
        │
        ├── Buzzer ON
        ├── Red LED
        └── EXAM OVER message

The completed examination can then be acknowledged and the system returned to IDLE.

Design Decisions

Bare-Metal Architecture

The project intentionally avoids:

HAL libraries

RTOS

High-level peripheral frameworks

Peripheral registers are configured directly to provide a clearer understanding of the ARM7 microcontroller and its hardware.

Modular Drivers

Each peripheral has its own source/header pair wherever applicable.

This avoids putting LCD, RTC, keypad, ADC, timer, and interrupt logic into one large application file.

Interrupt-Driven Display Refresh

The multiplexed 7-segment display is refreshed from Timer0 rather than relying on a long blocking delay in the main application.

This allows the application to continue handling other operations while the display is refreshed periodically.

State-Based Application Logic

The examination application is separated into logical states rather than treating every condition as an unrelated flag.

This makes transitions such as:

IDLE → CONFIG
IDLE → EXAM_RUNNING
EXAM_RUNNING → PAUSED
PAUSED → EXAM_RUNNING
EXAM_RUNNING → EXAM_END
EXAM_END → IDLE

easier to maintain and debug.

Testing Areas

The project can be tested at both driver and application levels.

Peripheral Tests

LCD initialization and text display

Keypad row/column scanning

RTC time/date read and write

ADC conversion

LM35 temperature measurement

7-segment digit output

Timer0 interrupt generation

EINT0 and EINT1 operation

Application Tests

Password authentication

Invalid time/date input

Examination scheduling

Countdown operation

LED threshold transitions

Pause/resume

Examination completion

Buzzer operation

Return to IDLE after completion

Project Highlights

Bare-metal ARM7 firmware

Direct register-level peripheral programming

Modular driver architecture

RTC-controlled automatic examination start

Interrupt-driven timing

Multiplexed 7-segment display

ADC-based LM35 temperature measurement

Password-protected configuration

External-interrupt pause/resume

Input validation

LED-based examination status indication

Audible examination-completion alert

Hardware-oriented embedded C implementation

Future Improvements

Possible extensions for a production-oriented version include:

Persistent storage of examination configuration

Examination history and event logging

UART-based monitoring from a PC

Remote examination-status reporting

Multiple examination-room support

More advanced date validation, including month-specific day limits

Improved fault handling and watchdog supervision

Author

Narra Nikhitha

B.Tech – Electronics and Communication Engineering

Embedded Systems / Firmware Development

Project Focus

This project demonstrates practical embedded firmware development using:

ARM7TDMI-S
     +
Embedded C
     +
Register-Level Programming
     +
Peripheral Drivers
     +
VIC Interrupts
     +
Timers
     +
RTC
     +
ADC
     +
GPIO
     +
State Machine

The emphasis is on understanding and controlling the microcontroller at the peripheral-register level rather than relying on high-level frameworks.
