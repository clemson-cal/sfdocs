# Code Configuration Reference

!!! abstract "Complete Parameter Guide"
    Comprehensive reference for all Sailfish configuration parameters, organized by functionality and usage patterns.

---

## Numerical & Time Integration

### Timestep Control

=== "Adaptive Timestep"
    ```cfg
    cfl = 0.3              # CFL number [0.01-0.4]
    dt_mode = 0.0          # Use adaptive timestep
    ```
    
    **CFL Condition**: Ensures numerical stability by limiting timestep based on wavespeeds:
    
    $$\Delta t = C_{CFL} \cdot \min\left(\frac{\Delta x}{|u| + c}\right)$$

=== "Fixed Timestep"
    ```cfg
    cfl = 0.3              # Still used for stability checks
    dt_mode = 1e-3         # Fixed timestep value
    ```
    
    **Fixed Mode**: When `dt_mode > 0.0`, use this exact timestep value instead of adaptive CFL.

### Temporal Accuracy

| Parameter | Type | Range | Description |
|-----------|------|-------|-------------|
| **`rk`** | `int` | `1, 2, 3` | Runge-Kutta integration order |
| **`theta`** | `double` | `1.0-2.0` | PLM slope limiter parameter for spatial reconstruction |

!!! tip "RK Order Selection"
    - **RK1**: First-order Euler method (fastest, least accurate)
    - **RK2**: Second-order midpoint method (balanced)
    - **RK3**: Third-order SSPRK3 method (most accurate, slowest)

---

## Simulation Domain & Time

### Time Control

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| **`tstart`** | `double` | `0.0` | Simulation start time |
| **`tfinal`** | `double` | `0.0` | Simulation end time |
| **`tunits`** | `double` | `1.0` | Time unit conversion factor |

!!! info "Time Unit System"
    **Raw Time**: Internal simulation time where orbital period at r=1 is 2π
    
    **User Time**: `t_user = t_raw / tunits`
    
    **Special Values**:
    
    - `tunits > 0`: Direct scaling factor
    - `tunits < 0`: Multiply by 2π (e.g., `tunits=-1` → one orbit = 1 time unit)
    - `tunits = 0`: **Invalid** (throws error)

### Spatial Domain

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| **`domain`** | `vec2` | `{0.5, 1.5}` | Radial domain extent `[r_inner, r_outer]` |
| **`dx`** | `double` | `1e-2` | Grid spacing (dlogr for polar, dx/dy for cartesian) |

---

## Physics Parameters

### Gravitational Setup

=== "Single Point Mass"
    ```cfg
    central_object = "single"
    ```
    
    **Configuration**: Single point mass at origin
    
    **Use Cases**: Isolated accretion disks, method validation

=== "Binary System"
    ```cfg
    central_object = "binary"
    mass_ratio = 1.0           # q = M2/M1 [0.0-1.0]
    a = 1.0                    # Binary separation
    eccentricity = 0.0         # Orbital eccentricity [0.0-1.0)
    ```
    
    **Configuration**: Two point masses with specified properties
    
    **Use Cases**: Circumbinary disks, gravitational wave astronomy

=== "Merger Evolution"
    ```cfg
    central_object = "merger"
    mass_ratio = 1.0           # Must specify mass ratio
    eccentricity = 0.0         # Must be exactly 0.0
    tunits = 1.0               # Must be exactly 1.0
    tstart = -100              # Start during inspiral (t < 0)
    tfinal = 10                # Continue post-merger (t > 0)
    ```
    
    **Configuration**: GW inspiral with separation evolution
    
    **Inspiral Law**: `a(t) = a₀ × (-t)^0.25` for t < 0
    
    **Merger**: At t = 0, binary coalesces into single point mass

### Viscosity Models

=== "No Viscosity"
    ```cfg
    alpha = 0.0
    nu = 0.0
    ```

=== "Alpha Viscosity"
    ```cfg
    alpha = 0.01               # Shakura-Sunyaev alpha parameter
    nu = 0.0                   # Must be zero when using alpha
    ```
    
    **Model**: `ν = α cs²/Ω` where cs is sound speed, Ω is orbital frequency

=== "Constant Kinematic Viscosity"
    ```cfg
    alpha = 0.0                # Must be zero when using nu
    nu = 0.01                  # Kinematic viscosity coefficient
    ```
    
    **Model**: `ν = constant` throughout the domain

### Stress Tensor Control

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| **`beta`** | `double` | `0.0` | Bulk/dynamic viscosity ratio (ζ/μ) |

!!! warning "Viscosity Tensor"
    **Stress Tensor**: `τᵢⱼ = μ (∂vⱼ/∂xᵢ + ∂vᵢ/∂xⱼ + (β - 2/3) δᵢⱼ ∇·v)`
    
    **Typical Values**:
    
    - **Disk simulations**: `beta = 0.0` (minimal bulk viscosity)
    - **Spreading ring**: `beta = 5/3` (significant bulk viscosity)

### Thermodynamics

=== "Cold (Pressureless)"
    ```cfg
    thermodynamics = "cold"
    ```
    
    **Model**: `P = 0` (pressure floor applied)
    
    **Incompatible**: Cannot use with alpha viscosity

=== "Locally Isothermal"
    ```cfg
    thermodynamics = "local_iso"
    mach = 10.0                # Orbital Mach number
    ```
    
    **Model**: `cs² = -Φ/M²` where Φ is gravitational potential
    
    **Pressure**: `P = ρcs²/γ` with γ = 5/3

=== "Adiabatic (Gamma Law)"
    ```cfg
    thermodynamics = "gamma_law"
    ```
    
    **Model**: Full adiabatic evolution with γ = 5/3

---

## Initial Conditions

### Setup Types

=== "Pringle Ring"
    ```cfg
    setup = "ring"
    nu = 0.01                  # Required for viscous spreading
    ```
    
    **Purpose**: Method validation using analytical Pringle solution
    
    **Physics**: Viscous spreading ring with known time evolution

=== "Steady Disk"
    ```cfg
    setup = "steady"
    ell = 1.0                  # Angular momentum parameter
    ```
    
    **Purpose**: Equilibrium viscous disk studies
    
    **Physics**: Steady accretion with `Ṁ` and `J̇` balance

=== "KITP Setup"
    ```cfg
    setup = "kitp"
    ```
    
    **Purpose**: Binary disk interactions (Santa Barbara comparison)
    
    **Physics**: Circumbinary disk with gap formation and spiral waves

---

## Sink & Softening

### Point Mass Accretion

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| **`sink_size`** | `double` | `0.05` | Radius of sink region |
| **`sink_rate`** | `double` | `10.0` | Accretion rate magnitude |
| **`sink_shape`** | `double` | `4.0` | Sink kernel shape exponent |

!!! note "Sink Model"
    **Accretion Kernel**: `K(r) = exp(-((r/R_sink)^sink_shape))`
    
    **Rate**: Proportional to viscous rate at sink radius

### Gravitational Softening

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| **`rsoft`** | `double` | `1.0` | Softening length scale factor |

!!! info "Softened Potential"
    **Formula**: `Φ = -GM/√(r² + ε²)` where `ε = dx × rsoft`
    
    **Purpose**: Prevent singularities and aid convergence

---

## Mesh Configuration

### Coordinate Systems

=== "Polar Coordinates"
    ```cfg
    mesh = "polar"
    domain = [1.0, 10.0]       # [r_min, r_max]
    dx = 0.01                  # Logarithmic radial spacing
    ```
    
    **Grid**: Logarithmic radial × uniform azimuthal
    
    **Advantages**: Natural for disk physics, efficient angular coverage

=== "Cartesian Coordinates"
    ```cfg
    mesh = "cartesian"
    domain = [0.5, 5.0]        # Sets domain extent
    dx = 0.01                  # Uniform grid spacing
    ```
    
    **Grid**: Uniform Cartesian grid
    
    **Advantages**: Simpler geometry, no coordinate singularities

### Advanced Mesh Features

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| **`mesh_omega`** | `double` | `0.0` | Mesh rotation frequency |
| **`mesh_contract`** | `bool` | `false` | Enable radial domain contraction |
| **`mesh_wobble`** | `bool` | `false` | Track binary motion |

---

## Boundary Conditions

### Inner Boundary (Polar Only)

=== "Inflow Boundary"
    ```cfg
    bc = "inflow"
    ```
    
    **Behavior**: Material can flow in from inner boundary
    
    **Implementation**: Ghost zones use initial condition values

=== "Outflow Boundary"
    ```cfg
    bc = "outflow"
    ```
    
    **Behavior**: Material can only flow outward
    
    **Implementation**: Extrapolation from interior

---

## Output & Diagnostics

### Output Control

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| **`fold`** | `int` | `1` | Iterations between status messages |
| **`cpi`** | `double` | `0.0` | Checkpoint interval (0 = disabled) |
| **`spi`** | `double` | `0.01` | Science products interval |
| **`tsi`** | `double` | `0.0` | Time series interval (0 = disabled) |
| **`outdir`** | `string` | `"."` | Output directory path |

### Diagnostic Sets

=== "Full Diagnostics"
    ```cfg
    diagnostic_set = "all"
    ```
    
    **Products**: All available fields (0-6)
    
    **Timeseries**: Complete set (0-21)

=== "Minimal Diagnostics"
    ```cfg
    diagnostic_set = "min"
    ```
    
    **Products**: Essential fields only
    
    **Timeseries**: Basic quantities

=== "Custom Selection"
    ```cfg
    diagnostic_set = "custom"
    sp = [0, 1, 2, 3]          # Selected product columns
    ts = [0, 10, 11, 20]       # Selected timeseries columns
    ```
    
    **Flexibility**: Choose exactly which outputs to generate

=== "No Diagnostics"
    ```cfg
    diagnostic_set = "none"
    ```
    
    **Products**: None
    
    **Timeseries**: None (minimal overhead)

---

## Parameter Validation

!!! warning "Configuration Constraints"
    
    **Merger Mode**:
    
    - `eccentricity` must be 0.0
    - `tunits` must be 1.0
    
    **Viscosity**:
    
    - Only one of `alpha` or `nu` can be non-zero
    - Alpha viscosity incompatible with cold thermodynamics
    
    **Time Units**:
    
    - `tunits` cannot be zero
    
    **Mesh**:
    
    - Polar mesh requires `nj` divisible by 4 (for symmetry analysis)

!!! tip "Best Practices"
    
    **Performance**:
    
    - Use `fold > 1` for long simulations to reduce I/O
    - Set `diagnostic_set = "custom"` for production runs
    
    **Stability**:
    
    - Keep `cfl ≤ 0.4` for stability
    - Use `rk = 2` or `rk = 3` for smooth flows
    
    **Accuracy**:
    
    - Set `theta = 1.5` for good TVD properties
    - Choose `dx` based on features you want to resolve