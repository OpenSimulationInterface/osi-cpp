Open Simulation Interface (OSI) C++ Bindings
============================================

This C++ package provides C++ bindings for the [ASAM Open Simulation Interface](https://github.com/OpenSimulationInterface/open-simulation-interface).

For more information on OSI see the [official documentation](https://opensimulationinterface.github.io/osi-antora-generator/asamosi/latest/specification/index.html) or the [class list](https://opensimulationinterface.github.io/osi-antora-generator/asamosi/latest/gen/annotated.html) for defined protobuf messages.

Usage
-----

For usage examples, please refer to the official documentation:
- [OSMP examples](https://opensimulationinterface.github.io/osi-antora-generator/asamosi/latest/sensor-model/setup/build_install_example.html), including the code found at [osi-sensor-model-packaging](https://github.com/OpenSimulationInterface/osi-sensor-model-packaging):
  - [OSMPDummySource](https://github.com/OpenSimulationInterface/osi-sensor-model-packaging/tree/master/examples/OSMPDummySource)
  - [OSMPDummySensor](https://github.com/OpenSimulationInterface/osi-sensor-model-packaging/tree/master/examples/OSMPDummySensor)
  - [OSMPCNetworkProxy](https://github.com/OpenSimulationInterface/osi-sensor-model-packaging/tree/master/examples/OSMPCNetworkProxy)

Installation
------------

### Checkout the repository with submodules

- Open a terminal.
- Clone the osi-cpp repository, including sub-modules:

  ```console
  git clone --recurse-submodules https://github.com/PMSFIT/osi-cpp.git
  ```

- Switch to the repository directory:

  ```console
  cd osi-cpp
  ```

### Using vcpkg for dependencies

Especially for building on Windows, the use of [vcpkg](https://vcpkg.io/) to handle dependencies is recommended.
This can be achieved easily through the use of the CMake `vcpkg` preset when configuring:

In order to use the preset the environment variable `VCPKG_ROOT` has to be set to the root of your vcpkg installation.
This can either be done manually, or through a CMake User preset that inherits from the vcpkg preset, i.e. by providing a `CMakeUserPresets.json` file with the following content:

```json
{
  "version": 3,
  "configurePresets": [
    {
      "name": "default",
      "inherits": "vcpkg",
      "environment": {
        "VCPKG_ROOT": "C:\\vcpkg"
      }
    }
  ]
}
```

When building on windows for inclusion into a DLL, e.g. for FMU generation, you can inherit from the `vcpkg-windows` preset instead of the `vcpkg` preset to build for static linking into a DLL.

The build can then be performed in the following manner:

- Configure the build:

  ```console
  cmake --preset default
  ```

- Perform the build:

  ```console
  cmake --build build
  ```

See the [vcpkg documentation](https://vcpkg.io/) for further information on the use of vcpkg in combination with CMake in various build environments.

### Building without vcpkg

To build without the use of vcpkg to handle dependencies, you have to ensure that necessary prerequisites are installed and detectable by CMake.

Once this has been achieved, the build can be performed in the following manner:

- Configure the build:

  ```console
  cmake -B build
  ```

- Perform the build:

  ```console
  cmake --build build
  ```

### Further information

For detailed installation instructions, please refer to the official documentation:
- [Setting up OSI](https://opensimulationinterface.github.io/osi-antora-generator/asamosi/latest/interface/setup/setting_up_osi.html)
  - [Installing _protobuf_ for static / dynamic linking](https://opensimulationinterface.github.io/osi-antora-generator/asamosi/latest/interface/setup/installing_prerequisites.html)
  - [Installing OSI for C++ on Linux](https://opensimulationinterface.github.io/osi-antora-generator/asamosi/latest/interface/setup/installing_linux_cpp.html)
  - [Installing OSI for C++ on Windows](https://opensimulationinterface.github.io/osi-antora-generator/asamosi/latest/interface/setup/installing_windows_cpp.html)
