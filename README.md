:globe_with_meridians: 
##geonames-for-postgis 
----------------------

__T__ his utility is for building the geonames database in PostGIS v. 2 in an automated fashion. 

####Assumptions: 

PostgreSQL v. 9.x  
PostGIS v. 2.x

**IMPORTANT:** verify PostgreSQL version (_e.g., 9.1, 9.2, 9.3, etc._) as well as PostGIS 
version (_e.g., 2.0, 2.1, etc._) and modify the variables PGVERSION and PGISVERSION
accordingly.  

For example:
PGVERSION="9.3"
PGISVERSION="2.1"

This is necessary to assure, amongst other things, the correct paths are specified.

This utility is best run as the postgresql superuser (e.g., postgres).  

* enable execute bit

```$ chmod +x build_geonames.sh```

* execute and redirect output to logfile 

```$ /path/to/build_geonames.sh > build_geonames.log 2>&1 ```

I like to have two terminals open in the same directory (i.e., ~postgres) and view 
the log file output as the utility is running using these steps...

**terminal 1**

```   $ rm build_geonames.log```

``` $ touch build_geonames.log```

``` $ tail -f build_geonames.log```

**terminal 2**

```$ /path/to/build_geonames.sh >> build_geonames.log 2>&1```

######NOTES: 
The log file can get large.  To quick check if there were any errors...
```$ grep ERROR build_geonames.log```

If any errors were found, open the log file in you favorite editor and serarch for 'ERROR'.
You will find a descriptive explanation there.

The entire process takes about an hour if none of the data files have been previously downloaded.

The script uses _wget_'s inherint timestamp/filesize checks to see if the file on the 
geoname portal (i.e., [Geonames dump files](http://download.geonames.org/export/dump/)) is newer 
than the equivalent file on the local disk.  If not, it uses the existing file, otherwise it downloads 
and overwrites the older version on your filesystem.  If no new files need to be downloaded the script 
completes in about 20 minutes on a standard (dual/quad) core workstation.

Some intersting queries can be run against the table 'geoname' using PostGIS 2's
inherent _'indexed nearest nieghbor search'_...

For example, to find 10 closest hotels to downtown Boulder and sort them by proximity
with their spatial component(s) as GeoJSON this works well...

```SELECT name, fcode, ST_AsGeoJSON(the_geom)
FROM geoname WHERE fcode = 'HTL'
ORDER BY the_geom <-> st_setsrid(st_makepoint(-105.27997,40.01789),4326)
LIMIT 10; ```

               name               | fcode |                    st_asgeojson                    
----------------------------------|-------|----------------------------------------------------
 Hotel Boulderado                 | HTL   | {"type":"Point","coordinates":[-105.2792,40.0169]}
 Marriott - Boulder               | HTL   | {"type":"Point","coordinates":[-105.2783,40.016]}
 St. Julien Hotel and Spa         | HTL   | {"type":"Point","coordinates":[-105.284,40.016]}
 Boulder University Inn           | HTL   | {"type":"Point","coordinates":[-105.2783,40.0133]}
 Quality Inn And Suites Boulder   | HTL   | {"type":"Point","coordinates":[-105.2687,40.0145]}
 Best Western Golden Buff Lodge   | HTL   | {"type":"Point","coordinates":[-105.2585,40.0178]}
 Millennium Harvest House Boulder | HTL   | {"type":"Point","coordinates":[-105.2588,40.011]}
 Boulder Outlook Hotel & Suites   | HTL   | {"type":"Point","coordinates":[-105.2587,40.0035]}
 Best Western Boulder Inn         | HTL   | {"type":"Point","coordinates":[-105.258,40.0015]}
 The Boulder Broker Inn           | HTL   | {"type":"Point","coordinates":[-105.253,39.9977]}
(10 rows)

Similarly, to find the 5 closest oilfields to Titusville, PA, (site of world's first oil well) and return as KML...

```SELECT name, fcode, ST_AsKML(the_geom) FROM geoname WHERE fcode = 'OILF' ORDER BY the_geom <-> st_setsrid(st_makepoint(-79.666667,41.633333),4326) LIMIT 5; ``

                     name                      | fcode |                                     st_askml                                     
-----------------------------------------------|-------|----------------------------------------------------------------------------------
 Church Run Oil Field                          | OILF  | <Point><coordinates>-79.653109999999998,41.679229999999997</coordinates></Point>
 Shamburg Oil Field                            | OILF  | <Point><coordinates>-79.581159999999997,41.569229999999997</coordinates></Point>
 Pleasantville Oil Field                       | OILF  | <Point><coordinates>-79.541160000000005,41.598390000000002</coordinates></Point>
 Colorado Goodwill Hill Grand Valley Oil Field | OILF  | <Point><coordinates>-79.545330000000007,41.683390000000003</coordinates></Point>
 Sugar Grove Oil Field                         | OILF  | <Point><coordinates>-79.305610000000001,41.978949999999998</coordinates></Point>
 
[Feature Codes for Geonames are here](http://www.geonames.org/export/codes.html).

Leave a comment if you find this useful and suggestions to make it better.
