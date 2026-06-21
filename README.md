# Wall.E - Autonomous Wall-Following Robot 

## Project Overview
Wall.E is an autonomous robotic vehicle engineered to navigate a complex physical track—featuring narrow maze-like corridors, sharp turns, and an elevated ramp—without any human intervention. Developed as part of the **ETE3007 Fundamentals of Robotics** course at Dokuz Eylül University, the robot dynamically reacts to its environment using real-time ultrasonic sensor feedback and a pre-mapped deterministic behavioral logic.

## Core Navigation Strategy
The system utilizes an **Adaptive Hybrid Navigation (Timed + Sensor-Guided)** approach. Early testing showed color sensors to be unreliable under mixed competition lighting, so the architecture was pivoted to a more robust, deterministic system:
* **Pre-mapped timed movements** calibrated specifically for the physical track layout.
* **Dynamic distance checkpoints** via a scanning ultrasonic sensor to trigger state transitions (turns, stops) at physical walls, rather than relying solely on open-loop delays.

## Hardware Architecture
The robot is built on a custom dual-tier wooden chassis featuring a modified Ackermann steering mechanism. 

| Component | Specification / Role |
| :--- | :--- |
| **MCU** | Arduino Uno R3 (16 MHz, ATmega328P) |
| **Distance Sensor** | HC-SR04 Ultrasonic (mounted on a panning servo) |
| **Motor Drivers** | 2× L298N Dual H-Bridges (splits the thermal load) |
| **Actuators** | 4× DC Gear Motors (1:48) for high-torque ramp climbing |
| **Servos** | 1x Steering (Ackermann), 1x Ultrasonic Pan (0°–180° scan) |
| **Power Supply** | 2× 18650 Li-ion cells (7.4V Nominal Series) |
| **Telemetry Display** | 16×2 I2C LCD for real-time status and distance readings |

## Software Architecture & State Machine
The robot operates using a Sequential Finite State Machine (FSM) written in Arduino C++. 

### Phase 1: Start Position Validation (`setup()`)
To ensure deterministic behavior, the robot performs a bilateral safety scan on power-up. The panning servo checks left (180°) and right (0°) lateral clearances. The FSM prevents the main loop from starting unless both readings are between 40–70 cm, ensuring the robot is perfectly aligned on the track.

### Phase 2: Main Navigation Loop (`loop()`)
Continuous forward drive interleaved with precise steering commands and sensor-triggered checkpoints:
* **`waitUntilCloserThan(int targetCm)`**: Implements a crucial debounce algorithm. It polls the HC-SR04 and requires two consecutive readings below the threshold to trigger a turn, effectively eliminating false positives from ultrasonic noise spikes.
* **`measureDistance()`**: Low-level hardware control sending 10µs trigger pulses and calculating precise echo durations.

## Pin Configuration & Wiring
* **Motor A (Left):** `enA = 6`, `in1 = 8`, `in2 = 7`
* **Motor B (Right):** `enB = 3`, `in3 = 5`, `in4 = 4`
* **Steering Servo:** Pin `10`
* **Ultrasonic Pan Servo:** Pin `A0`
* **HC-SR04 Sensor:** `Trig = 12`, `Echo = 2`
* **LCD Display:** I2C (`SDA` / `SCL`)

## Future Roadmap
* **Encoder-Based Odometry:** Replacing timed delays with wheel encoders for distance-accurate navigation that remains independent of battery voltage drops.
* **IMU Integration:** Adding an MPU-6050 for gyroscopic feedback to correct steering drift over long distances.
* **ESP32 Migration:** Upgrading the MCU to an ESP32 to enable wireless telemetry, real-time remote monitoring, and advanced IoT integrations.
* **PID Controller Implementation:** Adding side-facing ultrasonic sensors for true closed-loop lateral PID wall-following.

## Team Contributions (Group 8)
* **Olcay Karahasan:** Low-level sensor code (`measureDistance()`), HC-SR04 trigger/echo timing, I2C LCD telemetry, ultrasonic pan servo logic, and start-position validation algorithms.
* **Mohammad Sadra Najafi:** Chassis 3D design, power management routing, and L298N circuit integration.
* **Feryal Çetin:** Hardware assembly, motor mounting, structural integrity, and center-of-mass optimization for ramp stability.
* **Alp Kaya:** High-level navigation logic (`waitUntilCloserThan()`), FSM sequencing, steering angle calibration, and PWM motor control.
* **Rabia Bozkurt:** Project management, performance metrics tracking, test organization, and documentation.
