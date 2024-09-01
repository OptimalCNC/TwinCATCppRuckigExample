# Description
This repository present a simple example of using the modified [ruckig](https://github.com/huweiATgithub/ruckig) to generate motion in TwinCAT3 C++ project.

**Note**: We are using the C++ 17 standard. Please set it in the Visual Studio project settings.

# Steps to reproduce the project

## The `ruckig` library
We will compile the `ruckig` library as a static library and use it in the TwinCAT3 C++ project. For this, we will follow the steps below:

1. Clone the above mentioned modified `ruckig` repository as a submodule.
2. Create a new TwinCAT3 C++ static library project ruckig.
3. Add a filter "ruckig source" in the ruckig project and add the following source files as "Existing Item" in the filter:
    ```
    src/ruckig/brake.cpp
    src/ruckig/position_first_step1.cpp
    src/ruckig/position_first_step2.cpp
    src/ruckig/position_second_step1.cpp
    src/ruckig/position_second_step2.cpp
    src/ruckig/position_third_step1.cpp
    src/ruckig/position_third_step2.cpp
    src/ruckig/velocity_second_step1.cpp
    src/ruckig/velocity_second_step2.cpp
    src/ruckig/velocity_third_step1.cpp
    src/ruckig/velocity_third_step2.cpp
    ```
4. Since precompiled headers are used in the TwinCAT3 C++ project, we need to add the current project directory to the Additional Include Directories in the ruckig project settings so that the precompiled header file can be found.
5. As the original ruckig library put all its header files in `include/ruckig` directory and use `#include <ruckig/*.hpp>` to include them, we need to add the `include` directory to the Additional Include Directories in the ruckig project settings.
6. Build the project, make sure no Tc Sign is applied. You will see a static library `ruckig.lib` is generated.

There are still many template header files. You may instantiate them with your specific use case or defer the instantiation later to the precompiled header file in the TwinCAT3 C++ project.

## The Example Project

### Module `Example`
We present a simple example of using the above `ruckig.lib` in the TwinCAT3 versioned C++ project. For this, we will follow the steps below:

1. Create a new TwinCAT3 versioned C++ project `Example` and add a C++ Module.
2. Configure TwinCAT signing for the project.
3. Add the include path of submodule `ruckig` to the Additional Include Directories in the Example project settings.
4. Add the `ruckig.lib` file to the Additional Dependencies in the Example project settings.
5. Compose codes.
6. **Create Task and untick the "Floating point exception" in the Task settings.**
7. Set the Example Module's context to the created Task.
8. Activate: you may change the "Inputs.ExampleId" and see the result in "Outputs.MotionOutput" in the TwinCAT3 XAE. Currently, we have migrated three examples from the original ruckig library:
    - `ExampleId = 1`: A position example.
    - `ExampleId = 5`: A velocity example.


### Module `Interpolator1D`
We also present a simple example of using the `PositionInterpolator` and `VelocityInterpolator` (see [interpolator.hh](./TcRukig/ruckig/extern/ruckig/include/interpolator1d/interpolator.hh)). 

1. Create another C++ Module `Interpolator1D` in the Example project.
2. We put configurations like  `max_velocity`, `max_acceleration`, `max_jerk` in the Parameter section. You may set them in the Parameter(init) section of Object Setting.
    **Please be aware of the consistency of units of the parameters.**
3. We initialize the interpolators with parameters in the transition `PREOP` to `SAFEOP`. Note that we also create a "Context-based" parameter `CycleTime` to set the cycle time of the interpolators.
4. In the `CycleUpdate`, we use the interpolators to generate motion. The generated State is also placed in the `Outputs`.


For demo purposes, we have the Inputs:
- `InitialPosition`: The initial position for the PositionInterpolator.
- `InitialVelocity`: The initial velocity for the VelocityInterpolator.
The targets are all zero for the PositionInterpolator and zero velocity and acceleration for the VelocityInterpolator.
