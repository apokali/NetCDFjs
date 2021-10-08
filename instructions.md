# Preparation

## Install NetCDF4 From Source and C++ Dependencies
**Tips:** There might be package managers that help simplify NetCDF installtion process, but none of them seems to work at the time this guide was written. 
This guide will only present the method that works 100% feasible.
If you are interested in other ways, here is a reference:
* Using spack http://wiki.seas.harvard.edu/geos-chem/index.php/Check_if_netCDF_is_already_installed_on_your_system

### Install netCDF4 from Source (Updated Detailed Instructions)
Reference https://support.scinet.utoronto.ca/education/staticpublic/course177content327.html

**Warning:** The versions of many dependencies mentioned in the link above are legacy ones. However the insturctions below are up-to-date.

1. Create a place for local (non-system) libraries
```bash
# Create a place for local (non-system) libraries
mkdir $HOME/local
mkdir $HOME/local/bin
mkdir $HOME/local/include
mkdir $HOME/local/lib
export PATH="$PATH:$HOME/local/bin"
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:$HOME/local/lib"
export LIBRARY_PATH="$LIBRARY_PATH:$HOME/local/lib"
export CPATH="$CPATH:$HOME/local/include"
```
Also add the **last four lines** to your $HOME/.bashrc (if your shell is bash), so that these four environment variables are always set correctly.

2. Install zlib
```bash
# Install zlib from downloaded zlib-1.2.8.tar.gz
tar axvf zlib-1.2.8.tar.gz
cd zlib-1.2.8
./configure --prefix=$HOME/local
make
make check
make install
cd ..
```
Note that the make check step is **optional but highly recommended** to see if the library was correctly compiled.

3. Install HDF5
```bash
wget http://www.hdfgroup.org/ftp/HDF5/current/src/hdf5-1.10.5.tar.gz
# Install HDF5 from downloaded hdf5-1.10.5.tar.gz
tar axvf hdf5-1.10.5.tar.gz
cd hdf5-1.10.5
./configure --prefix=$HOME/local --enable-fortran --enable-c++
make
make check
make install
cd ..
```

4. Install the NetCDF4 C library
```bash
wget https://downloads.unidata.ucar.edu/netcdf-c/4.8.1/src/netcdf-c-4.8.1.tar.gz
# Install NetCDF4 from netcdf-4.8.1.tar.gz
tar axvf netcdf-4.8.1.tar.gz
cd netcdf-4.8.1
./configure --prefix=$HOME/local --enable-netcdf-4
make
make check
make install
cd ..
```

5. Install the NetCDF C++ binding
```bash
wget https://www.unidata.ucar.edu/downloads/netcdf/ftp/netcdf-cxx4-4.3.1.tar.gz
# Install NetCDF4 C++ interface from netcdf-cxx4-4.3.1.tar.gz
tar axvf netcdf-cxx4-4.3.1.tar.gz
cd netcdf-cxx4-4.3.1
./configure --prefix=$HOME/local
make
make check
make install
cd ..
```

## Install NetCDF4 Javascript on npm
Reference: https://github.com/parro-it/netcdf4 (More sytax and usage can be found)

**Warning:** Using **npm** alone is likely NOT to work according to (https://github.com/parro-it/netcdf4/issues/28#issuecomment-661298904).
```bash
npm install netcdf4
```

### Alternative
Clone this repo locally and do the following
```
git clone https://github.com/parro-it/netcdf4
npm install /netcdf4
```


# Test on NetCDF Files
Data Source used in thie guide https://www.frdr-dfdr.ca/repo/dataset/4bb24ee2-73e1-43a8-a929-126d2eb2bfa3

Here is an example to fetch data from a sample dataset (EMDNA_1980_mean.nc4, download it by `wget https://g-624536.53220.5898.data.globus.org/6/published/publication_270/submitted_data/EMDNA_estimate/1980/EMDNA_1980_mean.nc4`). We will use Python and Javascript to read the same sample netCDF file and compare their outputs.

**Warnings:** In the following example, the python script was run under pyserini conda environment with additional netCDF4 package installation. For more detailed guides, please reference https://towardsdatascience.com/read-netcdf-data-with-python-901f7ff61648. Additional `conda install netcdf4` is required.

While for JavaScript, the code was run under base environment as we did not install NetCDF dependencies in pyserini. There coule be a way to achieve this (using package manager). But reproduction is way too complicated.

* In Python (run under pyserini environment with additional netCDF4 package)
```py
import netCDF4 as nc
import numpy as np

# import dataset
fn = 'src/1980/EMDNA_1980_mean.nc4'
ds = nc.Dataset(fn)

# returns the information of the dataset imported
ds
```
Output
```
<class 'netCDF4._netCDF4.Dataset'>
root group (NETCDF4 data model, file format HDF5):
    dimensions(sizes): y(800), x(1300), time(366), const(1)
    variables(dimensions): int64 date(time), float32 latitude(y), float32 longitude(x), float32 prcp(time, x, y), float32 tmean(time, x, y), float32 trange(time, x, y)
    groups:
```

* In JavaScipt (run by using node)
```js
// import library
var netcdf4 = require("netcdf4");

// import dataset
var file = new netcdf4.File("src/1980/EMDNA_1980_mean.nc4", "r");


console.log(file);
console.log(file.root.dimensions);
```
which produces the following output
```
File {
  root: Group {
    fullname: '/',
    name: '/',
    subgroups: {},
    attributes: {},
    unlimited: {},
    dimensions: {
      y: [Dimension],
      x: [Dimension],
      time: [Dimension],
      const: [Dimension]
    },
    variables: {
      date: [Variable],
      latitude: [Variable],
      longitude: [Variable],
      prcp: [Variable],
      tmean: [Variable],
      trange: [Variable]
    },
    id: 65536
  }
}
{
  y: Dimension { name: 'y', length: 800, id: 0 },
  x: Dimension { name: 'x', length: 1300, id: 1 },
  time: Dimension { name: 'time', length: 366, id: 2 },
  const: Dimension { name: 'const', length: 1, id: 3 }
}
```

