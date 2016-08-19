# Cesium Buildings

## Introduction

iTowns2 is a JavaScript framework for the visualization of 3D geospatial data.

We will use it to view terrain, 3D buildings and point clouds.

## Server configuration

We have a short preprocessing step that organizes the data for faster viewing.

The server has to be pre-configured, you can check the configuration file: /home/oslandia/building-server/building_server/settings.py.

Most notably, you will find the database name, the table name, and the extent of the scene.

To prepare the database, launch the following script:

`python /home/oslandia/building-server/building_server/processDB.py wsXX` (XX being your assigned number)

## Client configuration

Open www/index.html with a text editor.

Find the "buildingLayerName" variable, and replace "montreal" by wsXX (XX being your assigned number).

## Running iTowns2

Open www/index.html in your web browser.

A running example is also available at http://3d.oslandia.com.

The top left panel can be used to show or hide the different layers of the demo.

Holding the "s" key and clicking on a building selects it and displays its attributes.

TODO : décrire la méthode de navigation
