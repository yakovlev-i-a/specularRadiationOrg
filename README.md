# specularRadiation — fvDOM specular BC for OpenFOAM.org v14

Specular-reflection boundary condition for the `fvDOM` radiation model,
ported from **OpenFOAM-v2306 (openfoam.com)** to **OpenFOAM.org-v14**.

It provides a mirror-like (specular) reflection of radiation intensity on
`wedge` and `symmetryPlane` patches, which is required to run 2D / axisymmetric
`fvDOM` cases correctly: without it, radiative flux leaks through the symmetry
plane (the classic `zeroGradient` behaviour).

## Reference

> Kumar, P., & Eswaran, V. (2013).
> *A methodology to solve 2D and axisymmetric radiative transfer problems using
> a general 3D solver.* Journal of Heat Transfer, 135(12).
> DOI: [10.1115/1.4024674](https://doi.org/10.1115/1.4024674)

## Requirements

- OpenFOAM.org **v14** (built against `FOAM_USER_LIBBIN`)
- `g++` / `wmake`

## Build

```bash
cd specularRadiatioOrg
./Allwmake
# or
wmake libso
```

This produces `libspecularRadiation.so` in `$FOAM_USER_LIBBIN`.

## Usage

1. Load the library in `system/controlDict`:

```cpp
libs            ("libspecularRadiation.so");
```

2. On every `symmetryPlane` / `wedge` patch that is not a real reflecting wall,
   set the radiation intensity boundary to specular:

```cpp
symmetryPlanePatch
{
    type            specularRadiation;
    // interpolate     true;   // optional: bi/linear ray-intensity interpolation
}
```

Set `interpolate true` to interpolate the reflected intensity from the
neighbouring rays (Kumar & Eswaran p.2); `false` (default) uses the closest
ray (Eq. 4).

> The condition only accepts `wedge` and `symmetryPlane` patches. Applying it
> to a physical wall throws a `FatalError`.

## Notes / deviations from v2306

- **Namespace**: `Foam` (the class is
  `Foam::specularRadiationMixedFvPatchScalarField`), matching the org-14
  radiation BC conventions. `fvDOM` is referenced as
  `Foam::radiationModels::fvDOM`.
- **`lerp()`**: OpenFOAM.org has no `lerp()` primitive; a local scalar-field
  helper is used.
- **Reflected intensity source**: the non-interpolated branch reads the
  reflected intensity from `IRayLambda(...).boundaryField()[patchi]`
  (boundary value), not `.internalField()` as in the v2306 sources.
- **`write()`**: persists only `type` (+ `patchType`), `interpolate` and
  `value`; the runtime-computed `refValue/refGradient/valueFraction` are not
  written back.

## License

GPL v3 (same as OpenFOAM). Original source: Copyright (C) 2023 OpenCFD Ltd.
