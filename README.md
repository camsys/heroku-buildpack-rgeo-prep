# heroku-buildpack-rgeo-prep

This buildpack overwrites Heroku's default .bundle/config to set BUNDLE_BUILD__RGEO to Heroku's build directory.

## Setup

Create this .buildpacks file in the root of your project.

    https://github.com/trailheadlabs/heroku-buildpack-vendorbinaries.git
    https://github.com/trailheadlabs/heroku-buildpack-rgeo-prep.git
    https://github.com/heroku/heroku-buildpack-ruby.git

Create this .vendor_urls file in the root of your project.

http://trailheadlabs.outerspatial.s3.amazonaws.com/buildpack.binaries/libjpeg-9a.tar.gz
http://trailheadlabs.outerspatial.s3.amazonaws.com/buildpack.binaries/gdal-1.10.1-1.tar.gz
http://trailheadlabs.outerspatial.s3.amazonaws.com/buildpack.binaries/proj-4.8.0-1.tar.gz
http://trailheadlabs.outerspatial.s3.amazonaws.com/buildpack.binaries/geos-3.4.1.tar.gz

Add these files to git.

Now, set up your heroku configuration.

    heroku config:set BUILDPACK_URL=https://github.com/trailheadlabs/heroku-buildpack-multi.git LD_LIBRARY_PATH=/app/lib

    heroku config:set GDAL_BIN=/app/bin
    
    heroku config:set GDAL_DATA=/app/share/gdal
    
If you haven't already set up your heroku database for postgis, you need to run the following steps. You currently must have a production level database to enable postgis.

Since postgis uses different settings in the database.yml, you need to modify the DATABASE_URL variable. Run the following command and extract the nessecary components out of it:

    $ heroku config:get DATABASE_URL 
    postgres://<username>:<password>@<host>:<port>/<database>

With those variables, run the following command

    $ heroku config:set DATABASE_URL="postgis://<username>:<password>@<host>:<port>/<database>?postgis_extension=true&search_schema_path=public,postgis"

Enable PostGIS

    $ heroku pg:psql
    => CREATE EXTENSION postgis;

Deploy

    git push heroku master
    
Verify it worked

    heroku run bash
    ~> bundle exec rails c

    >> RGeo::Geos.supported?
    => true
    >> RGeo::CoordSys::Proj4.supported?
    => true

If both of these are true, you should be ready to go.

## Tools

We found the heroku repo plugin to be useful during the debugging process :

https://github.com/heroku/heroku-repo

Running the following commands let us clear out the buildpacks so that we could ensure any changes we made would be used in the next deploy:

    heroku repo:purge_cache
    
    heroku repo:reset
    
## Credits

This is really just a fork for use in Trailhead Labs applications of :

* https://github.com/aaronrenner/heroku-buildpack-rgeo-prep

This solution draws from many people's research including

* https://github.com/jcamenisch/heroku-buildpack-rgeo
* https://github.com/davekapp for help with troubleshooting the extconf.rb build process
* https://gist.github.com/perplexes/5357663
