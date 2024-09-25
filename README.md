# $\textsf{\textbf{\color{purple}J\color{green}U\color{red}HPC}}$: Julia[up] for HPC  <!-- omit from toc -->


## Introduction: a community project for everyone - including end users

JUHPC is an attempt to convert the numerous efforts at different HPC sites for defining a suitable Julia HPC setup into a community scripting project... *which suits everyone*. The objective is to gather all the experience of the Julia HPC community and transform it in an portable automatic Julia HPC setup, enhanced and maintained jointly by the Julia HPC community. JUHPC can be used stand-alone (by end users) or as part of a recipe for automated software stack generation (by HPC sites) as, e.g., the generation of modules or [uenvs](https://eth-cscs.github.io/uenv/) (used on the ALPS supercomputer at the Swiss National Supercomputing Centre, see [here](https://confluence.cscs.ch/display/KB/UENV+user+environments)).

An important lesson learned by the Julia HPC community for providing Julia at HPC sites is not to preinstall any packages site wide. JUHPC pushes this insight even one step further and does not preinstall Julia either. Instead, Juliaup is leveraged and the installation of Juliaup, Julia and packages is preconfigured for being automatically executed by the end user. Furthermore, for maximal robustness, the preferences are created using the available API calls of the corresponding packages.

Concretely, JUHPC creates an HPC setup for Juliaup, Julia and some HPC key packages (MPI.jl, CUDA.jl, AMDGPU.jl, HDF5.jl, ADIOS2.jl, ...), including
- preferences for HPC key packages that require system libraries;
- a wrapper for Juliaup that will install Juliaup (and latest Julia) automatically in a predefined location (e.g., scratch) when the end user calls `juliaup` the first time;
- an activation script that sets environment variables for Juliaup, Julia and HPC key packages;
- optional execution of a site-specific post installation Julia script, using the project where preferences were set (e.g, to modify preferences or to create an uenv view equivalent to the activation script).

HPC sites can install the HPC setup into a folder in a location accessible to all users (which can also be part, e.g., of a uenv). HPC end users can install the HPC setup into any folder to their liking, accessible from the compute nodes; it is then enough to source the activate script in this folder in order to activate the HPC setup.


## Table of contents <!-- omit from toc -->
- [Introduction: a community project for everyone - including end users](#introduction-a-community-project-for-everyone---including-end-users)
- [Usage](#usage)
  - [1. Export environment variables for the installation of some HPC key packages](#1-export-environment-variables-for-the-installation-of-some-hpc-key-packages)
  - [2. Call JUHPC](#2-call-juhpc)
- [Examples: HPC setup installations on the ALPS supercomputer (CSCS)](#examples-hpc-setup-installations-on-the-alps-supercomputer-cscs)
  - [Example 1: using Cray Programming Environment](#example-1-using-cray-programming-environment)
  - [Example 2: using UENV](#example-2-using-uenv)
  - [Test of example 1](#test-of-example-1)
  - [Test of example 2](#test-of-example-2)
- [Your contributions](#your-contributions)
- [Contributors](#contributors)


## Usage
Installing a HPC setup for Juliaup, Julia and some HPC key packages, requires only two steps:
1. Export environment variables for the installation of some HPC key packages.
2. Call JUHPC.

Details are given in the following two subsections.

### 1. Export environment variables for the installation of some HPC key packages

- CUDA
  - `JUHPC_CUDA_HOME`: Activates HPC setup for CUDA and is used for CUDA.jl runtime discovery (set as `CUDA_HOME` in the activate script).
  - `JUHPC_CUDA_RUNTIME_VERSION`: Used to set CUDA.jl preferences (fixes runtime version enabling pre-compilation on login nodes).

- AMDGPU
  - `JUHPC_ROCM_HOME`: Activates HPC setup for AMDGPU and is used for AMDGPU.jl runtime discovery (set as `ROCM_PATH` in the activate script).

- MPI
  - `JUHPC_MPI_HOME`: Activates HPC setup for MPI and is used to set MPI.jl preferences. Incompatible with `JUHPC_MPI_VENDOR`
  - `JUHPC_MPI_VENDOR`: Activates HPC setup for MPI and is used to set MPI.jl preferences (currently only "cray" is valid, see [here](https://juliaparallel.org/MPI.jl/stable/configuration/#Notes-about-vendor-provided-MPI-backends)). Incompatible with `JUHPC_MPI_HOME`.
  - `JUHPC_MPI_EXEC`: Used to set MPI.jl preferences (exec command definition). Arguments are space separated, e.g., `"srun -C gpu"`.

- HDF5
  - `JUHPC_HDF5_HOME`: Activates HPC setup for HDF5 and is used to set HDF5.jl preferences.

- ADIOS2
  - `JUHPC_ADIOS2_HOME`: Activates HPC setup for ADIOS2 and is used to set ADIOS2.jl preferences.

> [!NOTE]
> The automatically defined preferences suitable for typical HPC needs can be modified with a post install Julia script (see `JUHPC_POST_INSTALL_JL` in next section). Also preferences for other packages could be added this way if needed. Of course, any of these preferences can later be overwritten by local preferences.

### 2. Call JUHPC

The `juhpc` bash script is called as follows:
```bash
juhpc $JUHPC_SETUP_INSTALLDIR $JULIAUP_INSTALLDIR [$JUHPC_POST_INSTALL_JL]
```
I.e., it takes the following arguments:
- `JUHPC_SETUP_INSTALLDIR`: the folder into which the HPC setup is installed, e.g., `"$SCRATCH/../julia/${HOSTNAME%%-*}/juhpc_setup"`.
- `JULIAUP_INSTALLDIR`: the folder into which Juliaup and Julia will automatically be installed the first time the end user calls `juliaup`. *User environment variables should be escaped* in order not to have them expanded during HPC setup installation, but during its usage by the end user, e.g., `"\$SCRATCH/../julia/\$USER/\${HOSTNAME%%-*}/juliaup"`.
- `JUHPC_POST_INSTALL_JL` (optional): site-specific post installation Julia script, using the project where preferences were set (e.g, to modify preferences or to create an uenv view equivalent to the activation script).

> [!NOTE]
> The above examples assume that `$SCRATCH/../julia` is a wipe out protected folder on scratch.

> [!IMPORTANT]
> Separate installation by `HOSTNAME` is required if different hosts with different architectures share file system used for installation (e.g., daint and eiger on ALPS).


## Examples: HPC setup installations on the ALPS supercomputer (CSCS)

In the following you can find two examples of HPC setup installations.

> [!TIP]
> More examples are found in the folder [examples](/examples/).

### Example 1: using Cray Programming Environment

```bash
# Load required modules (including correct CPU and GPU target modules)
module load cray
module switch PrgEnv-cray PrgEnv-gnu
module load cudatoolkit craype-accel-nvidia90
module load cray-hdf5-parallel
module list

# Environment variables for HPC key packages that require system libraries that require system libraries (MPI.jl, CUDA.jl, AMDGPU.jl, HDF5.jl and ADIOS2.jl)
export JUHPC_CUDA_HOME=$CUDA_HOME
export JUHPC_CUDA_RUNTIME_VERSION=$CRAY_CUDATOOLKIT_VERSION
export JUHPC_MPI_VENDOR="cray"
export JUHPC_MPI_EXEC="srun -C gpu"
export JUHPC_HDF5_HOME=$HDF5_DIR

# Call JUHPC
JUHPC_SETUP_INSTALLDIR=$SCRATCH/../julia/${HOSTNAME%%-*}/juhpc_setup
JULIAUP_INSTALLDIR="\$SCRATCH/../julia/\$USER/\${HOSTNAME%%-*}/juliaup"
VERSION="v0.2.0"
wget https://raw.githubusercontent.com/JuliaParallel/JUHPC/$VERSION/juhpc -O /tmp/juhpc
bash -l /tmp/juhpc $JUHPC_SETUP_INSTALLDIR $JULIAUP_INSTALLDIR
```

> [!TIP]
> The corresponding file is found [here](/examples/cscs/alps/gh200/craype_config).

### Example 2: using UENV

```bash
# UENV specific environment variables
export ENV_MOUNT={{ env.mount }}    # export ENV_MOUNT=/user-environment
export ENV_META=$ENV_MOUNT/meta
export ENV_EXTRA=$ENV_META/extra
export ENV_JSON=$ENV_META/env.json

# Environment variables for HPC key packages that require system libraries (MPI.jl, CUDA.jl, AMDGPU.jl, HDF5.jl and ADIOS2.jl)
export JUHPC_CUDA_HOME=$(spack -C $ENV_MOUNT/config location -i cuda)
export JUHPC_CUDA_RUNTIME_VERSION=$(spack --color=never -C $ENV_MOUNT/config find cuda | \
                                    perl -ne 'print $1 if /cuda@([\d.]+)/')
export JUHPC_MPI_HOME=$(spack -C $ENV_MOUNT/config location -i cray-mpich)
export JUHPC_MPI_EXEC="srun -C gpu"
export JUHPC_HDF5_HOME=$(spack -C $ENV_MOUNT/config location -i hdf5)
export JUHPC_ADIOS2_HOME=$(spack -C $ENV_MOUNT/config location -i adios2)

# Call JUHPC
JUHPC_SETUP_INSTALLDIR=$ENV_MOUNT/juhpc_setup
JULIAUP_INSTALLDIR="\$SCRATCH/../julia/\$USER/\${HOSTNAME%%-*}/juliaup"
JUHPC_POST_INSTALL_JL=$ENV_EXTRA/uenv_view.jl
VERSION="v0.2.0"
wget https://raw.githubusercontent.com/JuliaParallel/JUHPC/$VERSION/juhpc -O /tmp/juhpc
bash -l /tmp/juhpc $JUHPC_SETUP_INSTALLDIR $JULIAUP_INSTALLDIR $JUHPC_POST_INSTALL_JL
```

### Test of example 1

```bash
#!/bin/bash

# Variable set in craype_config
JUHPC_SETUP_INSTALLDIR=$SCRATCH/../julia/${HOSTNAME%%-*}/juhpc_setup

# Load required modules (including correct CPU and GPU target modules)
module load cray
module switch PrgEnv-cray PrgEnv-gnu
module load cudatoolkit craype-accel-nvidia90
module load cray-hdf5-parallel
module list

# Activate the HPC setup environment variables
. $JUHPC_SETUP_INSTALLDIR/activate

# Call Juliaup to install Juliaup and latest Julia on scratch
juliaup

# Call Juliaup to see its options
juliaup

# Call Julia Pkg 
julia -e 'using Pkg; Pkg.status()'

# Add CUDA.jl
julia -e 'using Pkg; Pkg.add("CUDA"); using CUDA; CUDA.versioninfo()'

# Add MPI.jl
julia -e 'using Pkg; Pkg.add("MPI"); using MPI; MPI.versioninfo()'

# Add HDF5.jl
julia -e 'using Pkg; Pkg.add("HDF5"); using HDF5; @show HDF5.has_parallel()'

# Test CUDA-aware MPI
cd ~/cudaaware
MPICH_GPU_SUPPORT_ENABLED=1 srun -Acsstaff -C'gpu' -N2 -n2 julia cudaaware.jl
```

> [!TIP]
> The corresponding file is found [here](/examples/cscs/alps/gh200/test_craype_config).


### Test of example 2

```bash
#!/bin/bash

# Start uenv with the view equivalent to the activation script
uenv start --view=julia julia

# Call Juliaup to install Juliaup and latest Julia on scratch
juliaup

# Call Juliaup to see its options
juliaup

# Call Julia Pkg 
julia -e 'using Pkg; Pkg.status()'

# Add CUDA.jl
julia -e 'using Pkg; Pkg.add("CUDA"); using CUDA; CUDA.versioninfo()'

# Add MPI.jl
julia -e 'using Pkg; Pkg.add("MPI"); using MPI; MPI.versioninfo()'

# Add HDF5.jl
julia -e 'using Pkg; Pkg.add("HDF5"); using HDF5; @show HDF5.has_parallel()'

# Test CUDA-aware MPI
cd ~/cudaaware
MPICH_GPU_SUPPORT_ENABLED=1 srun -Acsstaff -C'gpu' -N2 -n2 julia cudaaware.jl
```


## Your contributions

Any contribution is valuable to the Julia HPC community:
- contribute the HPC setup config file for your cluster or supercomputer in the folder [examples](/examples/) as an example for others;
- open a PR with some enhancement or fix;
- open an issue for a bug or an idea for enhancement.


## Contributors

The initial version of JUHPC was contributed by Samuel Omlin, Swiss National Supercomputing Centre, ETH Zurich (@omlins). The following people have provided valuable contributions over the years in the effort of defining a suitable Julia HPC setup (this is based on the list found [here](https://github.com/hlrs-tasc/julia-on-hpc-systems); please add missing people or let us know; the list is ordered alphabetically):
- Carsten Bauer (@carstenbauer)
- Alexander Bills (@abillscmu)
- Johannes Blaschke (@jblaschke)
- Valentin Churavy (@vchuravy)
- Steffen Fürst (@s-fuerst)
- Mosè Giordano (@giordano)
- C. Brenhin Keller (@brenhinkeller)
- Mirek Kratochvíl (@exaexa)
- Pedro Ojeda (@pojeda)
- Samuel Omlin (@omlins)
- Ludovic Räss (@luraess)
- Erik Schnetter (@eschnett)
- Michael Schlottke-Lakemper (@sloede)
- Dinindu Senanayake (@DininduSenanayake)
- Kjartan Thor Wikfeldt (@wikfeldt)
