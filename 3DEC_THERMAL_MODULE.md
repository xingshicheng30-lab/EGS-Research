# Thermal extension for the three-fracture double-well model

The geometry is configured with both `thermal` and `fluid` before either mode
is solved. The driver uses a deliberately staggered THM increment: fluid first
advances to the next measured-data time with mechanics following, then fluid is
made inactive and thermal advances to the same thermal time using the stored
fracture-flow field for convection. Mechanics remains active during the thermal
phase. The next fluid phase therefore sees the thermally updated contact state
and aperture.

## User parameters

Edit the thermal block in the `parameter` FISH function:

| Parameter | Default | SI meaning |
| --- | ---: | --- |
| `use_thermal` | `1` | Enable the thermal solve and temperature boundary. |
| `rock_thermal_conductivity` | `2.5` | Rock conductivity, W/(m K). |
| `rock_specific_heat` | `850` | Rock specific heat, J/(kg K). |
| `rock_thermal_expansion` | `1e-5` | Linear expansion, 1/K. |
| `fluid_specific_heat` | `4180` | Fluid specific heat, J/(kg K). |
| `fluid_thermal_conductivity` | `0.60` | Fluid conductivity, W/(m K). |
| `fluid_rock_heat_transfer` | `10` | Provisional heat-transfer coefficient, W/(m² K). |
| `rock_temperature_initial` | `60` | Initial rock temperature, °C. |
| `fluid_temperature_initial` | `60` | Initial fracture-fluid temperature, °C. |
| `injection_temperature` | `20` | Fixed INJ flowknot temperature, °C. |
| `thermal_timestep` | `0.20` | Fixed implicit thermal timestep, s. |

The numerical values are placeholders for model setup, not calibrated Bedretto
properties. Replace them with laboratory or field values before interpretation.
The default heat-transfer coefficient is deliberately inside the 1, 10, and
100 W/(m² K) range exercised by the official two-block verification example;
that range does not establish the correct field value for this model.

## Boundary conditions and outputs

The injection flowknots receive a fixed temperature. The production well has no
fixed temperature and therefore acts as a thermal observation well. Unspecified
outer rock boundaries are adiabatic. Histories record INJ, PRD, each fracture,
the rock midpoint temperatures, and the hydraulic aperture at each fracture
midpoint.

For a geothermal gradient, replace the uniform solid/fluid initialization with
the `gradient` options of `block gridpoint initialize temperature` and
`block insitu fluid-temperature`. For a fixed-temperature far boundary, add a
`block gridpoint apply temperature` range on the desired outer face.

## Staggered solve and validation limits

`advance_thm_to` advances the fluid and thermal clocks once each to the same
absolute target time. The thermal phase follows the official fracture examples:
the fluid process is inactive, but convection uses the previously established
flow field. Mechanical follower updates are retained in both phases so thermal
strain can change contact state and aperture before the following flow step.

This is a sequential (operator-split) THM scheme, not proof that the complete
model is validated. Before production studies, run the conduction/advection,
fluid/rock exchange, thermal-stress, and timestep-convergence checks described
by the five official examples. In particular, compare 0.05, 0.10, and 0.20 s
thermal timesteps and verify temperature, pressure, aperture, and energy trends.
