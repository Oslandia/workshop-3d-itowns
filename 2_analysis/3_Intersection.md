ALTER TABLE montreal ADD COLUMN in_radius smallint DEFAULT 0;

UPDATE montreal SET in_radius = 1 WHERE gid IN (SELECT gid FROM montreal WHERE ST_3DDistance(geom, ST_SetSRID(ST_MakePoint(300723,5041750,10),2950)) <= 500);
UPDATE montreal SET in_radius = 2 WHERE gid IN (SELECT gid FROM montreal WHERE in_radius = 1 AND ST_3DMaxDistance(geom, ST_SetSRID(ST_MakePoint(300723,5041750,10),2950)) >= 500);

iTowns (index.html):
* Fetch attributes in_radius
* Color function:
function(attributes) {
    if(attributes.in_radius === 1) {
        return new THREE.Vector3(1, 0, 0);
    } else if(attributes.in_radius === 2) {
        return new THREE.Vector3(1, 1, 0);
    }
    return new THREE.Vector3(1, 1, 1);
}
