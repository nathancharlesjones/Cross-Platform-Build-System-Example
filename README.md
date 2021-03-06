# Example Project using Cross-Platform Build System

## What is it?

A C project that demonstrates use of a [Cross-Platform Build System](https://github.com/nathancharlesjones/Cross-Platform-Build-System) I put together.

## How do I use it?

1) Clone this repo; use `git clone --recurse-submodules https://github.com/nathancharlesjones/Cross-Platform-Build-System-Example` to ensure that you also get the `Cross-Platform-Build-System` subfolder.
2) Follow steps 2 and 3 [here](https://github.com/nathancharlesjones/Cross-Platform-Build-System#how-do-i-use-it) to set up the required dependencies.
3) Run `./Cross-Platform-Build-System/make.py build_ninja` to create the build file.
4) Run `./Cross-Platform-Build-System/make.py build_target` to build the project's three targets.
5) Run `./build/x86/debug/blinky_x86_debug.exe` (or `./Cross-Platform-Build-System/make.py run -c './build/x86/debug/blinky_x86_debug.exe'`) to see the program running on your host machine.
6) Run `./Cross-Platform-Build-System/make.py debug -t <TARGET>` to start a debugging session. Replace `<TARGET>` with either `blinky_x86_debug`, `blinky_x86_release`, or `blinky_STM32F1_debug`.
	- Connect to gdbgui by opening a browser on the host machine and navigating to `localhost:5000`.
	- If you are debugging the STM32 target and want to run it on real hardware, connect an STM32F1 MCU to your host machine with a debug adapter (like a JLink or an OpenOCD-compatible debug adapter) and start a gdb server. Connect gdbgui/gdb to the debug adapter using one of the two gdb commands below, where `PORT` is the port number for the gdb server you just started:
		- Windows/Mac: `target extended-remote host.docker.internal:PORT`
		- Linux: `target extended-remote 172.17.0.1:PORT`

## How does it work?

[Cross-Platform Build System](https://github.com/nathancharlesjones/Cross-Platform-Build-System) is a project I put together after being dissatisfied with my other build systems. I felt that they were overcomplicated and nothing I found was able to give me a truly cross-platform building experience. I put together three components to create something that I felt was more straightforward and truly cross-platform:

1) The Python library `ninja_syntax`, which allows us to create a `build.ninja` file that defines how we want our project and each target to be built. Because it's Python, we can use standard Python syntax and data structures like dictionaries to easily define each target.
2) Docker, which provides us with a consistent, cross-platform way to build our project. Additionally, using [gdbgui](https://www.gdbgui.com/) we can debug our project from inside Docker (even if it's running on an MCU that's attached to the host machine with a USB debug adapter) without needing to recompile when moving between host OSes. (Of coure, if you're more comfortable using command-line gdb or the slightly-more-user-friendly [TUI mode](https://undo.io/resources/gdb-watchpoint/5-ways-reduce-debugging-hours/), you can edit `make.py` so that the `debug` action starts plain `gdb` instead of `gdbgui`.)
3) A Python CLI, which provides easy aliases for commonly needed commands.

See the [Cross-Platform Build System](https://github.com/nathancharlesjones/Cross-Platform-Build-System) repository for more information.

This project also demonstrates one way of writing a cross-platform project. It does this using a technique called ["link-time polymorphism"](https://github.com/nathancharlesjones/Comparison-of-OOP-techniques-in-C/tree/main/1c_Link-time-Polymorphism_ADT), in which a single header file (`hardware/include/hardwareAPI.h`) is given more than one implementation (`hardware/source/x86/x86.c` for the host machine and `hardware/source/STM32F1/source/STM32F1.c` for the STM32). To avoid multiple definition errors, only one of those source files is linked in with the final executable (notice the different set of source files listed in `project_targets.py` for each of our two targets), which is fine because when we compile the program we only _want_ a single definition for the hardware functions.

Notice how, in `source/main.c`, we need to implement a `delay_sec` function. This function is defined in `hardware/hardwareAPI.h` (and it would have a nice description if I was a better programmer), but we don't know, yet, how it's implemented. In `hardware/source/x86/x86.c`, `delay_sec` is implemented by using `struct timespec` and related functions from `time.h`, a Linux system header. And in `hardware/source/STM32F1/source/STM32F1.c` (a modified version of the `main.c` that was generated for this project from STM32CubeMX), `delay_sec` is implemented using the `HAL_Delay` function that the STM32 HAL provides us. Each one provides the needed functionality when used with the appropriate piece of hardware.