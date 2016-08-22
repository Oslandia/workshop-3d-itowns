# Land Use

The land use information will allow us to color building depending on their function.

In the database
---------------

First, create a new column in the building table to record the function fo the building.

```sql
ALTER TABLE yourschema.montreal ADD COLUMN landuse varchar;
```

Now, like we did with the districts, intersect the buildings with the various land use polygons:

```sql
WITH
d AS (
  SELECT categorie, geom FROM yourschema.landuse WHERE geom && ST_MakeEnvelope(298250, 5039250, 302750, 5043750)
),
b AS (
    SELECT ST_SetSRID(ST_CENTROID(Box2D(yourschema.montreal.geom)),2950) AS geom, gid FROM yourschema.montreal
),
t AS (
    SELECT d.categorie, b.gid FROM d, b WHERE ST_Intersects(d.geom, b.geom)
)
UPDATE yourschema.montreal SET landuse = t.categorie FROM t WHERE yourschema.montreal.gid = t.gid;
```

In iTowns
---------

In index.html, add "landuse" to the `attributes` array.

Modify the color function to color the buildings depending on the land use:

```Javascript
var colorFunction = function(attributes) {
	if(attributes.landuse === "mixte") {
		return new THREE.Vector3(0.5,0.5,0.5);
	} else if(attributes.landuse === "institution") {
		return new THREE.Vector3(1,0,0);
	} else if(attributes.landuse === "parc") {
		return new THREE.Vector3(0,1,0);
	} else if(attributes.landuse === "religieux") {
		return new THREE.Vector3(1,1,0);
	} else if(attributes.landuse === "emplois") {
		return new THREE.Vector3(0,0,1);
	} else if(attributes.landuse === "residentiel") {
		return new THREE.Vector3(1,0.5,0.5);
	} else if(attributes.landuse === "infrastructutre") {
		return new THREE.Vector3(1,0,1);
	} else if(attributes.landuse === "transport") {
		return new THREE.Vector3(0,1,1);
	}
	return new THREE.Vector3(1,1,1);
};
```

Open index.html in a web browser to see the result.
