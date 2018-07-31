
# Extended Django Serializer Module

## Working on Python 3, Django 2

### Install & Setup

If you have previous installed `wadofstuff-django-serializers`, then:

    pip uninstall wadofstuff-django-serializers

Else:

    pip install git+https://github.com/hasahmad/wadofstuff-django-serializers --upgrade

In you project `settings.py` file add this:

    SERIALIZATION_MODULES = { 'json': 'wadofstuff.django.serializers.json_reloaded' }

Then in your code `from django.core import serializers` and start using it.

### Description

The wadofstuff.django.serializers python module extends Django's built-in
serializers, adding 3 new capabilities inspired by the Ruby on Rails JSON
serializer. These parameters allow the developer more control over how their
models are serialized.

The additional capabilities are:

- excludes - a list of fields to be excluded from serialization. The
  excludes list takes precedence over the fields argument.
- extras - a list of non-model field properties or callables to be
  serialized.
- relations - a list or dictionary of model related fields to be followed
  and serialized.

# Example of serializing a relation

    >>> serializers.serialize('json', Group.objects.all(), indent=4, relations=('permissions',))
    [
        {
            "pk": 2,
            "model": "auth.group",
            "fields": {
                "name": "session",
                "permissions": [
                    {
                        "pk": 19,
                        "model": "auth.permission",
                        "fields": {
                            "codename": "add_session",
                            "name": "Can add session",
                            "content_type": 7
                        }
                    }
                ]
            }
        }
    ]
