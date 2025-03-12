<!-- -*-Mode: markdown;-*- -->
<!-- $Id: 4098d4ffce45696ec3497ad9e08e712906c9d8fe $ -->


DaYu
=============================================================================

**Home**:
  - https://github.com/pnnl/DaYu
  
  - [Performance Lab for EXtreme Computing and daTa](https://github.com/perflab-exact)

  - Related: 
  [DataLife](https://github.com/pnnl/DataLife)
  [DaYu](https://github.com/pnnl/DaYu)
  [FlowForecaster](https://github.com/pnnl/FlowForecaster)


**About**: 

The combination of ever-growing scientific datasets and distributed workflow complexity creates I/O performance bottlenecks due to data volume, velocity, and variety. Although the increasing use of descriptive data formats (e.g., HDF5, netCDF) helps organize these datasets, it also creates obscure bottlenecks due to the need to translate high level operations into file addresses and then into low-level I/O operations.

DaYu is a framework for analyzing (a) semantic relationships between logical datasets and file addresses, (b) how dataset operations translate into I/O, and (c) the combination across entire workflows. DaYu's analysis and visualization enables identification of critical bottlenecks and reasoning about remediation. With DaYu, one can extract workflow data patterns, develop insights into the behavior of data flows, and identify opportunities for both users and I/O libraries to optimize the applications.

The DaYu framework comprises three primary components:
* Data Semantic Mapper, which maps semantic datasets to I/O statistics, capturing essential data flow insights for analysis.
* Workflow Analyzer, which groups I/O statistics by high-level data semantics and visualizes the combination as semantic dataflow graphs, to give insights into holistic data dependence for I/O accesses
* Data Flow Diagnostics, which explores three real-world scientific workflows from distinct domains, generating visualization of dataflow and I/O semantics, revealing potential I/O improvement opportunities, and empowered with optimizations suggested by DaYu's insights

[1] DaYu (e.g. "Yu the Great") refers to a legendary Chinese king credited with taming floods through water control projects



**Contacts**: (_firstname_._lastname_@pnnl.gov)
  - Meng Tang (Illinois Institute of Technology) ([www](https://scholar.google.com/citations?user=KXC9NesAAAAJ&hl=en))
  - Nathan R. Tallent ([www](https://hpc.pnnl.gov/people/tallent)), ([www](https://www.pnnl.gov/people/nathan-tallent))
  - Lenny Guo ([www](https://www.pnnl.gov/people/luanzheng-guo))


**Contributors**:
  - Meng Tang (Illinois Institute of Technology) ([www](https://scholar.google.com/citations?user=KXC9NesAAAAJ&hl=en))
  - Lenny Guo ([www](https://www.pnnl.gov/people/luanzheng-guo))
  - Nathan R. Tallent (PNNL) ([www](https://hpc.pnnl.gov/people/tallent)), ([www](https://www.pnnl.gov/people/nathan-tallent))
  - Anthony Kougkas (Illinois Institute of Technology)
  - Xian-He Sun (Illinois Institute of Technology)


Details
=============================================================================

DaYu's Tracker monitors hdf5 program I/O from the Virtual Object Layer (VOL) level as well as the Vitual File Driver (VFD) level.


The VOL monitors objects accesses during program, implemented with the HDF5 Passthrough VOL.

The VFD monitors POSIX I/O operation during program, implemented with the HDF5 default sec2 I/O operations.


# Workflow Sankey Diagram Showcase
https://github.com/candiceT233/dayu-tracker/blob/main/flow_analysis/example_stat/README.md


# How to use

## Prerequisite
- HDF5 (1.14.+, require C, CXX and HDF5_HL_LIBRARIES) \
Install with spack (suggest spack version 0.20.+)
```bash
spack install hdf5@1.14+cxx+hl~mpi
```
- h5py==3.8.0
```bash
YOUR_HDF5_PATH="`which h5cc |sed 's/.\{9\}$//'`"
echo $YOUR_HDF5_PATH # make sure your path is correct
python3 -m pip uninstall h5py; HDF5_MPI="OFF" HDF5_DIR=$YOUR_HDF5_PATH python3 -m pip install --no-binary=h5py h5py==3.8.0
```

## Installation
```bash

git clone https://github.com/candiceT233/dayu-tracker.git
cd dayu-tracker 
git submodule update --init --recursive
YOUR_INSTALLATION_PATH="`pwd`" # you can use your own path

mkdir build 
cd build
ccmake -DCMAKE_INSTALL_PREFIX=$YOUR_INSTALLATION_PATH ..
```


## Setup program task name
Before running your program from a bash command, setup program task name two ways:
---
1. Setup with bash environment variable:
```shell
export CURR_TASK="my_program"
```
---
2. Setup with file in `/tmp` directory:
```bash

export WORKFLOW_NAME="my_program"
export PATH_FOR_TASK_FILES="/tmp/$USER/$WORKFLOW_NAME"
mkdir -p $PATH_FOR_TASK_FILES
> $PATH_FOR_TASK_FILES/${WORKFLOW_NAME}_vfd.curr_task # clear the file
> $PATH_FOR_TASK_FILES/${WORKFLOW_NAME}_vol.curr_task # clear the file

echo -n "$TASK_NAME" > $PATH_FOR_TASK_FILES/${WORKFLOW_NAME}_vfd.curr_task
echo -n "$TASK_NAME" > $PATH_FOR_TASK_FILES/${WORKFLOW_NAME}_vol.curr_task
```

## Dynamically load VFD and VOL libraries
```bash
TRACKER_SRC_DIR="../build/src" # dayu_tracker installation path
schema_file_path="`pwd`" #your_path_to_store_log_files
export HDF5_VOL_CONNECTOR="tracker under_vol=0;under_info={};path=$schema_file_path;level=2;format=" # VOL connector info string
export HDF5_PLUGIN_PATH=$TRACKER_SRC_DIR/vfd:$TRACKER_SRC_DIR/vol
export HDF5_DRIVER=hdf5_tracker_vfd # VFD driver name
export HDF5_DRIVER_CONFIG="${schema_file_path};${TRACKER_VFD_PAGE_SIZE}" # VFD info string

# Run your program
python h5py_write_read.py
```

## Optiona: Dynamically load only VFD
```bash
TRACKER_SRC_DIR="../build/src" # dayu_tracker installation path
schema_file_path="`pwd`" #your_path_to_store_log_files
export HDF5_PLUGIN_PATH=$TRACKER_SRC_DIR/vfd
export HDF5_DRIVER=hdf5_tracker_vfd # VFD driver name
export HDF5_DRIVER_CONFIG="${schema_file_path};${TRACKER_VFD_PAGE_SIZE}" # VFD info string

# Run your program
python h5py_write_read.py
```

## Optiona: Dynamically load only VOL
```bash
TRACKER_SRC_DIR="../build/src" # dayu_tracker installation path
schema_file_path="`pwd`" #your_path_to_store_log_files
export HDF5_VOL_CONNECTOR="tracker under_vol=0;under_info={};path=$schema_file_path;level=2;format="
export HDF5_PLUGIN_PATH=$TRACKER_SRC_DIR/vol

python h5py_write_read.py
```

# Use with Jarvis-cd
1. Jarvis-cd can be installed and initialized following steps from [here](https://github.com/candiceT233/jarvis-cd).
2. Add dayu-tracker to jarvis-cd
```bash
jarvis repo add /home/mtang11/scripts/vol-tracker/jarvis
```
