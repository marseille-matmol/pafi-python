         _______      _______      _______     _________
        (  ____ )    (  ___  )    (  ____ \    \__   __/
        | (    )|    | (   ) |    | (    \/       ) (
        | (____)|    | (___) |    | (__           | |
        |  _____)    |  ___  |    |  __)          | |
        | (          | (   ) |    | (             | |
        | )          | )   ( |    | )          ___) (___
        |/           |/     \|    |/           \_______/
        Projected    Average      Force        Integrator


v0.9 :copyright: TD Swinburne and M-C Marinica 2020 MIT License

swinburne at cinam.univ-mrs.fr

Beta version of code used in [this paper](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.120.135503)
> *Unsupervised Calculation of Free Energy Barriers in Large Crystalline Systems*   
> T.D. Swinburne and M.-C. Marinica, Physical Review Letters 120 (13), 135503, 2018

Please cite the above when publishing results using `PAFI`

This repository includes the [RapidXML](http://http://rapidxml.sourceforge.net) library for input parsing

# Installation


## Compile `LAMMPS` with `USER-MISC` package
1. `PAFI` is in the process of integration into `LAMMPS`. In the meantime it is best to download or clone [this](https://github.com/tomswinburne/lammps/) fork or simply execute
```bash
git clone https://github.com/tomswinburne/lammps.git
```
You could alternatively copy the following files into your LAMMPS distribution:
```bash
  src/library.cpp # overwrite (adding new functions)
  src/library.h # overwrite (adding new functions)
  src/USER-MISC/fix_pafi.cpp # new file
  src/USER-MISC/fix_pafi.h # new file
```
but this has no documentation so is not recommended

2. Install `USER-MISC` and any packages you desire (e.g. replica for `NEB`)
```bash
cd src
make yes-user-misc
make yes-replica # for NEB calculation
make yes-package_name # (i.e. manybody for EAM potentials etc)
```
3. In the appropriate Makefile add `-std=c++11` to `CCFLAGS` and `LINKFLAGS` and
add `-DLAMMPS_EXCEPTIONS` to `LMP_INC` to allow `PAFI` to read `LAMMPS` error messages.
This is very useful when running your own simulations. For `src/MAKE/Makefile.mpi` this reads
 ```make
CCFLAGS =	-g -O3 -std=c++11
LINKFLAGS =	-g -O3 -std=c++11
LMP_INC =	-DLAMMPS_GZIP -DLAMMPS_MEMALIGN=64  -DLAMMPS_EXCEPTIONS
```

4. Compile as a static library (and optionally binary for initial NEB calculation) Consult [LAMMPS documentation](http://lammps.sandia.gov/doc/Section_start.html) for details
```bash
   make mpi mode=lib # liblammps_mpi.a library for pafi
   make mpi # lmp_mpi binary for running initial NEB calculation if desired
```

4. Copy library to your local lib/ and headers to local include/, at e.g. ${HOME}/.local
```bash
  export PREFIX=${HOME}/.local # example value
  cp liblammps_mpi.a ${PREFIX}/lib
  mkdir ${PREFIX}/include/lammps
  cp *.h ${PREFIX}/include/lammps/
```

## Compile PAFI
0. If required, download and install cmake from https://cmake.org/download/

1. Specify environment variables in CMakeLists.txt:
```
   set(PREFIX your/PREFIX/value) # cmake doesn't see environment variables
   set(CMAKE_CXX_COMPILER path/to/mpic++)
```

2. Make pafi build folder, run cmake and make
```
   mkdir build
   cd build
   cmake ..
   make pafi
```

## Calculation of free energy barrier between states using PAFI++ (recommended)

0. Tarball in example folder has premade NEB calculation (SIA in EAM-Fe) for testing

1. First set up a LAMMPS neb calculation as described [here](http://lammps.sandia.gov/doc/neb.html)

2. In the LAMMPS script, use the "write_data" command to dump all the NEB knots, i.e.
```
variable u uloop N_NEB_IMAGES

neb etol ftol N1 N2 Nevery file-style arg keyword

write_data neb_knot_file.$u
```
3. Configure the configuration xml file, as shown in the examples

4. Run PAFI as e.g.
```
mkdir -p dumps
mpirun -np NPROCS ./pafi
```
where the first line ensures your dump folder (here the default value) actually exists

## Notes on choosing parameters
- The `CoresPerWorker` option in `config.xml` should be set as low as possible- whilst `LAMMPS` has excellent parallel scaling, ensemble averages have *perfect* parallel scaling.

- If you can add more cores, in general it is best to add more workers to reduce the ensemble average error

- More cores should only be used if the statistical error across the ensemble is already acceptably small and you wish to speed up the data aquisition. As a guide, when using an `EAM` potential with `LAMMPS`, linear scaling requires more than 5000-10000 atoms / core.

- The `nRepeats` option forces each worker to perform multiple independent sampling runs on each plane. This is useful if you are core-limited but can push to longer times

- The total number of sampling runs (independent data points) per plane is thus `nRepeats * NPROCS / CoresPerWorker`

- The total number of force calls per core over a whole run is `(ThermSteps+SampleSteps) * nRepeats * nPlanes * NPROCS / CoresPerWorker`. `PAFI` runs at essentially the same speed as normal molecular dynamics.

## Output

1. PAFI will try to write to the directory as specified in "DumpFolder" in config.xml. Each dump file has a suffix `_T_n`, where `T` is the temperature and `n` is the smallest integer that does not overwrite previous files.

2. For each temperaturer there will be a files `dev_r_T_n.dat` with the ensemble average and variance pathway deviation from each hyperplane and a file `free_energy_profile_T_` that has the integrated FEP.

3. See `error_analysis.pdf' for an expanation of the error bars used in PAFI and `example/sample_plot.py' for a simple plotting example

## Manual use (no recommended)
1. Compile the `pafi-lammps-path` binary
```
cd build
make pafi-lammps-path
```
2. Configure the configuration xml file, to specify the path then run
```
mkdir -p dumps
mpirun -np 1 ./pafi-lammps-path
```

3. This will make a set of files `dumps/pafipath.*.data` which can be run using the `PAFI` example script in the `examples/USER/misc/pafi` directory of the `LAMMPS` repository

## Coming Soon
1. Restart files from pathway deviations
2. Smoothed spline interpolation for more general reference pathways
