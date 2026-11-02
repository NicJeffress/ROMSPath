# ROMSPath (fork with fixes for periodic and curvilinear grids)

This is a fork of [imcslatte/ROMSPath](https://github.com/imcslatte/ROMSPath) at
upstream commit `6072ee9` (master, 2025-12-10). It adds periodic boundaries and
fixes found while tracking particles on periodic ROMS grids, one commit each, so
each can be offered upstream as a pull request. Everything else is upstream
ROMSPath, unchanged.

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

Each commit message explains the bug and the change.

**Check.** A periodic channel with uniform flow that speeds up from record to
record has an exact answer. After 2 h, ROMSPath with only the first commit is
48 m off (40 m on a grid rotated by 30 degrees); with all commits it is exact to
the output precision (0.0000 m on both grids).

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
