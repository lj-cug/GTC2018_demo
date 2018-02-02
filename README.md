# Installation of GTC 2018 Demo

Installation processes of the in-situ visualization Integrated RegESM is divided into three main sections:

1. **Preparing Working Environment:** In this case, set of external software libraries (i.e. [NetCDF](https://www.unidata.ucar.edu/software/netcdf/), [ESMF](https://www.earthsystemcog.org/projects/esmf/), [ParaView](https://www.paraview.org/download/)) must be installed or make them available in the host system that will be used to run the coupled model.

2. **Installing Model Components with Coupling Support:** Each model components must be installed with coupling support. Due to the various design of the standalone model components, the coupling support are achieved by different ways. GTC 2018 demo will use [RegCM](https://gforge.ictp.it/gf/project/regcm/) as an atmosphere model component and [ROMS](https://www.myroms.org) as an ocean model component.

3. **Installing RegESM:** In the last step, model components are merged into single executable using driver written using ESMF library. The driver mainly controls the model components and interactions among them.

## 1. Preparing Working Environment

This section includes detailed information about the installation of required libraries and tools on a Linux based environment. The users also note that the installation procedure might change based on the used computing environment, compiler and MPI implementation. 

**The rest of the document assumes that computing environment is a Linux Cluster (Centos based), compiler is Intel compiler and MPI implementation is Intel MPI.** 

The user defined **PROG** environment variable, which is used in this section, is mainly indicates the directory for the installation of external tools and libraries.

### 1.1. Hierarchical Data Format (HDF5)

Before installation of [HDF5](https://www.hdfgroup.org/HDF5/) library, it is necessary to install a compression library. The HDF5 supports both [zlib](http://zlib.net) and [szip](https://support.hdfgroup.org/ftp/lib-external/szip/2.1.1/src/szip-2.1.1.tar.gz) libraries but in this document we prefer to install zlib instead of szip library.

**To install zlib:**

```
cd $PROGS
wget http://zlib.net/fossils/zlib-1.2.8.tar.gz 
tar -zxvf zlib-1.2.8.tar.gz
cd zlib-1.2.8
export CC=icc
export FC=ifort
./configure --prefix=`pwd`
make
make install
```

**To install HDF5:**

```
cd $PROGS
wget https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.8/hdf5-1.8.11/src/hdf5-1.8.11.tar.gz
tar -zxvf hdf5-1.8.11.tar.gz
./configure --prefix=`pwd` --with-zlib=$PROGS/zlib-1.2.8 --enable-fortran --enable-cxx CC=icc FC=ifort CXX=icpc
make
make install
```

**Note:** Newer version of zlib and HDF5 can be also used but not tested yet.

### 1.2. Network Common Data Form (NetCDF)

The [NetCDF](https://www.unidata.ucar.edu/software/netcdf/) library is distributed separately for each programming language. Because of this restriction, it is necessary to follow specific order to install netCDF C (4.3.0), C++ (4.2) and Fortran (4.2) libraries.

**To install C interface:**

```
cd $PROGS
wget ftp://ftp.unidata.ucar.edu/pub/netcdf/old/netcdf-4.3.0.tar.gz
tar -zxvf netcdf-4.3.0.tar.gz
cd netcdf-4.3.0
mkdir src
mv * src/.
cd src
./configure --prefix=$PROGS/netcdf-4.3.0 CC=icc FC=ifort LDFLAGS="-L$PROGS/zlib-1.2.8/lib - L$PROGS/hdf5-1.8.11/lib" CPPFLAGS="-I$PROGS/zlib-1.2.8/include -I/$PROGS/hdf5- 1.8.11/include"
make
make install
export LD_LIBRARY_PATH=$PROGS/netcdf-4.3.0/lib:$LD_LIBRARY_PATH
```

**To install C++ interface:**

```
cd $PROGS
wget ftp://ftp.unidata.ucar.edu/pub/netcdf/netcdf-cxx-4.2.tar.gz
cd netcdf-cxx-4.2
mkdir src
mv * src/.
cd src
./configure --prefix=$PROGS/netcdf-cxx-4.2 CC=icc CXX=icpc LDFLAGS="-L$PROGS/zlib-1.2.8/lib - L$PROGS/hdf5-1.8.11/lib -L$PROGS/netcdf-4.3.0/lib" CPPFLAGS="-I$PROGS/zlib-1.2.8/include - I/$PROGS/hdf5-1.8.11/include -I$PROGS/netcdf-4.3.0/include"
make
make install
```

**To install Fortran interface:**

```
cd $PROGS
wget ftp://ftp.unidata.ucar.edu/pub/netcdf/netcdf-fortran-4.2.tar.gz
cd netcdf-fortran-4.2
mkdir src
mv * src/.
cd src
./configure --prefix=$PROGS/netcdf-fortran-4.2 CC=icc FC=ifort LDFLAGS="-L$PROGS/zlib-1.2.8/lib - L$PROGS/hdf5-1.8.11/lib -L$PROGS/netcdf-4.3.0/lib" CPPFLAGS="-I$PROGS/zlib-1.2.8/include - I/$PROGS/hdf5-1.8.11/include -I$PROGS/netcdf-4.3.0/include"
make
make install
```

Then, link content of include and library folders of C++ and Fortran under C interface installation directory to use all the libraries from single location (NETCDF environmant variable).

```
cd $PROGS/netcdf-4.3.0/lib
ln -s ../../netcdf-cxx-4.2/lib/* .
ln -s ../../netcdf-fortran-4.2/lib/* .
ln -s ../../netcdf-cxx-4.2/include/* .
ln -s ../../netcdf-fortran-4.2/include/* . 
export NETCDF=$PROGS/netcdf-4.3.0
export PATH=$NETCDF/bin:$PATH
```

### 1.3. Apache Xerces C++

This library is mainly required for **ESMF** installation. It is responsible to read/write grid definitions and attributes (field, component and state level) in XML format.

```
cd $PROGS
wget https://archive.apache.org/dist/xerces/c/3/sources/xerces-c-3.1.1.tar.gz
tar -zxvf xerces-c-3.1.1.tar.gz
cd xerces-c-3.1.1
./configure --prefix=$PROGS/xerces-c-3.1.1 CC=icc CXX=icpc
make
make install
```

### 1.4. Earth System Modeling Framework (ESMF)

The one of the main component of the coupled model is the coupling library, which is used to create driver to control the standalone model components. The detailed and up-to-date information of the installation procedure can be found [here](http://www.earthsystemmodeling.org/esmf_releases/last_built/ESMF_usrdoc/). To see the list of the tested working environments and specific flags for those architectures/platforms, please refer to [here](https://www.earthsystemcog.org/projects/esmf/platforms_7_0_2). 

Before starting to the installation of ESMF library, the users need to pay attention to the following issues;

* The coupled model needs special features of ESMF (>=7.1.0b30). This version is not current public release and its a development snapshot. 
* After netCDF version 4.3.0 the C++ interface is changed and ESMF is not compatible with it. So, it is better to use the <= 4.3.0 version of netCDF in this case.

** To get specific source code: **

```
cd $PROGS
git archive --remote=git://git.code.sf.net/p/esmf/esmf --format=tar --prefix=esmf/ ESMF_7_1_0_beta_snapshot_31 | tar xf -
mv esmf esmf-7.1.0b30
```

**Example environment variable definitions (sh/bash shell) for ESMF installations:**

```
export ESMF_OS=Linux
export ESMF_TESTMPMD=OFF
export ESMF_TESTHARNESS_ARRAY=RUN_ESMF_TestHarnessArray_default
export ESMF_TESTHARNESS_FIELD=RUN_ESMF_TestHarnessField_default
export ESMF_DIR=/okyanus/users/uturuncoglu/progs/esmf-7.1.0b30
export ESMF_TESTWITHTHREADS=OFF
export ESMF_INSTALL_PREFIX=$PROGS/esmf-7.1.0b30/install_dir
export ESMF_COMM=intelmpi
export ESMF_TESTEXHAUSTIVE=ON
export ESMF_BOPT=O
export ESMF_OPENMP=OFF
export ESMF_SITE=default
export ESMF_ABI=64
export ESMF_COMPILER=intel
export ESMF_PIO=internal
export ESMF_NETCDF=split
export ESMF_NETCDF_INCLUDE=$PROGS/netcdf-4.4.0/include
export ESMF_NETCDF_LIBPATH=$PROGS/netcdf-4.4.0/lib
export ESMF_XERCES=standard
export ESMF_XERCES_INCLUDE=$PROGS/xerces-c-3.1.4/include
export ESMF_XERCES_LIBPATH=$PROGS/xerces-c-3.1.4/lib
```

Also note that the environment variables must be changed to install ESMF library to other computing systems or clusters that use different MPI version and operating system.


**To install ESMF:**

```
cd $ESMF_DIR 
make >&make.log 
make install
```

After installation of the ESMF library, user can create a new environment variables like **ESMF_INC**, **ESMF_LIB** and **ESMFMKFILE** to help RegESM configure script to find the location of the required files in the ESMF library installation.

```
export ESMF_INC=$ESMF_INSTALL_PREFIX/include
export ESMF_LIB=$ESMF_INSTALL_PREFIX/lib/libO/Linux.intel.64.intelmpi.default
export ESMFMKFILE=$ESMF_INSTALL_PREFIX/lib/libO/Linux.intel.64.intelmpi.default/esmf.mk

```

Note that the **Linux.intel.64.intelmpi.default** directory could change based on the used architecture, compiler and MPI implementation.

## 2 Installation of Model Components

After installing the required libraries, the next step is installing individual model components with coupling support. In this case, user might install model components to any desired location (or directory) in the file system and point out the installation directories in the configuration phase of the coupled model (driver). 

For the GTC 2018 demo, we are using two-component modeling system (atmosphere and ocean) as well as co-processing component. 

In the user run space please create following directory structure:


```
mkdir $SCRATCH/COP_LR
cd COP_LR
mkdir -p src/atm
mkdir -p src/ocn
mkdir -p src/drv
mkdir input
mkdir output
```

**Note:** The user run space will be referred as **\$SCRATCH** in the rest of the document.

**To install atmosphere model:**

The RegCM is used as atmospheric model component and it requires netCDF library which is already installed in the previous section. To install model,

```
cd $SCRATCH/COP_LR/src/atm
wget -c 
tar -zxvf r6146.tar.gz
cd r6146
./bootstrap.sh
./configure --prefix=`pwd` --enable-cpl CC=icc FC=ifort MPIFC=mpiifort
make
make install
```

**To install ocean model:**

```
cd $SCRATCH/COP_LR/src/ocn
wget 
tar -zxvf roms-r809.tar.gz
```

To install ocean model component, user need to edit two files: **1)** build.sh and **2)** roms-r809/Compilers/Linux-ifort.mk (this file is architecture specific - Linux and Intel Compiler - and might change based on used computing architecture)





As it can be seen from the figure, each model components (including "driver" itself, which is called as "drv" in the figure) use its own directory for the source files, input and output. In addition to the directory structure, the configuration files, run script (OpenPBS, LSF etc.), input and output files can be placed in the main working directory (**BASE_DIR**). RegESM executable (**DRV_SRC**) placed in the working directory is the soft link and can be created with following commands, 

```
cd $BASE_DIR
ln -s $DRV_SRC/regesm.x
```