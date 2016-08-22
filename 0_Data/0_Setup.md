# Accessing the server

* Server: 3d.oslandia.com
* User: foss4g
* Password: foss4G2016!

# Accessing the database

Connect to the server by ssh using the given credentials and type :

`psql pc_montreal`

You will work in your own schema to avoid conflicts with other users. Choose a unique name for you schema. We will refer to it as *yourschema* in the rest of the workshop.

`CREATE SCHEMA yourschema`

You will need to copy a lot of queries during the workshop. A good part of them requires you to edit in the name of your schema. We advise you to copy the queries into a text editor and replacing the relevant fields there before copying them to your shell. Multi-line queries are not editable in postgres' command line interface.
