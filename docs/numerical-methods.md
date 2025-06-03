# Numerical Methods

!!! abstract "Overview"
    Sailfish employs state-of-the-art numerical methods for solving the compressible Euler equations with viscous and gravitational source terms.

---

## 🧮 Supported Methods

### Riemann HLLE Solver

!!! note "HLLE Method"
    The **Harten-Lax-van Leer-Einfeldt** solver approximates the Riemann problem using a two-wave model.

**Mathematical Foundation:**

The HLLE flux is computed as:

$$F_{HLLE} = \frac{S_R F_L - S_L F_R + S_L S_R (U_R - U_L)}{S_R - S_L}$$

where:

- $S_L, S_R$ are the left and right wave speeds
- $F_L, F_R$ are the left and right state fluxes  
- $U_L, U_R$ are the left and right conserved states

!!! tip "Wave Speed Estimation"
    ```cpp
    auto al = outer_wavespeeds(pl, nhat);  // [u-c, u+c] left
    auto ar = outer_wavespeeds(pr, nhat);  // [u-c, u+c] right
    auto am = min2(al[0], ar[0]);          // S_L
    auto ap = max2(al[1], ar[1]);          // S_R
    ```


---

### Piecewise Linear Method (PLM)

!!! info "High-Order Reconstruction"
    PLM achieves **second-order spatial accuracy** by reconstructing linear profiles within each cell.

**Minmod Slope Limiter:**

The slope in each cell is computed using the minmod function:

$$\Delta_i = \text{minmod}\left(\theta(\phi_i - \phi_{i-1}), \frac{\phi_{i+1} - \phi_{i-1}}{2}, \theta(\phi_{i+1} - \phi_i)\right)$$

where $\theta $ controls the limiter aggressiveness:


**Implementation:**
```cpp
return 0.25 * fabs(sign(a) + sign(b)) * (sign(a) + sign(c)) * minabs(a, b, c);
```

!!! success "TVD Property"
    The minmod limiter ensures **Total Variation Diminishing** (TVD) behavior, preventing spurious oscillations.

---

### Runge-Kutta Time Integration

!!! warning "Temporal Accuracy"
    Choose the appropriate RK order based on your accuracy requirements and computational budget.

=== "RK1 (Euler)"
    **First-order explicit method:**
    $$U^{n+1} = U^n + \Delta t \cdot L(U^n)$$
    

=== "RK2 (Midpoint)"
    **Second-order method:**
    $$\begin{align}
    U^* &= U^n + \Delta t \cdot L(U^n) \\
    U^{n+1} &= \frac{1}{2}(U^n + U^* + \Delta t \cdot L(U^*))
    \end{align}$$
    

=== "RK3 (SSPRK3)"
    **Third-order Strong Stability Preserving:**
    $$\begin{align}
    U^{(1)} &= U^n + \Delta t \cdot L(U^n) \\
    U^{(2)} &= \frac{3}{4}U^n + \frac{1}{4}U^{(1)} + \frac{1}{4}\Delta t \cdot L(U^{(1)}) \\
    U^{n+1} &= \frac{1}{3}U^n + \frac{2}{3}U^{(2)} + \frac{2}{3}\Delta t \cdot L(U^{(2)})
    \end{align}$$
    

## 🔬 Advanced Features

!!! tip "Adaptive Methods"
    
    **CFL Condition:**
    $$\Delta t = C_{CFL} \cdot \min\left(\frac{\Delta x}{|u| + c}\right)$$
    
    where $C_{CFL} \leq 0.4$ ensures numerical stability.

!!! note "Viscous Terms"
    Viscous fluxes are computed using central differences of the PLM-reconstructed gradients:
    
    $$\tau_{ij} = \mu \left(\frac{\partial v_i}{\partial x_j} + \frac{\partial v_j}{\partial x_i} + \left(\beta - \frac{2}{3}\right)\delta_{ij}\nabla \cdot \vec{v}\right)$$

---

