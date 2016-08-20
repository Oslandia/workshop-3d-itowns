
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

We have a short preprocessing step that organizes the data for faster viewing.

The server has been pre-configured, you can check the configuration file: /home/oslandia/building-server/building_server/settings.py.

Most notably, you will find the database name, the table name, and the extent of the scene.

To prepare the database, launch the following script:

`python /home/oslandia/building-server/building_server/processDB.py wsXX` (XX being your assigned number)

## Client configuration

* Download the content of the git repository hosting this workshop here https://github.com/Oslandia/workshop-3d-itowns/archive/master.zip (or clone the repository with git if your prefer).
* Extract the files on your computer and open `www/index.html` with your browser. You should see the example 3D scene.

The web page uses the same database as http://3d.oslandia.com. Point to your own database table:

* Open the `www/index.html` file and look for a JavaScript variable named `buildingLayerName`.
* The building server has been configured so that a table **wsXX_montreal** can be used by the name **wsXX**. Set the correct name to this variable so that it can point to your own database table.
* Test by opening `www/index.html` in your web browser. If buildings start to appear, the server was correctly configured.
