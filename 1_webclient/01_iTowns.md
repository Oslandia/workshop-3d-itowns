iTowns
======

Introduction
------------

iTowns2 is a JavaScript framework for the visualization of 3D geospatial data.

We will use it to view terrain, 3D buildings and point clouds.

Running iTowns2
-----------------

A running example is available at http://3d.oslandia.com.

The top left panel can be used to show or hide the different layers of the demo.

Holding the "s" key and clicking on a building displays its attributes.

TODO : décrire la méthode de navigation

Your own 3D scene
-----------------

On the server, in the home directory of the user 'foss4g', a subdirectory named 'www' can be found. It contains one subdirectory for each user of the workshop.

* Create a file /home/foss4g/www/wsXX/index.html with a simple text content ("Hello !") and check that it is displayed correctly when you go with your browser to http://3d.oslandia.com/foss4g/wsXX/

* In order to make your own 3D scene, we will start with the example that is shown on http://3d.oslandia.com by copying the HTML example file
 
`cp -R /home/foss4g/workshop-3d-itowns/www/dist/* /home/foss4g/www/wsXX/`

* Open the index.html file copied that way and look for a variable named **buildingLayerName**. The building server has been configured so that a table **wsXX_montreal** can be used by the name **wsXX**. Set the correct name to this variable so that it can point to your own database table and reload your browser to the page http://3d.oslandia.com/foss4g/wsXX/

