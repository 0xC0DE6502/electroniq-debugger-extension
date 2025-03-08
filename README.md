# Sourcelevel 6502 asm debugging for Electroniq

This VSCode extension enables sourcelevel 6502 asm debugging for [Electroniq](https://github.com/0xC0DE6502/electroniq), the Acorn Electron emulator. 

## Work in progress

This debugger extension is a work in progress and has only been tested with VSCode on Windows. Below is a quickstart to get you going. This is by no means a full user guide.

## Quickstart

1. Prerequisites: Windows, Visual Studio Code, [max65](https://github.com/0xC0DE6502/max65-releases) assembler (make sure `max65.exe` is in your `PATH`).

1. Install VSCode extension ["max65 Assembly Language"](https://marketplace.visualstudio.com/items?itemName=0xC0DE.max65-assembly-language).

1. Install VSCode extension ["Electroniq Debugger Extension"](https://marketplace.visualstudio.com/items?itemName=0xC0DE.electroniq-debugger-extension).

1. Check that the 6502 asm source files in your Acorn Electron project folder are correctly associated with the 'max65' language in VSCode (this also enables syntax highlighting).

1. Create/adapt `.vscode/tasks.json` and `.vscode/launch.json` in your project folder. See [tasks.json and launch.json](#tasksjson-and-launchjson) for details.

1. Set breakpoints in any 6502 asm source files in your project folder.

1. Press `F5` to first make and then debug your project in VSCode + Electroniq. Electroniq appears in a VSCode WebView panel or in your default browser, depending on the debug configuration selected.

1. Do usual debug stuff: step over (`F10`), step into (`F11`), set/clear breakpoints, run/pause/stop, hover over labels to see info, add watches, inspect CPU registers (variables section), etc.

## tasks.json and launch.json

Create or adapt `.vscode/tasks.json` in your project folder to make sure there is a task that assembles your project with `max65` and creates the final disk image. Example:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Build 'project'",
      "type": "shell",
      "command": "cmd.exe",
      "args": [
        "/c",
        "${workspaceFolder}/make.bat"
      ],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "problemMatcher": []
    }
  ]
}
```

The task above calls `make.bat` to assemble your project with `max65` (which also generates the required sourcemap) and then builds the disk image with [beebasm](https://github.com/stardot/beebasm). Of course you can use any other way you like to build the disk image. Example `make.bat`:

```batch
rem Assemble project and generate sourcemap using max65
max65.exe project.6502 -l listing.txt -s map.json -v
if %ERRORLEVEL% neq 0 pause & exit

rem Build disk image (using beebasm in this case)
beebasm.exe -i makedisk.6502 -do project.ssd -opt 3 -v -title PROJECT
if %ERRORLEVEL% neq 0 pause & exit
```

If you are using `beebasm` to create the disk image, adapt `makedisk.6502` to your project. In this example it contains:

```
puttext "!BOOT.txt", "!BOOT", 0, 0
putfile "PROG", "PROG", &2000, &2000
```

Create or adapt `.vscode/launch.json` in your project folder to have one or more debug configurations for your project. Example:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug 'project' (webview)",
      "type": "electroniq-debugger",
      "request": "launch",
      "image": "${workspaceFolder}/project.ssd",
      "map": "${workspaceFolder}/map.json",
      "electroniq": {
        "target": "https://0xc0de6502.github.io/electroniq",
        "args": "?dfs",
        "webview": true,
        "scale": "0.45"
      },
      "preLaunchTask": "Build 'project'"
    },
    {
      "name": "Debug 'project' (browser)",
      "type": "electroniq-debugger",
      "request": "launch",
      "image": "${workspaceFolder}/project.ssd",
      "map": "${workspaceFolder}/map.json",
      "electroniq": {
        "target": "https://0xc0de6502.github.io/electroniq",
        "args": "?dfs"
      },
      "preLaunchTask": "Build 'project'"
    }
  ]
}
```

## GPLv3 license

GNU GENERAL PUBLIC LICENSE  
Version 3, 29 June 2007  

Copyright (C) 2025 0xC0DE (0xC0DE6502@gmail.com)

This program is free software: you can redistribute it and/or modify  
it under the terms of the GNU General Public License as published by  
the Free Software Foundation, either version 3 of the License, or  
(at your option) any later version.  

This program is distributed in the hope that it will be useful,  
but WITHOUT ANY WARRANTY; without even the implied warranty of  
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the  
GNU General Public License for more details.  

You should have received a copy of the GNU General Public License  
along with this program. If not, see <https://www.gnu.org/licenses/>.
