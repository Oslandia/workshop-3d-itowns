
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


Download the content of the git repository hosting this workshop here https://github.com/Oslandia/workshop-3d-itowns/archive/master.zip (or clone the repository with git if your prefer).

Extract the files on your computer and open `www/index.html` with a text editor.

### Terrain

The terrain layer is composed of two components: the elevation and the imagery.

In index.html, add the elevation layer after the definition of the scene (search the "Elevation here" comment):

```javascript
itowns.viewer.addElevationLayer({
    url       : "http://3d.oslandia.com/dem.png",
    protocol  : 'single_image',
    id        : 'elevation',
    name      : 'MNT',
    projection : "EPSG:2950",
    bbox      : [266040.26, 4984521.41, 343559.74, 5095744.39],
    updateStrategy: {
        type: 0
    },
    options: {
        minmaxElevation: [0, 255],
        mimetype  : "image/png"
    }
});
```

Notable parameters:
* protocol single_image indicates that the MNT is described by a single image
* url thus indicates this image's address
* bbox is the bounding box of the image in the projection srs
* option.minmaxElevation is the image elevation scale

Now, add the imagery layer:

```javascript
itowns.viewer.addImageryLayer({
    url       : "http://3d.oslandia.com/mapserv?map=cmm_2013",
    protocol  : 'wms',
    version   : '1.3.0',
    id        : 'Imagery',
    name      : 'cmm_2013',
    style      : "",
    projection : "EPSG:2950",
    bbox      : [266040.26, 4984521.41, 343559.74, 5095744.39],
    heightMapWidth : 256,
    updateStrategy: {
        type: 3
    },
    options: {
        mimetype  : "image/jpeg",
        tileMatrixSet: 'WGS84G' // NE DEVRAIT PAS ETRE NECESSAIRE
    }
});
```

This time, the layer uses the WMS protocol. Notable new parameters:
* heightMapWidth: the resolution of the images that will be downloaded
* name: the name of the WMS ayer

### Buildings

In the `www/index.html` file, you will see the building layer has a symbolic name 'montreal' by default. It refers to a **building server** that serves vectorial data from a 3D postgis database to an optimized format suited for fast display.

The web page uses the same database as http://3d.oslandia.com. Point to your own database table:

* Open the `www/index.html` file and look for a JavaScript variable named `buildingLayerName`.
* Set the correct name (TODO_ID) to this variable so that it can point to your own database table (TODO_TABLE).
* Test by opening `www/index.html` in your web browser. If buildings start to appear, the server was correctly configured.

### Point cloud

Add the point cloud by adding this line after the scene definition:

```javascript
itowns.viewer.addPointCloud("greyhound://3d.oslandia.com/lopocs-itowns/greyhound/");
```
