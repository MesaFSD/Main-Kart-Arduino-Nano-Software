# Main-Kart-Arduino-Nano-Software
Sketches for the arduino nano, main kart

Wiring: 
<img width="653" height="287" alt="image" src="https://github.com/user-attachments/assets/1a73c7b8-15a5-455f-a43b-cbdf1dd9e681" />

Claude Context: 
FSD Autonomous Go-Kart — System Context (Current State)
Project Identity
MesaFSD autonomous go-kart for the AKS 2026 competition. Tier 1 MVP. Combines two parent platforms: the donkey car (proven autonomy stack) and the original RC kart (proven actuation stack).
Architecture Overview
Layered autonomy stack:
QGC (laptop)→Pixhawk Cube Orange+→Arduino Nano→Actuators\text{QGC (laptop)} \rightarrow \text{Pixhawk Cube Orange+} \rightarrow \text{Arduino Nano} \rightarrow \text{Actuators}QGC (laptop)→Pixhawk Cube Orange+→Arduino Nano→Actuators

Cube Orange+ runs ArduRover v4.6.3 (Ackermann frame). Handles all navigation: EKF, L1 path-following, mode arbitration (MANUAL/AUTO/HOLD), RC failsafe.
Arduino Nano (ATmega328P, 5V logic) is the low-level actuator controller and safety layer. Reads Cube PWM, drives steering and throttle, enforces software end-stops, runs failsafe. No navigation logic.

Control Flow
QGC creates waypoint missions and uploads to the Cube via MAVLink (USB on bench, future Raspberry Pi 5 + AI Hat MAVProxy bridge in the field). The Pi is NOT in the control loop, it only relays telemetry. If the Pi dies, the Cube continues the mission.
RC Chain
RadioMaster Pocket M2→ELRS 2.4GHzRP3 V2→SBUSCube RCIN\text{RadioMaster Pocket M2} \xrightarrow{\text{ELRS 2.4GHz}} \text{RP3 V2} \xrightarrow{\text{SBUS}} \text{Cube RCIN}RadioMaster Pocket M2ELRS 2.4GHz​RP3 V2SBUS​Cube RCIN
The Cube handles RC arbitration and failsafe, then outputs PWM on MAIN-OUT pins to the Nano. Chosen over RC-direct-to-Nano because it leverages ArduPilot's tested failsafe logic.
Sensors

Beitian BT-982K1 dual-antenna GNSS (Unicore UM982 chip) on Cube GPS2 port (SERIAL4). Configured with GPS_TYPE2=24\text{GPS\_TYPE2}=24
GPS_TYPE2=24 (UnicoreNMEA), COMPASS_ENABLE=0\text{COMPASS\_ENABLE}=0
COMPASS_ENABLE=0, EK3_SRC1_YAW=2\text{EK3\_SRC1\_YAW}=2
EK3_SRC1_YAW=2 (dual-antenna yaw, no compass needed).
REV Through Bore Encoder V1 (REV-11-1271), 2048 CPR, 8192 counts/rev in quadrature, on the steering column. Wired to Nano D2/D3 (hardware interrupts INT0/INT1).
IMU internal to the Cube.

Actuators
Steering (working)
Talon SRX motor controller driving 12V Toyota Prius steering motor. Accepts servo PWM (1000-2000 µs). No physical end-stops, software limits enforced by Nano via encoder counts.
Throttle (under test)
EZkontrol 5kW 48V BLDC controller driving a 48V BLDC traction motor. Accepts raw PWM on its throttle input (0-5V swing, ~490 Hz from Nano analogWrite). EZkontrol internally averages the PWM as analog. Reverse input is digital HIGH/LOW with the safety rule that throttle must be at zero before toggling.
Brake (not yet wired in current build)
12V solenoid, active-LOW signal, requires MOSFET driver + flyback diode.
Nano Pin Map (current)
PinFunctionNotesD2Encoder AINT0 hardware interrupt, blue wireD3Encoder BINT1 hardware interrupt, yellow wireD4Steering PWM infrom Cube MAIN-OUT 1 white, polledD5Throttle PWM infrom Cube MAIN-OUT 3 white, polledD6Steering PWM outto Talon SRXD9Throttle PWM outto EZkontrol throttle signal5VEncoder VCCred wireGNDStar pointCube blacks, Talon SRX signal GND, EZkontrol GND, encoder black
Power Domains

48V battery → EZkontrol → BLDC motor
12V rail → Talon SRX, steering motor, future brake solenoid
5V logic → Nano (via USB on bench)
All signal grounds meet at a single star point on the perf board

Steering Calibration (done)

ENCODER_MIN=−613\text{ENCODER\_MIN} = -613
ENCODER_MIN=−613 (full left)
ENCODER_MAX=770\text{ENCODER\_MAX} = 770
ENCODER_MAX=770 (full right)
Center = 78 counts (straight ahead)
Encoder is incremental, resets on every power cycle. Wheels must start straight before powering on.

Throttle Calibration (proven from old Mega build)

THROTTLE_MIN_PWM=51\text{THROTTLE\_MIN\_PWM} = 51
THROTTLE_MIN_PWM=51 (~1.0V floor, overcomes EZkontrol deadband)
THROTTLE_MAX_PWM=255\text{THROTTLE\_MAX\_PWM} = 255
THROTTLE_MAX_PWM=255 (5V max)
Output voltage: Vout≈PWM255×5VV_{out} \approx \frac{\text{PWM}}{255} \times 5V
Vout​≈255PWM​×5V

What is NOT Yet Implemented

Brake solenoid (12V, active-LOW, needs MOSFET driver)
Reverse signal (digital HIGH/LOW to EZkontrol)
PID closed-loop steering position control (intentionally skipped, Cube handles steering rate PID)
Automatic encoder homing on boot (currently manual, start with wheels straight)
Wireless E-stop (AKS-required, not yet built, must be on independent RF link with direct hardware path to EZkontrol enable line)

Safety Layers (current count)

Cube mode switch MANUAL/HOLD/AUTO (soft)
RC failsafe on the Cube (soft)
Nano failsafe on PWM loss → outputs neutral / throttle 0 (soft)
Physical kill button on kart (wiring needs verification)
Wireless E-stop (NOT BUILT, required)

The brake solenoid alone is NOT a kill switch. A real kill switch must cut motor power upstream at the EZkontrol enable line so the motor coasts and the brake fights only inertia.
Current Test Status

Open-loop steering with RC via Cube: working
Encoder counts and software end-stops: working
Throttle bench test: in progress (Nano D9 → EZkontrol direct PWM)
No brake, reverse, or wireless E-stop yet

Next Steps (in order)

Validate throttle PWM out path with EZkontrol on the bench
Full kart RC test with steering + throttle through the new Nano + Cube stack
Add brake solenoid
Add reverse signal
Wireless E-stop research and ordering
Full autonomous mission test with QGC waypoints

User Preferences
Accuracy-first, direct answers, point out mistakes. Practical step-by-step solutions. LaTeX with $ delimiters for math. Avoid em dashes. Short and simple.
