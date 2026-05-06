# Copilot instructions for SerialCommunication

## Build / Test / Lint
- Arduino sketch (primary target):
  - Build with Arduino CLI: `arduino-cli compile --fqbn <fqbn> C:\Users\tuurm\source\repos\SerialCommunication` 
    - Replace `<fqbn>` with the board FQBN for your board (e.g. `arduino:avr:uno`).
  - Upload with Arduino CLI: `arduino-cli upload -p <PORT> --fqbn <fqbn> C:\Users\tuurm\source\repos\SerialCommunication`
- Windows desktop project (SerialCommunication/):
  - Build with Visual Studio or MSBuild. For SDK-style projects try: `dotnet build SerialCommunication.slnx` or open the solution in Visual Studio.
- Tests / linters: None found in this repository. No test runner or linter configs detected.

> If you add tests, document single-test commands here (e.g., `dotnet test --filter FullyQualifiedName=...` or specific Arduino unit test tooling).

## High-level architecture (big picture)
- Two main components:
  1. Arduino firmware (root .ino + SerialCommand library):
     - SerialCommunication.ino is the Arduino sketch. It includes `SerialCommand.h/.cpp` (included in repo) and `analog.c`.
     - The sketch registers command handlers in `setup()` using `sCmd.addCommand(...)` and processes incoming serial input in `loop()` via `sCmd.readSerial()`.
     - Commands follow a tokenized pattern: handlers call `sCmd.next()` to consume arguments.
  2. Desktop/host app (SerialCommunication/ folder):
     - A Windows Forms application (C#) intended to communicate over a serial port with the Arduino device.
     - The solution and csproj files are present; open in Visual Studio to inspect UI and serial-port logic.

Data flow: host ↔ serial port ↔ Arduino SerialCommand tokenizer → command handlers.

## Key repository conventions and patterns
- Command registration: add new commands in `setup()` with `sCmd.addCommand("name", handler);` Handler functions should use `sCmd.next()` to read arguments.
- Command naming: commands use prefixes for pins: `dN` for digital (e.g., `d2`), `aN` for analog (e.g., `a0`), `pwmN` for PWM pins. See `onSet`, `onGet`, `onToggle` for examples.
- Resource strings: the sketch uses the `F("...")` macro to keep strings in flash memory—follow that pattern for large/permanent strings to reduce SRAM usage.
- SerialCommand limits: `SERIALCOMMANDBUFFER` (32) and `MAXSERIALCOMMANDS` (10) are defined in `SerialCommand.h`. Increase them only when necessary and keep in mind RAM constraints on AVR.
- Terminator and delimiters: the library defaults terminator to `\n` (set in SerialCommand constructor) and delimiter to space. Handlers expect tokenization around these settings.
- Inclusion of C source: `analog.c` is included directly into the sketch (`#include "analog.c"`) rather than compiled separately—keep this include as-is or move to a proper library if restructuring.
- Debugging: `SERIALCOMMANDDEBUG` is defined then immediately undefined in the header—debug prints are off by default. To enable library debug messages, edit `SerialCommand.h` (uncomment the `#undef SERIALCOMMANDDEBUG` or manage via build flags).

## AI assistant / other agent configs
- No CLAUDE.md, AGENTS.md, .cursorrules, .windsurfrules, CONVENTIONS.md, or other assistant rule files detected. If such files are added, copy high-priority rules into this file.

## Where to start when editing behavior
- Arduino: modify `SerialCommunication.ino` and add/remove handlers in `setup()` and implement logic in the corresponding functions (onSet, onGet...). Watch for SRAM usage.
- Host app: open `SerialCommunication.slnx` in Visual Studio to inspect serial communication code and UI.

---

If you add CI, tests, or a PlatformIO/Platform-specific project, update this file with explicit commands to build/upload/test on CI and a single-test invocation.
