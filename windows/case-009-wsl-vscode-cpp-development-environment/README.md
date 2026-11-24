# Case 009: WSL, VS Code, and C++ Development Environment

**Investigation Date:** September 2026
 

## Overview

This case documents the setup and troubleshooting of a C++ development environment on Windows using Windows Subsystem for Linux (WSL), Visual Studio Code, and Linux development tools.

The goal was to create a development workflow combining Windows desktop applications with a Linux environment for compiling and running C++ programs.

## Environment

- Host OS: Windows
- Linux environment: WSL
- Code editor: Visual Studio Code
- Programming language: C++
- Compiler: GCC / G++
- Build tools: Linux command-line tools
- Terminal: Windows Terminal / VS Code integrated terminal

*Exact versions and configuration details were not fully documented.*

## Objectives

- Configure WSL as a Linux development environment.
- Integrate WSL with Visual Studio Code.
- Install and use a C++ compiler.
- Create, compile, and run C++ programs.
- Understand the difference between Windows and Linux development environments.
- Establish a workflow for working with source code through VS Code and WSL.

## Environment Setup

### 1. Windows Subsystem for Linux

WSL provides a Linux environment running alongside Windows without requiring a traditional virtual machine setup.

The development workflow uses Linux tools inside WSL while retaining Windows applications such as Visual Studio Code.

### 2. Visual Studio Code Integration

VS Code serves as the editor, with its development environment connected to WSL.

The intended workflow:

1. Open a Linux terminal inside WSL.
2. Navigate to the project directory.
3. Launch VS Code from the WSL environment.
4. Open the project using the WSL integration.
5. Edit source files using the VS Code interface.
6. Compile and execute programs inside Linux.

This allows the editor to operate on files and tools belonging to the Linux environment rather than treating the project as a purely Windows-based workspace.

### 3. C++ Toolchain

The Linux environment requires a compiler and basic development utilities.

Typical installation command for Debian/Ubuntu-based distributions:

```bash
sudo apt update
sudo apt install build-essential
```

The package provides common development components, including GCC/G++, `make`, and related utilities.

Compiler availability can be checked with:

```bash
g++ --version
```

## Basic C++ Development Workflow

### Create a project directory

```bash
mkdir cpp-project
cd cpp-project
```

### Create a source file

```bash
nano main.cpp
```

Example program:

```cpp
#include <iostream>

int main() {
    std::cout << "Hello from C++ in WSL!" << std::endl;
    return 0;
}
```

### Compile

```bash
g++ main.cpp -o main
```

### Execute

```bash
./main
```

Expected output:

```text
Hello from C++ in WSL!
```

## VS Code Workflow

The intended workflow combines the VS Code interface with Linux-based compilation.

Typical project structure:

```text
cpp-project/
├── main.cpp
└── main
```

The source code is edited in VS Code, while compilation and execution are performed through the WSL terminal.

This approach provides access to Linux development tools without abandoning the Windows desktop environment.

## Challenges and Investigation Areas

### Windows and Linux Environment Separation

WSL introduces separate filesystem and execution environments.

Potential issues include:

- Confusion between Windows paths and Linux paths.
- Opening projects from the wrong filesystem location.
- Using Windows tools instead of Linux tools, or vice versa.
- Differences in permissions and executable behavior.
- Environment variables and PATH differences.

### VS Code Integration

The development workflow depends on the VS Code WSL integration being connected to the correct Linux distribution.

A VS Code window connected to Windows is not necessarily equivalent to one connected to WSL.

### Compiler and Build Configuration

Installing a compiler does not automatically configure every project to build.

Additional configuration may be needed for:

- Compiler paths
- Build tasks
- Debugging
- C++ extensions
- Multi-file projects
- CMake
- Build directories

These areas represent general configuration considerations; not every issue was confirmed to have occurred during the original investigation.

## Lessons Learned

- WSL provides a practical Linux development environment on a Windows computer.
- VS Code can act as a graphical editor while Linux tools perform compilation and execution.
- The location of project files matters when working across Windows and Linux.
- A working compiler is a prerequisite for building C++ programs.
- Editing, compiling, and debugging are separate stages that require appropriate configuration.
- Understanding which environment is currently active helps avoid tool and path confusion.

## Outcome

This case represents the development environment setup and exploration of a Windows–Linux C++ workflow.

The intended environment combines:

- Windows desktop
- WSL Linux shell
- VS Code editor
- GCC/G++ toolchain

The exact final configuration, versions, and extent of successful testing were not fully recorded.

## Future Work

- Record the exact WSL distribution and version.
- Document VS Code WSL extension configuration.
- Build a multi-file C++ project.
- Configure debugging with GDB.
- Explore CMake.
- Compare filesystem performance between Windows-mounted paths and the Linux filesystem.
- Create a reproducible setup guide for a new machine.

## Notes

This case documents environment setup and development workflow exploration rather than a verified advanced C++ project.

No unconfirmed troubleshooting results or final configuration details are presented as established facts.
