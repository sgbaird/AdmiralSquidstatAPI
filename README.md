# AdmiralSquidstatAPI

This repo includes the API bundle for public use as well as documentation for using the API.
The Admiral Instruments API gives more control of [our potentiostats](https://www.admiralinstruments.com/products), and gives you the
tools to integrate running our experiments in your pipeline and automating your workflow.

To read the documentation, please visit [the docs website](https://admiral-instruments.github.io/AdmiralSquidstatAPI).

## Installation

### Python API

To install the Python API (Linux):

1. Install system dependencies:
   ```bash
   sudo apt-get update
   sudo apt-get install -y build-essential libgl1-mesa-dev libxcb-cursor0
   ```

2. Install Python dependencies and the SquidstatPyLibrary:
   ```bash
   pip install -r requirements.txt
   pip install --no-deps SquidstatLibrary/Linux/pythonWrapper/SquidstatPyLibrary-1.10.3.0-py3-none-manylinux2014_x86_64.whl
   ```

Note: The `--no-deps` flag is required because the wheel contains hardcoded PySide6 URLs that conflict with PyPI packages. PySide6 is installed separately via requirements.txt.

For device access on Linux systems, you may also need to:
- Add your user to the dialout group: `sudo usermod -a -G dialout $USER`
- Set up udev rules (see `SquidstatLibrary/Linux/install_dependencies.sh` for details)
- Restart your computer for changes to take effect

These device permissions are only needed when connecting to actual Squidstat hardware.

### C++ API

The most convenient way to build and run the API and the provided examples is to utilize CMake with the CMake files we provide. After downloading the SquidstatLibrary folder, open it with an IDE with CMake support. For example, with Visual Studio 2019, if you have the C++ package installed with CMake, you can open the folder and the IDE will generate the project for you automatically. To then run any of the provided examples, set that example as the 'start up item'.