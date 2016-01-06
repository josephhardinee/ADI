ADI
===

ARM Data Integrator, ADI, is an open source framework that automates the process of retrieving and preparing data for analysis, simplifies the design and creation of output data products produced by the analysis, and provides a modular, flexible software development architecture for implementing algorithms.  These capabilities are supported through the use of a workflow for data integration, a source code generator that produces C, IDL and Python templates, and a graphical interface through which users can efficient define their data input, preprocessing, and output characteristics.  

ADI is used by the 'Atmospheric Radiation Measurement (ARM) Climate Research Facility <http://www.arm.gov>' to process raw data collected from ARM instruments, and to implement scientific algorithms that process one or more of ARM's existing data products to produce new, higher value, data products.

Important Links
===============

- Official source code repository: https://github.com/ARM-DOE/ADI
- HTML documentation: http://engineering.arm.gov/ADI_doc
- ADI Process Configuration Interface https://engineering.arm.gov/pcm/Main.html
- Instructions for installing a rh6 build https://engineering.arm.gov/~gaustad/instructions.html
- Examples: http://engineering.arm.gov/ADI_doc/algorithm.html#algorithm-development-tutorial
- ARM Data Archive for accessing ARM data http://archive.arm.gov

Dependencies
============

ADI has been tested to run under Red Hat Enterprise Linux 2.5 and 2.6.  Development in Python is for 2.7+, but does not include Python 3.  Development in IDL requires IDL 8.2+.  

To install a rhel6 build, the only other dependency not provided as part of the install of the build is:
* 'epl6 <https://fedoraproject.org/wiki/EPEL>'

Required packages provided in epel6 include:
- netcdf-4
- udunits2

The source code is freely availaable, but instructions and Makefiles to support compilation are not.  ARM's development environment uses a in house workflow management system to streamline development activities.  Generic Makefiles are not currently available, but will be provided at a later date. 

The required dependencies to install and compile source code are:
* 'Unidata NetCDF4-C <http://www.unidata.ucar.edu/downloads/netcdf>'
* 'PostgreSQL 8.4 <ttp://www.postgresql.org/download/linux/redhat/>'



Optional Dependences
====================

* 'Multiprotocol file transform library <http://curl.haxx.se/libcurl/>'_7.19.7 is used to access DSDB via a Web Service.

To develop new source code templates beyond those provided:
* 'Cheeta Pthon Template Engine <http://www.cheetahtemplate.org/>'

To develop algorithms in Python 2.7+:
* 'netCDF4 <http://code.google.com/p/netcdf4-python/>'_ 1.0.2+ 
* 'NumPy <http://www.scipy.org>'_ 1.6+

Install
=======

Installation and complilation of the source code is not fully documented, but will be in the future.  
However the latest source code for ADI can be obtained from the GitHub repository,
https://github.com/ARM-DOE/ADI by downloading and unpacking the zip file <https://github.com//ARM-DOE/ADI/archive/ADI_source.tar.gz>  of the source code.


## To Install a Build of ADI:

- Ensure your version of red hat 6 is up to date
- Install epel 6, which contains packages ADI depends on. You can download the rpm (for 64-bit RH6) from  [here](http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm)
- Open the .rpm file with Red Hat's Package Installer and follow the prompts to install it.
- Create a file ARM.repo in /etc/yum.repos.d/ARM.repo with the following entries:
    - `[ARMP6]` 
    - `name=ARM Production RH6 Repository`
    - `baseurl=http://yum.arm.gov/prod6`
    - `gpgcheck=0`
- run the following commands:
    - `$> yum -y groupinstall adi6`
    - `$> yum install netcdf-devel`
- if developing in IDL, idl82 is required.
- if developing in Python, Python 2.7 is required. Python 3 is not supported

## Set Up an Environment for ADI

- Download [adi_home](https://engineering.arm.gov/~gaustad/adi_home.tar.gz). This tarfile contains a directory structure to get you started running ADI quickly. Note that you can also configure the directory structure however you like on your own. 
  
  *A bit of explanation here: ADI's VAPs expect certain directories and environment variables to be present, so to run VAPs, you
  must re-create these directories and environment variables. `adi_home` gets this started for you; it contains all the
  directories needed to run vaps, and provides the required environment variables in the file env_vars. You can use the example1
  vap to verify the ADI installation worked properly.*

- untar the file and copy the resulting `adi_home` directory to somewhere you want to work from, say, `~/Documents` or your home directory in /Users, i.e. `~`.  These instructions will assume you copied it into `~`.
- A core database named dsdb has been provided in /usr/local/share.  Create the directory path ~/adi_home/data/db/sqlite, `mkdir -p ~/adi_home/data/db/sqlite`
- Copy the core dsdb into it`cp ???waiting_for_input????/dsdb/*core.sqlite   ~/adi_home/data/db/sqlite/dsdb.sqlite`
- Enter your home directory and create a new file named .db_connect with the following entries
  - `dsdb_data    sqlite    <your_home_directory>/adi_home/data/db/sqlite/dsdb.sqlite`
  - `dsdb_read    sqlite    <your_home_directory>/adi_home/data/db/sqlite/dsdb.sqlite`
- Enter the untarred `adi_home` directory
- To set the required environment variables for running from ~/adi_home, cat ~/adi_home/env_vars_bash_linux, and copy all the commands into your bash terminal. Later you may want to add these environment variables to your .bash_profile, as THESE WILL HAVE TO BE SET EACH TIME YOU ENTER A NEW TERMINAL.  You will probably want to update ~/adi_home/env_vars_bash_linux to set the locations explicitly so you can run your process from any location rather than from ~/adi_home.
- Setup the example vap process:
  - go into `dev/vap/src/adi_example1/process_dod_defs`
  - import the process by running `db_import_process -a dsdb_data -fmt json adi_example1.json`
  - import the output data definitions by running `db_load_dod -a dsdb_data cpc.json` and `db_load_dod -a dsdb_data met.json`

- Run C version of example1
  - Each time you open a new bash terminal you will need to setup the environment variables
    - `source ~/adi_home/env_vars_bash`
  - to to the ~/adi_home/adi_example1 directory
  - run `make clean; make`. If successful the binary ~/adi_home/dev/vap/bin/adi_example1_vap will be created.
  - run `adi_example1_vap -s sbs -f S2 -b 20110401 -e 20110402 -D 2 -R` this should complete successfully with an exit status of zero.
  - The output data created are:
    ~/adi_home/data/datastream/sbs/sbsadicpcexample1S2.a1/sbsadicpcexample1S2.a1.20110401.000000.cdf
    ~/adi_home/data/datastream/sbs/sbsadimetexample1S2.a1/sbsadimetexample1S2.a1.20110401.000000.cdf

- Run Python version of example1
  - Each time you open a new bash terminal you will need to setup the environment variables
    - `source ~/adi_home/env_vars_bash`
  - go to the ~/adi_home/adi_example1_py directory 
  - run `python adi_example1_vap.py -s sbs -f S2 -b 20110401 -e 20110402 -D 2 -R'
  - The output data created is the same as for the C run:
    ~/adi_home/data/datastream/sbs/sbsadicpcexample1S2.a1/sbsadicpcexample1S2.a1.20110401.000000.cdf
    ~/adi_home/data/datastream/sbs/sbsadimetexample1S2.a1/sbsadimetexample1S2.a1.20110401.000000.cdf

- Run IDL version of example1
  - If its first time you running an ADI process, from an IDL prompt:
    - IDL> pref_set, 'idl_path','<IDL_DEFAULT>:/usr/local/lib',/commit
    - IDL> pref_set, 'idl_dlm_path','<IDL_DEFAULT>:/usr/local/lib',/commit 
  - Each time you open a new terminal you will need to setup idl and the environment variables
    - `source /Applications/exelis/idl84/bin/idl_setup.bash`
    - `source ~/adi_home/env_vars_bash`
  - go to the ~/adi_home/adi_example1_idl directory 
  - run $> `idl -e "adi_example1_vap" -args -s sbs -f S2 -b 20110401 -e 20110402 -D 2'
  - or to run in debug mode 
    - $> `idl -args -s sbs -f S2 -b 20110401 -e 20110402 -D 2 -R`
    - $IDL> adi_example1_vap
  - The output data created is the same as for the C run:
    ~/adi_home/data/datastream/sbs/sbsadicpcexample1S2.a1/sbsadicpcexample1S2.a1.20110401.000000.cdf
    ~/adi_home/data/datastream/sbs/sbsadimetexample1S2.a1/sbsadimetexample1S2.a1.20110401.000000.cdf

## To Add More Process Definitions to the DSDB:
The process definitions for adi_example1 have been included in your adi_home area. To run additional VAPs against your local database, you will need to import their process information.

- Get the process definition from the PCM
  - Go to the <a href="https://engineering.arm.gov/pcm/Main.html" target="_blank">Processing Configuration Manager</a>
    and select the processes tab on the left hand side
  - Type the name of the process you want in the filter at the bottom, or find it by scrolling through the list
  - Double click the name of the process to bring it up on the right hand side
  - Click *Text Export/Import* in the lower right corner, and copy the text that appears to a file on your machine
- Set your enviornment variables as specified in `env_vars_bash` from the last section 
- run `db_import_process` for the definition you retrieved
  - `db_import_process -a dsdb_data -fmt json <process definition file name>`
- Load the DODs nessecary to run this process. The DODs used by a process are listed on that process's page in the PCM.
  - Load the DOD into the PCM datastream viewer.
  - Select the JSON format from the green export DOD icon at the top of the page to copy the DOD to your clipboard. 
    Copy this into a file on your local machine
  - Load the dods into the local database
    - `db_load_dod -a dsdb_data <dod file>`
