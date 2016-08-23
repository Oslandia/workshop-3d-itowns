# Intersection

In this file, we will try out some 3D operations on our city data.

First, we will find all the buildings that are inside and on the border of a 500 meters sphere.

## In the database

Let's start by adding a new column to our database:

```sql
psql pc_montreal
ALTER TABLE yourschema.montreal ADD COLUMN in_radius smallint DEFAULT 0;
```

The center of our sphere is the point of coordinate (300723,5041750,10). The *ST_3DDistance* function computes the minimum distance between two geometrie. *ST_3DMaxDistance* computes the maximum distance.

* Our first query selects all the geometries whose minimal distance to the center of the sphere is inferior to 500 meters.
* The second query selects all the geometries from the previous query whose maximal distance to the center of the sphere is superior to 500 meters: these are the geometries which are both in and out of the sphere.

```sql
UPDATE yourschema.montreal
SET in_radius = 1
WHERE gid
IN (
    SELECT gid
    FROM yourschema.montreal
    WHERE ST_3DDistance(geom, ST_SetSRID(ST_MakePoint(300723,5041750,10),2950)) <= 500
);

UPDATE yourschema.montreal
SET in_radius = 2
WHERE gid
IN (
    SELECT gid
    FROM yourschema.montreal
    WHERE in_radius = 1 AND ST_3DMaxDistance(geom, ST_SetSRID(ST_MakePoint(300723,5041750,10),2950)) >= 500
);
```

## In iTowns

Add "in_radius" to the list of attributes loaded by iTowns (attributes variable).

Modify the color function so the buildings in the radius of the sphere (in_radius = 1) are colored in red and the ones on the border of the sphere (in_radius = 2) are colored in yellow:

```js
function(attributes) {
    if(attributes.in_radius === 1) {
        return new THREE.Vector3(1, 0, 0);
    } else if(attributes.in_radius === 2) {
        return new THREE.Vector3(1, 1, 0);
    }
    return new THREE.Vector3(1, 1, 1);
};
```

We will also need to visualize the sphere to check if the result is correct. Luckily, iTowns is based on the ThreeJS 3D library, so it is really easy to add 3D objects to the scene. Add these lines after the scene instanciation:

```js
var geometry = new THREE.SphereGeometry( 500,20,20 );
geometry.translate(300723,5041750,10);
var material = new THREE.MeshBasicMaterial( {color: 0xffff00, wireframe: true} );
var sphere = new THREE.Mesh( geometry, material );
itowns.viewer.addObject(sphere);
```

* First we create a sphere geometry with a radius of 500 meters, composed of 20 x 20 segments (the resolution of the sphere)
* Then we translate it to the desired position in the world
* We create its material (which defines how the geometry will be rendered), and set the wireframe attribute to true to be able to see inside the sphere
* Finally, we create the mesh from the geometry and the material and add it to the scene

You can now visually check the correctness of the query.
