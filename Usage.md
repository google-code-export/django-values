# Importing #

Importing is as simple as any other app, simply add it to the import list at the top of `models.py`. The trick is to make sure you import the base of the package, not `dbsettings.models`. This is to prevent a name clash with `django.db.models` and also to make settings a bit more distinct from fields.

```
from django.db import models
import dbsettings
...
```

# Adding settings to your apps #

Settings are declared to subclasses of a special class, `dbsettings.Group`. This class is then instantiated wherever you'd like to use it. This may be done in the main module namespace, or within the namespace of an individual model. In fact, a single `Group` class may be instantiated in multiple places, even in multiple apps. Each instantiation creates a new set of values at the specified location.

The syntax used for individual settings is nearly identical to that of fields. Simply declare it using Django's standard declarative syntax, making sure to use `dbsettings` instead of `models`.

```
class ThreadOptions(dbsettings.Group):
    popularity_threshold = dbsettings.PositiveIntegerValue()

class Thread(models.Model):
    title = models.CharField(maxlength=255)

    options = ThreadOptions()

    def is_popular(self):
        return self.posts.count() > Topic.options.popularity_threshold

class Post(models.Model):
    thread = models.ForeignKey(Thread, related_name='posts')
    author = models.ForeignKey(User)
    body = models.TextField()
```

You may supply any name you like for the model attribute that will hold your settings. The example uses "options," but any name will suffice.

By default, the value description that will be shown in the editor is derived from the attribute name (just like fields). However, also like fields, settings may also take a positional argument for a description. This description will be used in the editor instead of the default, which is based on the attribute name. It's best to start the description with a lower-case letter, as it will be captialized automatically when necessary.

```
class ProductOptions(dbsettings.Group):
    sale_length = dbsettings.DurationValue("how long a sale should last")
options = ProductOptions()

class ProductOptions(values.Options):
    preferred_markup = values.PercentValue("how much markup I'd like to get for an item")

class Product(models.Model):
    name = models.CharField(maxlength=255)
    description = models.TextField()
    price = models.FloatField("Retail Price", max_digits=5, decimal_places=2)
    cost = models.FloatField("Store Cost", max_digits=5, decimal_places=2)

    options = ProductOptions()

    @property
    def profit(self):
        return self.price - self.cost

    def should_promote(self):
        return (self.profit / self.cost) > Product.options.preferred_markup
```

Individual settings may also take a list of choices that will be available in the editor, which is particularly useful for apps meant for distribution. The syntax for setting choices is the same as for choices in model fields and the newforms `ChoiceField`.

```
verbosity_choices = (
    (0, 'Silent'),
    (1, 'Warnings messages only'),
    (2, 'Warnings and status messages'),
)

class LogginOptions(dbsettings.Group):
    enabled = dbsettings.BooleanValue()
    verbosity = dbsettings.IntegerValue(choices=verbosity_choices)
```

**Note:** At the moment, module-level options should be instantiated at the top of the module, before any models that use options. Functionally, they can be defined anywhere in the file, but the editor currently doesn't show any kind of heading for them, so they're hard to identify if they're not at the top of the file. This limited is expected to be lifted in a future release, but backwards-compatibility will not be impacted.

`Group` objects may also be added together to apply multiple sets of values to a single model or module. This can be especially useful if other contributed apps supply their own options, as they can be easily combined in a single model. The standard addition operator can be used for this, as follows:

```
class TimeOptions(dbsettings.Group):
    maximum_length = dbsettings.DurationValue()

class VolumeOptions(dbsettings.Group):
    auto_normalize = dbsettings.BooleanValue("Normalize audio on upload")
    normalized_volume = dbsettings.IntegerValue()

class Song(models.Model):
    name = models.CharField(maxlength=255)
    file = models.FileField(upload_to="/path/to/audio")

    options = TimeOptions() + VolumeOptions()
```

# Assigning permissions #

In order to delegate the task of editing settings to different users, every model using a value also receives an additional permission, which controls the available values when a user views the value editor. These additional permissions must be added to the database in order to be utilized, so sipmly run `syncdb` again and these permissions will be added.

Once added to the database, these are permissions like any other, and can be assigned to users or groups; superusers automatically have permission to edit all values.

**Note:** For various technical reasons, permissions are only added for model-level options. Module-level options are therefore only editable by superusers.

# Editing settings #

Once settings are added to your apps, simply start your server and they'll be automatically found and made available to the value editor. Simply point your browser to !http://localhost:8000/settings/ (substitute your own domain/port and path) and edit at your leisure.

Settings will appear in the editor in the order they were provided to your code. The order of `Group` instantiation is most important, with the order of setting declarations taking effect after that. If multiple `Group` classes are added together, their settings will appear in the editor according to their `Group` classes from left to right. Try it out, you'll get the hang of it.

# Referencing settings in Python #

As the above examples demonstrate, Python code can use these settings as standard class attributes. Internally, dbsettings use descriptors to take these references and return the current value in the proper type, so your code will always get the right value in the right type. This helps keep your model methods clean and concise, by using a notation that already makes perfect sense.

**Note:** Settings groups defined on models are only available on the model itself, _not on any individual instances_. Attempting to access a group on an instance of your model will raise an `AttributeError`. This is to ensure that programmers don't get confused by referencing an attribute on an object when that attribute doesn't relate to that specific object. Basically, it's my way of enforcing some level of "good" programming practice.