**Terrastories minimal** is a stripped down version of the [**Terrastories**](http://terrastories.io) application which runs just the tilebuilder and tileserver, in order to render an offline map using MBtiles styled in Mapbox Studio. 

## Table of Contents
0. [Running the application: quickstart guide](#running-the-application-quickstart-guide)

1. [Creating and updating the Tileserver Map](#creating-and-updating-the-tileserver-map)

2. [Docker prerequisites](#docker-prerequisites)

3. [Setup](#setup)

4. [Building map tiles](#build-the-map-tiles)

5. [Running the server](#make-it-go)

## Running the application: quickstart guide
1. Install and start Docker. 
2. Download the `terrastories-minimal` code and put it in a directory (like `D:\terrastories-minimal\`)
3. Put the shapefiles in `tilebuilder\shapefiles\` and the Mapbox style.json file in `\tileserver\data\styles\`. See [Creating and updating the Tileserver Map](#creating-and-updating-the-tileserver-map) for more information on this process.
4. In a terminal application like PowerShell for Windows, navigate to that directory. 
5. Run `docker-compose build` to build the container for the application.
6. Run `docker-compose run tilebuilder` to generate the MBtiles from the shapefiles.
7. Run `docker-compose up -d nginx` to start the application.
8. Now, the map should be runnning in `http://localhost:3000/map/`
Note: Once you've built the application and created the tiles, you only ever have to run `docker-compose up -d nginx` for it to work. You can also run `docker-compose down` to close the application.

## Creating and updating the Tileserver Map
### Step 1: preparing content in Mapbox Studio

Terrastories is designed to render a basemap as designed and styled in Mapbox Studio. There are two different components: shapefiles (the spatial data without any styling properties) and styles (the look and feel of the map, as designed in Mapbox Studio, exported in json format). The basic workflow is as follows:

1) upload the shapefile content to [Mapbox Studio](https://www.mapbox.com/mapbox-studio/), and use the Studio interface to lay out the map. You have to have a Mapbox account to use Mapbox Studio (creating and designing maps using Mapbox Studio is free up to certain file size limitations). To learn how to use Mapbox Studio, you can refer to the manuals and tutorials made available by Mapbox [here](https://www.mapbox.com/help/studio-manual-tutorials/) or other resources on the web.

2) download the style.json from Mapbox Studio via the Mapbox Studio styles interface [here](https://www.mapbox.com/studio/styles/)

3) copy both the shapefiles and style.json into the respective directories on Terrastories. Do not rename shapefiles or feature names in Mapbox Studio throughout this process -- the naming conventions must be consistent (aside from the additional -###### hash added by Mapbox Studio; more on that later).

Note: the user must provide their own shapefile content. It is not possible to use any of the standard OpenStreetMap (OSM) content used in the standard styles made available by Mapbox, unless the user first downloads that OSM content and converts it to shapefile first.

### Step 2: adding new or updating shapefiles to Terrastories

To add new shapefiles or update existing shapefiles, there are two steps:

1) Include the new files in your `shapefile` directory (\tilebuilder\shapefiles])

2) You will need to re-run the tilebuilder, following the instructions here: https://github.com/rubyforgood/terrastories/blob/master/tilebuilder/README.md

### Step 3: adding or updating style to the Map

To add or update the map style,

1) download the style.json from Mapbox Studio via the Mapbox Studio styles interface [here](https://www.mapbox.com/studio/styles/)

2) at this point, we have to edit the style.json a little. When you upload shapefiles to Mapbox Studio, it actually adds on an additional six alphanumeric characters preceded by a dash (-), which is called "hash." For example, a shapefile called "South_America" might be called "South_America-a2027z" in Mapbox Studio. And then in style.json file, all of the names for this layer will have "–a2027z" added to it. This is a problem because there is a discrepancy between the names of the shapefile you added in Step 2, which does no include "-a2027z." So, you have to go into the json and look for "source-layer": "South_America-a2027z", and take out the "-a2027z", and do the same for each layer.

In the future, we will create an automatic script that will take care of this process.

3) copy the style into your `styles` directory (tileserver\data\styles])

4) make sure that config.json in \tileserver\data\ is pointing to the right style file.

## Docker Prerequisites
Install docker. On linux, you may have to install docker-compose separately.
 - https://docs.docker.com/install/
 - https://docs.docker.com/compose/install/

On Windows, all terminal docker commands need to be run using Windows PowerShell, not Command Prompt.
PowerShell comes bundled with Windows.

## Setup
In your terminal program, navigate to the directory where you put the `terrastories-minimal` 
files, after you've downloaded it. You always have to be in this directory to run any of the docker
commands.

Docker will automatically build images as needed when running `docker-compose up`,
but to confirm everything builds correctly, run the following and check that the
output ends with something like this.
```
$ docker-compose build
  ...
> mariadb uses an image; skipping
  ...
> Successfully built 0123456789
> Successfully tagged terrastories/tilebuilder:latest
> tileserver uses an image; skipping
```

## Build The Map Tiles
The tilebuilder service will need to be run once to populate the `mbtiles`
shared volume that the tileserver will read from. The tilebuilder does not need
to stay running along with the other services. Building map tiles may take quite
a long time, but it should show progress similar to the following and eventually
get to 100%, exiting with code 0.
```
$ docker-compose run tilebuilder
...
> wwww features, xxxx bytes of geometry, yyyy bytes of separate metadata,
zzzz bytes of string pool
> 99.9% 11/2222/3333
```

Any time the shapefiles change and require regenerating the mbtiles file,
this service will need to be run again and the tileserver restarted once the
tilebuilder finishes (just run `docker-compose restart tileserver`).

## Make It Go
In `docker-compose.yml`, the tileserver service is listed
as dependencies for the nginx service. So to start the whole thing up
(omitting tilebuilder, which only needs to run once) just run the following.
```
$ docker-compose up -d nginx
```

Once the service fully starts up, you can view tileserver running at `localhost:3000`
or an alternative port specified in `.env` if one exists. See `.env.example` for
available options and reasonable starting values. 

To view a map equipped with bookmarks and other customizations, visit `localhost:3000/map/`.

Omit the `-d` flag if you prefer to see the nginx server output. You can always
tail the output of any service with `docker-compose logs`. 

To spin all the services back down run the following.
```
$ docker-compose down
```
