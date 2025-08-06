# Sailfish-v0.8

!!! abstract "Project Overview"
    A 2D hydrodynamics code for simulating fluid dynamics on cylindrical/cartesian grids. Designed for high-resolution astrophysical simulations with advanced numerical schemes.

[**Source Code - GitHub**](https://github.com/clemson-cal/sailfish-v0.8){ .md-button .md-button--primary }

---

## Features

<div class="grid cards" markdown>

-   :material-chart-line-variant: **Advanced Numerics**
    
    ---
    
    - Piecewise Linear Method with minmod slope limiter
    - HLLE Riemann solver for shock capturing  
    - 2nd order accuracy with TVD properties

-   :material-border-none-variant: **Boundary Conditions**
    
    ---
    
    - Flexible inflow and outflow boundaries
    - Periodic azimuthal conditions
    - Customizable sink regions

-   :material-cpu-64-bit: **High Performance**
    
    ---
    
    - GPU acceleration via VAPOR library
    - OpenMP parallelization 
    - Optimized for modern hardware

-   :material-grid: **Mesh Flexibility**
    
    ---
    
    - Cartesian and polar coordinate systems
    - Rigid mesh rotation capabilities
    - Adaptive radial contraction

-   :material-telescope: **Astrophysical Models**
    
    ---
    
    - Single, binary, and merger scenarios
    - Viscous disk physics (α-model)
    - Gravitational wave inspiral dynamics

-   :material-file-chart: **Data Output**
    
    ---
    
    - HDF5 format for efficient storage
    - Comprehensive diagnostic outputs
    - Time series and checkpoint data

</div>

---

## Getting Started

!!! tip "Quick Setup"
    Follow these steps to get Sailfish running on your system.

### Prerequisites

!!! warning "Requirements"
    - **GCC** version 10 or higher
    - **CUDA** support (optional, for GPU mode)
    - **Python** dependencies for visualization

### Installation

=== "Clone Repository"
    ```bash
    git clone --recurse-submodules git@github.com:clemson-cal/sailfish-v0.8.git
    cd sailfish-v0.8
    ```

=== "Configure Vapor"
    Create the required configuration files:
    
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
        "modes": ["dbg", "cpu", "omp", "gpu"],
        "libs": [],
        "omp_flags": "-Xpreprocessor -fopenmp",
        "lomp": "-lgomp",
        "nvcc_ccbin": "c++"
    }
    ```

=== "Build & Test"
    ```bash
    # Configure build system
    ./configure
    
    # Compile (choose your target)
    make cpu    # CPU version
    make omp    # OpenMP version  
    make gpu    # GPU version (requires CUDA)
    
    # Test installation
    ./bin/sailfish_dbg presets/steady.cfg
    ```

!!! example "Platform Notes"
    
    === "Linux"
        ```bash
        # Install OpenMP
        sudo apt install libomp-dev
        # Use: "lomp": "-lgomp" 
        ```
    
    === "macOS (Apple Silicon)" 
        ```bash
        # Install OpenMP via Homebrew
        brew install libomp
        # Use: "lomp": "-lomp"
        ```
    
    === "Windows"
        Windows support via WSL2 recommended.

---

## Key Capabilities

!!! success "Simulation Types"

    | Setup Type | Description | Use Case |
    |------------|-------------|----------|
    | **Ring** | Viscous spreading ring | Method validation |
    | **Steady** | Equilibrium disk | Long-term evolution |
    | **KITP** | Binary disk interaction | Santa-Barbara setup |

!!! info "Performance Metrics" 
    Typical performance on modern hardware:
    
    - **GPU**: ~5B zones/second on 5x H100's for a binary simulation with dx = 1e-3

---

## Quick Links

<div class="grid cards" markdown>

-   [:material-book-open-page-variant: **Documentation**](setups.md)
    
    ---
    
    Learn about different simulation setups

-   [:material-code-braces: **Code Examples**](code-examples.md)
    
    ---
    
    Explore implementation details

-   [:material-math-integral: **Numerical Methods**](numerical-methods.md)
    
    ---
    
    Understand the algorithms

-   [:material-cog: **Configuration**](definitions.md)
    
    ---
    
    Configure your simulations

</div>

