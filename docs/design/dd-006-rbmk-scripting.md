# RBMK Scripting Support Design Document

|              |                                                |
|--------------|------------------------------------------------|
| Author       | [@bassosimone](https://github.com/bassosimone) |
| Last-Updated | 2024-12-01                                     |

This document describes the scripting support design
for RBMK, which builds on top of the core functionality
described in [dd-005-rbmk.md](dd-005-rbmk.md).

## Overview

RBMK scripting support enables defining measurement
algorithms as shell scripts. The `rbmk` tool provides
Unix-like commands and a POSIX shell interpreter
that work consistently across platforms, including Windows.

## Core Principles

1. Scripts are the primary way to define measurement algorithms
2. Consistent behavior across platforms
3. Self-contained minimal-shell execution environment
4. Clear working directory conventions
5. Simple data management

## Command Set

We extend RBMK with Unix-like commands to support scripting:

- `rbmk sh`: POSIX shell interpreter with `RBMK_EXE` environment variable
- `rbmk mkdir`: Create directories
- `rbmk rm`: Remove files and directories
- `rbmk tar`: Create and compress TAR archives
- `rbmk cat`: Concatenate files
- `rbmk timestamp`: Generate filesystem-friendly timestamps
- `rbmk ipuniq`: Sort and format IP addresses

These commands work consistently across operating systems and handle
platform-specific details (e.g., path separators).

## Working Directory

By default, scripts running from the `rbmk` repository should save
measurement results inside the `Workspace` directory of the
repository, which conceptually provides:

- Consistent relative paths
- Separation between the `rbmk` source code and data
- Clarity about where results are stored
- Support for multiple concurrent measurements

A future version of this design document will further specify the
organization of the `Workspace` directory.

## Script Generation

Complex measurements are defined through Python scripts
that generate POSIX shell scripts. This approach:

1. Separates measurement definition from execution
2. Makes measurement steps explicit
3. Supports different execution environments
4. Allows shell-script inspection before running

The generators should strive to emit correctly indented and
commented scripts, to facilitate inspection.

## Usage Examples

Generating measurement script:

```bash
./Workspace/scripts/check-https -i example.com > ./Workspace/measure.sh
```

Copying the script to a remote machine:

```bash
scp ./Workspace/measure.sh user@remote:
```

Running the script on Unix:

```bash
./rbmk sh measure.sh
```

Running the script on Windows:

```cmd
rbmk.exe sh measure.sh
```

The separation between generation and execution allows for more
operational agility in the measurement process. Scripts generation
can happen inside an uncensored environment, while execution has
minimal runtime requirements. When copying script files across
long-distance networks, the script files can be compressed, and
occupy much less bandwidth than the `rbmk` binary itself or
the measurement data. As a result, it will be possible to iterate
quickly on updating and re-running the measurement scripts.

## Design Choices

1. Shell Interpreter
- Use `mvdan.cc/sh` for portable, consistent behavior
- Provide `RBMK_EXE` environment variable so that a script can
invoke `rbmk` subcommands

2. Working Directory
- Clear separation of concerns
- Fixed structure

3. Script Generation
- Python for complex logic
- Shell for execution
- Clear separation of roles

4. Unix-like Commands
- Minimal but complete set
- Platform-independent behavior
- Focus on measurement needs

## Future Extensions

1. Script Templates
- Common measurement patterns
- Best practices examples
- Reusable components

2. Result Processing
- Standard analysis tools
- Report generation
- Data visualization

3. Unix Commands
- More commands as needed

## References

1. [RBMK Core Design](dd-005-rbmk.md)
2. [mvdan.cc/sh/v3 Documentation](https://pkg.go.dev/mvdan.cc/sh/v3)
