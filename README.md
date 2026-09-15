\# Modular Water Treatment System

\# Automated Cam-Operated Control Valve



\## 1.0 Project Overview



This repository contains the software assets and Functional Design Specification (FDS) for a modular water treatment control valve. The system integrates with a host controller to cycle a servo-driven, cam-operated valve through distinct treatment phases (Fill, Drain, and Flush) utilizing precise timing logic and position feedback.



\*\*Target Hardware/Environment:\*\* RSLogix 500 Micro Starter Lite - free license

\*\*Programming Language:\*\* Ladder Diagram (LD)



\## 2.0 System Architecture \& Process Flow



The mechanical architecture utilizes a single servo motor rotating a cam mechanism. Valve position is monitored by four dedicated microswitches mounted along the cam's rotational path.



\*Please refer to the process diagram:\* `Process\_Diagram\_Water\_Treatment.png`



\## 3.0 Control Philosophy



The control system executes a timed, sequential batch process upon receiving a cycle call from the host system:



1\. \*\*Home Position Seeking:\*\* Upon system initialization or reset, the servo energizes continuously until the Home microswitch actuates.

2\. \*\*Cycle Initiation:\*\* A momentary signal from the host system triggers the treatment cycle, advancing the servo.

3\. \*\*Sequential Processing:\*\*

&#x20;   \* \*\*Fill Stage:\*\* Servo reaches the Fill microswitch and de-energizes for exactly 10 seconds.

&#x20;   \* \*\*Drain Stage:\*\* Servo advances to the Drain microswitch and de-energizes for exactly 20 seconds.

&#x20;   \* \*\*Flush Stage:\*\* Servo advances to the Flush microswitch and de-energizes for exactly 10 seconds.

4\. \*\*Cycle Completion:\*\* Following the flush stage, the servo returns to the Home position and waits for the next cycle call.

5\. \*\*State Tracking (Bonus Implementation):\*\* A dedicated integer file (`N7:0`) actively reports the current valve position to the host system (0 = HOME, 1 = FILL, 2 = DRAIN, 3 = FLUSH, 4 = TRAVELLING).

6\. \*\*System Interrupt (Bonus Implementation):\*\* A manual reset/interrupt bit (`B3:0/0`) allows operators to abort an active process, bypassing remaining timers and forcing the servo to immediately seek the Home position.



\## 4.0 I/O Allocation Schedule



| Tag / Address | I/O Type | Device Description | Field State Definition |

| :--- | :--- | :--- | :--- |

| `I:0/0` | Digital Input  | ZS-01: Home Microswitch | Closed = Valve at Home |

| `I:0/1` | Digital Input  | ZS-02: Fill Microswitch | Closed = Valve at Fill |

| `I:0/2` | Digital Input  | ZS-03: Drain Microswitch | Closed = Valve at Drain |

| `I:0/3` | Digital Input  | ZS-04: Flush Microswitch | Closed = Valve at Flush |

| `I:0/4` | Digital Input  | HS-01: Host Cycle Call | Momentary Close = Start Cycle |

| `O:0/0` | Digital Output | M-01: Cam Servo Motor | 1 = Energized / Rotating |

| `N7:0`  | Integer File   | System State Register | 0=Home, 1=Fill, 2=Drain, 3=Flush, 4=Travel |

| `B3:0/0`| Internal Bit   | System Reset/Interrupt | Toggle = Abort Cycle to Home |



\## 5.0 Simulation \& Verification Protocol (Dry Run Test)



The logic was validated via RSLogix Emulate software. The system successfully passed the following Dry Run Test simulation states:



| Test State | Condition / Input Action | Expected Output | Status |

| :--- | :--- | :--- | :---: |

| \*\*1. Startup / Homing\*\* | Initial power-up. <br> All inputs (0). | Servo `O:0/0` \*\*ENERGIZES\*\* <br> State `N7:0` = \*\*4\*\* | ✅ PASS |

| \*\*2. False Position\*\* | Force Fill switch ON. <br> `I:0/1` (1) | Servo `O:0/0` \*\*REMAINS ENERGIZED\*\* (seeking home) <br> State `N7:0` = \*\*1\*\* | ✅ PASS |

| \*\*3. Home Reached\*\* | Force Fill OFF, Home ON. <br> `I:0/1` (0), `I:0/0` (1) | Servo `O:0/0` \*\*DE-ENERGIZES\*\* <br> State `N7:0` = \*\*0\*\* | ✅ PASS |

| \*\*4. Cycle Call\*\* | Pulse Host Call. <br> `I:0/4` (1 -> 0) | Servo `O:0/0` \*\*ENERGIZES\*\* <br> State `N7:0` = \*\*0\*\* | ✅ PASS |

| \*\*5. Fill Sequence\*\* | Force Home OFF, Fill ON. <br> `I:0/0` (0), `I:0/1` (1) | `N7:0` transitions \*\*4 -> 1\*\*. <br> Servo \*\*DE-ENERGIZES\*\* for 10s, then auto-starts. | ✅ PASS |

| \*\*6. Drain Sequence\*\* | Force Fill OFF, Drain ON. <br> `I:0/1` (0), `I:0/2` (1) | `N7:0` transitions \*\*4 -> 2\*\*. <br> Servo \*\*DE-ENERGIZES\*\* for 20s, then auto-starts. | ✅ PASS |

| \*\*7. Flush Sequence\*\* | Force Drain OFF, Flush ON. <br> `I:0/2` (0), `I:0/3` (1) | `N7:0` transitions \*\*4 -> 3\*\*. <br> Servo \*\*DE-ENERGIZES\*\* for 10s, then auto-starts. | ✅ PASS |

| \*\*8. Cycle Complete\*\* | Force Flush OFF, Home ON. <br> `I:0/3` (0), `I:0/0` (1) | `N7:0` transitions \*\*4 -> 0\*\*. <br> Servo \*\*DE-ENERGIZES\*\* and holds. | ✅ PASS |

| \*\*9. Interrupt Test\*\* | Pulse Cycle Call, Toggle `B3:0/0`, Force Home OFF, Fill ON. | Servo \*\*REMAINS ENERGIZED\*\* (bypasses fill timer to seek home). <br> State `N7:0` = \*\*1\*\* | ✅ PASS |

| \*\*10. Abort Complete\*\* | Force Fill OFF, Home ON. <br> `I:0/1` (0), `I:0/0` (1) | `N7:0` transitions \*\*4 -> 0\*\*. <br> Servo \*\*DE-ENERGIZES\*\* and holds. | ✅ PASS |



\## 6.0 Software Assets



\* The raw ladder logic project file (`.RSS`) can be found in the `/src/` directory.

\* A complete PDF export of the ladder logic program is available in the `/docs/` directory.



\## 7.0 Acknowledgements \& My Learning Journey



This project is a reflection of my ongoing, highly structured learning journey in PLC programming. The online courses I undertook made it incredibly easy to grasp the fundamentals and challenged me to apply critical thinking—specifically the 80/20 rule. By truly mastering just 20% of the core instruction sets, we can effectively execute 80% of real-world automation tasks. 



Special thanks to the course instructor for the exercise materials:

\* \*\*Course Detail:\*\* Applied Logic (via Udemy) by Paul Lynn

\* \*\*Project Concept:\*\* The base process flow, diagrams, and core test criteria were provided as part of his excellent course materials.

