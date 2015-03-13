To get the latest copy of the code, just perform a checkout from SVN :

```
svn checkout http://django-values.googlecode.com/svn/trunk/dbsettings/
```

# INSTALLED\_APPS #

Add it to your `INSTALLED_APPS` setting to make sure Django knows about it.

```
INSTALLED_APPS = (
    ...
    'dbsettings',
    ...
)
```

# URLconf #

In order to modify your options on the fly, you'll need access to the setting editors, so add it in to your urls.py.

```
urlpatterns = patterns('',
    ...
    (r'^settings/', include('dbsettings.urls')),
    ...
)
```

# Syncing the database #

There is one model that needs to be installed into the database. Just run `syncdb` and the table will be created as normal.

**Note:** This does _not_ have to be done prior to adding settings to your models. If you run `syncdb` _after_ setting up your value attributes, dbsettings won't complain. However, you will definitely need to run `syncdb` before attempting to edit any settings.

# That's it! #

The rest of the work is done in each individual models.py, and is covered under [Usage](Usage.md).