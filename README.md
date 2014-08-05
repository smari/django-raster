Rasters for Django
==================

Django-raster provides the simplest possible raster data integration for Django projects using PostGIS database backends.

**Note: This package requires PostGIS >= 2.0**

Setup
-----

1. Install package with `pip install django-raster`

2. Add "raster" to your INSTALLED_APPS setting like this::

        INSTALLED_APPS = (
            ...
            'raster',
        )

3. Run `python manage.py migrate` to create the raster models.

Description
-----------
Django-raster provides the simplest possible integration of raster
data in Django. Raster files can be uploaded and parsed through the admin interface. The raw raster data can be parsed asynchronously if [Celery](http://celeryproject.org) is integrated into the project.

Once a raster file is uploaded, the parser will extract the data in the raste files and store the rasters in tiles of 100x100 pixels in a PostGIS raster table. 

For this, the package defines two models and one field:

* ``RasterLayer`` - storing the raw raster files and metadata (for example rasterfile=raster.tif and srid=4326)

* ``RasterTile`` - storing the parsed raster in PostGis. The raster data is split into tiles of 100x100 pixels and each tile is stored as an instance of RasterTile. The raster data itself is stored in a *RasterField* within the RasterTile model.

* ``RasterField`` - an extension of the Django `BinaryField` class to store the raster data. The only difference to the BinaryField class is that the *db_type* is redefined as *raster*, such that the data is stored as raster in PostGIS.

Due to the simplicity of the implementation, no spatial querying can be done on the raster data through python. This package is not integrated with GeoDjango and has none of the features that GeoDjango spatial models have. Proper integration of rasters into Geodjango with spatial querying etc. is beyond the scope of this package. If required however, custom SQL can be used to make spatial queries on the raster data.

Usage
-----
After setting the package up, you can upload raster files through the admin interface using the RasterLayer model. Specify a layer name, the raster data type (continuous, categorical, mask or rank ordered), the raster's srid, the nodata value and the raster file to be uploaded.

Upon saving the raster file, django-raster automatically loads the raster data from the file into a raster column in the PostGIS database. These tiles will be stored in the RasterTile model, which should not be edited directly but only through adding and deleting entire RasterLayers instances.

The RasterLayer instances have a *parse_log* field, which stores information about the parsing process.


### Asynchronous parsing with Celery

Note that for large rasters this parsing step might take a while, so your html request might time out. It is therefore recommented to use [Celery](http://celeryproject.org) in combination with django-raster.

If celery is found on the python path of your project, it is automatically used for parsing the raster asynchronously. After uploading a file the parsing routine is added to the celery queue of your application.
