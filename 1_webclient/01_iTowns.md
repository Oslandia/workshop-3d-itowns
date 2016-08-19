
# iTowns

## Introduction

iTowns2 is a JavaScript framework for the visualization of 3D geospatial data.

We will use it to view terrain, 3D buildings and point clouds.

## Server configuration

We have a short preprocessing step that organizes the data for faster viewing.

The server has to be pre-configured, you can check the configuration file: /home/oslandia/building-server/building_server/settings.py.

Most notably, you will find the database name, the table name, and the extent of the scene.

To prepare the database, launch the following script:

`python /home/oslandia/building-server/building_server/processDB.py wsXX` (XX being your assigned number)

## Testing iTowns2

A running example is available at http://3d.oslandia.com.

The top left panel can be used to show or hide the different layers of the demo.

Holding the "s" key and clicking on a building selects it and displays its attributes.

TODO : décrire la méthode de navigation

## Your own 3d scene

* Download the content of the git repository hosting this workshop here https://github.com/Oslandia/workshop-3d-itowns/archive/master.zip (or clone the repository with git if your prefer).
* Extract the files on your computer and open `www/index.html` with your browser. You should see the example 3D scene.

## Point to your own database table

* Open the www/index.html file and look for a variable named `buildingLayerName`.
* The building server has been configured so that a table **wsXX_montreal** can be used by the name **wsXX**. Set the correct name to this variable so that it can point to your own database table.

