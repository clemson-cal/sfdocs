## Code Examples

This page provides code snippets from key functions in `sailfish` to illustrate how different components work.

---

## Initial Conditions

### KITP Setup Initial Conditions

```cpp title="KITP Setup Initial Conditions" linenums="1"
// Relevant part of initial_model_t::initial_primitive
// for the KITP setup
case Setup::kitp: {
    // binary_orbit_size(t) is a method of initial_model_t which calculates 
    // the appropriate orbital separation 'at' based on the central_object 
    // type (single, binary, merger) and time 't'.
    double at = binary_orbit_size(t); 
    double sigma = kitp_sigma(r, at);
    double vr = kitp_vr(r, phi, at);
    double vp = kitp_vphi(mach, r, at);
    double cs2 = -stars.potential(x) / mach / mach; // x is the field_point
    double p = sigma * cs2 / ADIABATIC_INDEX; // Corrected pressure calculation
    double vx = vr * cos(phi) - vp * sin(phi);
    double vy = vr * sin(phi) + vp * cos(phi);
    return vec(sigma, vx, vy, p);
}
// Note: The functions kitp_sigma, kitp_vr, kitp_vphi, and stars.potential 
// are defined elsewhere and used here to construct the primitive state.
```

### Ring Setup Initial Conditions

```cpp title="Pringle Ring Setup" linenums="1"
// Initial conditions for viscous spreading ring (Pringle solution)
case Setup::ring: {
    double sigma = pringle_ring_sigma(r, t, 1.0, 1.0, nu) + ZERO_SIGMA;
    double vr = pringle_ring_vr(r, t, 1.0, 1.0, nu);
    double vp = pringle_ring_vphi(r);
    double p = ZERO_PRESSURE;
    double vx = vr * cos(phi) - vp * sin(phi);
    double vy = vr * sin(phi) + vp * cos(phi);
    return vec(sigma, vx, vy, p);
}
```

### Steady Disk Setup

```cpp title="Steady Disk Initial Conditions" linenums="1"
// Initial conditions for steady-state viscous disk
case Setup::steady: {
    double nu = visc_model.kinematic_viscosity(1.0, r);
    double at = binary_orbit_size(t);
    double vr = steady_disk_vr(r, ell, nu, at);
    double vp = steady_disk_vphi(mach, r);
    double sigma = steady_disk_sigma(r, ell, nu, at);
    double cs2 = -stars.potential(x) / mach / mach;
    double p = sigma * cs2 / ADIABATIC_INDEX;
    double vx = vr * cos(phi) - vp * sin(phi);
    double vy = vr * sin(phi) + vp * cos(phi);
    return vec(sigma, vx, vy, p);
}
```

---

## Numerical Methods

### PLM Minmod Slope Limiter

```cpp title="PLM Minmod Function" linenums="1"
// Piecewise Linear Method with minmod slope limiter
static HD double plm_minmod(
    double yl,     // Left cell value
    double yc,     // Center cell value  
    double yr,     // Right cell value
    double plm_theta)  // Limiter parameter [1.0-2.0]
{
    double a = (yc - yl) * plm_theta;
    double b = (yr - yl) * 0.5;
    double c = (yr - yc) * plm_theta;
    return 0.25 * fabs(sign(a) + sign(b)) * (sign(a) + sign(c)) * minabs(a, b, c);
}
```

### HLLE Riemann Solver

```cpp title="HLLE Riemann Solver" linenums="1"
// Approximate Riemann solver using HLLE (Harten-Lax-van Leer-Einfeldt) method
static HD auto riemann_hlle(
    prim_t pl, prim_t pr,    // Left and right primitive states
    cons_t ul, cons_t ur,    // Left and right conserved states
    dvec_t<2> nhat,          // Interface normal vector
    double v_face) -> cons_t // Face velocity for moving mesh
{
    auto fl = dot(prim_and_cons_to_flux(pl, ul), nhat);
    auto fr = dot(prim_and_cons_to_flux(pr, ur), nhat);
    auto al = outer_wavespeeds(pl, nhat);
    auto ar = outer_wavespeeds(pr, nhat);
    auto am = min2(al[0], ar[0]);  // Minimum wave speed
    auto ap = max2(al[1], ar[1]);  // Maximum wave speed
    
    if (v_face < am) {
        return fl - ul * v_face;
    }
    if (v_face > ap) {
        return fr - ur * v_face;
    }
    auto u_hll = (ur * ap - ul * am + (fl - fr)) / (ap - am);
    auto f_hll = (fl * ap - fr * am - (ul - ur) * ap * am) / (ap - am);
    return f_hll - u_hll * v_face;
}
```

---

## Physical Models

### Viscous Stress Tensor

```cpp title="Viscous Stress Tensor Calculation" linenums="1"
// Compute viscous stress tensor from velocity gradients
HD auto viscous_stress_tensor(
    vec_t<dvec_t<2>, 2> gradv,  // Velocity gradient tensor
    double mu,                   // Dynamic viscosity
    double beta) -> vec_t<dvec_t<2>, 2>  // Bulk viscosity parameter
{
    auto tau = vec_t<dvec_t<2>, 2>{};
    auto div = gradv[0][0] + gradv[1][1];  // Velocity divergence

    for (uint i = 0; i < 2; ++i) {
        for (uint j = 0; j < 2; ++j) {
            tau[i][j] = mu * (gradv[i][j] + gradv[j][i] + (beta - 2./3) * (i == j) * div);
        }
    }
    return tau;
}
```

### Point Mass Gravitational Potential

```cpp title="Point Mass Potential" linenums="1"
// Calculate gravitational potential from a point mass
HD auto point_mass_potential(point_mass_t mass, dvec_t<2> field_point) -> double
{
    auto dx = field_point - mass.x;      // Distance vector
    auto r2 = dot(dx, dx);               // Squared distance
    auto e2 = mass.soft * mass.soft;     // Softening parameter squared
    return -mass.mass / sqrt(r2 + e2);   // Softened potential
}
```

---

## State Conversion Functions

### Primitive to Conserved Variables

```cpp title="Primitive to Conserved Conversion" linenums="1"
// Convert primitive variables [ρ, vx, vy, p] to conserved variables [ρ, ρvx, ρvy, E]
static HD auto prim_to_cons(prim_t p) -> cons_t
{
    double rho = p[0];  // Density
    double ux = p[1];   // x-velocity
    double uy = p[2];   // y-velocity  
    double pre = p[3];  // Pressure
    
    double px = rho * ux;  // x-momentum
    double py = rho * uy;  // y-momentum
    double E = 0.5 * (ux * ux + uy * uy) * rho + pre / (ADIABATIC_INDEX - 1.0);  // Total energy
    
    return vec(rho, px, py, E);
}
```

### Conserved to Primitive Variables

```cpp title="Conserved to Primitive Conversion" linenums="1"
// Convert conserved variables [ρ, ρvx, ρvy, E] to primitive variables [ρ, vx, vy, p]
static HD auto cons_to_prim(cons_t u) -> optional_t<prim_t>
{
    double rho = u[0];                    // Density
    double px = u[1];                     // x-momentum
    double py = u[2];                     // y-momentum
    double E = u[3];                      // Total energy
    
    double ux = px / rho;                 // x-velocity
    double uy = py / rho;                 // y-velocity
    double p_squared = px * px + py * py;
    double pre = (E - 0.5 * p_squared / rho) * (ADIABATIC_INDEX - 1.0);  // Pressure
    
    if (pre < 0.0) {
        pre = ZERO_PRESSURE;  // Apply pressure floor
    }
    if (rho <= 0.0) {
        return none<prim_t>();  // Invalid state
    }
    return some(vec(rho, ux, uy, pre));
}
```

---

## Configuration and Orbital Mechanics

### Binary Orbital Elements

```cpp title="Orbital State Calculation" linenums="1"
// Calculate binary orbital state from eccentric anomaly
auto orbital_state_from_eccentric_anomaly(double eccentric_anomaly) const -> vec_t<point_mass_t, 2>
{
    auto w = orbital_frequency();
    auto m1 = m / (1.0 + q);    // Primary mass
    auto m2 = m - m1;           // Secondary mass
    auto ck = cos(eccentric_anomaly);
    auto sk = sin(eccentric_anomaly);
    
    // Positions relative to center of mass
    auto x2 = +a / (1.0 + q) * (e - ck);
    auto y2 = -a / (1.0 + q) * (sk) * sqrt(1.0 - e * e);
    auto x1 = -x2 * q;
    auto y1 = -y2 * q;
    
    // Velocities
    auto vx2 = +a / (1.0 + q) * w / (1.0 - e * ck) * sk;
    auto vy2 = -a / (1.0 + q) * w / (1.0 - e * ck) * ck * sqrt(1.0 - e * e);
    auto vx1 = -vx2 * q;
    auto vy1 = -vy2 * q;
    
    auto p1 = point_mass_t{m1, 1.0, 1.0, 1.0, {x1, y1}, {vx1, vy1}};
    auto p2 = point_mass_t{m2, 1.0, 1.0, 1.0, {x2, y2}, {vx2, vy2}};
    return vec(p1, p2);
}
```

!!! tip "Using These Examples"
    These code snippets show the core computational kernels of Sailfish. The functions are designed to work together: initial conditions set up the primitive state, which gets converted to conserved variables, advanced using numerical methods like PLM and HLLE, and updated with physical effects like viscosity and gravity.

!!! note "Function Signatures" 
    - `HD` macro indicates functions that can run on both CPU and GPU (via CUDA)
    - `vec_t<T, N>` represents N-dimensional vectors of type T
    - `optional_t<T>` represents values that may or may not be present (for error handling)
