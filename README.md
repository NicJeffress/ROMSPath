# ROMSPath (fork with fixes for periodic and curvilinear grids)

This is a fork of [imcslatte/ROMSPath](https://github.com/imcslatte/ROMSPath) at
upstream commit `6072ee9` (master, 2025-12-10). I use ROMSPath to track
particles (for example, water residence time in a lagoon) in idealized coral-reef
simulations with COAWST (ROMS coupled to SWAN), which run on periodic,
sometimes rotated, grids with waves. Everything not listed here is upstream
ROMSPath, unchanged.

## What I changed

- **Added periodic boundaries and ROMS's Stokes drift names.** Idealized reef
  domains repeat alongshore, so particles must leave one side and come back on
  the other; ROMSPath had no periodic option. It also now reads the 3-D Stokes
  drift under ROMS's own names (`u_stokes`, `v_stokes`), so it uses COAWST's
  wave-driven transport directly.
- **Fixed four timing bugs in the particle clock.** From the third step on, the
  flow fields were one record behind the particles; the first step placed its
  records one interval early; and the Runge-Kutta (RK4) step took all four
  stages from the velocities at the start of the step, with the last stage
  half a step short. Each moved particles with the wrong flow whenever the
  flow changes in time.
- **Fixed two grid bugs.** On Cartesian rotated or curvilinear grids, release
  positions were converted to grid cells as if grid lines ran along x and y,
  and the periodic wrap used a period one cell short, so every crossing moved
  a particle by one cell.
- **Made it faster and fixed a smaller bug.** Per-particle CPU timers now run only
  when timing output is requested (29-43 % faster, same results), and two
  end weights of the vertical-turbulence smoothing spline are now set.

**How I checked it.** A periodic channel whose uniform flow speeds up from
record to record has an exact answer. After 2 h, the fork with only the periodic
boundaries added is 48 m off (40 m on a grid rotated by 30 degrees); with all
the fixes it is exact to the output precision (0.0000 m on both grids). The test
is [`tests/romspath/periodic-channel`](https://github.com/NicJeffress/coawst-cloud/tree/main/tests/romspath/periodic-channel)
in coawst-cloud, which also builds this fork into a container image.

Each change is its own commit, with a message that explains the bug and the
fix, so each can be offered upstream as a pull request:

| Commit | What it changes |
|---|---|
| Periodic boundaries, ROMS Stokes names, gfortran date format | namelist switches `PeriodicBoundary_X` / `PeriodicBoundary_Y` (group `hydroparam`) with a wrap across the seam; 3-D Stokes drift read as `u_stokes` / `v_stokes`; makefile `-DSTOKES` and debug flags `-g -O0` |
| Periodic wrap by the ROMS period | wrap by Lm = xi_rho - 2 (was xi_rho - 3, one cell lost per crossing), periodic interpolation stencils, no open boundary in a periodic direction |
| Position to grid index on Cartesian curvilinear grids | `spherical = 0`: particles placed correctly on rotated or curvilinear grids |
| Record counter after the initial reads | the hydro fields were one record behind the particle clock from the third external step on |
| RK4: fourth stage at X + k3 dt | was X + k3 dt/2 |
| VTurb: set the smoothing-spline end weights | `dy(1)`, `dy(np)` were never set |
| Per-particle CPU timers only with WriteModelTiming | 29-43 % faster, same results |
| First external step: records at their own times | records were placed one interval early in the first step |
| RK4: evaluate the stages at their own times | all four stages used the velocities at the step start |

**Building.** As upstream (`make` with gfortran and NetCDF-Fortran). The makefile
keeps the debug flags `-g -O0`; for production runs use optimisation, e.g.
`FFLAGS = -O3 -march=native -ffree-line-length-none -cpp $(CPPFLAGS) -I$(NETCDF_INCDIR)`
(about 3-4 times faster).

**Licence.** Upstream ROMSPath has no licence file. Until it has one, ROMSPath
remains the work of its authors (Hunter et al.); this fork carries their code
only so that the changes can be reviewed and offered back upstream.

---

# ROMSPath
Offline particle tracking (OPT) is a widely used tool for the analysis of data in oceanographic research. Given the output of a hydrodynamic model, OPT can provide answers to a wide variety of research questions involving fluid kinematics, zooplankton transport, the dispersion of pollutants, and the fate of chemical tracers, among others. In this paper, we introduce ROMSPath, an OPT model designed to complement the Regional Ocean Modelling System (ROMS). Based on the Lagrangian TRANSport (LTRANS) model (North et al., 2008), ROMSPath is written in Fortran 90 and provides advancements in functionality and efficiency compared to LTRANS.  First, ROMSPath now calculates particle trajectories using the ROMS native grid, which provides advantages in interpolation, masking, and boundary interaction, while improving accuracy. Second, ROMSPath enables simulated particles to pass between nested ROMS grids, which are an increasingly popular tool to simulate the ocean over multiple scales.  Third, the ROMSPath vertical turbulence module enables the turbulent (diffusion) time step and advection time step to be specified separately, adding flexibility and improving computational efficiency.  Lastly, ROMSPath includes new infrastructure enabling input of auxiliary parameters for added functionality. In particular, Stokes drift can be input and added to particle advection. Here we describe the details of these updates and improvements. 
