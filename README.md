# Using Rez on Windows: A Comprehensive Guide

This guide provides step-by-step instructions for installing and configuring Rez on a Windows system, addressing common issues and offering solutions to ensure a smooth setup.

## Table of Contents

1. [Downloading Rez](#1-downloading-rez)
2. [Installing Rez](#2-installing-rez)
3. [Configuring Environment Variables](#3-configuring-environment-variables)
4. [Binding Standard Packages](#4-binding-standard-packages)
5. [Handling Python Package Binding Issues](#5-handling-python-package-binding-issues)
6. [Adding Autodesk Maya as a Rez Package](#6-adding-autodesk-maya-as-a-rez-package)
7. [Running Maya with Rez](#7-running-maya-with-rez)
8. [Integrating External Libraries (e.g., NumPy) into Rez Packages](#8-integrating-external-libraries-eg-numpy-into-rez-packages)
9. [Running Maya with Integrated Libraries](#9-running-maya-with-integrated-libraries)
10. [Additional Suggestions](#10-additional-suggestions)

## 1. Downloading Rez

Begin by obtaining the latest version of Rez from its official repository:

- **GitHub Repository**: [https://github.com/AcademySoftwareFoundation/rez](https://github.com/AcademySoftwareFoundation/rez)

You can either clone the repository using Git:

```bash
git clone https://github.com/AcademySoftwareFoundation/rez.git
```

Or download the ZIP archive directly from the repository's main page.

## 2. Installing Rez

After downloading, follow these steps to install Rez:

1. **Extract the Archive**: If you downloaded the ZIP file, unzip it to a directory of your choice.

2. **Navigate to the Root Directory**: Open Command Prompt and change to the directory where Rez is located:

```bash
cd path\to\rez
```

3. **Run the Installation Script**: Execute the following command to install Rez:

```bash
python ./install.py
```

> **Note**: Ensure you're using Python 3.7 or above, as it's required for Rez installation.

## 3. Configuring Environment Variables

To make Rez accessible from any command prompt, add its bin directory to your system's PATH environment variable:

1. **Locate the Installation Path**: By default, Rez installs to `/opt/rez`. If you've installed it elsewhere, adjust the path accordingly.

2. **Add to PATH**:
   - Open the Start Menu and search for "Environment Variables."
   - Select "Edit the system environment variables."
   - In the System Properties window, click on "Environment Variables."
   - In the System Variables section, find and select the Path variable, then click "Edit."
   - Click "New" and add the path to Rez's bin directory (e.g., `C:\path\to\rez\bin`).
   - Click "OK" to save and apply the changes.

## 4. Binding Standard Packages

Rez uses the rez-bind tool to create packages that reference software already installed on your system. To bind a set of standard packages:

**Run the Quickstart Binding**:

```bash
# Run PowerShell as Administrator for this step
rez-bind --quickstart
```

> **Important**: This command must be run in PowerShell with Administrator privileges because it creates symbolic links that require elevated permissions on Windows systems.

This command binds essential packages like platform, arch, os, and python into your Rez environment.

## 5. Handling Python Package Binding Issues

On Windows, binding the Python package can encounter issues due to symbolic link creation restrictions. To address this:

1. **Understand the Issue**: Windows has limitations with symbolic links, which can cause the `rez-bind python` command to fail.

2. **Enable Developer Mode**: To allow the creation of symbolic links without administrative privileges:
   - Go to Settings > Privacy & Security > For Developers.
   - Turn on Developer Mode.

3. **Alternative Solution - Use rezify-python**: If enabling Developer Mode isn't feasible, consider using rezify-python, a tool designed to create full Python packages on Windows without relying on symbolic links.

   - Download rezify-python: https://github.com/techartorg/rez_utils/tree/main/rezify_python
   - Install rezify-python:
     - Clone the repository or download the ZIP archive.
     - Navigate to the rezify_python directory.
     - Run the installation script as per the provided instructions.
   - Create the Python Package:

     ```bash
     rezify_python -v 3.11.9
     ```
     
     Replace 3.11.9 with your desired Python version.

## 6. Adding Autodesk Maya as a Rez Package

To integrate Autodesk Maya into your Rez environment:

1. **Create the Package Definition**: In your Rez packages directory (e.g., `C:\Users\YourUsername\packages`), create a folder named `maya` and within it, a subfolder named `2023`.

2. **Create the package.py File**: Inside the `2023` folder, create a `package.py` file with the following content:

```python
name = 'maya'
version = '2023'
description = 'Autodesk Maya 2023'
authors = ['Your Name']

def commands():
    import os
    
    maya_location = r'C:\Program Files\Autodesk\Maya2023'
    maya_bin = os.path.join(maya_location, 'bin')
    env.PATH.append(maya_bin)
    
    # Set critical environment variables
    env.MAYA_LOCATION = maya_location
    env.PYTHONPATH.append(os.path.join(maya_location, 'Python', 'Lib', 'site-packages'))
```

Adjust the `maya_location` path if Maya is installed in a different directory.

## 7. Running Maya with Rez

With the Maya package defined, you can launch Maya within a Rez environment:

```bash
rez-env maya -- maya
```

This command sets up the environment as specified in the `package.py` file and then launches Maya.

## 8. Integrating External Libraries (e.g., NumPy) into Rez Packages

To add external Python libraries like NumPy to your Rez environment for use in Maya:

1. **Create a New Package**: In your Rez packages directory, create a folder named `maya_numpy` and within it, a subfolder named `1.23.0`.

2. **Create the package.py File**: Inside the `1.23.0` folder, create a `package.py` file with the following content:

```python
name = "maya_numpy"
version = "1.23.0"
requires = ["maya-2023"]
description = "NumPy installed for Maya 2023 compatibility"

build_command = "python {root}/build.py {install_root}"

def commands():
    # Add the installed NumPy to PYTHONPATH
    import os
    env.PYTHONPATH.append(os.path.join("{root}", "lib", "site-packages"))
```

3. **Create the build.py File**: In the same `1.23.0` folder, create a `build.py` file with this script:

```python
import os
import subprocess
import sys

# Get the installation root directory from the command line arguments
package_root = sys.argv[1]

# Path to Maya's Python interpreter
mayapy = os.path.join(os.environ.get("MAYA_LOCATION", "C:/Program Files/Autodesk/Maya2023"), "bin", "mayapy.exe")

# Create the site-packages directory
install_dir = os.path.join(package_root, "lib", "site-packages")
os.makedirs(install_dir, exist_ok=True)

# Install NumPy using pip with Maya's Python interpreter
numpy_version = "1.23.0"
subprocess.check_call([mayapy, "-m", "pip", "install", f"numpy=={numpy_version}", "-t", install_dir])
```

4. **Build the Package**: To build the package, use the following command:

```bash
rez-build --install
```

Run this command from within the `maya_numpy/1.23.0` directory.

## 9. Running Maya with Integrated Libraries

Once you have defined both the Maya and NumPy packages, you can run Maya with NumPy support:

```bash
rez-env maya maya_numpy -- maya
```

This command creates an environment with both Maya and NumPy available.

## 10. Additional Suggestions

- **Create a Central Package Repository**: Consider setting up a central network location for shared Rez packages in team environments.

- **Version Control**: Keep your package definitions under version control to maintain consistency across different systems.

- **Build scripts**: For more complex packages, use build scripts to automate the installation process.

- **Documentation**: Maintain clear documentation for custom packages, specifying dependencies and usage instructions.

## Troubleshooting

- **Path Issues**: If you encounter "command not found" errors, double-check that Rez's bin directory is correctly added to your PATH.

- **Permission Errors**: When binding packages, ensure you have the necessary permissions or run the command with administrative privileges.

- **Package Resolution Failures**: Use `rez-context --explain` to diagnose dependency resolution issues.

- **Symbolic Link Errors**: Many Rez operations on Windows require either Administrator privileges or Developer Mode enabled due to Windows' restrictions on symbolic link creation.

## Resources

- [Official Rez Documentation](https://github.com/AcademySoftwareFoundation/rez/wiki)
- [Rez Package Examples](https://github.com/AcademySoftwareFoundation/rez/tree/main/example)
