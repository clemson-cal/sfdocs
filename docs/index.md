# Sailfish-v0.8

This project involves the development of a 2D hydrodynamics code for simulating fluid dynamics on a cylindrical/cartesian grid. The code includes advanced numerical schemes such as the Piecewise Linear Method (PLM) and handles both inflow and outflow boundary conditions. It is designed for high-resolution simulations, particularly useful in astrophysical contexts.

[Source Code - Github](https://github.com/clemson-cal/sailfish-v0.8)

## Features
- Piecewise Linear Method with a minmod slope limiter for 2nd order accuracy
- Godunov-type Riemann solver for shock capturing
- Inflow and outflow boundary conditions
- MPI parallelization for efficient computation on multiple cores
- Support for both cylindrical and cartesian grids
- 2D visualization of simulation results using Matplotlib
- Based on the VAPOR C++ library
- Concise and expressive functional programming style
- Supports azimuthal mesh rotation [rigid]
- Supports radial mesh contraction
- Fifth order WENO reconstruction (WIP)
- Output results in HDF5 format

## Getting Started
- Clone the repository with __vapor__ submodule:

`git clone --recurse-submodules git@github.com:clemson-cal/sailfish-v0.8.git`

    1. gcc version 10 or higher is required
    2. Install the python plotting dependencies using the requirements.txt file

- Setup the project.json and system.json files for the vapor library:

```json title="project.json"
{
	"src": "src",
	"bin": "bin",
	"build": "build",
	"vapor": "vapor",
	"programs": {
		"sailfish": {
			"deps": ["hdf5"]
		}
	}
}
```
```json title="system.json"
{
    "modes": ["dbg", "cpu", "omp", "gpu"],  // "gpu" mode requires CUDA support
    "libs": [],
    "omp_flags": "-Xpreprocessor -fopenmp",
    "lomp": "-lgomp",
    "nvcc_ccbin": "c++"
}
```
??? warning "gpu mode and lomp flag"

    You would need CUDA support for the "gpu" mode to work. If you don't have CUDA support, you can remove the "gpu" mode from the system.json file.
    If you are using a Mac with Apple Silicon, you would need to install the libomp library and set the "lomp" flag in the system.json file to "-lomp".
    For Linux machines, you can install the libomp library and set the "lomp" flag in the system.json file to "-lgomp".

- Configure the build (place the __configure__ file within the vapor library in your main work directory):

    run `./configure`

- Compile the code:

    1. For __debug__ mode: `make dbg`
    2. For __cpu__ mode: `make cpu`
    3. For __openmp__ mode: `make omp`
    4. For __gpu__ mode: `make gpu`

- Test the code:

    __For a dbg build__: run `./bin/sailfish_dbg presets/ring.cfg`

