Fork fork fork fork!
--------------------

Forked from `dreipol's repo`_ in order to get out of using ``gevent``. I'm
looking at using ``django-storages``'s boto3 backend (instead of the older boto
backend) for uploading to Amazon S3. boto3 uses thread locks for multipart
up/downloads. Attempting to use dreipol's collectfaster with a boto3 backend
fails -- gevent throws a ``LoopExit: This operation would block forever`` when
spawning workers.

An alternate tactic here would be to convince boto3 to not use threads -- I did
this by messing with ``boto3.s3.inject`` and making all of the
``TransferConfig()`` calls use ``use_threads=False`` (see `Boto 3 docs`_). I
didn't see a nice way to configure ``django-storages`` to pass along something
like that, though.

So, here we are.

====================
django-collectfaster
====================

This package extends Django's ``collectstatic`` management command with a ``--faster`` argument that activates the
parallel file copying. The speed improvement is especially helpful for remote storage backends like S3.

Quickstart
----------

Install django-collectfaster::

    pip install django-collectfaster

Configure installed apps in your ``settings.py`` and make sure ``collectfaster`` is listed before ``django.contrib.staticfiles``::

    INSTALLED_APPS = (
        ...,
        'collectfaster',
        'django.contrib.staticfiles',
        'storages',
        ...,
    )

If you are using S3 with ``django-storages`` you probably already have this configured in your ``settings.py``::

    AWS_S3_HOST = 's3-eu-west-1.amazonaws.com'
    AWS_STORAGE_BUCKET_NAME = '<your_aws_bucket_name>'

Set the storage backends for your static and media files in the ``settings.py``::

    STATICFILES_STORAGE = 'collectfaster.backends.S3StaticStorage'
    DEFAULT_FILE_STORAGE = 'collectfaster.backends.S3MediaStorage'


You should split your static and media files on your S3 in different folders and configure it in the ``settings.py``::

    STATICFILES_LOCATION = 'static'
    MEDIAFILES_LOCATION = 'media'


Set the ``STATIC_URL`` at least on your production settings::

    STATIC_URL = 'https://%s/%s/%s/' % (AWS_S3_HOST, AWS_STORAGE_BUCKET_NAME, STATICFILES_LOCATION)


Usage
-----

Collect your static files parallel::

    python manage.py collectstatic --faster


Set the amount of workers to 30::

    python manage.py collectstatic --faster --workers=30


Credits
-------

Tools used in rendering this package:

*  Cookiecutter_
*  `cookiecutter-djangopackage`_

.. _Cookiecutter: https://github.com/audreyr/cookiecutter
.. _`cookiecutter-djangopackage`: https://github.com/pydanny/cookiecutter-djangopackage
.. _`dreipol's repo`: https://github.com/dreipol/django-collectfaster
.. _`Boto 3 docs`: http://boto3.readthedocs.io/en/latest/reference/customizations/s3.html#boto3.s3.transfer.TransferConfig
