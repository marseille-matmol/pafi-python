         _______      _______      _______     _________
        (  ____ )    (  ___  )    (  ____ \    \__   __/
        | (    )|    | (   ) |    | (    \/       ) (
        | (____)|    | (___) |    | (__           | |
        |  _____)    |  ___  |    |  __)          | |
        | (          | (   ) |    | (             | |
        | )          | )   ( |    | )          ___) (___
        |/           |/     \|    |/           \_______/
        Projected    Average      Force        Integrator


v0.8 :copyright: TD Swinburne and M-C Marinica 2019 MIT License

swinburne at cinam.univ-mrs.fr


Beta version of code used in [this paper](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.120.135503)
> *Unsupervised Calculation of Free Energy Barriers in Large Crystalline Systems*   
> T.D. Swinburne and M.-C. Marinica, Physical Review Letters 120 (13), 135503, 2018

Please cite the above when publishing results using PAFI

This PAFI repository includes the [RapidXML](http://http://rapidxml.sourceforge.net) library for input parsing

# Installation

## Compile LAMMPS with USER-PAFI package
1.  RECOMMENDED: Download or clone fork of LAMMPS with USER-PAFI package [here](https://github.com/tomswinburne/lammps/)
ALTERNATIVE: [Download LAMMPS](http://lammps.sandia.gov/download.html) then, before installing any packages, in the LAMMPS root directory (parent directory of src folder) run the pafi patch, adding the USER-PAFI package and support for PAFI atom styles
```
cd /path/to/lammps_repo/
patch -p0 < /path/to/user-pafi_lammps_Aug18.patch # modifies /path/to/lammps_repo/src folder
```

2. Install USER-PAFI and any packages you desire (e.g. replica for NEB)
```
cd src
make yes-user-pafi
make yes-replica # for NEB calculation
make yes-package_name # (i.e. manybody for EAM potentials etc)
```

3. Compile as a static library (and optionally binary initial NEB calculation) Consult [LAMMPS documentation](http://lammps.sandia.gov/doc/Section_start.html) for details
```
   make mpi mode=lib # liblammps_mpi.a library for pafi
   make mpi # lmp_mpi binary for running initial NEB calculation if desired
```
4. Copy library to your local lib/ and headers to local include/, at e.g. ${HOME}/.local
```
   cp liblammps_mpi.a ${HOME}/.local/lib
   mkdir ${HOME}/.local/include/lammps
   cp *.h ${HOME}/.local/include/lammps/
```

## Compile PAFI
0. If required, download and install cmake from https://cmake.org/download/

1. Specify environment variables in CMakeLists.txt:
```
   set(HOME your/home/path)
   set(CMAKE_INCLUDE_PATH ${HOME}/.local/include)
   set(CMAKE_LIBRARY_PATH ${HOME}/.local/lib)
   set(CMAKE_CXX_COMPILER path/to/mpic++)
```

2. Make pafi build folder, run cmake and make
```
   mkdir build
   cd build
   cmake ..
   make
```

## Calculation of free energy barrier between states

0. Tarball in example folder has premade NEB calculation (SIA in EAM-Fe) for testing

1. First set up a LAMMPS neb calculation as described [here](http://lammps.sandia.gov/doc/neb.html)

2. In the LAMMPS script, use the "write_data" command to dump all the NEB knots, i.e.
```
variable u uloop N_NEB_IMAGES

neb etol ftol N1 N2 Nevery file-style arg keyword

write_data neb_knot_file.$u
```
3. Move the knots to folder and include and configure the configuratio xml file, as shown in the examples

4. Run PAFI as e.g.
```
mpirun -np NPROCS ./pafi
```
## Output

1. PAFI will try to write to the directory as specified in "DumpFolder" in config.xml. Each dump file has a suffix `_T_n`, where `T` is the temperature and `n` is the smallest integer that does not overwrite previous files.

2. For each temperaturer there will be a files `dev_r_T_n.dat` with the ensemble average and variance pathway deviation from each hyperplane and a file `free_energy_profile_T_` that has the integrated FEP.

## Coming Soon
1. Restart files from pathway deviations
2. Smoothed spline interpolation for more general reference pathways
3. tbc....
