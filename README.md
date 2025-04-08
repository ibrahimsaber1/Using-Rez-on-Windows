# Using Rez with DaVinci Resolve: A Comprehensive Guide

This guide provides step-by-step instructions for installing and configuring Rez to work with DaVinci Resolve on a Windows system, addressing common issues and offering solutions to ensure a smooth setup.

## Table of Contents

1. [Downloading Rez](#1-downloading-rez)
2. [Installing Rez](#2-installing-rez)
3. [Configuring Environment Variables](#3-configuring-environment-variables)
4. [Binding Standard Packages](#4-binding-standard-packages)
5. [Handling Python Package Binding Issues](#5-handling-python-package-binding-issues)
6. [Adding DaVinci Resolve as a Rez Package](#6-adding-davinci-resolve-as-a-rez-package)
7. [Determining DaVinci Resolve's Python Version](#7-determining-davinci-resolves-python-version)
8. [Creating a Compatible Python Package](#8-creating-a-compatible-python-package)
9. [Adding PySide2 for DaVinci Resolve](#9-adding-pyside2-for-davinci-resolve)
10. [Running DaVinci Resolve with Rez](#10-running-davinci-resolve-with-rez)
11. [Additional Suggestions](#11-additional-suggestions)
12. [Troubleshooting](#troubleshooting)

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
     rezify_python -v 3.10.0
     ```
     
     Replace 3.10.0 with the Python version used by your DaVinci Resolve (see Section 7).

## 6. Adding DaVinci Resolve as a Rez Package

To integrate DaVinci Resolve into your Rez environment:

1. **Create the Package Definition**: In your Rez packages directory (e.g., `C:\Users\YourUsername\packages`), create a folder named `davinci_resolve` and within it, a subfolder named `18.6.3` (adjust version if needed).

2. **Create the package.py File**: Inside the `18.6.3` folder, create a `package.py` file with the following content:

```python
name = 'davinci_resolve'
version = '18.6.3'
description = 'Blackmagic Design DaVinci Resolve 18.6.3'
authors = ['Your Name']

def commands():
    import os
    
    resolve_location = r'C:\Program Files\Blackmagic Design\DaVinci Resolve'
    resolve_bin = resolve_location
    env.PATH.append(resolve_bin)
    
    # Set critical environment variables
    env.RESOLVE_LOCATION = resolve_location
    # Add the Resolve Python modules path
    # Note: Resolve doesn't have a direct mayapy equivalent
    env.PYTHONPATH.append(os.path.join(resolve_location, 'fusionscript.dll'))
```

Adjust the `resolve_location` path if DaVinci Resolve is installed in a different directory.

## 7. Determining DaVinci Resolve's Python Version

Unlike Maya, DaVinci Resolve doesn't include a standalone Python executable like `mayapy.exe`. To determine which Python version DaVinci Resolve uses:

1. **Open DaVinci Resolve**

2. **Access the Console**: 
   - Go to Workspace > Console
   - Or use the keyboard shortcut (typically Shift+F12)

3. **Check Python Version**:
   - In the console, type and execute:
   ```python
   import sys
   print(sys.version)
   ```

4. **Note the Version**: The output will show the Python version used by Resolve. In this example, it's Python 3.10.0.

This information is crucial for creating compatible packages to use with DaVinci Resolve.

## 8. Creating a Compatible Python Package

To create a Python package compatible with DaVinci Resolve:

1. **Create or Use an Existing Python Package**: Ensure you have a Python package in your Rez environment that matches the version used by DaVinci Resolve (3.10.0 in this case).

2. **Example of Creating a Python Package**:
   - If using rezify-python:
   ```bash
   rezify_python -v 3.10.0
   ```
   - This creates a standalone Python package compatible with DaVinci Resolve's Python version.

## 9. Adding PySide2 for DaVinci Resolve

To add PySide2 for use with DaVinci Resolve:

1. **Create the Package Structure**: In your Rez packages directory, create a folder named `py310_PySide2` and within it, a subfolder named `5.15.2`.

2. **Create the package.py File**: Inside the `5.15.2` folder, create a `package.py` file with the following content:

```python
# -*- coding: utf-8 -*-

name = 'py310_PySide2'
version = '5.15.2'
description = "PySide installed for DaVinci Resolve 18.6.3 compatibility"
requires = ['davinci_resolve-18.6.3']

# Path to your Python 3.10.0 executable in your Rez packages
build_command = [
    r"D:\new\packages\python\3.10.0\platform-windows\arch-AMD64\python.exe",
    r"{root}\build.py",
    "{install}"
]

def commands():
    import os
    env.PATH.append(os.path.join("{root}", "build", "install", "lib", "site-packages"))
    env.PYTHONPATH.append(os.path.join("{root}", "build", "install", "lib", "site-packages"))
```

3. **Create the build.py File**: In the same `5.15.2` folder, create a `build.py` file with this script:

```python
import os
import subprocess
import sys

package_root = sys.argv[1]
install_dir = os.path.join(package_root, "lib", "site-packages")
os.makedirs(install_dir, exist_ok=True)

# Path to your Python 3.10.0 executable in your Rez packages
python_cmd = r"D:\new\packages\python\3.10.0\platform-windows\arch-AMD64\python.exe"
package_name = "PySide2"
package_version = "5.15.2.1"

print("*-"*50)
print(f"Installing {package_name} {package_version} for Python 3.10")
print("*-"*50)
print('Python version:', sys.version)
print("*-"*50)

subprocess.check_call([
    python_cmd, "-m", "pip", "install",
    f"{package_name}=={package_version}",
    "--target", install_dir
])

print(f"{package_name} {package_version} installed successfully to {install_dir}")
```

4. **Build the Package**: To build the package, navigate to the `py310_PySide2/5.15.2` directory and run:

```bash
rez-build --install
```

## 10. Running DaVinci Resolve with Rez

With all packages defined, you can launch DaVinci Resolve within a Rez environment:

```bash
rez-env davinci_resolve py310_PySide2-5.15.2 -- "C:\Program Files\Blackmagic Design\DaVinci Resolve\Resolve.exe"
```

This command sets up the environment with both DaVinci Resolve and PySide2 available.

## 11. Additional Suggestions

- **Create a Central Package Repository**: Consider setting up a central network location for shared Rez packages in team environments.

- **Version Control**: Keep your package definitions under version control to maintain consistency across different systems.

- **Custom Plugins**: For DaVinci Resolve plugins, create separate Rez packages that depend on your davinci_resolve package.

- **Documentation**: Maintain clear documentation for custom packages, specifying dependencies and usage instructions.

- **Multiple Python Packages**: If you work with multiple applications that require different Python versions, create separate Python packages for each version needed.

## Troubleshooting

- **Path Issues**: If you encounter "command not found" errors, double-check that Rez's bin directory is correctly added to your PATH.

- **Permission Errors**: When binding packages, ensure you have the necessary permissions or run the command with administrative privileges.

- **Package Resolution Failures**: Use `rez-context --explain` to diagnose dependency resolution issues.

- **Python Version Mismatch**: Make sure the Python version in your packages matches the one used by DaVinci Resolve. Use the console method described in Section 7 to verify.

- **DLL Load Failures**: If DaVinci Resolve complains about missing DLLs when using PySide2, check that all dependencies are properly installed.

- **Symbolic Link Errors**: Many Rez operations on Windows require either Administrator privileges or Developer Mode enabled due to Windows' restrictions on symbolic link creation.

## Resources

- [Official Rez Documentation](https://github.com/AcademySoftwareFoundation/rez/wiki)
- [Rez Package Examples](https://github.com/AcademySoftwareFoundation/rez/tree/main/)
- [Blackmagic DaVinci Resolve API Documentation](https://documents.blackmagicdesign.com/DeveloperManuals/Fusion_Scripting_Guide.pdf)
