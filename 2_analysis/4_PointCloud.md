# Point Cloud

As seen previously, a point cloud is already stored for Montreal in the *pa*
table within the *pc_montreal* database. You may access the database with
`psql pc_montreal`.

You can view it either in [iTowns](http://3d.oslandia.com/) or in [PotreeViewer](http://3d.oslandia.com/potree/examples/lopocs.html).

## PGPointCloud basics

PGPointCloud does not store points directly in the database. Instead, to relieve postgres from the load caused by the massive amount of points, it stores patches of points.

To count the number of patches within the *pa* table:

`SELECT count(*) from pa;`

Each patch is made of several hundred of neighboring points thanks to the chipper filter of PDAL. You can retrieve the size of a patch thanks to the *pc_numpoints* function:

`SELECT pc_numpoints(pa) from pa LIMIT 1;`

Note that it's possible to store PGPointCloud patches with 4 kinds of compression : none, dimensional, GHT or LAZ.

Each point has a number of dimension, such as 3D coordinates (x,y,z), classification or color.

Using pc_summary, we can have the definition of these dimensions as well as additional informations concerning the patch (min, max, average values for each dimension, total number of points...):

`SELECT pc_summary(pa) FROM pa WHERE id = 1 LIMIT 1;`

We can retrieve the first point of the patch with id = 1 by exploding the patch:

`SELECT pc_astext(pc_explode(pa)) FROM pa WHERE id = 1 LIMIT 1;`

This returns a JSON object, containing two attributes:
* **pcid**, which is a reference to the type of formating of the point
* **pt**, the value for each of the dimensions of the point

*pc_get* allows us to retrieve the value of one of the dimension:

```sql
WITH tmp as (
    SELECT
        pc_explode(pa) as pts
    FROM pa
    WHERE id = 3225684
)
SELECT pc_get(pts, 'Classification') AS classification,
       -- note that once the point is converted to a geometry we
       -- can use any PostGIS function, like getting the z value
       ST_Z(pts::geometry(pointz, 2950)) as z
FROM tmp;
```

## Simple queries

### Average altitude of a patch

There is at least 2 ways to retrieve the average altitude of a patch.

Either we use the *pc_summary* function:

```sql
WITH tmp as (
    SELECT json_array_elements(pc_summary(pa)::json->'dims') as dims
    FROM pa
    WHERE id = 1
)
SELECT dims->'stats'->'avg'
FROM tmp
WHERE dims->>'name' = 'Z';
```

or we compute it:

```sql
WITH tmp as (
    SELECT pc_get(pc_explode(pa), 'z') as z
    FROM pa
    WHERE id = 1
)
SELECT avg(z)
FROM tmp;
```

### Filtering the patches

We can filter spatially the patches by using *pc_intersects*:

```sql
SELECT count(*)
FROM pa
WHERE
 pc_intersects(pa,
     st_geomfromtext('polygon
     ((298250 5039250,
     298250 5039350,
     298350 5039350,
     298350 5039250,
     298250 5039250))',
     '2950'));
```
*pc_filterequals* outputs a patch with only the points that match the filter. For example, we can filter based on the point's classification.

The points of our dataset are classified into three categories: ground (2), vegetation (5) and building (6).

The following query counts the number of points of the vegetation type.

```sql
SELECT sum(pc_numpoints(pc_filterequals(pa, 'Classification', 5)))
FROM pa
WHERE
 pc_intersects(pa,
     st_geomfromtext('polygon
     ((298250 5039250,
     298250 5039350,
     298350 5039350,
     298350 5039250,
     298250 5039250))',
     '2950'));
```

## Analysis

In this section, we will compute the height of a building.

Start by selecting a roof in iTowns in a place where the point cloud is defined.

In this example, we chose a roof whose gid is 60449.

The first step consists in selecting all the patches that intersect our roof.

We start by selecting the polygons of the roof.

```sql
SELECT ST_GeometryN(geom, n) AS geom
FROM yourschema.montreal
CROSS JOIN generate_series(1,ST_NumGeometries(geom)) n
WHERE gid = 60449;
```

This query breaks down a polyhedral surface in all its polygons.
* ST_GeometryN retrieves the nth polygon
* ST_NumGeometries retrieves the number of geometries of the polyhedral surface
* generate_series creates a list of integer from 1 to ST_NumGeometries. By cross-joining with the building database, we obtain a tuple for each polygon.

Once we have all these polygons, we can select the relevant patches:

```sql
WITH roofs as (
    SELECT ST_GeometryN(geom, n) AS geom
    FROM yourschema.montreal
    CROSS JOIN generate_series(1,ST_NumGeometries(geom)) n
    WHERE gid = 60449)
SELECT count(DISTINCT id)
FROM pa, roofs
WHERE pc_intersects(pa,geom);
```

And get the maximum height:

```sql
WITH roofs as (
    SELECT ST_GeometryN(geom, n) AS geom
    FROM yourschema.montreal
    CROSS JOIN generate_series(1,ST_NumGeometries(geom)) n
    WHERE gid = 60449)
SELECT max(pc_patchmax(pa, 'z'))
FROM pa, roofs
WHERE pc_intersects(pa,geom);
```
