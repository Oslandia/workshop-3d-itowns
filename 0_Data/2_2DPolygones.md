# 2D polygons

We will now import the districts and land usage of the city, stored in a GeoJSON and shape file, into our database.

## Importing the data

The ogr2ogr program allows us to load all of the data contained in the GeoJSON file in our database.

`ogr2ogr -f "PostgreSQL" PG:"dbname=pc_montreal" "/home/oslandia/data/montreal/geojson/district.json" -nln yourschema.districts`

The shape file can be imported using the shp2pgsql command line.

`shp2pgsql -I -s 2950 "/home/oslandia/data/montreal/shp/affectation-du-sol/Affectation du sol 20140729.shp" yourschema.landuse | psql pc_montreal`


We can check the result:

```
psql pc_montreal
SELECT num, nom FROM yourschema.districts;
SELECT DISTINCT categorie FROM yourschema.landuse;
```

You should see 34 districts and 9 categories in the tables.

## Transforming the data

The district data is in the geographic coordinate system. We will project it in our local srs: epsg:2950.

```
psql pc_montreal
SELECT AddGeometryColumn('yourschema', 'districts', 'geom', 2950, 'MULTIPOLYGON', 2);
UPDATE yourschema.districts SET geom = ST_TRANSFORM(wkb_geometry, 2950);
```
