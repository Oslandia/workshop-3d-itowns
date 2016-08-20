# Districts

We will now import the districts of the city, stored in a GeoJSON file, into our database.

The districts are represented geometrically by 2D polygons.

## Importing the data

The ogr2ogr program allows us to load all of the data contained in the GeoJSON file in our database.

`ogr2ogr -f "PostgreSQL" PG:"dbname=gml" "/home/oslandia/data/montreal/geojson/district.json" -nln wsXX_districts -append`

We can check the result:

```
psql gml
SELECT num, nom FROM wsXX_districts;
```

You should see 34 districts in the table.

## Transforming the data

The district data is in the geographic coordinate system. We will project it in our local srs: epsg:2950.

```
psql gml
SELECT AddGeometryColumn('wsXX_districts', 'geom', 2950, 'MULTIPOLYGON', 2);
UPDATE wsXX_districts SET geom = ST_TRANSFORM(wkb_geometry, 2950);
```
