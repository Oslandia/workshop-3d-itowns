# PointCloud

In this section, we are going to talk about how to load point cloud data in
database thanks to PDAL and the pgpointcloud PostgreSQL extension.

The size of the point cloud and the time it requires to import it all does not
make it possible to realize the import during this session. Nevertheless, we
will explain the steps needed to do it.

## LAS files

Generally speaking, point cloud data is often distributed through LAS file. LAS
is a binary public file format which supports a free and lossless compression
named LAZ (LASzip).

To play with these files, some very useful tools are available in libLAS or
LAStools according to the needs (http://www.liblas.org/lastools.html):
- las2las
- las2txt
- lasinfo
- ...

For example, in the case of Montreal, open LAZ files can be downloaded thanks
to an interactive map http://montreal.maps.arcgis.com/apps/Viewer/index.html?appid=0e6890209c024ef88a73cbc0bb6a3308.

Then thanks to the *lasinfo* tool, a user is able to retrieve information on
the content:

```bash
> lasinfo 284-5042_2015_2-5-6.laz
  ---------------------------------------------------------
  Header Summary
  ---------------------------------------------------------

  Version:                     1.3
  Source ID:                   0
  Reserved:                    0
  Project ID/GUID:             '00000000-0000-0000-0000-000000000000'
  System ID:                   ''
  Generating Software:         'LASzip DLL 2.4 r0 (150731)'
  File Creation Day/Year:      116/2016
  Header Byte Size             235
  Data Offset:                 335
  Header Padding:              0
  Number Var. Length Records:  1
  Point Data Format:           1
  Number of Point Records:     9032350
  Compressed:                  True
  Compression Info:            LASzip Version 2.4r0 c2 50000: POINT10 2 GPSTIME11 2
  Number of Points by Return:  7968817 944776 112030 6577 150
  Scale Factor X Y Z:          0.01000000000000 0.01000000000000 0.01000000000000
  Offset X Y Z:                -0.00 -0.00 -0.00
  Min X Y Z:                   284000.00 5042000.00 2.47
  Max X Y Z:                   285000.00 5043000.00 70.57
  Spatial Reference:           None

  ---------------------------------------------------------
  VLR Summary
  ---------------------------------------------------------
    User: 'laszip encoded' - Description: 'LASzip DLL 2.4 r0 (150731)'
    ID: 22204 Length: 46 Total Size: 100
  ---------------------------------------------------------
  Schema Summary
  ---------------------------------------------------------
  Point Format ID:             1
  Number of dimensions:        13
  Custom schema?:              false
  Size in bytes:               28

  Dimensions
  ---------------------------------------------------------
  'X'                            --  size: 32 offset: 0
  'Y'                            --  size: 32 offset: 4
  'Z'                            --  size: 32 offset: 8
  'Intensity'                    --  size: 16 offset: 12
  'Return Number'                --  size: 3 offset: 14
  'Number of Returns'            --  size: 3 offset: 14
  'Scan Direction'               --  size: 1 offset: 14
  'Flightline Edge'              --  size: 1 offset: 14
  'Classification'               --  size: 8 offset: 15
  'Scan Angle Rank'              --  size: 8 offset: 16
  'User Data'                    --  size: 8 offset: 17
  'Point Source ID'              --  size: 16 offset: 18
  'Time'                         --  size: 64 offset: 20

  ---------------------------------------------------------
  Point Inspection Summary
  ---------------------------------------------------------
  Header Point Count: 9032350
  Actual Point Count: 9032350

  Minimum and Maximum Attributes (min,max)
  ---------------------------------------------------------
  Min X, Y, Z:    284000.00, 5042000.00, 2.47
  Max X, Y, Z:    285000.00, 5043000.00, 70.57
  Bounding Box:   284000.00, 5042000.00, 285000.00, 5043000.00
  Time:     364643.713623, 555926.876164
  Return Number:  1, 6
  Return Count:   0, 6
  Flightline Edge:  0, 0
  Intensity:    655, 65535
  Scan Direction Flag:  0, 1
  Scan Angle Rank:  -25, 23
  Classification: 2, 6
  Point Source Id:  61, 256
  User Data:    205, 205
  Minimum Color (RGB):  0 0 0
  Maximum Color (RGB):  0 0 0

  Number of Points by Return
  ---------------------------------------------------------
  (1) 7968817 (2) 944776  (3) 112030  (4) 6577  (5) 149 (6) 1

  Number of Returns by Pulse
  ---------------------------------------------------------
  (0) 44438 (2) 6516737 (4) 1981461 (6) 489714

  Point Classifications
  ---------------------------------------------------------
  4865876 Ground (2)
  2014655 High Vegetation (5)
  2151819 Building (6)
  -------------------------------------------------------
    0 withheld
    0 keypoint
    0 synthetic
  -------------------------------------------------------
```

With the output of the previous command, we now know the extent of data, the
number of points, the available classification levels and so on.

By downloading the full area for Montreal, we have 684 LAZ files.

## PDAL and pgpointcloud

PDAL is an abstraction library which provides some tools to manipulate point
cloud data. An important thing to understand in PDAL is the concept of
*pipeline*. In fact, it's a custom processing chain described by a JSON
file.

In our case, we want to fill a database using the pgpointcloud extension. Thus,
the pipeline is the following:
- read a LAS/LAZ file
- use the chipper filter to store points in pgpointcloud patchs (collection of points)
- write the pgpointcloud patchs in database

With the PDAL JSON formalism, the pipeline *pipeline.json* looks like:

```js
{
  "pipeline":[
    {
      "type":"readers.las",
      "filename":"file.laz"
    },
    {
      "type":"filters.chipper",
      "capacity":400
    },
    {
      "type":"writers.pgpointcloud",
      "connection":"dbname=mydatabase user=myuser",
      "table":"pa"
    }
  ]
}
```

Then, we just have to run the pipeline:

```bash
> pdal pipeline pipeline.json
```

To process our 684 LAZ files, the first idea would be to merge the data
in one big file in order to facilitate the pipeline handling. It's fully
possible thanks to PDAL:

```bash
> pdal merge f1.laz f2.laz f3.laz merged.laz
```

However, according to the output of the *lasinfo* tool, a LAS file may contains
several tens of millions of points. Thus, with 684 LAZ files, the result would
be a file containing several billions of points. So, it's not really a good
idea because of the amount of memory which would be necessary to process a
file of this size.

So, we have to create a specific pipeline for each LAZ file. But, the
pgpointcloud writer coming with PDAL needs a new option to not drop the
table at each processing step:

```js
{
  "type":"writers.pgpointcloud",
  "connection":"dbname=mydatabase user=myuser",
  "table":"pa",
  "overwrite":"false"
}
```

Then, by running the pipeline on each LAZ file, we obtain a database with
several millions of pgpointcloud patchs which themselves contain about
400 points (chipper filter capacity).

The table *pa* of the *pc_montreal* database has been built in this way and
contains 15906335 pgpointcloud patchs or by around 6 billions of points.

If you are curious on how to work with point cloud data, you can take a look
to our dedicated workshop : https://github.com/Oslandia/workshop-pointcloud.
