# COVID-19 - time-series utilities

This repo contains several utilities for wrangling COVID-19 data from John Hopkins University.

* A shell script that converts the JHU COVID-19 daily-report data to a time-series database using TimescaleDB.
* Several OpenRefine projects that "unpivots" (normalizes) JHU's COVID-19 time-series csv file

## Requirements
* For time-series ingestion script
  - A working instance of [TimescaleDB](https://docs.timescale.com) in PostgreSQL v10+
  - [csvkit](https://csvkit.readthedocs.io/en/latest/)
  - [git](https://git-scm.com/)
  - Unix/Linux operating system with bash 
* OpenRefine time-series automation
	- [OpenRefine](http://openrefine.org) - installed automatically
  - [openrefine-batch](https://github.com/opencultureconsulting/openrefine-batch) - included
  - a [Geocode.earth](https://geocode.earth) API key - [install](https://github.com/pelias/pelias) or [free trial](https://geocode.earth/invite/request?referrer=datHere). Used to enrich geographic data.

## Cloning
A note on cloning this repo, since the COVID19 directory is a git submodule:

* after cloning, you must initiate the submodule. In the top level directory for the project, run `git submodule init` and `git submodule update` to clone the JHU Repo as a submodule 

## Content
The files in this directory and how they're used:

* `covid-19_ingest.sh`: Bash script to read daily-report data from [JHU COVID-19 Github](https://github.com/CSSEGISandData/COVID-19), and upsert the data into TimescaleDB.
* `schema.sql`: Data definition (DDL) to create the necessary tables & hypertables.
* `covid-refine`: OpenRefine automation to create fully normalized COVID-19 time-series data, with expanded geographic features 

## Using the Timescale covid-19_ingest script
1. Create a TimescaleDB instance - [download](https://docs.timescale.com/latest/getting-started/installation) or [signup](https://www.timescale.com/cloud-signup)
2. Create a database named `covid_19`, and an application user `covid_19_user`

```
  psql
  create database covid_19;
  create user covid_19_user WITH PASSWORD 'your-password-here';
  alter database covid_19 OWNER TO covid_19_user;
  \quit
```

3. Run `schema.sql` as the `covid_19_user`. VACUUM/ANALYZE require owner privs 

   `psql -U covid_19_user -h <the.server.hostname> -f schema.sql covid_19`
   
   
4. Install csvkit

    - Ubuntu: `sudo apt-get install csvkit`
    - MacOS: Using [homebrew](https://brew.sh/) run `brew install csvkit`

5. Using a text editor, replace the environment variables for `PGHOST`, `PGUSER` and `PGPASSWORD` in `covid-19_ingest.sh`

6. Run the script 

   `bash covid-19_ingest.sh`

7. (OPTIONAL) add shell script to crontab to run daily

8. Be able to slice-and-dice the data using the full power of PostgreSQL along with Timescale's time-series capabilities!

## Using the COVIDrefine 
See the detailed [README](covid19-refine/README.md).

## NOTES
 - the JHU COVID-19 repository is a git submodule. This was required to automate getting the latest data from their repo.
 - the script will only work in \*nix environment (Linux, Unix, MacOS)
 - the script creates a hidden directory called `~/.covid-19` in your home directory where it stores the date of the last csv it processed in a file named `lastcsvprocessed`.  Delete that file to process all the files from the beginning, or change the date in the file to start processing files AFTER the entered date.  
 - if you need the normalized COVID-19 time-series data on another operating system, use the OpenRefine projects.  OpenRefine is cross-platform and has pre-built binaries for Windows, Mac and Linux.

## TODO
 - add postgis to schema for spatial analysis
 - use [postgREST](http://postgrest.org) to add a REST API in front of TimescaleDB database
 - create a Grafana dashboard
 - create a Carto visualization
 - create a Superset visualization

 ## ACKNOWLEDGEMENTS
  - thanks to Avtar Sewrathan (@avthars), Prashant Sridharan (@CoolAssPuppy) and Mike Freedman (@mfreed) at [Timescale](https://timescale.com) for their help & support to implement this project from idea to implementation in 5 days!
  - thanks to Julian Simioni (@orangejulius) at [Geocode.earth](https://geocode.earth) for allowing us to use the Geocode.earth API!


Shield: [![CC BY-SA 4.0][cc-by-sa-shield]][cc-by-sa]

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0
International License][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
[cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg
