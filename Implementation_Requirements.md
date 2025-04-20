# Pimoroni Trackball PIM447 Implementation Requirements

## Overview

This document outlines the requirements for implementing the Pimoroni PIM447 trackball driver in our ZMK configuration. The implementation is based on work done by user cdc-mkb in the following pull request:

- [Pimoroni PIM447 trackball support (#961)](https://github.com/zmkfirmware/zmk/compare/main...cdc-mkb:zmk:mouse-pim447)

As this PR is several years old and ZMK has evolved significantly since then, we need to carefully adapt the approach to be compatible with current ZMK architecture.

## Pull Request Analysis

### Files Created/Modified in PR

1. **Driver Source Files**:
   - `app/drivers/sensor/trackball_pim447/trackball_pim447.c`
   - `app/drivers/sensor/trackball_pim447/CMakeLists.txt`
   - `app/drivers/sensor/trackball_pim447/Kconfig`
   
2. **Device Tree Bindings**:
   - `app/dts/bindings/sensor/pimoroni,trackball_pim447.yaml`
   - `app/dts/bindings/zmk,trackball-mode.yaml`
   
3. **Header Files**:
   - `app/include/dt-bindings/zmk/trackball_pim447.h`
   
4. **Behaviors**:
   - `app/src/behaviors/behavior_trackball_mode.c`
   
5. **Core ZMK Changes**:
   - Additions to `app/drivers/CMakeLists.txt`
   - Additions to `app/drivers/sensor/CMakeLists.txt`
   - Additions to `app/drivers/sensor/Kconfig`
   - Modifications to `app/Kconfig` for mouse/trackball configuration

6. **Keymap Example**:
   - Example implementation in a user configuration

### Key Components

1. **Driver Implementation**: 
   - I²C communication with the Pimoroni trackball
   - Sensor data processing
   - Trackball LED control

2. **Behavior Implementation**:
   - Mode switching (movement vs. scrolling)
   - Configuration options for sensitivity
   
3. **Device Tree Integration**:
   - Binding definitions
   - Device properties

4. **Kconfig Options**:
   - `CONFIG_ZMK_TRACKBALL_PIM447` - Main driver enable option
   - Related mouse/pointing device options

## Implementation Strategy

Given that the original PR is older and ZMK has evolved, we'll take the following approach:

1. **Create a Custom Module**:
   - Instead of directly modifying ZMK, we'll implement a custom module in our config repository
   - This allows integration without forking the main ZMK project

2. **Directory Structure**:
   ```
   config/modules/
       pim447/
           zephyr/module.yml        # Define the module
           CMakeLists.txt           # Build instructions
           Kconfig                  # Configuration options
           dts/bindings/           # Device tree bindings
           include/                # Header files
           src/                    # Driver source code
   ```

3. **Integration Points**:
   - Update `west.yml` to include our custom module
   - Create device tree overlays referencing the driver
   - Update keymaps to utilize the trackball behaviors

## Implementation Steps (Updated)

### User Stories

#### US-01: Module Setup
**As a** keyboard developer,  
**I want to** set up the basic module structure following ZMK conventions,  
**So that** the driver can be correctly recognized by ZMK/Zephyr build system.

**Intent:**  
To define a comprehensive blueprint for the module directory structure and files needed to implement the Pimoroni trackball driver. This structure must follow ZMK conventions for modules and incorporate all necessary code components from the original pull request while adapting to current ZMK architecture.

**Acceptance Criteria:**
- The proposed structure complies with ZMK module conventions and integration methods
- All necessary file paths and file types are identified and documented
- The structure includes proper locations for driver source, DT bindings, behaviors, and configurations
- The structure incorporates all components from PR #961 with appropriate adaptations
- The structure is validated against current ZMK module requirements
- The directory structure is documented in a way that makes implementation straightforward
- No actual implementation is done in this story, only the blueprint is created

**Details:**
Based on the analysis of the pull request and ZMK documentation, the module structure should include the following components:

```
config/modules/
    zmk-driver-pim447/                          # Main module directory
        zephyr/
            module.yml                          # Module definition with build settings
        CMakeLists.txt                          # Main build file
        Kconfig                                 # Configuration options
        
        dts/                                    # Device tree files
            bindings/
                sensor/
                    pimoroni,trackball_pim447.yaml  # Device tree binding for sensor
                zmk/
                    behaviors/
                        zmk,behavior-trackball-mode.yaml  # DT binding for behavior
        
        include/                                # Public headers
            dt-bindings/
                zmk/
                    trackball_pim447.h          # Constants and defines
        
        src/                                    # Source code
            drivers/
                sensor/
                    trackball_pim447/
                        trackball_pim447.c      # Driver implementation
                        CMakeLists.txt          # Driver build configuration
                        Kconfig                 # Driver configuration options
            
            behaviors/
                behavior_trackball_mode.c       # Behavior implementation
        
        README.md                               # Documentation
```

This structure incorporates all components from the original PR while following ZMK's current module conventions and organizing the files in a way that maintains proper separation of concerns between driver code, behavior code, and configuration.

#### US-02: Device Tree Binding Definition
**As a** keyboard developer,  
**I want to** define proper device tree bindings for the Pimoroni trackball,  
**So that** it can be correctly configured in the device tree.

**Intent:**  
To create comprehensive device tree binding definitions for both the Pimoroni trackball sensor and the trackball mode behavior, ensuring they can be properly referenced and configured in device tree overlays while following ZMK's current device tree binding conventions.

**Acceptance Criteria:**
- Device tree binding yaml files are properly structured according to ZMK/Zephyr conventions
- The binding defines all necessary properties for configuring the Pimoroni trackball
- The binding includes appropriate documentation for each property
- The binding is compatible with ZMK's current pointing device architecture
- The binding for the trackball mode behavior is properly defined
- The binding includes standard and required properties for Zephyr sensor devices

**Tasks:**
1. [ ] Create the sensor binding file (`zmk-driver-pim447/dts/bindings/sensor/pimoroni,trackball_pim447.yaml`)
   - Define compatible string property
   - Define I²C address property
   - Define sensitivity property
   - Define LED color properties
   - Define optional axis inversion properties
   - Include documentation for all properties

2. [ ] Create the behavior binding file (`zmk-driver-pim447/dts/bindings/zmk/behaviors/zmk,behavior-trackball-mode.yaml`)
   - Define compatible string property
   - Define binding cells property
   - Define mode selection properties (move/scroll)
   - Include documentation for all properties

3. [ ] Review existing ZMK sensor bindings for reference structure
   - Identify required properties for Zephyr sensor devices
   - Follow ZMK binding naming conventions
   - Ensure compatibility with current ZMK DTS architecture

4. [ ] Define property constraints
   - Set appropriate data types (int, boolean, etc.)
   - Set valid ranges for numeric properties
   - Identify which properties are required vs. optional

5. [ ] Document how the DTS bindings map to driver functionality
   - Clear connection between binding properties and driver behavior
   - Document default values for optional properties

**Details:**
Based on the PR analysis, the device tree bindings should define the following:

**Trackball Sensor Binding** (`pimoroni,trackball_pim447.yaml`):
```yaml
description: Pimoroni Trackball (PIM447) I2C sensor
compatible: "pimoroni,trackball_pim447"

include: i2c-device.yaml

properties:
  reg:
    required: true
    type: int
    description: I2C device address (0x0A)
  
  sensitivity:
    type: int
    default: 64
    description: Trackball sensitivity (0-255)

  led-red:
    type: int
    default: 0
    description: Red component of LED color (0-255)

  led-green:
    type: int
    default: 0
    description: Green component of LED color (0-255)

  led-blue:
    type: int
    default: 0
    description: Blue component of LED color (0-255)

  invert-x:
    type: boolean
    description: Invert X-axis direction
  
  invert-y:
    type: boolean
    description: Invert Y-axis direction
```

**Trackball Mode Behavior Binding** (`zmk,behavior-trackball-mode.yaml`):
```yaml
description: Trackball Mode Behavior
compatible: "zmk,behavior-trackball-mode"

include: behavior.yaml

properties:
  "#binding-cells":
    type: int
    const: 1
    required: true
    description: Number of binding parameters

  default-mode:
    type: int
    default: 0
    description: Default trackball mode (0=move, 1=scroll)
```

These bindings will need to be compatible with ZMK's current pointing device architecture including support for input listeners and input processors.

#### US-03: Driver Implementation
**As a** keyboard developer,  
**I want to** implement the Pimoroni trackball driver as a Zephyr sensor,  
**So that** it provides movement data to the ZMK pointing system.

**Intent:**  
To create a functional I²C driver for the Pimoroni PIM447 trackball that follows Zephyr sensor driver conventions and interfaces correctly with ZMK's pointing device system. The implementation will be based directly on the code from PR #961, adapted as needed for current ZMK architecture.

**Acceptance Criteria:**
- Driver properly initializes and communicates with the PIM447 trackball
- Movement data is accurately captured along X and Y axes
- Trackball button press is detected
- RGB LED control is functional
- Driver integrates with ZMK's sensor system
- Proper error handling for communication issues
- Driver is configurable via the properties defined in the device tree binding

**Tasks:**
1. [ ] Create driver source file (`zmk-driver-pim447/src/drivers/sensor/trackball_pim447/trackball_pim447.c`)
   - Implement driver based on PR #961 code
   - Update includes to match current Zephyr paths (logging, etc.)
   - Implement I²C communication with proper error handling

2. [ ] Create driver build files
   - `zmk-driver-pim447/src/drivers/sensor/trackball_pim447/CMakeLists.txt`
   - `zmk-driver-pim447/src/drivers/sensor/trackball_pim447/Kconfig`

3. [ ] Implement register interaction functions
   - Read functions for movement and button data
   - Write functions for LED control
   - Functions to reset accumulated data

4. [ ] Implement Zephyr sensor interface functions
   - `sample_fetch()` - Read current data from the trackball
   - `channel_get()` - Return requested data channel (X, Y, button)

5. [ ] Configure proper initialization sequence
   - I²C device binding
   - Initial LED configuration
   - Error handling for initialization failures

6. [ ] Update any deprecated API calls from the PR code
   - Replace `device_pm_control_nop` with `NULL` as per Zephyr updates
   - Update any other deprecated function calls identified during implementation

**Details:**
The driver implementation will be based directly on the code from PR #961, with necessary adaptations for current ZMK. Key components from the PR include:

1. **Register Definitions**:
```c
#define TRACKBALL_PIM447_REG_LEFT 0x04
#define TRACKBALL_PIM447_REG_RIGHT 0x05
#define TRACKBALL_PIM447_REG_UP 0x06
#define TRACKBALL_PIM447_REG_DOWN 0x07
#define TRACKBALL_PIM447_REG_SWITCH 0x08
```

2. **Data Structure**:
```c
struct trackball_pim447_data {
    const struct device *i2c_dev;
    int32_t dx;
    int32_t dy;
    int32_t dz;
};
```

3. **Core Functions to Implement**:
- `trackball_pim447_read_reg` - Read from trackball registers
- `trackball_pim447_write_reg` - Write to trackball registers
- `trackball_pim447_read_axis` - Process directional movement data
- `trackball_pim447_sample_fetch` - Get latest data from the device
- `trackball_pim447_channel_get` - Get specific data channel
- `trackball_pim447_init` - Initialize the device

4. **Device Definition**:
```c
DEVICE_DT_INST_DEFINE(0, &trackball_pim447_init, NULL, 
                     &trackball_pim447_data, NULL, POST_KERNEL,
                     CONFIG_SENSOR_INIT_PRIORITY, &trackball_pim447_api);
```

The driver will need to be updated to use current ZMK practices, including proper error handling, modern API calls, and integration with the current pointing device architecture.

#### US-04: Trackball Mode Behavior
**As a** keyboard user,  
**I want to** be able to switch the trackball between movement and scrolling modes,  
**So that** I can use the trackball for different functions without changing layers.

**Intent:**  
To implement a behavior that allows users to toggle the trackball between cursor movement and scrolling modes, giving the trackball dual functionality without requiring layer changes.

**Acceptance Criteria:**
- Users can toggle between movement and scrolling modes with a keypress
- The current mode is tracked and maintained correctly
- The behavior properly integrates with the trackball driver
- Mode toggling is responsive and reliable
- The behavior is configurable in device tree and keymap files

**Tasks:**
1. [ ] Create trackball mode constants header file (`zmk-driver-pim447/include/dt-bindings/zmk/trackball_pim447.h`)
   - Define mode constants (MOVE, SCROLL)
   - Define any other public enums or constants needed

2. [ ] Create behavior source file (`zmk-driver-pim447/src/behaviors/behavior_trackball_mode.c`)
   - Implement behavior based on PR #961 code
   - Create mode tracking system
   - Implement proper ZMK behavior hooks
   - Handle mode transition logic

3. [ ] Implement integration with trackball driver
   - Ensure driver can respond to mode changes
   - Implement method for behavior to communicate with driver

4. [ ] Extend ZMK's behavior registration system
   - Ensure behavior is properly registered
   - Handle initialization sequence

5. [ ] Implement user feedback mechanism (optional)
   - Add LED color change to indicate current mode
   - Or other visual feedback if available

**Details:**
The behavior implementation will enable users to toggle the trackball's functionality between:
1. **MOVE mode**: Standard mouse cursor movement (X/Y axes control the mouse pointer)
2. **SCROLL mode**: Scrolling functionality (X/Y axes control horizontal and vertical scrolling)

The behavior will be triggered by a keypress defined in the user's keymap, allowing for easy mode switching without needing to change layers. The behavior should be stateful, remembering the current mode across multiple uses.

Sample keymap usage might look like:
```
// In keymap file
&pim447_mode MOVE_TOGGLE  // Toggle between move and scroll modes
```

This behavior is one of the key usability features that make the trackball more versatile, as it allows a single compact input device to handle both pointer movement and document scrolling.

#### US-05: Input Listener Implementation
**As a** keyboard developer,  
**I want to** implement proper input listeners for the trackball,  
**So that** the movement data is correctly processed and sent to the host.

**Intent:**  
To configure the appropriate input listeners that connect the Pimoroni trackball driver with ZMK's input system, ensuring movement data is properly processed, transformed if necessary, and sent to the host computer.

**Acceptance Criteria:**
- Input listener is properly configured in device tree overlays
- Movement data from the trackball is correctly captured by the input system
- Input processing is efficient and responsive
- Integration with mode behavior (movement vs. scrolling)
- Optional transforms (axis inversion, sensitivity) can be applied through input processors

**Tasks:**
1. [ ] Create device tree overlay templates for trackball input listeners
   - Define listener node structure
   - Configure proper device references

2. [ ] Implement integration with ZMK's input system
   - Connect driver's sensor data to input events
   - Ensure proper event types are generated

3. [ ] Add support for input processors (if needed)
   - Implement axis inversion through input processors
   - Implement sensitivity scaling through input processors

4. [ ] Integrate with trackball mode behavior
   - Ensure different actions based on current mode (scroll vs. move)
   - Handle mode transitions properly

**Details:**
The input listener is a critical component that connects the raw hardware driver with ZMK's input system. In ZMK's current architecture, every input device needs an associated listener to process events from the device.

For our trackball implementation, we would create an input listener in the device tree overlay like this:

```dts
/ {
    trackball_listener {
        compatible = "zmk,input-listener";
        device = <&trackball_pim447>;
        
        /* Optional transforms */
        input-processors = <&transform_processor>;
    };
};
```

This listener connects to our trackball driver and ensures that movement data is properly captured and processed by ZMK's input system. The listener can also include optional input processors for transformations like axis inversion or sensitivity scaling.

The implementation must also account for different operating modes (move vs. scroll) by either:
1. Having mode-specific listeners that are enabled/disabled
2. Having a single listener that routes events differently based on the current mode

This component is essential for the trackball to function correctly within ZMK's pointing device framework.

#### US-06: Split Keyboard Integration
**As a** split keyboard user,  
**I want to** have the trackball on the peripheral half communicate with the central half,  
**So that** trackball data is properly sent to the host computer.

**Intent:**  
To implement the necessary configuration to allow trackball data from the peripheral (right) half to be properly transmitted to the central (left) half and then to the host computer, ensuring smooth operation in a split keyboard setup.

**Acceptance Criteria:**
- Trackball data is successfully transmitted between keyboard halves
- No data loss or significant latency in pointer movement
- Proper configuration on both central and peripheral halves
- Coordination between halves is maintained even during heavy use
- BLE connection between halves is stable for pointer data

**Tasks:**
1. [ ] Implement input split device on peripheral half (right side)
   - Configure input split device in device tree overlay
   - Ensure device properly captures trackball data

2. [ ] Implement input split device on central half (left side)
   - Configure input split device in device tree overlay
   - Ensure device properly receives trackball data

3. [ ] Configure data transmission between halves
   - Ensure proper event encoding/decoding
   - Handle potential packet loss or transmission errors

4. [ ] Test and optimize data transmission
   - Ensure low latency for pointer movements
   - Optimize data rate if needed

**Details:**
ZMK's current pointing device architecture requires specific configuration for split keyboards. When a pointing device is on a split peripheral (like the right half of a Corne), we need to ensure the data is properly transmitted to the central half.

The implementation requires adding input split device nodes to both halves:

**Right Half Overlay**:
```dts
/ {
    /* Input listener for the trackball */
    trackball_listener {
        compatible = "zmk,input-listener";
        device = <&trackball_pim447>;
    };
    
    /* Input split device for the right half */
    right_split_device {
        compatible = "zmk,input-split-device";
    };
};
```

**Left Half Overlay**:
```dts
/ {
    /* Input split device for the left half */
    left_split_device {
        compatible = "zmk,input-split-device";
    };
};
```

The input split devices allow ZMK to transmit pointing events between halves. ZMK will handle most of the complexity of the transmission, but we need to ensure the device tree is configured correctly.

#### US-07: Keymap Integration
**As a** keyboard user,  
**I want to** configure the trackball behaviors in my keymap,  
**So that** I can easily use and control the trackball functions.

**Intent:**  
To provide clear examples and templates for integrating the Pimoroni trackball behaviors into user keymaps, enabling customization of the trackball functions and ensuring users can control both the trackball mode and other related settings.

**Acceptance Criteria:**
- Clear keymap integration examples are provided
- Mode switching behavior is properly configurable in keymap
- Documentation for all available behaviors and settings
- Keymap integration works with standard ZMK keymap structure
- User-friendly naming and parameters for behaviors

**Tasks:**
1. [ ] Create keymap integration templates
   - Example configuration for the trackball in keymap file
   - Example device tree overlay for the trackball

2. [ ] Document available behaviors
   - Mode switching behavior (`&pim447_mode`)
   - Any other custom behaviors

3. [ ] Create layer examples
   - Example mouse layer configuration
   - Example scroll layer configuration (if needed)

4. [ ] Define configuration options
   - Document any configurable parameters
   - Explain valid values for each parameter

**Details:**
The keymap integration will allow users to incorporate the trackball functionality into their keyboard layout. At minimum, users will need to add:

1. A behavior binding to toggle between movement and scrolling modes:
```
// In keymap file
&trackball_mode MOVE_TOGGLE  // Toggle between move and scroll modes
```

2. Proper device tree configuration in their overlay file:
```dts
&i2c0 {
    status = "okay";
    
    trackball_pim447: trackball@0A {
        compatible = "pimoroni,trackball_pim447";
        reg = <0x0A>;
        sensitivity = <64>;
        led-red = <255>;   // Customize LED color
        led-green = <0>;
        led-blue = <0>;
    };
};

/ {
    trackball_listener {
        compatible = "zmk,input-listener";
        device = <&trackball_pim447>;
    };
    
    // For split keyboards
    right_split_device {
        compatible = "zmk,input-split-device";
    };
};
```

The keymap documentation should also include typical usage patterns, such as creating a dedicated mouse layer with additional buttons for mouse clicks, and configuring mode switching to easily alternate between cursor movement and scrolling.

#### US-08: Documentation and Testing
**As a** keyboard developer,  
**I want to** have clear documentation and test procedures,  
**So that** I can verify the implementation is working correctly and maintainable.

**Intent:**  
To create comprehensive documentation and testing procedures that ensure the Pimoroni trackball driver is working correctly and can be maintained over time as ZMK evolves.

**Acceptance Criteria:**
- Clear README documentation for the module
- Hardware connection guidelines
- Configuration options documentation
- Troubleshooting guide
- Test procedures to verify functionality
- Example configurations for common use cases
- Version compatibility information

**Tasks:**
1. [ ] Create README documentation
   - Module overview and purpose
   - Installation instructions
   - Configuration options
   - Hardware connection diagrams/instructions

2. [ ] Develop testing procedures
   - Basic functionality tests (movement, button clicks)
   - Mode switching tests
   - Split keyboard tests
   - LED control tests

3. [ ] Create troubleshooting guide
   - Common issues and solutions
   - Diagnostic procedures
   - Hardware verification steps

4. [ ] Document compatibility information
   - ZMK version compatibility
   - Hardware compatibility
   - Limitations or known issues

5. [ ] Create example configurations
   - Standard configuration example
   - Advanced configuration example
   - Split keyboard configuration example

**Details:**
Good documentation is crucial for both initial implementation and long-term maintenance. The documentation should cover:

1. **Hardware Configuration**:
   - Required I²C connections on the Pro Micro or other controller
   - Minimum power requirements
   - Wiring diagram for trackball connection

2. **Software Configuration**:
   - Module installation in west.yml
   - Required device tree configuration
   - Keymap configuration examples
   - Available configuration options

3. **Testing**:
   - Step-by-step verification procedures
   - Expected behavior descriptions
   - Debugging tools and logs

4. **Maintenance**:
   - Version tracking information
   - Update procedure for new ZMK versions
   - Contribution guidelines

This documentation will ensure that both the initial implementation and future maintenance are well-supported, reducing the risk of compatibility issues or feature regression as ZMK evolves.

## Concerns and Considerations (Updated)

Based on current ZMK documentation research, here are the refined concerns and their mitigation strategies:

1. **API Compatibility**:
   - **Concern**: The PR uses older ZMK APIs that may have changed
   - **Status**: Valid concern - ZMK's pointing device API has evolved significantly
   - **Mitigation**:
     - Use the current `zmk,input-listener` approach instead of older device registration methods
     - Utilize the modern input processor system (`input-processors` in DT) for transformations
     - Implement the driver as a standard Zephyr sensor driver, integrating with current ZMK pointing device architecture
     - For split keyboards, implement `zmk,input-split-device` nodes on both halves

2. **Building and Testing**:
   - **Concern**: Build system integration needs careful testing
   - **Status**: Valid, but addressable with proper module structure
   - **Mitigation**:
     - Follow the established module creation pattern documented in ZMK
     - Ensure proper `zephyr/module.yml` configuration with correct `dts_root` setting
     - Use a specific directory structure that aligns with Zephyr module expectations
     - For any `CMakeLists.txt`, follow similar patterns from existing ZMK driver modules

3. **Hardware Requirements**:
   - **Concern**: Ensure proper I²C wiring and configuration
   - **Status**: Valid and hardware-dependent
   - **Mitigation**:
     - Document required connections in a README
     - Implement clear device tree configurations that specify pin settings
     - Add configuration options for common hardware configurations (e.g., Pro Micro pins)
     - Implement fault detection in driver if supported by hardware

4. **Future Updates**:
   - **Concern**: Maintainability as ZMK continues to evolve
   - **Status**: Valid ongoing concern
   - **Mitigation**:
     - Implement as a standalone module separate from core ZMK
     - Follow current best practices for module organization
     - Document key integration points and dependencies clearly
     - Use clear versioning for the module to track compatibility with ZMK releases

## Module Structure (Updated)

Based on ZMK's current module documentation, here's the optimized structure for our implementation:

```
config/modules/
    zmk-driver-pim447/                 # Follow ZMK naming conventions
        zephyr/module.yml              # Module definition with dts_root setting
        CMakeLists.txt                 # Build system integration
        Kconfig                        # Configuration options
        dts/
            bindings/
                sensor/
                    pimoroni,trackball_pim447.yaml  # DT binding
                zmk/
                    behaviors/
                        zmk,behavior-trackball-mode.yaml  # Behavior binding
        include/
            dt-bindings/zmk/trackball_pim447.h  # Public header
        src/
            drivers/
                sensor/
                    trackball_pim447.c          # Driver implementation
            behaviors/
                behavior_trackball_mode.c       # Behavior implementation
        README.md                      # Documentation
```

## Resources

- [Original PR](https://github.com/zmkfirmware/zmk/compare/main...cdc-mkb:zmk:mouse-pim447)
- [ZMK Module Creation Guide](https://zmk.dev/docs/development/module-creation)
- [ZMK Pointing Device Documentation](https://zmk.dev/docs/development/hardware-integration/pointing)