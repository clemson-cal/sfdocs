# Science Products & Timeseries Reference

!!! abstract "Output Data Guide"
    Complete reference for Sailfish science products (2D fields) and timeseries (integrated quantities) outputs, including physical meanings and applications.

---

## Science Products (2D Fields)

Science products are 2D spatial arrays saved to HDF5 files at intervals specified by `spi`. Each product represents a physical quantity distributed across the computational domain.

### Available Products

| Column | Name | Symbol | Units | Description |
|--------|------|--------|-------|-------------|
| **0** | `mass_density` | ρ | M L⁻² | Surface mass density of gas |
| **1** | `x_velocity` | vₓ | L T⁻¹ | Velocity component in x-direction |
| **2** | `y_velocity` | vᵧ | L T⁻¹ | Velocity component in y-direction |
| **3** | `gas_pressure` | P | M L⁻¹ T⁻² | Gas pressure (depends on thermodynamics) |
| **4** | `gravitational_torque_density` | τ | M L T⁻² | Gravitational torque density per unit area |
| **5** | `x_vertices` | x | L | x-coordinates of mesh vertices |
| **6** | `y_vertices` | y | L | y-coordinates of mesh vertices |
| **7** | `M1_x` | x₁ | L | x-position of primary mass |
| **8** | `M1_y` | y₁ | L | y-position of primary mass |
| **9** | `M2_x` | x₂ | L | x-position of secondary mass |
| **10** | `M2_y` | y₂ | L | y-position of secondary mass |

## Timeseries Data

Timeseries are globally integrated quantities computed at intervals specified by `tsi`. Each timeseries represents a single scalar value characterizing the entire simulation domain.

### Angular Momentum & Torques

| Column | Name | Symbol | Units | Description |
|--------|------|--------|-------|-------------|
| **0** | `Jdot_grav` | J̇ₘᵣₐᵥ | M L² T⁻³ | Total gravitational torque on disk |
| **1** | `Jdot_excised` | J̇ₑₓc | M L² T⁻³ | Gravitational torque excluding inner region |
| **8** | `Jdot_accrete` | J̇ₐcc | M L² T⁻³ | Angular momentum loss to accretion |
| **9** | `Jdot_spin` | J̇ₛₚᵢₙ | M L² T⁻³ | Spin torque (not implemented) |
| **11** | `Ldisk` | L | M L² T⁻¹ | Total angular momentum of disk |
| **15** | `Jdisk_excised` | Lₑₓc | M L² T⁻¹ | Angular momentum excluding inner region |

!!! info "Angular Momentum Budget"
    **Conservation**: `dL/dt = J̇ₘᵣₐᵥ + J̇ₐcc + J̇ₛₚᵢₙ`
    
    **Physical Interpretation**:
    
    - `Jdot_grav`: Torque from binary gravitational field
    - `Jdot_accrete`: Angular momentum carried away by accreted material
    - `Jdot_excised`: Excludes regions near binary (r < a)

### Mass & Accretion

| Column | Name | Symbol | Units | Description |
|--------|------|--------|-------|-------------|
| **6** | `Mdot1` | Ṁ₁ | M T⁻¹ | Mass accretion rate onto primary |
| **7** | `Mdot2` | Ṁ₂ | M T⁻¹ | Mass accretion rate onto secondary |
| **10** | `Mdisk` | M_disk | M | Total mass in disk |
| **12** | `Mdot_buffer` | Ṁ_buf | M T⁻¹ | Buffer zone mass flux (not implemented) |
| **14** | `Mdisk_excised` | M_exc | M | Disk mass excluding inner region |
| **21** | `Mdot_inner_boundary` | Ṁ_ᵢₙ | M T⁻¹ | Mass flux through inner boundary |

### Momentum & Forces

| Column | Name | Symbol | Units | Description |
|--------|------|--------|-------|-------------|
| **16** | `Pdot_x_grav` | Ṗₓ,ₘᵣₐᵥ | M L T⁻³ | x-momentum change from gravity |
| **17** | `Pdot_y_grav` | Ṗᵧ,ₘᵣₐᵥ | M L T⁻³ | y-momentum change from gravity |
| **18** | `Pdot_x_accrete` | Ṗₓ,ₐcc | M L T⁻³ | x-momentum loss to accretion |
| **19** | `Pdot_y_accrete` | Ṗᵧ,ₐcc | M L T⁻³ | y-momentum loss to accretion |

!!! note "Momentum Budget"
    **Forces**: Gravitational and accretion forces on disk
    
    **Applications**: Disk migration, orbital evolution feedback

### Disk Structure Analysis

| Column | Name | Symbol | Units | Description |
|--------|------|--------|-------|-------------|
| **2** | `psi_re` | ℜ(ψ) | M | Real part of m=1 density mode |
| **3** | `psi_im` | ℑ(ψ) | M | Imaginary part of m=1 density mode |
| **4** | `ex` | eₓ | - | x-component of eccentricity vector |
| **5** | `ey` | eᵧ | - | y-component of eccentricity vector |

!!! abstract "Mode Analysis"
    **m=1 Density Mode**: `ψ = ∫ ρ e^(iφ) dA`
    
    **Eccentricity Vector**: `e⃗ = (1/M_nominal) ∫ ρ e⃗_orbital dA`
    
    **Applications**: Disk precession, spiral wave tracking, asymmetry detection

### Time Coordinate

| Column | Name | Symbol | Units | Description |
|--------|------|--------|-------|-------------|
| **20** | `time` | t | T | Simulation time in user units |

!!! info "Time Conversion"
    **User Time**: `t_user = t_raw / tunits`
    
    **Orbital Periods**: For `tunits = -1`, one orbit = 1 time unit

---

## Diagnostic Set Configurations

### Pre-defined Sets

=== "All Diagnostics"
    ```cfg
    diagnostic_set = "all"
    ```
    
    **Products**: 0, 1, 2, 3, 4, 5, 6 (all available)
    
    **Timeseries**: 0-21 (complete set)
    
    **Use**: Development, comprehensive analysis

=== "Custom Selection"
    ```cfg
    diagnostic_set = "custom"
    sp = [0, 1, 2, 3]          # Density + velocities + pressure
    ts = [0, 10, 11, 20]       # Torque + mass/momentum + time
    ```
    
    **Products**: User-specified columns
    
    **Timeseries**: User-specified columns
    
    **Use**: Production runs, focused studies

=== "Minimal Output"
    ```cfg
    diagnostic_set = "min"
    ```
    
    **Products**: None defined (empty set)
    
    **Timeseries**: None defined (empty set)
    
    **Use**: Performance testing, minimal I/O

=== "No Diagnostics"
    ```cfg
    diagnostic_set = "none"
    ```
    
    **Products**: None
    
    **Timeseries**: None
    
    **Use**: Fastest execution, checkpoints only

---

## Output File Structure

### HDF5 Products Format

```
product_NNNNNN.h5:
├── __time__           # User time value
├── mass_density/      # 2D array [ni × nj]
├── x_velocity/        # 2D array [ni × nj]
├── y_velocity/        # 2D array [ni × nj]
├── gas_pressure/      # 2D array [ni × nj]
└── ...               # Additional selected fields
```

