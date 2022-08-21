---
title: "Visual Studio Code setup for C/C++ development in Docker on Windows"
header:
  teaser: /assets/images/posts/vscode-docker.png
toc: true
categories:
  - Programming
  - IDE Setup
tags:
  - c++
  - vscode
  - docker
  - cmake
  - windows
  - linux
  - wsl
  - clangd
---

![VS Code / Docker](/assets/images/posts/vscode-docker.png){: .align-center .img-large}

C/C++ development environment setup for local/remote development in Visual Studio Code, using Docker/WSL-2 on Windows.

<!-- more -->

## 1. Introduction

This is my personal setup that I use for my private home projects (although I'm also using/have used some of this setup professionally at work).
It is mainly for my future reference, but hopefully can be helpful for others as well.

As I use Windows on my primary laptop, it will be mostly focused on that particular operating system.
However some parts (the general VS Code setup etc.) would work on Linux and MacOS as well (all of these also have Docker available, although the installation details differ).

The reasons why I'm choosing this particular setup:

***Why Visual Studio Code***:
- free to use (including commercially)
- available for all major OS (Windows, Linux, macOS)
- large base of extensions
- very fast for remote development (presumably thanks to the remote VS Code server) - in my experience best responsiveness even over slow connections (like VPN), compared to other full-featured IDEs (like e.g. Eclipse/RSE)
- excellent Docker integration

***Why Docker***:
- encapsulating all the development environment, dependencies etc.
- therefore can be set up / recovered quickly even after full system reinstall
- the whole development environment setup can be versioned
- the builds and environment are reproducible
- there were some licensing changes, but at the time of writing the article still free for home and small business use

All this goes into the direction to not have to set up a lot of stuff manually every time the development environment needs to be re-created (e.g. after nuking the whole OS for whatever reason).

***Disadvantages of this setup***:
- not very suitable for Windows (macOS) native development - predominately using all Linux under the hood

## 2. Prerequisites

The article will not focus on installing the prerequisites in too much detail, as it is fairly straightforward, described in a lot of other articles and would make the article very long.
The main purpose is the development environment setup itself.

You can see for example [Setting-up a local development environment using VS Code, Docker, and WSL 2](https://dev.to/jnous5/setting-up-a-local-development-environment-using-vs-code-docker-and-wsl-2-5c0i) for more details of the prerequisites setup under Windows.

### 2.1. Windows Subsystem for Linux (WSL)

- only necessary for Windows (Linux and macOS don't need this step)
- in a nutshell:

```powershell
# use admin command line or PowerShell console
C:\Windows\system32> wsl --install
```

Refer to the following links for further details:
- ***recent Windows 10***: [Install Linux on Windows with WSL](https://docs.microsoft.com/en-us/windows/wsl/install)
- ***older Windows versions***: [Manual installation steps for older versions of WSL](https://docs.microsoft.com/en-us/windows/wsl/install-manual)

***Resource limits** (optional)*:

- to make sure the WSL will not consume all the computer resources
- set the limits in the "_.wslconfig_" file in the user profile directory:

```ini
[wsl2]
processors=6
memory=6GB
```

My usual setup:
- CPU: 75% of virtual cores (example: 8 total => 6)
- Memory: 75% of the total physical memory  
  (8GB total => 6GB, 16GB total => 12GB)

See [Configuration setting for .wslconfig](https://docs.microsoft.com/en-us/windows/wsl/wsl-config#configuration-setting-for-wslconfig) for further details and examples.

### 2.2. Docker

- the installation packages: [Docker Desktop Download](https://www.docker.com/products/docker-desktop/)
- older Docker versions: [Docker Desktop Release Notes](https://docs.docker.com/desktop/release-notes/)

Under Windows, make sure to use the "***WSL 2 based engine***" (should be the default).

{% capture notice_contents %}
Note that Docker for Windows can also be used without the WSL by utilizing the Hyper-V hypervisor.

However the WSL usage is preferred - enabling the Hyper-V might create some issues and incompatibilities with other virtualization solutions (e.g. VMware).
{% endcapture %}

{% include notice level="warning" %}

### 2.3. Visual Studio Code

- the installation packages: [Download Visual Studio Code](https://code.visualstudio.com/download)

For Windows:
- usually using the "System Installer" package (64-bit)
- in case of not having the admin privileges on the machine can still use the "User Installer" (to install to your personal user profile) or the ZIP package to use from anywhere  
  (but in such case you'd probably not be able to install the Docker)

## 3. Development environment setup

{% capture notice_contents %}
**<a name="src-location">Source code location considerations</a>**:

For Windows/WSL, the important consideration is the source code location.
There are 2 primary options:
- storing the source code on the Windows host filesystem and mounting into Docker
- storing the source code on a WSL/Docker volume directly

The second option is generally being recommended for better performance.
However I prefer to use the first option for simpler access from the host Windows (using Windows native tools like GitExtensions etc.) and also because of having the source code on an encrypted volume.

But note that if you'd like to use the WSL native filesystem, the files can still be accessed from the Windows host by using the "\\\\wsl$\\" internal share.

In my experience the performance difference of the source code location is not very noticeable, but still strongly recommend to put the output files like the compiled objects and linked executables onto a Docker volume.
{% endcapture %}

{% include notice level="warning" %}

{% capture notice_contents %}

**<a name="all-setup">GitHub repository available</a>**:

The entire sample setup (including an example CMake C++ project) is available here:  
[GitHub/EmDroid: Sample VScode Docker C++](https://github.com/EmDroid/sample-vscode-docker-cpp)
{% endcapture %}

{% include notice level="info" %}

### 3.1. VS Code Extensions

Launch the VS Code and install the following extensions:
- **Remote - Containers**: required, adding support for running into a Docker container
- **Remote - WSL**: needed in case the VS Code will be started from inside WSL and the source code stored there

Note that you don't need to install the other extensions (like C/C++, CMake etc.) at this point, as those only need to be installed into the Docker instance.

### 3.2. Opening the project

Launch the VS Code:
- if using Windows host filesystem, launch directly from host
- if using WSL natively, launch from the WSL console

Once started, use "File/Open Folder" to open either an existing project, or a new folder to start a new project.

### 3.3. Launching under Docker

The information how to launch the project under the Docker is provided in the "_.devcontainer_" subdirectory of the project.

The setup can be done directly in the "_.devcontainer/devcontainer.json_" file, but I prefer to use docker compose for that (more flexibility, and can also be used directly from the command line then).

Create the following files in the project (can be done directly in the VS Code):

**a) The "_.devcontainer/devcontainer.json_" file**:

- this is the VS Code setup file for running under the Docker container

```json
// See https://aka.ms/vscode-remote/containers for the
// documentation about the devcontainer.json format

{
    "name": "My Project Name",
    "dockerComposeFile": "docker-compose.yml",
    "service": "dev",
    "workspaceFolder": "/workspace",

    "settings": {
        "terminal.integrated.shell.linux": "/bin/bash"
    },

    "extensions": [
        "ms-vscode.cpptools"
    ]
}
```

{% capture notice_contents %}
**<a name="extension-id-tip">Tip: The extension ID</a>**:

If you want to find out the extension identifier of your favorite extension, you can find it on the extension description page:
- search the extension in the Extensions sidebar
- click on the extension to open the info page
- locate the "Identifier" description on the right side panel of the page
{% endcapture %}

{% include notice level="info" %}

**b) The "_.devcontainer/Dockerfile_" file**:

- the file describing the Docker image

```Dockerfile
FROM debian:bullseye
LABEL Description="Build environment"

RUN apt-get -qq update \
    && apt-get -y --no-install-recommends install \
        build-essential \
        ca-certificates \
        gdb \
        git \
        less \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*
```

- you can choose whatever Linux distribution and version you like
- but note there might be differences if you select a different distro  
  (e.g. rhel/centos using "yum" instead of "apt" etc.)
- feel free to add any other tools you might need (like e.g. vim), but at the same point keep in mind that the image should ideally be kept as small as possible

**c) The "_.devcontainer/docker-compose.yml_" file**:

- the file describing the whole Docker setup (including the mount points etc.)

```yaml
version: '3'

volumes:
  bldvol:

services:

  dev:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ..:/workspace   # bind mount the host project directory
      - bldvol:/build   # store build artifacts on a docker volume
    # keep open (do not auto-close)
    tty: true
    # alternatively ("tty" not working in some cases):
    # command: sleep infinity
```

- there is a separate build volume to store the build artifacts
- the "tty" or sleep command is needed to keep the Docker container open  
  (otherwise it would close immediately after the startup and fail the VS Code startup)

**d) Load the new configuration**:

- restart the VS Code
- the VS Code will ask to re-open the project in Docker ("Reopen in container")
- it will then build your Docker image and start the project from inside it
- the project directory will be mounted as "_/workspace_"

### 3.4. CMake support

There are various build systems, I prefer to use the CMake for my personal projects (and even for professional work project, if I can help it), as it is the de-facto standard solution across the board and has the most support and resources available.

**a) Adding the CMake extension**:

- the extension needs to be added into the "_.devcontainer/devcontainer.json_" file:

```json
    ...
    "extensions": [
        ...
        "ms-vscode.cmake-tools",
        ...
    ]
```

**b) Updating the docker image**:

- the cmake needs to be added to the "_.devcontainer/Dockerfile_" file to make it available in the image

- I also prefer to use the "Ninja" make (is usually faster than the "default" GNU make):

```Dockerfile
...
RUN apt-get -qq update \
    && apt-get -y --no-install-recommends install \
        ...
        cmake \
        ...
        ninja-build \
        ...
...
```

**c) The VS Code CMake settings**:

- update or create the "_.vscode/settings.json_" file:

```json
{
    ...
    "cmake.buildDirectory": "/build/${buildType}",
    "cmake.generator": "Ninja",
    ...
}
```

**d) Applying the new configuration**:

- reload the window (Ctrl+Shift+P, "Reload Window")
- will ask to rebuild the image
- when complete, CMake will ask for the build kit
- should only have one that can be selected  
  (if you'd also install some other toolchain like "clang", you might have more build kits available)
- then press on "Build" on the bottom bar to configure and build the project
- the output files will be generated in the "_/build/${buildType}_" directory  
  (for example: "_/build/Debug_")
- when using multiple kits, can also use "_/build/${buildKit}/${buildType}_" to separate the output directories for the different kits
- can also launch or debug the project executable from the bottom bar
- when tests are configured, can then run the tests from there as well

### 3.5. Setting up C/C++ tools

We already installed the Microsoft C++ extension ("ms-vscode.cpptools") which can be used for code assist (Intellisense) etc., but here we'll setup the Clangd extension that has some advanced features, including code formatting and static analysis tools (clang-tidy).

Note that Clangd works best together with CMake (in particular, the CMake compile database is needed) - if not using CMake, it might be better to stay with the Microsoft C++ extension.

**a) Adding the Clangd extension**:

- the extension needs to be added into the "_.devcontainer/devcontainer.json_" file:

```json
    ...
    "extensions": [
        ...
        "llvm-vs-code-extensions.vscode-clangd",
        ...
    ]
```

**b) Updating the docker image**:

- the Clangd should to be added to the "_.devcontainer/Dockerfile_" file to make it available in the image:

```Dockerfile
...
RUN apt-get -qq update \
    && apt-get -y --no-install-recommends install \
        ...
        clangd \
        ...
...
```

- if not added here, the Clangd extension would ask to install it's own private version

**c) The VS Code CMake settings**:

- update the "_.vscode/settings.json_" file:

```json
{
    ...
    "clangd.arguments": [
        "-background-index",
        "-compile-commands-dir=/build/Debug",
        "--completion-style=detailed",
        "--header-insertion=never"
    ],
    // disabling the standard C++ extension Intellisense
    // (will use Clangd instead)
    "C_Cpp.intelliSenseEngine": "Disabled",

    // these can also be moved inside the [cpp] section
    // to only apply to the C/C++ files
    "editor.formatOnSave": true,
    "editor.formatOnSaveMode": "modifications",
    "editor.insertSpaces": true,
    "editor.tabSize": 4,
    "editor.rulers": [80,120],

    "[cpp]": {
        "editor.defaultFormatter": "llvm-vs-code-extensions.vscode-clangd",
    },
    ...
}
```

***Compile commands directory***>
- the CMake compile commands path needs to be set up properly for the code assist to work
- that is the directory containing the "_compile_commands.json_" file generated during the CMake build
- in the above example it is set to "_/build/Debug_" (the Debug build path)
- if using multiple build kits (e.g. both GCC and Clang), you might need to add the build kit folder into the path (depending on the CMake build setup)

**d) Setting up the clang-format and clang-tidy** (optional):

- setting up the clang-format and clang-tidy is recommended especially for new projects

- will start working in VS Code Clangd as soon as you add the "_.clang-format_" and "_.clang-tidy_" into your project root folder

- "_.clang-format_" example:

```yaml
---
# Using defaults from the Mozilla style
BasedOnStyle: Mozilla
---
# C++ settings different from the default style
Language:       Cpp

Standard:       Cpp03

ColumnLimit:    0
IndentWidth:    4
ConstructorInitializerIndentWidth: 4
ContinuationIndentWidth: 8
AccessModifierOffset:   -4

BreakBeforeBraces:  Custom
BraceWrapping:
  AfterClass:       true
  AfterControlStatement: true
  AfterCaseLabel:   true
  AfterEnum:        true
  AfterFunction:    true
  AfterNamespace:   false
  AfterStruct:      true
  AfterUnion:       true
  BeforeCatch:      true
  BeforeElse:       true
  IndentBraces:     false
  SplitEmptyFunction: false
  SplitEmptyRecord: false
  SplitEmptyNamespace: true

BinPackArguments:   true
BinPackParameters:  true

SortIncludes:   false
ReflowComments: false
FixNamespaceComments: true

MaxEmptyLinesToKeep:  2
SpacesBeforeTrailingComments: 2
SpaceAfterCStyleCast: true

AlignConsecutiveAssignments:  true
AlignEscapedNewlines: Left
---
```

- "_.clang-tidy_" example:

```yaml
---
Checks: >
  -*,
  bugprone-*,
  clang-diagnostic-*,
  clang-analyzer-*,
  cppcoreguidelines-*,
  google-*,
  hicpp-*,
  modernize-*,
  performance-*,
  portability-*,
  readability-*,
  -modernize-use-trailing-return-type,

WarningsAsErrors: >
  modernize-*,
  cppcoreguidelines-*,
  boost-*,
  google-build-using-namespace,
  readability-else-after-return,
  google-readability-todo,

FormatStyle: 'file'

CheckOptions:
  - key: bugprone-argument-comment.StrictMode
    value: 1
  - key: cppcoreguidelines-non-private-member-variables-in-classes.IgnoreClassesWithAllMemberVariablesBeingPublic
    value: 1
  - key: cppcoreguidelines-special-member-functions.AllowSoleDefaultDtor
    value: 1
  - key: cppcoreguidelines-macro-usage.CheckCapsOnly
    value: 1
  - key: hicpp-special-member-functions.AllowSoleDefaultDtor
    value: 1
  - key: readability-identifier-length.IgnoredParameterNames
    value: '^[n]|ex$'
...
```

- note that there are known issues with some macro-based libraries (like for example GoogleTest) that could trigger quite a lot of clang-tidy warnings and errors; in such cases you can add a "`// NOLINT`" comment after the line to ignore

### 3.6. Optional: Using ccache

The "ccache" is a tool that allows to cache the compiler artifacts (mostly object files), so that they do not need to be rebuild if the exact same configuration is used.
This can be beneficial when e.g switching between multiple source code branches frequently.

The setup I'm usually using is the following:
- setting up the CMake for using ccache
- using a separate volume for storing the ccache files

**a) Updating the docker image**:

- the ccache needs to be added to the "_.devcontainer/Dockerfile_" file to make it available in the image

```Dockerfile
...
RUN apt-get -qq update \
    && apt-get -y --no-install-recommends install \
        ...
        ccache \
        ...
...
```

**b) Adding the ccache volume**:

- update the "_.devcontainer/docker-compose.yml_" file:

```yaml
version: '3'

volumes:
  ...
  cchvol:

services:

  dev:
    ...
    volumes:
      ...
      - cchvol:/ccache  # ccache docker volume
    environment:
      - CCACHE_DIR=/ccache
    ...
```

- this will add the separate volume and use it for the ccache output files (compiled objects)

**c) The VS Code CMake settings**:

- update the "_.vscode/settings.json_" file:

```json
{
    ...
    "cmake.configureArgs": [
        "-DCMAKE_CXX_COMPILER_LAUNCHER=ccache",
        "-DCMAKE_C_COMPILER_LAUNCHER=ccache"
    ],
    ...
}
```

- this will instruct the CMake to use the ccache when compiling the source code

### 3.7. Optional: User volume persistence

Additional separate volume can be used to persist the user settings (for example bash command history etc.).
It will then also persist any other settings you might need - this includes the ".vscode-server" setup, so it doesn't need to be recovered after the image rebuild.

The volume can be added in the "_.devcontainer/docker-compose.yml_" file:

```yaml
version: '3'

volumes:
  ...
  usrvol:
  ...

services:

  dev:
    ...
    volumes:
      ...
      - usrvol:/root    # persist the home directory (bash_history etc.)
      ...
```

### 3.8. Optional: Auto build on save

This optional setup can be somewhat controversial and personal preference, not everyone likes it.

I personally like it when the build + tests are run after each save automatically, so that I don't have to trigger them explicitly and can always see the effect of my changes immediately.

However for it to be useful, it is best for the project to meet some criteria:
- the incremental build needs to be fast (in the matter of seconds, max. up to a minute): this is a separate topic, but there are techniques to achieve that (precompiled headers, forward declarations, shared libs to avoid long link times etc.)
- if also running the unit tests, those also need to be fast (ideally seconds, which you should always strive for - you might also run just some "fast" subset of unit tests by default)

In general, it might not be the best for very large projects, but can be beneficial for smaller ones (or small sub-projects) that build and test very fast.
Note also that the trigger can be set up so that if you save a new version of a file when the build is still running, it will stop the current build and start a new one ("restart" the build).

**a) Adding the TriggerTaskOnSave extension**:

- the extension is added into the "_.devcontainer/devcontainer.json_" file:

```json
    ...
    "extensions": [
        ...
        "gruntfuggly.triggertaskonsave",
        ...
    ]
```

**b) Adding the task to be run**:

- update or create the "_.vscode/tasks.json_" file:

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build all",
            "type": "shell",
            "command": "time ninja all",
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "options": {
                "cwd": "/build/Debug"
            },
            "presentation": {
                "clear": true
            },
            "problemMatcher": {
                "owner": "cpp",
                "fileLocation": "absolute",
                "pattern": {
                    "regexp": "(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
                    "file": 1,
                    "line": 2,
                    "column": 3,
                    "severity": 4,
                    "message": 5
                }
            }
        }
    ]
}
```

- the "command" option is the one specifying what particular command will be executed

**c) Setting up the automatic task**:

- update the "_.vscode/settings.json_" file:

```json
{
    ...
    "triggerTaskOnSave.tasks": {
        "build all check": [
            "**/*.hpp",
            "**/*.inl",
            "**/*.cpp",
            "**/CMake*.*",
            "**/*.cmake",
        ]
    },
    "triggerTaskOnSave.selectedTask": "build all",
    "triggerTaskOnSave.on": true,
    "triggerTaskOnSave.restart": true,
    "triggerTaskOnSave.showNotifications": true,
    ...
}
```

## Resources and references

- [Microsoft: Install Linux on Windows with WSL](https://docs.microsoft.com/en-us/windows/wsl/install)
- [Microsoft: Advanced settings configuration in WSL](https://docs.microsoft.com/en-us/windows/wsl/wsl-config)
- [Microsoft: Using Docker in WSL 2](https://code.visualstudio.com/blogs/2020/03/02/docker-in-wsl2)
- [Setting-up a local development environment using VS Code, Docker, and WSL 2](https://dev.to/jnous5/setting-up-a-local-development-environment-using-vs-code-docker-and-wsl-2-5c0i)

{% include abbrev domain="computers" %}
