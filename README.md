# QGIS Server Docker Example

A Docker Compose setup for running QGIS Server with example WMS and WFS services.

Based on [docker/qgis/qgis-server](https://hub.docker.com/r/qgis/qgis-server).
Maybe replace with [docker/camptocamp/qgis-server](https://hub.docker.com/r/camptocamp/qgis-server).
Might offer more convenience and is used more often, but differences are currently not clear or obvious.

## Setup

Based on [qgis/qgis-docker/docker-compose.yml (example)](https://github.com/qgis/qgis-docker/blob/main/docker-compose.yml), [qgis/qgis-docker/server/README.md](https://github.com/qgis/qgis-docker/blob/main/server/README.md) and [Anita Graser - QGIS Server Docker Edition](https://anitagraser.com/2024/04/20/qgis-server-docker-edition/).

### Sample data

```shell
mkdir data
cd data
wget https://github.com/qgis/QGIS-Training-Data/archive/master.zip
unzip master.zip
cp -r QGIS-Training-Data-master/exercise_data/qgis-server-tutorial-data/ world/
rm -r QGIS-Training-Data-master
rm master.zip
```

### Example requests

Vendor concepts (specific to QGIS Server)

- Parameter MAP: define the QGIS project file to use (mandatory if no default project specified in env vars), path must be absolute or relative to server executable `qgis_mapserv.fcgi`
- Parameter FILE_NAME: If this vendor parameter is set, the server response will be sent to the client as a file attachment with the specified file name.
- Short name: Features can be requested by their short name, if one was configured for them in the QGIS project

#### WFS API

http://localhost/wfs3/

#### OGC Services

```bash
http://localhost/ogc/<your-query>
```

**Request default project**

http://localhost/ogc/world?LAYERS=countries,airports,places&SERVICE=WMS&VERSION=1.3.0&REQUEST=GetMap&CRS=EPSG:4326&WIDTH=1200&HEIGHT=600&BBOX=40,-5,50,15

**Request specific project (mandatory if default project is not set)**

http://localhost/ogc/?MAP=/io/data/world/world.qgs&LAYERS=countries,airports,places&SERVICE=WMS&VERSION=1.3.0&REQUEST=GetMap&CRS=EPSG:4326&WIDTH=1200&HEIGHT=600&BBOX=40,-5,50,15

**JPEG with compression instead of PNG**

http://localhost/ogc/world?LAYERS=countries,airports,places&SERVICE=WMS&VERSION=1.3.0&REQUEST=GetMap&CRS=EPSG:4326&WIDTH=1200&HEIGHT=600&BBOX=40,-5,50,15&FORMAT=image/jpeg&IMAGE_QUALITY=75

**WMTS - single layer**

- Layer needs to be activated first: Project Properties > QGIS Server > WMTS
- TILEMATRIX == Zoom, TILEROW == y, TILECOL == x

http://localhost/ogc/?SERVICE=WMTS&REQUEST=GetTile&LAYER=countries&FORMAT=image/png&TILEMATRIXSET=EPSG:4326&TILEMATRIX=4&TILEROW=3&TILECOL=16

**WMTS - layer group**

- Needs some configuration first: group all layers into a group called "group", then publish the group under Project Properties > QGIS Server > WMTS
- Shows all layers in the group, no matter their activation state in the QGIS Project!

http://localhost/ogc/?SERVICE=WMTS&REQUEST=GetTile&LAYER=group&FORMAT=image/png&TILEMATRIXSET=EPSG:4326&TILEMATRIX=6&TILEROW=16&TILECOL=64

## To Do

- integrate with [MapProxy](https://www.mapproxy.org/) tile cache server which can also act as a security layer! QGIS Server also does tile caching though.
- add customized OpenStreetMap Tiles with [OpenMapTiles](https://openmaptiles.org/) and [TileServer GL](https://openmaptiles.org/docs/host/tileserver-gl/); style inspiration can be taken from [OSM Hiking Maps](https://wiki.openstreetmap.org/wiki/Hiking_maps) and [Geofabrik](https://www.geofabrik.de/maps/rendering.html)

## Configuration

- [Catalog](https://docs.qgis.org/3.40/en/docs/server_manual/catalog.html)
- [Advanced Configuration](https://docs.qgis.org/3.40/en/docs/server_manual/config.html#advanced-configuration): ENVIRONMENT VARIABLES, pg_service file, fonts, logging

## Troubleshooting

### Check QGIS Server logs:
```bash
docker-compose logs qgis-server
```

### Validate QGIS project:
```bash
docker-compose exec qgis-server qgis_mapserv.fcgi "MAP=/io/data/example-project.qgs&SERVICE=WMS&REQUEST=GetCapabilities"
```

### Test QGIS Server Executable

```bash
docker compose exec qgis-server /usr/lib/cgi-bin/qgis_mapserv.fcgi --version
```

## References

### Technical Documentation

- [QGIS Server Documentation](https://docs.qgis.org/latest/en/docs/server_manual/)
- [OGC WFS Standard](https://www.ogc.org/standards/wfs)
- [OGC WMS Standard](https://www.ogc.org/standards/wms)
- [OGC WMTS Standard](https://www.ogc.org/standards/wmts)
