
# iTowns

## Introduction

iTowns2 is a JavaScript framework for the visualization of 3D geospatial data.

It is able to view terrain, 3D buildings and point clouds.

## Testing iTowns2

A running example is available at http://3d.oslandia.com.

The top left panel can be used to show or hide the different layers of the demo.

The navigation uses solely the keyboard:
* Arrow keys to move the camera on the horizontal plan
* R/F to move up and down
* W/A/X/D to rotate the camera
* Left shift to move faster

Holding the S key and clicking on a building selects it and displays its attributes.

## Your own 3D scene

### Server configuration

We have a short preprocessing step that organizes the data for faster viewing. It classifies the buildings into a quadtree, allowing their progressive transmission.

First, you have to create a configuration file for your building layer. The configuration files are located in /home/oslandia/building-server/building_server/cities. There already is a configuration file for the demo at http://3d.oslandia.com. Copy it, and give it a unique name: `cp /home/oslandia/building-server/building_server/cities/montreal_default.py /home/oslandia/building-server/building_server/cities/<My_config_file>.py`

This is the content of the file:

```python
import cities

cities.CITIES["TODO_ID"] = {
    "tablename": "TODO_TABLE",
    "extent": [[297250,5038250], [303750,5044750]],
    "maxtilesize": 2000,
    "srs":"EPSG:2950",
    "featurespertile":20
}
```

* TODO_ID: replace this by the identifier by which you wish your building layer to be referred to
* TODO_TABLE: replace this by the name of the table that contains the building data
* extent: the extent covered by the buildings
* maxtilesize: the size of the root tiles of the quadtree
* srs: the geometries' spatial reference system
* featurespertile: the maximum number of buildings per tile of the quadtree

You only need to modify TODO_ID and TODO_TABLE.

To prepare the database, launch the following script. Don't forget to replace TODO_ID.

`python /home/oslandia/building-server/building_server/processDB.py TODO_ID`

The configuration of all the cities can be retrieved at http://3d.oslandia.com/building?query=getCities. You can test if your configuration has been taken into account by going on this page. This can take up to one minute to update.

## Client configuration

* Download the content of the git repository hosting this workshop here https://github.com/Oslandia/workshop-3d-itowns/archive/master.zip (or clone the repository with git if your prefer).
* Extract the files on your computer and open `www/index.html` with your browser. You should see the example 3D scene.

The web page uses the same database as http://3d.oslandia.com. Point to your own database table:

* Open the `www/index.html` file and look for a JavaScript variable named `buildingLayerName`.
* The building server has been configured so that a table **wsXX_montreal** can be used by the name **wsXX**. Set the correct name to this variable so that it can point to your own database table.
* Test by opening `www/index.html` in your web browser. If buildings start to appear, the server was correctly configured.
