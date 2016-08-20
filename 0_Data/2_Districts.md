# Districts

We will now import the districts of the city, stored in a GeoJSON file, into our database.

The districts are represented geometrically by 2D polygons.

## Importing the data

The ogr2ogr program allows us to load all of the data contained in the GeoJSON file in our database.

`ogr2ogr -f "PostgreSQL" PG:"dbname=pc_montreal" "/home/oslandia/data/montreal/geojson/district.json" -nln yourschema.districts -append`

We can check the result:

```
psql pc_montreal
SELECT num, nom FROM yourschema.districts;
```

You should see 34 districts in the table.

## Transforming the data

The district data is in the geographic coordinate system. We will project it in our local srs: epsg:2950.

```
psql pc_montreal
SELECT AddGeometryColumn('yourschema.districts', 'geom', 2950, 'MULTIPOLYGON', 2);
UPDATE yourschema.districts SET geom = ST_TRANSFORM(wkb_geometry, 2950);
```
