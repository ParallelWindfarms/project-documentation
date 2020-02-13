# OpenFOAM and hdf5

## Approach 1: IOHwrite

- https://github.com/parallelwindfarms/IOH5Write
- Project discontinued in 2014

### Troubleshooting

#### hdf5.h not available
There is a bug in `src/postProcessing/functionObjects/IOh5Write/Make/options`. Line 5 should be

```
H5INC    = -I$(HDF5_DIR)

# Instead of
# H5INC    = -I$(HDF5_DIR)/include

```
#### Fatal error while executing `Allwmake`
Missing `OutputFilterFunctionObject.H`.

This seems to be a problem with the OpenFOAM version. A workarond seems to be the following renamings:

```
#include "OutputFilterFunctionObject.H" -> #include "IOOutputFilter.H"
```

```
typedef OutputFilterFunctionObject<h5Write> -> typedef IOOutputFilter<h5Write>
```

The above-mentioned calls appear in `src/postProcessing/functionObjects/IOh5Write/h5Write/h5WriteFunctionObject.H`

## Approach 2: foamToHDF5

- https://github.com/nicolasedh/foamToHDF5
- Project discontinued in 2016

### Troubleshooting

#### Poorly formatted readme

Fixed in [our fork](https://github.com/parallelwindfarms/foamToHDF5)

#### hdf5.h not available
Quick and dirty fix:

```
# Rename
HDF5_DIR=$(WM_PROJECT_INST_DIR)/installs/hdf5 -> HDF5_DIR=/home/<user>/code/installs/hdf5

# Both in
# exportFoamToHDF5/Make/options
#
# and
#
# foamToHDF5/Make/options
```

#### Fatal error while executing `Allwmake`

Same as in approach 1

## Approach 3: let Python do the work

#### Links
- [How Python helps us to avoid contact with OpenFOAMT](http://www.thevisualroom.com/_static/02_pyFoamFifthWorkshop.pdf)

## Approach 4: do it yourself

### Adding custom `functionObject`

Add a dictionary to the `controlDict` file:

```
functions
{
    Co1
    {
        type                CourantNo;
        libs                ("libfieldFunctionObjects.so");
        executeControl      timeStep;
        executeInterval     2;
        writeControl        writeTime;
    }

}
```

The functions are stored at `./src/functionObject/`

#### Links
- [Guide to functionObjects](http://www.tfd.chalmers.se/~hani/kurser/OS_CFD_2017/SankarRajuNarayanasamy/Sankar_OS_CFD_Report.pdf)
