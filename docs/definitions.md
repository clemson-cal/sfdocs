# Code Configuration

This section describes the meaning of Sailfish configuration parameters.

---

## The `tunits` parameter

!!! info "Time Coordinate System"
    The `State` struct contains a `time` member representing **"raw" simulation time**. With G=1 and M=1, gas parcels at r=1 have orbital periods of ==2π== in raw time.

For user convenience, there's a **"user time"** `t = state.time / tunits` where `tunits` has default value `1.0`. Setting `tunits=6.283185` means status messages show `t=1` after one orbit.

!!! tip "Key Features"
    * **Negative values**: `tunits` values are multipliers of 2π (e.g., `tunits=-1` equals `tunits=6.283185`)
    * **Product files**: Store user time in HDF5 key `__time__`
    * **Time series**: Output user time in "time" column
    * **Checkpoints**: Store raw simulation time in `state/time`

---

## The `central_object` parameter

!!! note "Point Mass Configurations"
    Three configurations control the gravitational setup:

=== "Single"
    **`"single"`** - Single point mass at grid center
    
    Perfect for studying accretion disks around isolated objects.

=== "Binary" 
    **`"binary"`** - Two point masses with separation `a` (default 1.0, configurable)
    
    * Mass ratio set by `mass_ratio` parameter (secondary/primary, range 0.0-1.0)
    * Ideal for circumbinary disk simulations

=== "Merger"
    **`"merger"`** - Like binary, but separation decreases following GW inspiral
    
    * For t < 0: `a(t) = a0 * pow(-t, 0.25)` where `a0 = orbit_size_at_unit_time(q)`
    * At t=0: Merger occurs forming single point mass

---

## The `beta` parameter

!!! warning "Viscosity Control"
    Controls the bulk vs. dynamic viscosity ratio in the stress tensor formulation.

The stress tensor is calculated as:

```cpp title="Viscous Stress Tensor"
τij = μ (∂xj/∂vi + ∂xi/∂vj + (β - 2/3) δij ⋅ ∇⋅v)
```

where **β = ζ/μ** (bulk/dynamic viscosity ratio).

!!! example "Typical Values"
    
    | Setup Type | `beta` Value | Purpose |
    |------------|--------------|---------|
    | **Disk setups** (steady, KITP) | `0.0` | Minimal bulk viscosity |
    | **Spreading ring** | `5/3` | Significant bulk viscosity |

!!! success "Quick Reference"
    * Use `tunits=-1` for orbital time units
    * Set `central_object=binary` for circumbinary disks  
    * Choose `beta=0` for most disk simulations

