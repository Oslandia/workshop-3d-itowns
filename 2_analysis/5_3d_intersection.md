## 3D intersection

The 3D buildings extracted from CityGML data are noisy: it is hard to get correct volumes out of them. This is more a soup of polygons than correctly connected components.
Moreover, computing 3D functions on detailed polygons may take time (especially when 20 people are doing the same thing on the same server during this workshop session).

We are going to approximate the 3d buildings with some 3d functions.

```sql
create table yourschema.approx (
       gid serial primary key,
       geom2d geometry('polygon', 2950),
       geom geometry('polyhedralsurfaceZ', 2950),
       minz real,
       height real
       );
```

* geom2d will represent an approcimation of the building projected on the ground
* we will use geom as an approximate of the 3D buliding
* height will be the height of the building

```sql
insert into yourschema.approx (gid, geom2d, minz, height) select
       gid,
       st_convexhull(st_force2d(st_forcecollection(geom))),
       st_zmin(geom),
       st_zmax(geom)-st_zmin(geom)+1
       from yourschema.montreal;
```

* st_forcecollection will transform a 'PolyhedralSurface' to a set of Polygon, because lots of postgis functions do not support (yet) polyhedral surfaces.
* st_force2d will project the point on the ground
* st_convexhull will compute a convex enveloppe

```sql
update yourschema.approx set geom = st_extrude(geom2d, 0, 0, height);
```

* st_extrude will make a volume out of a 2d polygon and a vector of extrusion

Then add this layer to your settings.py file

```python
cities.CITIES["TODO_ID"] = {
    "tablename": "TODO_TABLE",
    "extent": [[297250,5038250], [303750,5044750]],
    "maxtilesize": 2000,
    "srs":"EPSG:2950",
    "featurespertile":20
}
```

Then run:

`python /home/oslandia/building-server/building_server/processDB.py yourlayer`

