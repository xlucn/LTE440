# LTE440
Lunar Time Ephemerides based on DE440. Provide an ephemeris to compute transformations between the time scale of Lunar Reference System, i.e., Lunar Coordinate Time (TCL), and other time scales such as Barycentric Dynamical Time (TDB).

## Download
The files are available in the [Releases](https://github.com/xlucn/LTE440/releases) page.

## File structure
The released product contains the following data and demo files:
- Data files. These files include the actuall data of LTE440 and follow the [SPICE](https://naif.jpl.nasa.gov/naif/aboutspice.html) format. Common toolkits to access the data include [SPICE Toolkit](https://naif.jpl.nasa.gov/naif/toolkit.html), [CALCEPH](https://calceph.imcce.fr/) and [libephaccess](https://gitlab.iaaras.ru/iaaras/ephemeris-access)
  - `lte440.bsp`: binary SPK kernel
  - `lte440.tpc`: text PCK kernel
- Demo files. These files provide examples for accessing the data with certain toolkits, and also serve as test cases for anyone to check their results. Note that the output is slightly different given the same input, due to the toolkits provide slightly different results given the same ephemeris file.
  - With SPICE toolkit
    - `lte440.py`: demo script
    - `lte440.in`: demo input
    - `lte440.out`: demo output
  - With CALCEPH
    - `lte440_2.py`: demo script
    - `lte440.in`: demo input
    - `lte440_2.out`: demo output

## Usage

The demo scripts `lte440.py` or `lte440_2.py` provide a function `tclmtdb` to access the ephemeris data and compute the final transformation of TCL-TDB and TCL-TCB. Basically it reads the periodic part from the binary bsp file and linear rate from the text tpc file, and then compute the combination of the two parts. The code can be easily ported to other toolkits or programing languages.

As a simplified example
```py
from calcephpy import CalcephBin, Constants

jd = 2451545.0
T0 = 2443144.5003725 + (-65.5e-6) / 86400

# Load ephemeris files
eph = CalcephBin.open(['lte440.bsp', 'lte440.tpc'])
unit = Constants.UNIT_SEC | Constants.UNIT_KM | Constants.USE_NAIFID

# Extract periodic part
# Function `compute_unit` returns [X, Y, Z, VX, VY, VZ]
# LTE data is in the X coordinate
periodic = eph.compute_unit(jd, 0, 1000000005, 1000000000, unit)[0]

# Calculate the linear part
# Read drift rate from the PCK kernel, name is "BODY<target>_RATE"
linear_rate = eph.getconstant('BODY1000000005_RATE')
linear = linear_rate * (jd - T0) * 86400

# Calculate the total TCL-TCB by combining periodic and linear terms
tcl_tdb = periodic + linear
print("TCL-TDB at T0+TDB0 is:", tcl_tdb, "seconds")
```
should produce the following output
```
TCL-TDB at T0+TDB0 is: 0.49330749643254945 seconds
```

## References
Refer to [arXiv:2506.19213](https://arxiv.org/abs/2506.19213) and [aa57345-25](https://doi.org/10.1051/0004-6361/202557345) for more information.

To cite our work
```
@ARTICLE{2025A&A...704A..76L,
       author = {{Lu}, Xu and {Yang}, Tian-Ning and {Xie}, Yi},
        title = "{Lunar time ephemeris LTE440: Definitions, algorithm, and performance}",
      journal = {\aap},
     keywords = {ephemerides, time, Instrumentation and Methods for Astrophysics, Earth and Planetary Astrophysics},
         year = 2025,
        month = dec,
       volume = {704},
          eid = {A76},
        pages = {A76},
          doi = {10.1051/0004-6361/202557345},
archivePrefix = {arXiv},
       eprint = {2509.18511},
 primaryClass = {astro-ph.IM},
       adsurl = {https://ui.adsabs.harvard.edu/abs/2025A&A...704A..76L},
      adsnote = {Provided by the SAO/NASA Astrophysics Data System}
}

@ARTICLE{2025arXiv250619213L,
       author = {{Lu}, Xu and {Yang}, Tian-Ning and {Xie}, Yi},
        title = "{Lunar Time Ephemeris LTE440: User Manual}",
      journal = {arXiv e-prints},
     keywords = {Instrumentation and Methods for Astrophysics, Earth and Planetary Astrophysics},
         year = 2025,
        month = jun,
          eid = {arXiv:2506.19213},
        pages = {arXiv:2506.19213},
          doi = {10.48550/arXiv.2506.19213},
archivePrefix = {arXiv},
       eprint = {2506.19213},
 primaryClass = {astro-ph.IM},
       adsurl = {https://ui.adsabs.harvard.edu/abs/2025arXiv250619213L},
      adsnote = {Provided by the SAO/NASA Astrophysics Data System}
}
```
