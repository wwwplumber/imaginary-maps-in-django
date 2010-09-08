========================
Maps of Imaginary Lands
========================

Introduction
=============

Supporting code and slides for a talk originally given at DjangoCon-US,
September 2010 (in Portland, Oregon, USA).

Short Description
------------------

The GIS features of Django aren't restricted to being applied to real world
maps and planets. This talk will show how to display and interact with maps of
imaginary lands, such as game maps or lands in science fiction novels. We'll
uncover a bit of how Django GIS works in the process, separating the map
display from the modeling.

Abstract
---------

Whilst `django.contrib.gis` isn't particularly difficult to get started with,
particularly if you follow the tutorials, it can sometimes seem a bit
overwhelming in the way it goes from zero to fancy maps in only a couple of
steps. I'd like to demystify some of the pieces of the stack, pulling apart the
modeling support — specifying the data are we trying to work with — from the
display and client-side portion.

To make this more than a dry technical dive, I'll show how to add extras to an
imaginary map, rather than something pulled from Google Maps or Open Street
Map. We'll take on the task of plotting features on a landscape from a
potential role-playing game and show how the GIS data manipulation features,
such as calculating region intersections, nearby points, and Javascript
client-side display work the in a familiar way against this slightly unusual
background.

Some basic familiarity with Django's GIS features would be useful for this
talk, although it might also serve as a motivating introduction to trying
things out.

Setting up
===========

This code is intended as a self-contained, runnable small example, written with
reasonably professional coding standards in mind.

That being said, a few pre-requisites need to be installed in order to run the
example. I am assuming you have worked through the `django.contrib.gis`
tutorial_ and thus have all the necessary pre-requisites installed. If you can
view any Django admin page that involves GIS information, you have met the
requirements here.

Secondly, you'll need to have Mapnik_ installed (and Mapnik's Python bindings).
Major Linux distributions will ship these as packages that won't require
anything more than a `yum install ...` or `aptitude install...`. I've been led
to believe it isn't amazingly difficult to get the necessary pieces installed
on a Mac, as well. I have on idea about the level of difficulty for a Windows
install (feedback welcome). If you can run the following at a Python prompt and
no exception is raised, you have the necessary components installed::

    >>> from mapnik import ogcserver

.. _tutorial: http://docs.djangoproject.com/en/1.2/ref/contrib/gis/tutorial/
.. _Mapnik: http://mapnik.org/

The included settings file is set up to use Postgis as the database. I don't
believe anything Postgis-specific is used in the code, however, so any spatial
database backend should work (again, being able to work through the Django GIS
tutorial is probably both necessary and sufficient).

To do a setup from scratch (assuming Postgis for the first step), I ran the
following steps, in order:

 1. Create the GIS-aware database: `createdb -T template_postgis
    imaginary_lands`. Assumes you already have `template_postgis` created as per
   Geodjango setup instructions.
 2. `python manage.py syncdb --noinput` to do the basic model creation. This
    will load an initial fixtures file and create and admin user, with both
    username and password being "*admin*" (without the quotes).
 3. `python manage.py import_land` and `python manage.py import_adventures` to
    load initial shape data into the GIS models.
 4. Create the GeoTiff version of the base map (only the PNG version is checked
    in)::

        bin/create_tiff.sh

    This is a shell script that uses some GDAL utilities to make the conversion
    (Windows users may need to find an equivalent alternative).

You then need to start both the Django development webserver and a local Mapnik
server. Start the Mapnik server by::

    cd map_server
    python wms.py

That will launch the server on port 8001. Start Django's development server
with the usual::

    python manage.py runserver

You can now browse to http://localhost:8000/ to see the island map with a few
features enabled (try zooming in or panning around), or
http://localhost:8000/admin/ to view the same data in the admin interface, with
the imaginary island base map, instead of the normal world map.

Navigating the code
====================

Hopefully the code is written and commented clearly enough that somebody
familiar with Django can follow along. I have tried to write in a non-throwaway
style, so any code you see here is how I would do something in production.  I
guess, there's one exception to this: the Javascript in
`interface/templates/interface/simple.py` is fairly throwaway code to make the
main page map display work. I would tidy that up a lot in a production
deployment.

The standalone Mapnik server configuration is all in the `map_server/`
directory. The only piece that is particular to a local development
installation is `wms.py`. Otherwise one would do a normal `mod_wsgi`
installation (for example) to call `map_factory.py` in a production environment.

The `lands/` and `adventure/` directories are two small Django apps that purely
manage the GIS data. Have a look at the models there, as well as the data
import scripts in `lands/management/commands/import_land.py` and similarly in
the `adventure/` directory. Fairly standard GeoDjango import techniques being
used there. Also note the `admin.py` files in both directories and how the
overrides to use my imaginary map is set up via `utils/admin_helper.py`. Each
directory contains a `data/` subdirectory that contains the raw shape files
that are imported into GeoDjango. You can inspect those with tools like
`ogrinfo`, as described in the GeoDjango tutorial.

The `interface/` directory hides the main view and the views that are called by
OpenLayers to populate the data. These would be a fair bit more fleshed out in
a "real world" application, but they are correct for the small-scale operation
here. The javascript code in `interface/templates/interface/simple.html` is
also a key part of this equation.
