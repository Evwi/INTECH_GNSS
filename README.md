# IMPORTANT

CDDIS is an important GNSS data archive. Because of the way that CDDIS has 
implemented security restrictions, we have had to change our download access. 
For this reason we strongly urge that you install **wget** on your machine.
You will only have very limited analysis abilities without it.

# NEWS

I have added defaults so you don't have to think quite so much. The defaults are that  
you are using GPS and have a fairly standard site (not super tall, < 20 meters).
Pleae note changes to **rinex2snr**, **quickLook**, and **gnssir**.
There are still optional inputs that allow you to vary things.  

# gnssrefl

This package is a new version of my GNSS interferometric reflectometry (GNSS_IR) code. 

The main difference bewteen this version and previous versions is that I am
attempting to use proper python packaging rules, LOL. I have separated out the main
parts of the code and the command line inputs so that you can use the gnssrefl libraries
yourself or do it all from the command line. This should also - hopefully - make
it easier for the production of Jupyter notebooks. The latter are to be developed
by UNAVCO with NASA GNSS Science Team funding.

# For those of you who don't like reading documentation

I recommend you use the web app [I developed](https://gnss-reflections.org). It 
can show you how the technique works without installing any code. It also picks up 
the data for you and provides results in less than 10 seconds. 

# For those of you who don't like python

I have a [working matlab version on github](https://github.com/kristinemlarson/gnssIR_matlab_v3), 
but I will not be updating it.

# Overview Comments

The goal of this python repository is to help you compute (and evaluate) GNSS-based
reflectometry parameters using geodetic data. This method is often
called GNSS-IR, or GNSS Interferometric Reflectometry. There are three main codes:


* **rinex2snr** translates RINEX files into SNR files needed for analysis.

* **gnssir** computes reflector heights (RH) from GNSS data.

* **quickLook** gives you a quick (visual) assessment of a file without dealing
with the details associated with **gnssir**.

There is also a RINEX download script **download_rinex** and an orbit download script 
**download_orbits**, but you are not required to use them.

The rest of this file is about how to install and run the code. *It is not a class about 
reflectometry.* If you are unsure about why various restrictions are being applied, you need
to read Roesler and Larson (2018) and similar. I am committed in principle to set up some online
courses to teach people about reflections, but funding for these couress is not in hand at the moment.  

# Environment Variables 

You should define three environment variables:

* EXE = where various RINEX executables will live.

* ORBITS = where the GPS/GNSS orbits will be stored. They will be listed under directories by 
year and sp3 or nav depending on the orbit format.

* REFL_CODE = where the reflection code inputs (SNR files and instructions) and outputs (RH)
will be stored (see below). Both SNR files and results will be saved here in year subdirectories.

However, if you do not do this, the code will assume your local working directory (where you installed
the code) is where you want everything to be. The orbits, SNR files, and periodogram results are stored in 
directories in year, followed by type, i.e. snr, results, sp3, nav, and then by station name.

# Python

If you are using the version from gitHub:

* make a directory, cd into that directory, set up a virtual environment, a la python3 -m venv env 
* activate your virtual environment
* git clone https://github.com/kristinemlarson/gnssrefl 
* pip install .

If you use the PyPi version:  

* make a directory, cd into that directory, set up a virtual environment 
* activate the virtual environment
* pip install gnssrefl

To use **only** python codes, you will need to be sure that your RINEX files are uncompressed (i.e.
not using Hatanaka compression, which end in a d instead of an o). Since **many** archives use 
Hatanaka compression, this will significantly
limit what you can do. Second thing to know is that you should use the -fortran False flag 
when you make SNR files because the default behavior is to assume you are using fortran 
translators (gnssSNR.e or gpsSNR.e). Finally, you will not be able to use RINEX 3 files because
I rely on the gfzrnx RINEX3 to RINEX2 translator.

# Non-Python Code 

All executables should be stored in the EXE directory.  If you do not define EXE, 
it will look for them in your local working directory.  The Fortran translators are 
much faster than using python. But if you don't want to use them, 
they are optional, that's fine. FYI, the python version is slow 
not because of the RINEX - it is because you need to calculate
a crude model for satellite coordinates in this code. And that takes cpu time....

* Required Translator for compressed RINEX files. CRX2RNX, http://terras.gsi.go.jp/ja/crx2rnx.html

* Optional Fortran RINEX Translator for GPS, the executable must be called gpsSNR.e, https://github.com/kristinemlarson/gpsonlySNR

* Optional Fortran RINEX translator for multi-GNSS, the executable must be called gnssSNR.e, https://github.com/kristinemlarson/gnssSNR

* Optional datatool, teqc, is highly recommended.  There is a list of static executables at the
bottom of [this page](http://www.unavco.org/software/data-processing/teqc/teqc.html)

* Optional datatool, gfzrnx is required if you plan to use the RINEX 3 option. Executables available from the GFZ,
http://dx.doi.org/10.5880/GFZ.1.1.2016.002


# rinex2snr - making SNR files from RINEX files

The international standard for sharing GNSS data is called the RINEX format.
A RINEX file has extraneous information in it (the data used for positioning, LOL) - and it 
does not provide some of the information needed for reflectometry (elevation and azimuth angles). 
The first task you have is to translate a from RINEX into what I will call the SNR format - and 
to calculate those geometric angles. For the latter you will need an **orbit** file. **rinex2snr**
will go get an orbit file for you. You can override the default orbit choice by selections given below.

There is no reason to save ALL the RINEX data as the reflections are only useful at the lower elevation
angles. The default is to save all data with elevation lower than 30 degrees (this is called SNR format 66).
Another SNR choice is 99, which saves elevation angle data between 5 and 30.  

There are rules for naming RINEX files. I **require** that RINEX (version 2) be lowercase.
The filename must be 12 characters long (ssssddd0.yyo), where ssss is station name, 
ddd is day of year, followed by a zero, yy is the two character year and o stands for observation. 
If you have installed the CRX2RNX code, you can also provide a compressed RINEX format file, which ends in a d.
I think my code allows you to gzip the RINEX files if you are providing them.


You can run **rinex2snr** at the command line. You required inputs are:

- station name
- year
- day of year

If you installed the fortran translator gpsSNR.e, a sample call for a station called p041, restricted 
to GPS satellites, on day of year 132 and year 2020 would be:

*rinex2snr p041 2020 132*

If the RINEX file for p041 is in your local directory, it will translate it.  If not, 
it will check four archives (unavco, sopac, cddis, and sonel) to find it. 

If you did not install a fortran translator, the command would be:

*rinex2snr p041 2020 132 -fortran False* 

The code will also search ga (geoscience Australia), nz (New Zealand), 
ngs, and bkg if you invoke -archive, e.g.

*rinex2snr tgho 2020 132 -archive nz*

What if you want to run the code for all the data for a given year?  

*rinex2snr tgho 2019 1  -archive nz -doy_end 365* 
 
If your station name has 9 characters (lower case please), the code assumes you are looking for a 
RINEX 3 file. However, my code will store the SNR data using the normal
4 character name. **You must install the gfzrnx executable that translates RINEX 3 to 2 to use RINEX 3 files
in my code.** *rinex2snr* currently only looks for RINEX 3 files at CDDIS (30 sec) and UNAVCO (15 sec).  There 
are more archive options in **download_rinex** and someday I will merge these. If you do have your own RINEX 3
files, I use the community standard, that is upper case except for the extension (which is rnx).  I know, it is weird.

Here are some examples for RINEX 3 conversions:

*rinex2snr onsa00swe 2020 298*

*rinex2snr at0100usa 2020 55*

*rinex2snr mdo100usa 2020 290*

*rinex2snr mkea00usa 2020 290*

*rinex2snr p36000usa 2020 290*

The snr options are mostly based on the need to remove the "direct" signal. This is 
not related to a specific site mask and that is why the most frequently used 
options (99 and 66) have a maximum elevation angle of 30 degrees. The
azimuth-specific mask is decided later when you need to run **gnssir**.  The SNR choices are:

- 66 is elevation angles less than 30 degrees (the default)
- 99 is elevation angles of 5-30 degrees  
- 88 is elevation angles of 5-90 degrees
- 50 is elevation angles less than 10 degrees (good for very tall sites, high-rate applications)


*orbit file options for general users:*

- gps : will use GPS broadcast orbits **the default**
- gps+glos : will use JAXA orbits which have GPS and Glonass (usually available in 48 hours)
- gnss : will use GFZ orbits, which is multi-GNSS (available in 3-4 days?)

*orbit file options for experts:*

- nav : GPS broadcast, perfectly adequate for reflectometry. 
- igs : IGS precise, GPS only
- igr : IGS rapid, GPS only
- jax : JAXA, GPS + Glonass, within a few days
- gbm : GFZ Potsdam, multi-GNSS, not rapid
- grg: French group, GPS, Galileo and Glonass, not rapid
- wum : Wuhan, multi-GNSS, not rapid

Other questions:

- What if you do not want to install the fortran translators?  Use -fortran False on the command line.

- What if you are providing the RINEX files and you don't want the code to search for 
the files online? Use -nolook True

There is a **rate** command line input that has two values, high or low. However, if you invoke high,
it currently only looks at unavco. Please beware - it takes a long time to download a 
highrate GNSS RINEX file (even when it is compressed). 
And it also takes a long time to compute orbits for it (and thus create a SNR file).

# quickLook 

Before using the **gnssir** code, I recommend you try **quickLook**. This allows you
to quickly test various options (elevation angles, frequencies, azimuths).
The required inputs are station name, year, and doy of year. 

If the SNR file has not been previously stored, you can provide a properly named RINEX file
(lowercase only) in your working directory. If it doesn't find a file in either of these places, it
will try to pick up the RINEX data from various archives (unavco, sopac, sonel, and cddis) and translate it for
you into the correct SNR format (note: this feature might make use of the Fortran translators). 
There are stored defaults for analyzing the spectral characteristics of the SNR data. 
If you want to override those, use *quickLook -h*

Here are some examples:

*quickLook p041 2020 150*  (uses defaults)

*quickLook gls1 2011 271*  (uses defaults)

The defaults are inappropriate for many sites. For example, smm3 is about 15 
meters tall. If you use the defaults you won't see anything useful. Here I am overriding
the defaults to require that the analysis code only look at reflector heights between 
8 and 20 meters. (I am also telling it to use snr format 99).

*quickLook smm3 2018 271 -snr 99 -h1 8 -h2 20*  

# gnssir

This is the main driver for the reflectometry code.  

You need a set of instructions which can be made using **make_json_input**.  
At a minimum **make_json_input** needs the station name (4 char), the latitude (degrees), 
longitude (degrees) and ellipsoidal height (meters). This location does not 
have to be cm-level for the reflections code.
Within a few hundred meters is sufficient.  


*make_json_input p101 41.692 -111.236 2016.1* 

It will use defaults for other parameters if you do not provide them. Those defaults 
tell the code an azimuth and elevation angle mask (i.e. which directions you want 
to allow reflections from), and which frequencies you want to use, and various quality control metrics. 
Right now the default frequencies are GPS L1 and L2C and a peak to noise ratio of 2.7 is set.
This is fine for water, but I would suggest higher for snow (3.5). GPS L5 provides excellent data,
but very few geodesists track it, so it is not currently a default. The output file will be put in 
$REFL_CODE/input/p101.json. You should look at it to get an idea of the kinds of inputs the code will be using.

You can edit the json file directly, or you can set some of the parameters from the command line.
For example, if you only want to use elevation angles between 5 and 10 degrees:

*make_json_input p101 41.692 -111.236 2016.1 -e1 5 -e2 10* 

Things that are helpful to know for the json and commandline inputs:

*Names for the GNSS frequencies*

- 1,2,5 are GPS L1, L2, L5
- 20 is GPS L2C 
- 101,102 Glonass L1 and L2
- 201, 205, 206, 207, 208: Galileo frequencies
- 302, 306, 307 : Beidou frequencies

*Reflection parameters settings in the json file:*

- e1 and e2 are the min and max elevation angle, in degrees
- minH and maxH are the min and max allowed reflector height, in meters
- desiredP, desired reflector height precision, in meters
- PkNoise is the periodogram peak divided by the periodogram noise ratio.  
- reqAmp is the required periodogram amplitude value, in volts/volts
- polyV is the polynomial order used for removing the direct signal
- freqs are selected frequencies for analysis
- delTmax is the maximum length of allowed satellite arc, in minutes
- azval are the azimuth regions for study, in pairs (i.e. 0 90 270 360 means you want to evaluate 0 to 90 and 270 to 360).

*Other json inputs:*

- wantCompression, boolean, compress SNR files
- screenstats, boolean, whether minimal periodogram results come to screen
- refraction, boolean, whether simple refraction model is applied.
- plt_screen: boolean, whether SNR data and periodogram are plotted to the screen
- seekRinex: boolean, whether code looks for RINEX at an archive


Simple example for my favorite GPS site [p041](https://spotlight.unavco.org/station-pages/p042/eo/scientistPhoto.jpg)

- *make_json_input p041 39.949 -105.194 1728.856* (use defaults and write out a json instruction file)
- *rinex2snr p041 2020 150* (pick up and translate RINEX file from unavco, uses snr=66 default)
- *gnssir p041 2020 150* (calculate the reflector heights) 
- *gnssir p041 2020 150 -fr 5 -plt True* (override defaults, only look at L5 SNR data, and periodogram plots come to the screen)

Where are the files for this example?

- json is stored in $REFL_CODE/input/p041.json
- SNR files are stored in $REFL_CODE/2020/snr/p041
- Reflector Height (RH) results are stored in $REFL_CODE/2020/results/p041
- I do not save RINEX files. In fact, my code deletes them.

If you want a multi-GNSS solution, you need make a new json file and use multi-GNSS orbits. And the RINEX you select 
must have multi-GNSS SNR observations in it. p041 currently has multi-GNSS data in the RINEX file, so you 
can use it as a test.

- *make_json_input p041 39.949 -105.194 1728.856 -allfreq True* 
- *rinex2snr p041 2020 151 -orb gnss* 
- *gnssir p041 2020 151 -fr 201 -plt True* (look at the lovely Galileo L1 data) 

What should the periodogram plots look like? Until we have Jupyter notebooks, I 
recommend you look at 
[the paper I wrote with Carolyn Roesler](https://link.springer.com/article/10.1007/s10291-018-0744-8) 
or the [question section of my web app.](https://gnss-reflections.org). Note that a failed
arc is shown as gray in the periodogram plots. And once you know what you are doing (have picked
the azimuth and elevation angle mask), you won't be looking at plots anymore.

# Bugs/Features I know about 

I have been using **teqc** to reduce the number of observables and to decimate. I have removed the former 
because it unfortunately- by default - removes Beidou observations in Rinex 2.11 files. If you request decimation 
and fortran is set to True, unfortunately this will still occur. I am working on removing my 
code's dependence on **teqc**.

If there is interest, I will ask UNAVCO to implement the Fortran translation code automatically (i.e. download
and compile it for you as part of the pypi install). But doing this myself is well beyond my skillset. 

No phase center offsets have been applied to these reflector heights. While these values are relatively small,
we do plan to remove them in subsequent versions of the code.

The L2C and L5 satellite lists are not time coded as they should be. I currently use a list from 2020.

# Helper Codes

**download_rinex** can be useful if you want to download RINEX v2 or 3 files (using the version flag) without using 
the reflection-specific codes. Sample calls:

- *download_rinex p041 2020 6 1* downloads the data from June 1, 2020

- *download_rinex p041 2020 150 0* downloads the data from day of year 150 in 2020

- *download_rinex p041 2020 150 0 -archive sopac* downloads the data from sopac archive on day of year 150 in 2020


**daily averages** is a helper code for cryosphere people interested in daily snow 
accumulation. It can be used for lake levels. It is not for tides!

**download_orbits** does what it sounds like. Feel free to use it. 

**ymd** is for those annoying days when you don't know what day of year is October 15.

# Publications

There are A LOT of publications about GPS and GNSS interferometric reflectometry.
If you want something with a how-to flavor, try this paper, 
which is [open option](https://link.springer.com/article/10.1007/s10291-018-0744-8)

Also look to the publications page on my [personal website](https://kristinelarson.net/publications)

# How can I import the libraries in this package?

I will be adding more documentation and examples here.

If you wanted to run the gnssir code without the command line interface, here is 
an example for station p041 where the json instructions exist and the SNR file has already been created.  

```sh
# my internal libraries you need
import gnssrefl.gps as g
import gnssrefl.gnssir as guts


station = 'p041' 
extension = ''  

# instructions for the Lomb Scargle Periodogram
lsp = guts.read_json_file(station, extension)

# set the year, doy, and type of snr file
year = 2020; doy = 150; snr_type =  99 
guts.gnssir_guts(station,year,doy, snr_type, extension, lsp)
```


# Acknowledgements

People that helped me with this code include Radon Rosborough, Joakim Strandberg, and Johannes Boehm. 
I also thank Peter Shearer and Lisa Tauxe for some very nice Python lecture notes.

Updated September 25, 2020

Kristine M. Larson
