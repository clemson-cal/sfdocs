##Setups

```cpp title="KITP Setup" linenums="1"
case Setup::kitp: {
    double at;
    switch (central_object) 
    {
    case CentralObject::single:
    case CentralObject::binary:
        at = 1.0;
        break;
    case CentralObject::merger:
        if (t < 0.0) {
            auto q = stars.masses[1].mass / stars.masses[0].mass;
            auto a0 = orbit_size_at_unit_time(q);
            at = a0 * pow(-t / t0, 0.25);
        } else {
            at = 1.0;
        }
    }
    double sigma = kitp_sigma(r, at);
    double vr = kitp_vr(r, phi, at);
    double vp = kitp_vphi(mach, r, at);
    double cs2 = -stars.potential(x) / mach / mach;
    double p = sigma * cs2;
    double vx = vr * cos(phi) - vp * sin(phi);
    double vy = vr * sin(phi) + vp * cos(phi);
    return vec(sigma, vx, vy, p);
    }
    default: return {};
    }
}
```
