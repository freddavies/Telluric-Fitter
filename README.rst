TelFit Code
===========

This code will fit the telluric absorption spectrum in data, using
LBLRTM. More details can be found in the `documentation`_ and examples, but the
installation and run procedure are outlined below. If you use this code,
please cite `my paper`_.

Installation
------------

This code originally only ran on python 2, but should now ostensibly run on python 3. 

The following packages are required (version numbers are a carryover from the python 2 version):

  - matplotlib
  - **numpy v1.6 or greater**
  - scipy v0.13 or greater
  - astropy v0.2 or greater
  - lockfile
  - pysynphot v0.7 or greater
  - fortranformat
  - **cython**
  - **requests**

The bolded entries are required *before* installation, so make sure you get them from pip, apt-get/yum, or conda (depending on your OS and python distribution). The setup script will attempt to install the rest if you don't have them, but I suggest doing it yourself just to make sure nothing goes wrong. Once you have the dependencies, then run

.. code:: bash

    git clone https://github.com/freddavies/Telluric-Fitter-py3.git
    cd Telluric-Fitter-py3
    python setup.py install

to install TelFit. It may take a while, as it needs to build the LBLRTM code and some of its standard input files.

If there are Fortran compile errors
------------

If you are running gfortran version 8 or higher, you will very likely run into some errors when LBLRTM compiles. Fortunately, there is a way out. The TelFit installation places the LBLRTM and LNFL codes in the directory `~/.TelFit/` and attempts to compile them there. The compilation statement for whatever system you are running needs to be modified to include the `-std=legacy` flag, and then run python setup.py install once more to finish the installation.

**WARNING**: Any changes you make to the makefiles, as described below, will be overwritten if you simply run `python setup.py install` again. Comment out line 210 in `setup.py` to bypass re-un-packing the tar files,

.. code:: bash

    #subprocess.check_call(["tar", "-xzf", '{}{}'.format(TELLURICMODELING, fname), '-C', TELLURICMODELING])

Here are examples for modifying the makefiles. For my system, in the file `~/.TelFit/lblrtm/build/makefile.common`, the relevant block is:

.. code:: bash

    osxGNUsgl:
	${MAKE} -f ${MAKEFILE} all P_TYPE=sgl FC_TYPE=gnu \
	PLTFRM=OS_X \
	FC=gfortran \
	FCFLAG="-frecord-marker=4" \
	UTIL_FILE=util_gfortran.f90

To fix it, I add the `std=legacy` flag to FCFLAG, like so:

.. code:: bash

    osxGNUsgl:
	${MAKE} -f ${MAKEFILE} all P_TYPE=sgl FC_TYPE=gnu \
	PLTFRM=OS_X \
	FC=gfortran \
	FCFLAG="-frecord-marker=4 -std=legacy" \
	UTIL_FILE=util_gfortran.f90

The same flag must also be added to FCFLAG in the file `~/.TelFit/lnfl/build/makefile.common`:

.. code:: bash

    osxGNUsgl:
	${MAKE} -f ${MAKEFILE} all P_TYPE=sgl FC_TYPE=gnu \
	PLTFRM=OS_X \
	FC=gfortran \
	FCFLAG="-Wall -frecord-marker=4 -std=legacy" \
	UTIL_FILE=util_gfortran.f

After changing the makefiles, re-run `python setup.py install` to compile LBLRTM and finish installing TelFit.

Remember: you **must** comment out the line in `setup.py` that unpacks the original tar files, or these changes will be overwritten!

Running TelFit
--------------

To run TelFit, you should create a script like in the examples. The key
parts of the script are the inputs to the TelluricFitter class. You
should:

-  Initialize fitter: fitter = TelluricFitter()
-  Define variables to fit: must provide a dictionary where the key is
   the name of the variable, and the value is the initial guess value
   for that variable. Example: fitter.FitVariable({“ch4”: 1.6, “h2o”:
   45.0})
-  Edit values of constant parameters: similar to FitVariable, but the
   variables given here will not be fit. Useful for settings things like
   the telescope pointing angle, temperature, and pressure, which will
   be very well-known. Example: fitter.AdjustValue({“angle”: 50.6})
-  Set bounds on fitted variables (fitter.SetBounds): Give a dictionary
   where the key is the name of the variable, and the value is a list of
   size 2 of the form [lower\_bound, upper\_bound]
-  Import data (fitter.ImportData): Copy data as a class variable. Must
   be given as a DataStructures.xypoint instance
-  Perform the fit: (fitter.Fit): Returns a DataStructures.xypoint
   instance of the model. The x-values in the returned array are the
   same as the data.

.. _my paper: http://adsabs.harvard.edu/abs/2014AJ....148...53G
.. _documentation:  http://telfit.readthedocs.org/en/latest/
