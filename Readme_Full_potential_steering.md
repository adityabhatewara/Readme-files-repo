#  Full Auto Steering for 4-Wheel Rover (ROS2)

## Overview

This ROS Python node (`drive_arc`) enables **manual and autonomous steering and drive control** for a 4-wheeled rover. It supports joystick input, encoder feedback, and autonomous commands to compute steering angles and drive PWM signals.

---

## Features

-  Manual and Autonomous control switching  
-  Runtime mode switching via joystick  
-  4 types of driving/steering logic  
-  Customizable joystick mappings  
-  Encoder feedback-based closed-loop steering  
-  Velocity and omega smoothing using `queue.Queue`  

---

## Node: `drive_arc`

### ✅ Publishers

| Topic        | Type               | Description                              |
|--------------|--------------------|------------------------------------------|
| `/motor_pwm` | `Int32MultiArray`  | [4x drive PWM, 4x steer PWM]             |
| `/state`     | `std_msgs/Bool`    | 0 → Manual, 1 → Autonomous               |

### 📥 Subscribers

| Topic        | Type                   | Used for                          |
|--------------|------------------------|-----------------------------------|
| `/joy`       | `sensor_msgs/Joy`      | Joystick input                    |
| `/enc_auto`  | `Float32MultiArray`    | Encoder readings                  |
| `/motion`    | `custom/WheelRpm`      | Autonomous velocity/omega         |
| `/rot`       | `std_msgs/Int8`        | Rotate-in-place trigger           |

---

## 🕹️ Joystick Mapping

| Action                        | Button/Axis            | Description                                  |
|------------------------------|------------------------|----------------------------------------------|
| Mode up/down                 | `RB` / `LB`            | Increase/decrease drive mode                |
| Toggle autonomous            | `Start/Menu` button    | Switch between manual and autonomous         |
| Toggle steering lock         | Axis trigger           | Sets `steer_islocked`                        |
| Toggle full potential        | Axis trigger           | Sets `full_potential_islocked`               |
| Forward/Backward drive       | Left stick Y-axis      | Drive direction                              |
| Lateral movement             | Left stick X-axis      | Strafing                                     |
| Curve steering               | Axis 3                 | Adds angular motion                          |
| Same-direction PWM steering  | Right stick Y-axis     | Applies same PWM to all steering motors      |
| Opp-direction PWM steering   | Right stick X-axis     | Steers wheels in opposite PWM directions     |
| Individual wheel PWM axes    | Custom axes            | Controls each wheel's angle (4 axes)         |

---

## 🚗 Drive + Steering Modes

| Mode | `steer_islocked` | `full_potential_islocked` | Description                              |
|------|------------------|---------------------------|------------------------------------------|
| 1    |  True          |  True                   | Locked steering (forward, perp, rotate)  |
| 2    |  False         |  True                   | Unlocked steering using joystick axes    |
| 3    |  True          |  False                  | Full potential per-wheel PWM steering    |
| 4    |  False         |  False                  | Invalid / fallback                       |

---

## 🔁 Core Logic

### `joyCallback`
- Reads button and axis values  
- Controls mode switches and internal toggles (`state`, `steer_islocked`, etc.)

### `encoderCallback`
- Collects encoder values from subscribed topic

### `motionCallback`
- Updates `velocity` and `omega` from autonomous source

### `rotCallback`
- Triggers rotate-in-place action

---

## 🔧 Steer Function

```python
steer(initial_angles, final_angles, mode)
