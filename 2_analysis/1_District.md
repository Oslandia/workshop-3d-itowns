# District

We will now make the link between buildings and the district they belong to.
The goal is then to be able to use that information in iTowns2 to classify each building by their district.

In the database
---------------

```sql
ALTER TABLE wsXX_montreal ADD COLUMN district_num integer;
ALTER TABLE wsXX_montreal ADD COLUMN district_name varchar;

WITH
d AS (
  SELECT num, nom, geom FROM wsXX_districts WHERE geom && ST_MakeEnvelope(298250, 5039250, 302750, 5043750)
),
t AS (
  SELECT d.num, d.nom, wsXX_montreal.gid from d, wsXX_montreal WHERE ST_Force3D(d.geom) && ST_CENTROID(Box2D(wsXX_montreal.geom))
)
UPDATE wsXX_montreal SET district_num = t.num, district_name = t.nom FROM t WHERE wsXX_montreal.gid = t.gid;
```

In iTowns
---------

Now you need to modify the javascript code found in index.html so that the building attributes stored in the newly created columns are extracted from the building server with the geometries. The attributes values can then be used in a function that sets a color to each feature.

* Look for the variable `attributes`. It should be an array of string. We need to fetch two new attributes coming from the data table: "district_num" and "district_name" `var attributes = ["district_num", "district_name"];`
* Look for the variable `colorFunction`. This function will be called for each vector feature with a dictionary of attributes as its first parameter. A vector of 3 dimension is expected in return. It gives the 3 R,G,B components of the color (between 0.0 and 1.0):

```Javascript
function(attributes) {
    if(attributes.district_num === 20) {
        return new THREE.Vector3(1, 1, 0);
    } else if(attributes.district_num === 21) {
        return new THREE.Vector3(1, 0, 1);
    } else if(attributes.district_num === 22) {
        return new THREE.Vector3(0, 1, 1);
    }
    return new THREE.Vector3(1, 1, 1);
}
```

Open index.html in a web browser to see the result. Selecting a building (s + click) will display its gid, district number and district name.
