geonames-for-postgis
====================

This utility is for building the geonames database in PostGIS v. 2 in an automated fashion. 

Assumptions:

PostgreSQL v. 9.+
PostGIS v. 2.+

This utility is best run as the postgresql superuser (e.g., postgres).  

<enable execute bit>

$ chmod +x build_geonames.sh

<execute and redirect output to logfile>

$ /path/to/build_geonames.sh > build_geonames.log 2>&1 

I like to have two terminals open in the same directory (i.e., ~postgres) and view 
the log file output as the utility is running using these steps...

TERMINAL 1
$ rm build_geonames.log
$ touch build_geonames.log
$ tail -f build_geonames.log

TERMINAL 2
$ /path/to/build_geonames.sh >> build_geonames.log 2>&1

NOTES: 
The log file can get large.  To quick check if there were any errors...
$ grep ERROR build_geonames.log

If any errors were found, open the log file in you favorite editor and serarch for 'ERROR'.
You will find a descriptive explanation for the error there.

The entire script takes about an hour if none of the data files have been previously downloaded.

The script uses wget's inherint timestamp/filesize checks to see of the file on the 
geoname server (i.e., http://download.geonames.org/export/dump/) is newer than the 
file on the local disk.  If its not, it uses the existing file, otherwise it downloads 
and overwrites the existing file.  If no new files need to be the script completes 
in about 20 minutes.

Some intersting queries can be run against the table 'geoname' using PostGIS 2's
inherent 'indexed nearest nieghbor search'...

For example, to find 10 closest hotels to downtown Boulder and sort them by proximity
this query works well...

SELECT name, fcode
FROM geonames WHERE fcode = 'HTL'
ORDER BY geom <-> st_setsrid(st_makepoint(-105.27997,40.01789),4326)
LIMIT 10; 

Have fun!
