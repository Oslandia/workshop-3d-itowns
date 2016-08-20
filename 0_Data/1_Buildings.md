# Buildings

In this section, we will import building data contained in CityGML files into our postGIS database.

Here are the instructions for extracting the geometries from the files and importing them into a postGIS database.

## Preparing the database

We need to create a new table for the buildings.

`psql gml -c "CREATE TABLE wsXX_montreal(gid SERIAL PRIMARY KEY, geom GEOMETRY('POLYHEDRALSURFACEZ', 2950))"`

## Importing the data

We use the citygml2pgsql script to extract geometries from cityGML files into SQL queries.

You can safely ignore the warnings.

`citygml2pgsql /home/oslandia/data/montreal/gml/data/pmr/PMR01_2013.gml 2 2950 geom wsXX_montreal | psql gml`

* /home/oslandia/data/montreal/gml/data/pmr/PMR01_2013.gml is the CityGML file to parse
* 2 is the level of detail that we wish to load
* 2950 is the SRS of the data
* geom is the name of the column that will contain the geometry
* wsXX_montreal is the name of the table in which the data will be imported

This process can be pretty long. To save some time, the other SQL scripts have already been created.

`sh /home/oslandia/data/montreal/scripts/load_citygml.sh wsXX_montreal`

You can check that the data has been correctly loaded:

```
psql gml
SELECT count(*) FROM wsXX_montreal;
```

The result should be 80127.
