We will now make the link between buildings and the district they belong to.
The goal is then to be able to use that information in iTowns2 to classify each building by their district.

```sql
ALTER TABLE wsXX_montreal ADD COLUMN district_num integer;
ALTER TABLE wsXX_montreal ADD COLUMN district_name varchar;

WITH
d AS (
  SELECT num, nom, geom FROM wsXX_district WHERE geom && ST_MakeEnvelope(298250, 5039250, 302750, 5043750)
),
t AS (
  SELECT d.num, d.nom, wsXX_montreal.gid from d, wsXX_montreal WHERE ST_Force3D(d.geom) && ST_CENTROID(Box2D(wsXX_montreal.geom))
)
UPDATE wsXX_montreal SET district_num = t.num, district_name = t.nom FROM t WHERE wsXX_montreal.gid = t.gid;
```

iTowns (index.html):
* Fetch attributes district_num, district_name
* Color function:
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
