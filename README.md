# osm2pgsql : True Append Version #

**This is a simple fork of osm2pgsql that allows extending an existing dataset in PostGIS by safely handling duplicate OSM ID's in adjacent/overlapping input areas.** For example, two adjacent countries that share ways (like roads) can be imported separately into the same database with `--append`. Adding a feature that already exists in the database simply overwrites (updates) that feature. Note: this version is slower than the standard osm2pgsql, due to the performance of the SQL queries used.

osm2pgsql is a tool for loading OpenStreetMap data into a PostgreSQL / PostGIS
database suitable for applications like rendering into a map, geocoding with
Nominatim, or general analysis.

## Features ##

* Converts OSM files to a PostgreSQL DB
* Conversion of tags to columns is configurable in the style file
* Able to read .gz, .bz2, .pbf and .o5m files directly
* Can apply diffs to keep the database up to date
* Support the choice of output projection
* Configurable table names
* Gazetteer back-end for [Nominatim](http://wiki.openstreetmap.org/wiki/Nominatim)
* Support for hstore field type to store the complete set of tags in one database
  field if desired

## Installing ##

The latest source code is available in the OSM git repository on github
and can be downloaded as follows:

```sh
$ git clone git://github.com/openstreetmap/osm2pgsql.git
```

## Building ##

Osm2pgsql uses the [GNU Build System](http://www.gnu.org/software/automake/manual/html_node/GNU-Build-System.html)
to configure and build itself and requires 

* [expat](http://www.libexpat.org/)
* [geos](http://geos.osgeo.org/)
* [proj](http://proj.osgeo.org/)
* [bzip2](http://www.bzip.org/)
* [zlib](http://www.zlib.net/)
* [PostgreSQL](http://www.postgresql.org/) client libraries
* [Lua](http://www.lua.org/) (Optional, used for [Lua tag transforms](docs/lua.md))

It also requires access to a database server running
[PostgreSQL](http://www.postgresql.org/) and [PostGIS](http://www.postgis.net/).

Make sure you have installed the development packages for the libraries
mentioned in the requirements section and a C++ compiler which supports C++11.
Both GCC 4.8 and Clang 3.4 meet this requirement.

First install the dependencies.

On a Debian or Ubuntu system, this can be done with:

```sh
sudo apt-get install make cmake g++ libboost-dev libboost-system-dev \
  libboost-filesystem-dev libboost-thread-dev libexpat1-dev zlib1g-dev \
  libbz2-dev libpq-dev libgeos-dev libgeos++-dev libproj-dev lua5.2 \
  liblua5.2-dev
```

To on a Fedora system, use

```sh
sudo yum install cmake gcc-c++ boost-devel expat-devel zlib-devel bzip2-devel \
  postgresql-devel geos-devel proj-devel proj-epsg lua-devel
```

On RedHat / CentOS first run `sudo yum install epel-release` then install
dependencies like on Fedora.

On a FreeBSD system, use

```sh
pkg install devel/cmake devel/boost-libs textproc/expat2 \
  databases/postgresql94-client graphics/geos graphics/proj lang/lua52
```

Once dependencies are installed, use CMake to build the Makefiles in a separate folder

```sh
mkdir build && cd build
cmake ..
```

If some installed dependencies are not found by CMake, more options may need
to be set. Typically, setting `CMAKE_PREFIX_PATH` to a list of appropriate
paths is sufficient.

When the Makefiles have been successfully built, compile with

```sh
make
```

The compiled files can be installed with

```sh
sudo make install
```

By default, the Release build with debug info is created and no tests are compiled.
You can change that behavior by using additional options like following:

    cmake .. -G "Unix Makefiles" -DCMAKE_BUILD_TYPE=Debug -DBUILD_TESTS=ON

## Usage ##

Osm2pgsql has one program, the executable itself, which has **42** command line
options.

Before loading into a database, the database must be created and the PostGIS
and optionally hstore extensions must be loaded. A full guide to PostgreSQL
setup is beyond the scope of this readme, but with reasonably recent versions
of PostgreSQL and PostGIS this can be done with

```sh
createdb gis
psql -d gis -c 'CREATE EXTENSION postgis; CREATE EXTENSION hstore;'
```

A basic invocation to load the data into the database ``gis`` for rendering would be

```sh
osm2pgsql --create --database gis data.osm.pbf
```

This will load the data from ``data.osm.pbf`` into the ``planet_osm_point``,
``planet_osm_line``, ``planet_osm_roads``, and ``planet_osm_polygon`` tables.

When importing a large amount of data such as the complete planet, a typical
command line would be

```sh
osm2pgsql -c -d gis --slim -C <cache size> \
  --flat-nodes <flat nodes> planet-latest.osm.pbf
```
where
* ``<cache size>`` is 24000 on machines with 32GiB or more RAM
  or about 75% of memory in MiB on machines with less
* ``<flat nodes>`` is a location where a 24GiB file can be saved.

The databases from either of these commands can be used immediately by
[Mapnik](http://mapnik.org/) for rendering maps with standard tools like
[renderd/mod_tile](https://github.com/openstreetmap/mod_tile),
[TileMill](https://www.mapbox.com/tilemill/), [Nik4](https://github.com/Zverik/Nik4),
among others. It can also be used for [spatial analysis](docs/analysis.md) or
[shapefile exports](docs/export.md).

[Additional documentation is available on writing command lines](docs/usage.md).

## Alternate backends ##

In addition to the standard [pgsql](docs/pgsql.md) backend designed for
rendering there is also the [gazetteer](docs/gazetteer.md) database for
geocoding, principally with [Nominatim](http://www.nominatim.org/), and the
null backend for testing. For flexibility a new [multi](docs/multi.md)
backend is also avialable which allows the configuration of custom
postgres tables instead of those provided in the pgsql backend.

Any questions should be directed at the osm dev list
http://wiki.openstreetmap.org/index.php/Mailing_lists

## Contributing ##

We welcome contributions to osm2pgsql. If you would like to report an issue,
please use the [issue tracker on GitHub](https://github.com/openstreetmap/osm2pgsql/issues).

More information can be found in [CONTRIBUTING.md](CONTRIBUTING.md).

General queries can be sent to the tile-serving@ or dev@
[mailing lists](http://wiki.openstreetmap.org/wiki/Mailing_lists).
