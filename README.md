## **Deploy New MapServer**

Install Postgres SQL

    sudo apt-get install postgresql-server-dev-all postgresql-client postgresql-contrib pgadmin3

Install PostGIS extension

    sudo apt-get install osm2pgsql

Install NodeJS

    curl -sL https://deb.nodesource.com/setup_13.x | sudo -E bash -
    sudo apt-get install -y nodejs

Install the MapServer with instructions from this git:

> https://github.com/mapserver/mapserver/wiki/Install-mapserver-on-ubuntu-18.04-from-official-repository

Create Postgres user and database

    sudo su - postgres
    psql -c "CREATE USER osm WITH PASSWORD 'osm';"
    psql -c "CREATE DATABASE osm WITH owner osm ENCODING 'UTF8';"
    psql -d osm -c "CREATE EXTENSION postgis;"
    exit

Clone the repository

    git clone https://github.com/minhduccse/deploy-new-mapserver.git

Import shapefiles

    cd deploy-new-mapserver/normalized
    shp2pgsql -I -s 4326 gadm36_VNM_2.shp vnm_2 | psql -h localhost -p 5432 -U osm -d osm
    shp2pgsql -I -s 4326 gadm36_VNM_3.shp vnm_3 | psql -h localhost -p 5432 -U osm -d osm
    cd ../income_point
    shp2pgsql -I -s 4326 point.shp heat_data | psql -h localhost -p 5432 -U osm -d osm

***Run npm script:***

    cd ../postgis-elasticsearch-cronjob

Install dependency

     npm install

**Config your connection in `db/esConnection` `db/pgConnection` and `cronjob/dropData.js` `cronjob/fetchData.js` line 7-10 and line 23**

Run npm script to create tables

    npm run drop-data
    npm run population
    npm run income
    npm run create-hash-table

Run the cronjob

    node index.js

Run the API

    npm start

Your url to mapserver for google map will be: 

    http://your_hostname/cgi-bin/mapserv?map=/home/[user]/deploy-new-mapserver/mapserver-server/google_maps_demo.map&


API table:
Url to API is `your_hostname:3000` + `[url]`
|Url|Type|Details|Sample Request|
|--|--|--|--|
|`/map/get/income`|GET|Get income map info||
|`/map/get/population`|GET|Get population map info||
|`/map/update/income`|POST|Update income color by id|`{"id":5, "color_type": STATIC", "color_value": "#0C2808"}`|
|`/map/update/population`|POST|Update population color by gid|`{"gid": 251, "population": 71000, "color_value": "#000000"}`|

