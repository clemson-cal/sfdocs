# Presets

## Ring

```cfg title="ring.cfg"
fold = 1
rk = 1
beta = 5.0/3.0
nu = 0.01
ell = 1.0
theta = 1.5
tstart = 0.01
tfinal = 0.01
tunits = 8.3333333333 # this is t_visc = 1 / (12 nu / r0^2); tau = t / t_visc
cfl = 0.3
cpi = 0
spi = 0.05
mesh_omega = 0
dx = 0.01
domain = 0.5 1.5
sp = 0,1,2,3,4
diagnostic_set = all
outdir = .
setup = ring
thermodynamics = cold
central_object = single
```

## Steady Disk

`Change central_object to binary to run a binary system.`

### central_object = single

```cfg title="steady.cfg"
fold = 1
rk = 2
beta = 0
alpha = 0
nu = 0.01
mach = 10
mass_ratio = 0.2
ell = 0.0
theta = 1.5
tstart = -1
tfinal = 0
tunits = 1
cfl = 0.3
cpi = 0
spi = 0.1
tsi = 0
mesh_omega = 0
sink_size = 0.2
rsoft = 1.0
dx = 0.01
domain = 1.25 10
sp = 0,1,2,3,4
diagnostic_set = custom
ts =
outdir = .
setup = steady
mesh = polar
thermodynamics = local_iso
central_object = single
```

## KITP

### central_object = binary

```cfg title="santa_barbara.cfg"
fold = 1
rk = 1
beta = 0.00
nu = 0.001
theta = 1.5
tstart = 0.0
tfinal = 1.0
tunits = -1.0 # this is one orbit, 2 * pi units of raw simulation time
cfl = 0.3
cpi = 1.0
spi = 1.0
mesh_omega = 0
sink_size = 0.2
sink_rate = 10
rsoft = 1.0
dx = 0.01
domain = 1.0 10.0
sp = 0,1,2,3,4
diagnostic_set = all
outdir = .
ell = 0.0
setup = kitp
thermodynamics = local_iso
central_object = binary
```

## Merger

### central_object = merger

```cfg title="inspiral.cfg"
fold = 1
rk = 1
beta = 0
alpha = 0
nu = 0.001
mach = 10
mass_ratio = 0.1
ell = 0.0
theta = 1.5
tstart = -100
tfinal = 0
tunits = 1
cfl = 0.3
cpi = 0
spi = 0.1
tsi = 0
mesh_omega = 0
sink_size = 1.0
sink_rate = 10
rsoft = 1.0
dx = 0.01
domain = 0.2, 10
diagnostic_set = all
sp = 0,1,2,3,4
ts =
outdir = .
setup = kitp
mesh = polar
thermodynamics = local_iso
central_object = merger
bc = outflow
```