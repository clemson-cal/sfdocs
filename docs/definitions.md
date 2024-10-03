# Code Configuration

This section describes the meaning of Sailfish configuration items.

## The `tunits` parameter

 - The Sailfish `State` struct contains a data member called `time` — this is called the “raw” simulation time. 
 - The meaning of this time is easiest to understand through an example. In the code the gravitational constant is always set to G=1. It is also likely that the mass of a gravitating central object, whether a point mass, or a binary, is also M=1. In the case of a gravitating point mass with M=1, gas parcels on a Keplerian circular orbit at radius r=1 have orbital periods in raw simulation time of 2pi. In the case of a binary with total mass M=1 and semi-major axis a=1, the binary orbital period is again 2pi in raw simulation time.
 - By default, the code writes status messages to stdout reporting the time in terms of raw simulation time. By default it also interprets the config items `cpi`, `spi`, and `tsi` as intervals of raw simulation time. However, depending on the problem setup, it can be much more convenient for user interaction (input and output) to use a different time coordinate. For this reason there is another variable called the “user time”, `t = state.time / tunits`, where `tunits` is a config item with default value 1.0. If you supplied on the command line `tunits=6.283185`, then the status message would indicate `t=1` when a gas parcel had orbited once around a central M=1 point mass, or when an M=1 and a=1 binary had completed a single orbit. Similarly the intervals for checkpoints, products, and time series samples are also supplied as user times. Some notes on the use of this feature:

* Since 2pi is a common choice for `tunits`, the code interprets a negative value as being a multiplier of 2pi. Setting `tunits=6.283185` is thus equivalent to setting `tunits=-1`
* The products files contain a root-level HDF5 key called `__time__` — this is a user time
* The time series files output a column called “time” — this column is also the user time [for consistency maybe we should change this column name from time to t]
* The checkpoint file contains a root-level HDF5 group called `state`, under which is nested a variable called `time`. That time is the raw simulation time, reflecting the numerical value of `state.time` 

## The `central_object` parameter

There are three different point mass configurations to choose in the `central_object` parameter:

1. “single” - where a single point mass formed at the center of the grid to simulate a single black hole accreting from a circumbinary disk: if it’s around a single point mass then it’s not a circumbinary disk
2. “binary” - where two point masses are formed with a fixed separation of a=1.0 in code units. The relative masses of the black holes can be altered by changing the `mass_ratio` parameter, secondary mass divided by primary mass, to a value between 0.0-1.0
3. “merger” - similar to the “binary” `central_object` , however the black holes have a decreasing separation over time that follows the inspiral of binary black holes due to gravitational wave emission. Time is in units of decoupling timescales and binary separation is in units of decoupling separation. If `tstart=-1e4` then $$
a(t) = r_{dec} \cdot \left( \frac{-t}{t_{dec}} \right)^{\frac{1}{4}}
$$ for a_0 = 10. For t < 0, the binary will inspiral but when t=0 the merger occurs and a single point mass forms to replace the binary

## The `beta` parameter

The `beta` parameter in the `viscous_stress_tensor` function controls the relationship between bulk and dynamic viscosity based on the Bird-Stewart-Lightfoot stress tensor formulation ⤵

    τij = μ (∂xj / ∂vi + ∂xi / ∂vj + (β - 2./3) δij ⋅ ∇⋅v)

It represents the ratio β = ζ / μ, where  ζ  is the bulk viscosity and μ is the dynamic viscosity. In different setups, the value of `beta` determines how compressible stresses are handled.

1. For a disk setup, such as a steady or KITP, `beta` is set to 0.00, which minimizes bulk viscosity.
2. For a spreading ring setup, `beta` is set to 5/3, reflecting significant bulk viscosity.

