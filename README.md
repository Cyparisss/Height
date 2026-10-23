# Project README & Technical Documentation

---

## 1. Project Overview & Deliverables

This repository contains the complete Unreal Engine project and source files for a playable prototype sandbox. The prototype features interactive physics manipulation (grabbing and launching objects), dynamic camera zoom controls, and an environment-aware traversal system featuring Ledge/Bar Grabbing and Edge Jumping (Ledge Jump).

The player starts inside an enclosed box arena and must climb out to reach the finish area by choosing between two distinct parkour routes of varying difficulty.

### Summary of Deliverables
- **Unreal Project**: Complete, functional project containing all source files required for evaluation.
- **Playable Prototype**: Testable sandbox map featuring the box escape environment and all implemented character and physics mechanics.
- **Technical Documentation**: Comprehensive overview of project architecture, interaction design, level mechanics, and technical choices.
- **AI Statement of Intent**: Detailed disclosure of artificial intelligence tools utilized, scope of usage, benefits, and limitations encountered.

---

## 2. Technical Documentation

### 2.1 Overall Project Architecture

The architecture relies primarily on a centralized actor model centered around **`BP_ThirdPersonCharacter`**. This main Blueprint integrates player input, camera management components, physics handles, and trace-based environmental detection routines for ledge grabbing and jumping.

```
+-----------------------------------------------------------------------+
|                        BP_ThirdPersonCharacter                        |
+-----------------------------------------------------------------------+
|  - SpringArmComponent    --> Controls dynamic camera zoom bounds      |
|  - PhysicsHandleComp     --> Manages physics object grabbing/holding  |
|  - Ledge Tracing System  --> Handles wall/ledge detection & alignment |
|  - Ledge Jump System     --> Manages Anim Notify detach & launch Z     |
|  - Wall Clearance Check  --> Validates geometry height for feet alignment|
+-----------------------------------------------------------------------+
```

---

### 2.2 Main Classes, Blueprints & Components

- **`BP_ThirdPersonCharacter` (Blueprint Class)**  
  The core player actor responsible for handling locomotion, mapping input actions, managing character states (e.g., ground movement, falling, hanging/ledge grab), raycast routines, and jumping off ledges.
- **`SpringArmComponent` (Component)**  
  Attached to the character mesh to maintain camera offset and smoothing. Dynamically adjusts its target arm length based on player scroll input.
- **`PhysicsHandleComponent` (Component)**  
  Attached to the character root to manipulate 3D physics-enabled objects (e.g., interactive cubes) within the world.

---

### 2.3 Interaction & Traversal Systems

#### A. Camera Zoom System
- **Input**: Mouse Wheel Up / Mouse Wheel Down.
- **Logic**:
  * Modifies the `TargetArmLength` property of the `SpringArmComponent`.
  * Decrements distance on **Mouse Wheel Up** and increments distance on **Mouse Wheel Down** in steps of **25 units**.
  * Math is clamped using a `Clamp (Float)` node between **300 and 500 units**, ensuring the player's view remains within optimal bounds.

#### B. Ledge & Bar Grabbing Mechanic
- **Execution Flow**:
  1. **State Condition**: Evaluates whether the character is actively in a **Falling** movement state.
  2. **Wall Detection**: Fires a forward line trace from the character vector to detect nearby vertical wall or bar geometry.
  3. **Ledge Detection**: Upon hitting a wall, a secondary downward trace locates the top horizontal edge.
  4. **Wall Clearance Check**: Verifies that the wall below the ledge provides sufficient vertical clearance for the character's feet-landing animation. If clearance is insufficient, the grab is rejected and the character continues falling.
  5. **State Freeze & Alignment**: Halts all linear movement immediately (`Stop Movement Immediately`), transitions the character into a looping ledge-grab animation, snaps local offset, and rotates the actor perpendicular to the surface.

#### C. Edge Jump Mechanic (Ledge Jump)
- **Execution Flow**:
  1. **Trigger**: Input key pressed while in the **Ledge Grab** state.
  2. **Animation Montage**: Plays the `JumpFromLedge` montage.
  3. **Anim Notify (`Jump`)**: Fires a notification at the precise frame of physical push-off.
  4. **Detach & Launch**: Executes a `Detach` node to release the character from the ledge and applies a vertical impulse via `Launch Character` ($Z = 850.0$ with `ZOverride` enabled for consistent jump height).
  5. **Airborne Re-grab**: Allows the character to detect and grab subsequent higher or adjacent ledges/bars during descent.

---

### 2.4 Level Design & Pathing System

#### Box Arena Escape & Dual Difficulty Routes
The prototype map places the character inside a structural box arena. To reach the goal at the top and escape, the player can choose between two distinct parkour paths:

1. **Path 1 (Easy / Standard Route)**:
   - Features closely spaced ledges and bars.
   - Does not require high-precision Edge Jumps.
   - Provides straight vertical progression with clear foothold clearance.
2. **Path 2 (Hard / Advanced Route)**:
   - Features wider vertical and horizontal gaps between ledges.
   - Requires chaining **Edge Jumps** ($Z = 850$ impulse) and camera rotation mid-air to reach higher, offset ledges.
   - Includes narrow geometries that test the Wall Clearance Check.

---

### 2.5 Physics Systems Developed

#### Object Grabbing & Launching
- **Detection & Capture**:
  * Triggering the **Grab** input casts a **300-unit line trace** forward from the camera's perspective.
  * If the trace hits a physics-simulated object, the `PhysicsHandleComponent` grabs the target component at the impact location.
- **Holding Behavior**:
  * On `Event Tick`, the target location of the held object is recalculated continuously to hover **300 units** in front of the camera forward vector.
- **Release & Impulse Launching**:
  * Releasing the input detaches the object while maintaining its linear and angular velocity.
  * Rapid camera movement prior to release transfers momentum to the object, allowing players to launch interactive cubes across the sandbox.

---

### 2.6 Main Technical Choices

- **Centralized Character Blueprint**: Chosen for simplified component communication during rapid prototyping of camera, movement, traversal, and physics interactions.
- **Anim Notify-Driven Detach & Launch**: Synchronizes character physics release and upward impulse ($Z = 850.0$) with the visual animation frame for maximum responsiveness and polish.
- **Wall Clearance Verification**: Prevents clipping bugs and unrealistic hanging poses by validating vertical surface depth beneath detected ledges.
- **Dual-Path Box Environment**: Demonstrates mechanical versatility and player agency within a controlled, enclosed sandbox.

---

## 3. AI Statement of Intent

### 3.1 Overview
During the development of this project, **Gemini AI** was utilized as an assistive technology for troubleshooting logic, optimizing mathematical formulas, and brainstorming gameplay mechanics.

### 3.2 Breakdown of AI Usage

| Category | Details |
| :--- | :--- |
| **Tools Used** | Gemini AI (Google) |
| **Where AI Was Used** | Blueprint error diagnosing, trace vector math optimization, mechanic brainstorming (Edge Jump, Wall Clearance), and concept referencing. |
| **Why AI Was Used** | To streamline bug fixing and explore standard implementation patterns for character-environment interactions. |
| **How AI Was Used** | Provided targeted prompts regarding line-trace setups, spring arm clamping logic, Anim Notify handling, and velocity retention. Suggestions were manually translated and tested inside Unreal Engine. |
| **Benefits Obtained** | Accelerated debugging cycles, clearer insight into Unreal trace logic, and inspiration for environmental traversal mechanics. |
| **Limitations Encountered** | AI guidance occasionally referenced node names or configurations incompatible with specific Unreal Engine context, requiring manual revision and logic adaptation. |

### 3.3 Verification & Compliance
All Blueprint logic, math nodes, and physics implementations assisted by AI were **reviewed, integrated, fully understood, tested, and verified** directly within the Unreal Engine project environment.
