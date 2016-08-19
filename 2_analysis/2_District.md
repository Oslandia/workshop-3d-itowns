ALTER TABLE montreal ADD COLUMN district_num integer;
ALTER TABLE montreal ADD COLUMN district_name varchar;

WITH d AS (SELECT num, nom, geom FROM districts WHERE geom && ST_MakeEnvelope(298250, 5039250, 302750, 5043750)),
t AS (SELECT d.num, d.nom, montreal.gid from d, montreal WHERE ST_Force3D(d.geom) && ST_CENTROID(Box2D(montreal.geom)))
UPDATE montreal SET district_num = t.num, district_name = t.nom FROM t WHERE montreal.gid = t.gid;

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
