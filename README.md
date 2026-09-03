<div align="center">
  <img src="img/fmm_social.jpg">
</div>

| Linux / macOS | Windows | Wiki          | Docs        |
| ------------- | ------- | ------------- | ----------- |
| [![Build Status](https://travis-ci.org/cyang-kth/fmm.svg?branch=master)](https://travis-ci.org/github/cyang-kth/fmm) | [![Build status](https://ci.appveyor.com/api/projects/status/8qee5c8iay75j1am?svg=true)](https://ci.appveyor.com/project/cyang-kth/fmm) | [![Wiki](https://img.shields.io/badge/wiki-website-blue.svg)](https://fmm-wiki.github.io/) | [![Documentation](https://img.shields.io/badge/docs-doxygen-blue.svg)](https://cyang-kth.github.io/fmm/) |

FMM is an open source map matching framework integrating hidden Markov models and precomputation. It solves the problem of matching noisy GPS data to a road network. By maximizing performance and functionality, FMM allows for map matching algorithms that are both efrficient and scalable to large volumes of data.

FMM provides Python and C++ APIs and can be used in the command line, in Jupyter notebooks, or in web app.

### Table of Contents
- [Features](#features)
- [Benchmark (fork vs origin)](#benchmark-fork-vs-origin)
- [Screenshots of notebook](#screenshots-of-notebook)
- [Requirements](#requirements)
- [Getting Started](#getting-started)
- [Documentation](#documentation)
- [Code docs for developer](#code-docs-for-developer)
- [Contact and citation](#contact-and-citation)

### Features
- **High performance**: C++ implementation using Rtree, optimized routing, parallel computing (OpenMP).
- **Python API**: [jupyter-notebook](example/notebook) and [web app](example/web_demo)
- **Scalability**: millions of GPS points and millions of road edges.  
- **Multiple data formats**:
  - Road network in OpenStreetMap or ESRI shapefile.
  - GPS data in Point CSV, Trajectory CSV and Trajectory Shapefile ([more details](https://fmm-wiki.github.io/docs/documentation/input/#gps-data)).
- **Detailed matching information**: traversed path, geometry, individual matched edges, GPS error, etc. More information at [here](https://fmm-wiki.github.io/docs/documentation/output/).
- **Multiple algorithms**: [FMM](http://www.tandfonline.com/doi/full/10.1080/13658816.2017.1400548) (for small and middle scale networks) and [STMatch](https://dl.acm.org/doi/abs/10.1145/1653771.1653820) (for large scale road networks)
- **Platform support**: Unix (ubuntu) , Mac and Windows(cygwin environment).
- **Hexagon match**: :tada: Match to the uber's [h3](https://github.com/uber/h3) Hexagonal Hierarchical Geospatial Indexing System. Check the [demo](example/h3).

We encourage contribution with feature request, bug report or developping new map matching algorithms using the framework.

### Benchmark (fork vs origin)

Test map matching of connected vehicle GPS points in three
sub-regions in Michigan, with the settings `k = 8`, 
search radius 3000 m, GPS error 100 m, all output fields
written). The time is the `Time takes` value reported by
`FastMapMatch.match_gps_file`, i.e. reading the input CSV, matching and
writing the result file, measured on an AMD Ryzen 9 9950X (16 cores,
32 threads) with 32 OpenMP threads.

| Region | Area (E-W x N-S) | Road network | GPS points | Trajectories | `19ef34e` | This version | Speedup |
|--------|------------------|--------------|-----------:|-------------:|----------:|-------------:|--------:|
|   1    | 4.4 x 8.9 km     | 1,862 edges  |  5,368,249 |       75,595 |    13.0 s |        1.0 s |     13x |
|   2    | 13.7 x 9.3 km    | 4,990 edges  | 15,387,122 |      150,458 |    72.7 s |        3.1 s |     23x |
|   3    | 9.0 x 5.8 km     | 2,836 edges  |  8,020,427 |       92,083 |    34.5 s |        1.6 s |     22x |

- [`19ef34e`](https://github.com/cyang-kth/fmm/commit/19ef34e1f57ff2f2484231aa0d01dfffea986ec1) is the code
  before the changes of this fork, built with the newest tool chain it
  compiles with (GCC 13, Boost 1.74, GDAL 3.4.3, Python 3.10).
- *This version* is built with GCC 15, Boost 1.92, GDAL 3.12 and Python 3.14.
  The tool chain upgrade by itself did not change the timings; the gain comes
  from the candidate search (a bounding box bug that made the rtree return
  most of the network for every point, a k-nearest search that skips the
  edges which cannot be among the k best, and an adaptive query box), an
  open addressing UBODT hash table, faster CSV parsing and writing, and
  reading the next block of trajectories while the current one is matched.
- Both versions produce the same matching results for these data sets
  (verified by comparing the complete output files).

### Screenshots of notebook

Map match to OSM road network by drawing

![fmm_draw](https://github.com/cyang-kth/fmm-examples/blob/master/img/fmm_draw.gif?raw=true)

Explore the factor of candidate size k, search radius and GPS error

![fmm_explore](https://github.com/cyang-kth/fmm-examples/blob/master/img/fmm_explore.gif?raw=true)

Explore detailed map matching information

![fmm_detail](https://github.com/cyang-kth/fmm-examples/blob/master/img/fmm_detail.gif?raw=true)

Explore with dual map

![dual_map](https://github.com/cyang-kth/fmm-examples/blob/master/img/dual_map.gif?raw=true)

Map match to hexagon by drawing

![hex_draw](https://github.com/cyang-kth/fmm-examples/blob/master/img/hex_draw.gif?raw=true)

Explore the factor of hexagon level and interpolate

![hex_explore](https://github.com/cyang-kth/fmm-examples/blob/master/img/hex_explore.gif?raw=true)

Source code of these screenshots are available at https://github.com/cyang-kth/fmm-examples.

### Requirements
- C++ Compiler supporting c++17 and OpenMP
- CMake >=3.16: provides cross platform building tools
- GDAL >= 2.2 (tested up to 3.13): IO with ESRI shapefile, Geometry data type
- Boost Graph >= 1.70 (tested up to 1.92): routing algorithms used in UBODT Generator
- Boost Geometry >= 1.70: Rtree, Geometry computation
- Boost Serialization >= 1.70: Serialization of UBODT in binary format
- ~~Libosmium: a library for reading OpenStreetMap data. Requires expat and bz2.~~ (The direct input with OSM file is removed since the raw dataset may contain
topology errors. It is suggested to use shapefile input downloaded from osmnx.)
- swig >= 4.0: used for building Python bindings
- Python 3 (tested up to 3.14) with the development headers

### Getting Started
These instructions are for the Ubuntu platform. For installation on Windows and Mac, refer to the [installation instructions](https://fmm-wiki.github.io/docs/installation/) here.

1. **Install Requirements**
    - Update the ppa to install the required version (>=2.2) of GDAL.
    
      ```shell
      sudo add-apt-repository ppa:ubuntugis/ppa
      sudo apt-get -q update
      ```
    - Then, install all the requirements with

      ```shell
      sudo apt-get install libboost-dev libboost-serialization-dev \
      gdal-bin libgdal-dev make cmake libbz2-dev libexpat1-dev swig python-dev
      ```

2. **Install C++ program and Python bindings**
    - Build and install the program with cmake.

      ```shell
      # Under the project folder
      mkdir build
      cd build
      cmake ..
      make -j4
      sudo make install
      ```
      This will build executable files under the `build` folder, which are installed to `/usr/local/bin`:
      - `ubodt_gen`: the Upper bounded origin destination table (UBODT) generator (precomputation) program
      - `fmm`: the program implementing the fast map matching algorithm
      - `stmatch`: the program implementing the STMATCH algorithm, no precomputation needed
      
      It will also create a folder `python` under the build path, which contains fmm bindings(`fmm.py` and `_fmm.so`) that are installed into the Python site-packages location (e.g., `/usr/lib/python2.7/dist-packages`).
      
3. **Verification of Installation**
    - Run command line map matching
      Open a new terminal and type `fmm`. You should see the following output:
      
      ```shell
      ------------ Fast map matching (FMM) ------------
      ------------     Author: Can Yang    ------------
      ------------   Version: 2020.01.31   ------------
      ------------     Applicaton: fmm     ------------
      A configuration file is given in the example folder
      Run `fmm config.xml` or with arguments
      fmm argument lists:
      --ubodt (required) <string>: Ubodt file name
      --network (required) <string>: Network file name
      --gps (required) <string>: GPS file name
      --output (required) <string>: Output file name
      --network_id (optional) <string>: Network id name (id)
      --source (optional) <string>: Network source name (source)
      --target (optional) <string>: Network target name (target)
      --gps_id (optional) <string>: GPS id name (id)
      --gps_geom (optional) <string>: GPS geometry name (geom)
      --candidates (optional) <int>: number of candidates (8)
      --radius (optional) <double>: search radius (300)
      --error (optional) <double>: GPS error (50)
      --pf (optional) <double>: penalty factor (0)
      --log_level (optional) <int>: log level (2)
      --output_fields (optional) <string>: Output fields
        opath,cpath,tpath,ogeom,mgeom,pgeom,
        offset,error,spdist,tp,ep,all
      For xml configuration, check example folder
      ------------    Program finished     ------------
      ```
  
    - Run python script
      To verify that the Python bindings are working:
  
      ```shell
      # Change to the parent folder which contains fmm_test.py
      cd ../example/python
      python fmm_test.py
      ```

    Refer to the [Q&A](https://fmm-wiki.github.io/docs/installation/qa) for any installation errors.

### Documentation

- Check [https://fmm-wiki.github.io/](https://fmm-wiki.github.io/) for installation, documentation.
- Check [example](example) for simple examples of fmm.
- :tada: Check [https://github.com/cyang-kth/fmm-examples](https://github.com/cyang-kth/fmm-examples)
for interactive map matching in notebook.

### Code docs for developer

Check [https://cyang-kth.github.io/fmm/](https://cyang-kth.github.io/fmm/)

### Contact and citation

Can Yang, Ph.D. student at KTH, Royal Institute of Technology in Sweden

Email: cyang(at)kth.se

Homepage: https://people.kth.se/~cyang/

FMM originates from an implementation of this paper [Fast map matching, an algorithm integrating hidden Markov model with precomputation](http://www.tandfonline.com/doi/full/10.1080/13658816.2017.1400548). A post-print version of the paper can be downloaded at [link](https://people.kth.se/~cyang/bib/fmm.pdf). Substaintial new features have been added compared with the original paper.  

Please cite fmm in your publications if it helps your research:

    Can Yang & Gyozo Gidofalvi (2018) Fast map matching, an algorithm
    integrating hidden Markov model with precomputation, International Journal of Geographical Information Science, 32:3, 547-570, DOI: 10.1080/13658816.2017.1400548

Bibtex file

```bibtex
@article{Yang2018FastMM,
  title={Fast map matching, an algorithm integrating hidden Markov model with precomputation},
  author={Can Yang and Gyozo Gidofalvi},
  journal={International Journal of Geographical Information Science},
  year={2018},
  volume={32},
  number={3},
  pages={547 - 570}
}
```
