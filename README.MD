
# Extended Django Serializer Module

## Working on Python 3, Django 2

### Install & Setup

If you have previous installed `wadofstuff-django-serializers`, then:

    pip uninstall wadofstuff-django-serializers

Else:

    pip install git+https://github.com/hasahmad/wadofstuff-django-serializers --upgrade

If you are using pipenv, then use this command:

    pipenv install git+https://github.com/hasahmad/wadofstuff-django-serializers#egg=wadofstuff-django-serializers

Or if that is not working, then copy this to Pipfile:

    wadofstuff-django-serializers = {git = "https://github.com/hasahmad/wadofstuff-django-serializers"}

In you project `settings.py` file add this:

    SERIALIZATION_MODULES = { 'json': 'wadofstuff.django.serializers.json_reloaded' }

Then in your code `from django.core import serializers` and start using it.

### Description (from <https://wadofstuff.blogspot.com/2009/02/django-full-serializers-part-i.html>)

The wadofstuff.django.serializers python module extends Django's built-in
serializers, adding 3 new capabilities inspired by the Ruby on Rails JSON
serializer. These parameters allow the developer more control over how their
models are serialized.

The additional capabilities are:

- excludes - a list of fields to be excluded from serialization. The excludes list takes precedence over the fields argument.
- extras - a list of non-model field properties or callables to be serialized.
- relations - a list or dictionary of model related fields to be followedand serialized.

# Backwards Compatibility

The Wad of Stuff serializers are 100% compatible with the Django serializers when serializing a model.

    >>> from django.contrib.auth.models import Group
    >>> from django.core import serializers
    >>> print serializers.serialize('json', Group.objects.all(), indent=4)
    [
        {
            "pk": 2,
            "model": "auth.group",
            "fields": {
                "name": "session",
                "permissions": [
                    19
                ]
            }
        }
    ]

# Excludes

    >>> print serializers.serialize('json', Group.objects.all(),
                indent=4, excludes=('permissions',))
    [
        {
            "pk": 2,
            "model": "auth.group",
            "fields": {
                "name": "session"
            }
        }
    ]

# Extras

The extras option allows the developer to serialize properties of a model that are not fields. These properties may be almost any standard python attribute or method. The only limitation being that you may only serialize methods that do not require any arguments.

For demonstration purposes in this example I monkey patch the Group model to have a get_absolute_url() method.

    >>> def get_absolute_url(self):
    ...     return u'/group/%s' % self.name
    ...
    >>> Group.get_absolute_url = get_absolute_url
    >>> print serializers.serialize('json', Group.objects.all(),
            indent=4, extras=('__unicode__','get_absolute_url'))
    [
        {
            "pk": 2,
            "model": "auth.group",
            "extras": {
                "get_absolute_url": "/group/session",
                "__unicode__": "session"
            },
            "fields": {
                "name": "session",
                "permissions": [
                    19
                ]
            }
        }
    ]

# Relations

The Wad of Stuff serializers allow you to follow and serialize related fields of a model to any depth you wish. This is why it is considered a "full serializer" as opposed to Django's built-in serializers that only return the related fields primary key value.

When using the relations argument to the serializer you may specify either a list of fields to be serialized or a dictionary of key/value pairs. The dictionary keys are the field names of the related fields to be serialized and the values are the arguments to pass to the serializer when processing that related field.

This first example shows the simplest way of serializing a related field. The Group model has a ManyToManyField to the Permission model. In this case there is only one permission but if there were more then each would be serialized.

    >>> print serializers.serialize('json', Group.objects.all(),
            indent=4, relations=('permissions',))
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

The simple case may be all you need but if you want more control over exactly which fields or extras are included, excluded, and the depth of relations to follow then you need to pass a dictionary in the relations option. This dictionary is a series of nested dictionaries that are unrolled and passed as arguments when serializing each related field.

    >>> print serializers.serialize('json', Group.objects.all(),
            indent=4, relations={'permissions':{'fields':('codename',)}})
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
                            "codename": "add_session"
                        }
                    }
                ]
            }
        }
    ]

The relations option in this example roughly translates to a call to `serialize('json', permissions_queryset, fields=('codename',))` when the permissions field is serialized.

## Serializing deeper relations

The power of the relations option becomes obvious when you see it in action serializing related fields that are 2 or more levels deep. Below the content_type ForeignKey field on the Permission model is also serialized.

    >>> print serializers.serialize('json', Group.objects.all(),
            indent=4, relations={'permissions':{'relations':('content_type',)}})
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
                            "content_type": {
                                "pk": 7,
                                "model": "contenttypes.contenttype",
                                "fields": {
                                    "model": "session",
                                    "name": "session",
                                    "app_label": "sessions"
                                }
                            }
                        }
                    }
                ]
            }
        }
    ]

## Combining options

You may also combine the other options when serializing related fields. In the example below I am excluding the `content_type.app_label` field from being serialized.

    >>> print serializers.serialize('json', Group.objects.all(),
            indent=4, relations={'permissions':{'relations':{'content_type':{'excludes':('app_label',)}}}})
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
                            "content_type": {
                                "pk": 7,
                                "model": "contenttypes.contenttype",
                                "fields": {
                                    "model": "session",
                                    "name": "session"
                                }
                            }
                        }
                    }
                ]
            }
        }
    ]
