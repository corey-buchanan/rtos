# Lab 0: Build ALL THE THINGS!
In this lab, we'll be setting up the development environment. This environment will allow you to build, test, deploy, and manage your code.
## Learning objective:
- Navigate documentation
- Setup toolchain
- Create a project.
- Commit to source control
- Build project
- Run tests
- Flash a program
- Use the debugger
- Setup CICD
# Prelab
Complete the following tasks before lab. **This will require a large download and long installation, so do it before lab**. At the end of this section, you should have the tools installed and a working project skeleton.

## Reading
https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests


## Install Visual Studio Code
### Overview
We will use the Visual Studio Code integrated development environment (IDE). Modern software development leverages many tools - the IDE integrates them into a single environment. You'll use the IDE to write, manage, build, deploy, and debug your code.

We will also be heavily using the command line. The IDE automates and provides GUI shortcuts, but these are just wrappers over the underlying commands available. **Don't use the GUI when a command line or keyboard shortcut is available**.
### Tasks
Detailed installation instructions are available in the chapters 2 and 3 in the getting started datasheet https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf

1. Download and install VSCode, available at the following link. https://code.visualstudio.com/
1. Install these plugins
    - https://marketplace.visualstudio.com/items?itemName=raspberry-pi.raspberry-pi-pico
    - https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools
    - https://marketplace.visualstudio.com/items?itemName=twxs.cmake
    - https://marketplace.visualstudio.com/items?itemName=ms-vscode.makefile-tools
1. Download and install Renode, available at the following link https://renode.io/#downloads
We'll be using the VSCode extension integration to setup the SDK and the build toolchain.

If you want to install the SDK manually, there are instructions on https://github.com/raspberrypi/pico-sdk

#### Windows specific issues
NOTE: If you are using a windows device, install this: https://zadig.akeo.ie/
Use this to install a driver for the RP2 USB-Device, after you have plugged in your RP2040 from the stockroom to your computer.

#### Linux specific issues
You may need to install this udev rule to allow non-root access over USB.

https://github.com/raspberrypi/picotool/blob/master/udev/99-picotool.rules

You may need to add your user to the dialout group to access serial devices.

`sudo usermod -a -G dialout $USER`

## Create a new project
### Overview

Build tools use a set of conventions to setup a project. This usually defines a consistent organization for source code, header, and test code. The configuration of the project, library dependencies, along with any custom build tasks, will be located in the build file. The Raspberry Pi Pico uses the CMake tool to configure and run the build process. The build file `CMakeLists.txt` is located at the root of the project. The build file will be committed to your git repository with the rest of your source code. Anyone else working on the project will be able to use the same project configuration by simply checking out the code.

### Tasks
You should be able to load the example code onto the board following the instructions.

- Follow the chapter 4 introduction and section 4.1 of the getting started guide. https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf

You can build the project using the GUI as outlined in the documentation. We will explore the command line in the next lab (see Appendix C).

# Lab
# Commit project
## Overview
Now that we have our project, let's get it into source control.
## Tasks
1. Create an ssh key for authentication with github
    1. Generate: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
    1. Add to your account: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account
1. Create repository on github
    1. Do *not* initialize the repository with a README, gitignore, or license.
1. Initialize the local repository.
    1. From your project directory, run `git init`. This sets up the repository metadata.
    1. Stage all of the files in the current working directory `git add .`
    1. Commit the staged files with a descriptive commit message `git commit -m "Initialize Pico project"`
    1. Create a main branch `git branch -M main`
    1. Add your github repository as a remote. `git remote add origin git@github.com:username/lab0.git`
    1. Push the main branch and the commit to the remote. `git push -u origin main`

# Making changes to a project

## Tasks
1. Create a new git branch. You can call it whatever you like, but it is useful to use a descriptive name for what is being developed in the branch.`git checkout -b compilation-demo`
1. Make the modifications in section 4.2 of the getting started datasheet.
1. Build and deploy the changes.
1. Commit the changes to the main file, and push the branch to your github remote.
.
# Setup Continuous Integration
## Overview
Continuous Integration (CI), often paired with Continuous Delivery (CI/CD), is a development pattern to rapidly deliver consistent working software. The basic principle is to always keep your project in a working state, with small incremental changes. The project should be monitored with automated tests and performance instrumentation. When code is pushed to the central repository, a build system will run the automated tests. If the tests pass, the code is ready to be reviewed and then deployed.

Github provides an automated build system called Github Actions. We will set up your project to build and run tests on push. We will then show the passing or failing status of your tests on your repository README.
## Tasks
1. Update your CMakelists.txt file to fix some paths. Replace these lines, which you'll find towards the top of the file.
```
set(PICO_SDK_PATH ${USERHOME}/.pico-sdk/sdk/2.0.0)
set(PICO_TOOLCHAIN_PATH ${USERHOME}/.pico-sdk/toolchain/13_2_Rel1)
```
With these lines. This will allow us to override the paths to the SDK and toolchain with correct paths in the CI/CD environment.
```
if(DEFINED ENV{PICO_SDK_PATH})
    set(PICO_SDK_PATH $ENV{PICO_SDK_PATH})
else()
    set(PICO_SDK_PATH ${USERHOME}/.pico-sdk/sdk/2.0.0)
endif()

if(DEFINED ENV{PICO_TOOLCHAIN_PATH})
    set(PICO_TOOLCHAIN_PATH $ENV{PICO_TOOLCHAIN_PATH})
else()
    set(PICO_TOOLCHAIN_PATH ${USERHOME}/.pico-sdk/toolchain/13_2_Rel1)
endif()
```

1. Create the metadata directories. In the root of your repository, create a directory `.github` (note the leading dot). Inside that directory, create another directory `workflows`.
1. Create a workflow.
    1. Worflows are defined in yaml configuration files, which define a series of steps that will be executed in response to an event, such as a push.
    1. Create a file `.github/workflows/main.yml`. Add the following configuration.
```
name: CMake
on: [push, pull_request]

env:
  # Customize the CMake build type here (Release, Debug, RelWithDebInfo, etc.)
  BUILD_TYPE: Release

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Install GCC
        id: arm-none-eabi-gcc-action
        uses: carlosperate/arm-none-eabi-gcc-action@v1
        with:
          release: '13.2.Rel1'

      - name: Clean workspace
        run: |
          echo "Cleaning up previous run"
          rm -rf "${{ github.workspace }}"
          mkdir -p "${{ github.workspace }}"

      - name: Checkout pico-sdk/develop
        uses: actions/checkout@v2
        with:
          repository: raspberrypi/pico-sdk
          ref: develop
          path: ${{github.workspace}}/pico_sdk
          submodules: true

      - uses: actions/checkout@v3
        with:
          path: ${{github.workspace}}/source

      - name: Create Build Environment
        # Some projects don't allow in-source building, so create a separate build directory
        # We'll use this as our working directory for all subsequent commands
        working-directory: ${{github.workspace}}
        run:  cmake -E make_directory ${{github.workspace}}/build

      - name: Configure CMake
        # Use a bash shell so we can use the same syntax for environment variable
        # access regardless of the host operating system
        shell: bash
        working-directory: ${{github.workspace}}
        # Note the current convention is to use the -S and -B options here to specify source
        # and build directories, but this is only available with CMake 3.13 and higher.
        # The CMake binaries on the Github Actions machines are (as of this writing) 3.12
        run: PICO_SDK_PATH=${{github.workspace}}/pico_sdk PICO_TOOLCHAIN_PATH=${{ steps.arm-none-eabi-gcc-action.outputs.path }} cmake -DCMAKE_BUILD_TYPE=$BUILD_TYPE -B ./build ./source

      - name: Build
        working-directory: ${{github.workspace}}
        shell: bash
        # Execute the build.  You can specify a specific target with "--target <NAME>"
        run: cmake --build ./build --config $BUILD_TYPE --parallel $(nproc)
```

https://docs.github.com/en/actions/learn-github-actions

1. Commit the file and push.
1. add test status badge to your repo README.
    1. Create a file named `README.md` in the root of your project.
    1. Add a brief description of you project.
    1. Add a badge image showing the current status of the workflow
```
![example workflow](https://github.com/<OWNER>/<REPOSITORY>/actions/workflows/main.yml/badge.svg)
```
https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/adding-a-workflow-status-badge
    1. Commit your README and push.
    1. Open a pull request to merge your branch into main branch.
