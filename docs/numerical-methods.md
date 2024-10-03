# Numerical Methods

## Supported

### Riemann HLLE Solver

The HLLE (Harten, Lax, van Leer, Einfeldt) solver is an approximate method for solving the Riemann problem at cell interfaces in the Euler equations. It tackles the initial value problem, which models the evolution of a discontinuous state in fluid dynamics. As a two-wave approach, the HLLE solver uses the left and right eigenvectors of the Jacobian matrix to compute fluxes efficiently. Its simplicity and versatility make it a popular choice in computational fluid dynamics, offering reliable accuracy across a broad range of flow conditions.

### Piecewise Linear Method (PLM)

The Piecewise Linear Method (PLM) is a numerical technique used to solve partial differential equations (PDEs) by approximating the solution as a piecewise linear function over a grid of cells. This method is particularly useful for problems with smooth solutions and is commonly used in computational fluid dynamics (CFD) and finite element methods (FEM).

The PLM method is combined with the minmod limiter (TVD) in the code. This combination gives us a second-order accurate scheme for solving the fluxes in the Euler equations.

## WIP

### Fifth order WENO (Weighted Essentially Non-Oscillatory)

The Weighted Essentially Non-Oscillatory (WENO) scheme is a high-order numerical method used to solve partial differential equations (PDEs) with a high degree of accuracy while maintaining stability and avoiding the formation of spurious oscillations. The WENO scheme is a non-linear reconstruction method that combines the advantages of several lower-order schemes to achieve higher-order accuracy.
