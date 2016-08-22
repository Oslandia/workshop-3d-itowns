## 3D intersection

The 3D buildings extracted from CityGML data are noisy: it is hard to get correct volumes out of them. This is more a soup of polygons than correctly connected components.
Moreover, computing 3D functions on detailed polygons may take time (especially when 20 people are doing the same thing on the same server during this workshop session).

We are going to approximate the 3d buildings thanks to postgis 3d functions.

```sql
create table yourschema.approx (
       gid serial primary key,
       geom2d geometry('polygon', 2950),
       geom geometry('polyhedralsurfaceZ', 2950),
       height real
       );
```

* geom2d will represent an approximation of the building's extent projected on the ground
* we will use geom as an approximate of the 3D building
* height will be the height of the building

```sql
insert into yourschema.approx (gid, geom2d, height) select
       gid,
       st_convexhull(st_force2d(st_forcecollection(geom))),
       st_zmax(geom)-st_zmin(geom)+1
       from yourschema.montreal;
```

* st_forcecollection will transform a 'PolyhedralSurface' to a set of polygons, because lots of postgis functions do not support (yet) polyhedral surfaces.
* st_force2d will project the points on the ground
* st_convexhull will compute a convex envelope

```sql
update yourschema.approx set geom = st_extrude(geom2d, 0, 0, height);
```

* st_extrude will make a volume out of a 2d polygon and a vector of extrusion

Then add this layer to your settings.py file

```python
cities.CITIES["yourlayer"] = {
    "tablename": "yourschema.approx",
    "extent": [[297250,5038250], [303750,5044750]],
    "maxtilesize": 2000,
    "srs":"EPSG:2950",
    "featurespertile":20
}
```

Then run:

`python /home/oslandia/building-server/building_server/processDB.py yourlayer`

## Flood (rough) simulation

Get parts of the buildings above a flood of 30 meters.

```sql
create table yourschema.flood as
with
-- the whole 'plane'
e as (select st_extent(geom) as geom from mercier.approx),
-- a big box at 30 meters above the ground
ee as (select st_translate(st_extrude(e.geom, 0, 0, 300), 0, 0, 30) as geom from e)
select gid,
       st_3dintersection(a.geom, ee.geom) as geom
from ee, mercier.approx as a
-- start with a small subset (50m around a point)
-- don't go further than 200 m (too slow)
where geom2d && st_makeenvelope(299765-50,5040838-50,299765+50,5040838+50)
```

Exercise: there is also a function named `st_3ddifference(geom1, geom2)`, use it instead of `st_3dintersection`
