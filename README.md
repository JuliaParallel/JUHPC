# JUHPC
Create an HPC setup for juliaup, julia and some HPC key packages (MPI.jl, CUDA.jl, HDF5.jl, ADIOS2.jl, ...), including
- preferences for HPC key packages that require system libraries;
- a wrapper for juliaup that installs juliaup (and latest julia) in an appropriate location (e.g., scratch) if it is not already installed;
- an activation script that sets environment variables for juliaup, julia and HPC key packages;
- optionally executing a site-specific post installation julia script, using the project where preferences were set (e.g, to modify preferences or to create an uenv view equivalent to the activation script).
