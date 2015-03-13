Because database-backed settings can only be edited by staff members, value types only need to differentiate basic data types, rather than validate specific inputs. However, some number-validing types have been included for ease of administration.

**Note:** I'm open to adding new value types, these are simply the only ones that I've run into a need for so far.

# BooleanValue #

Displays a checkbox in the editor, while providing `True` or `False` to Python code, depending on its status.

# DurationValue #

Accepts and validates a length of time, which would be made available to Python as a `timedelta` object. Particularly useful for timeouts and delays. For instance, a `DurationValue` could determine how long stories may be displayed on the front page before being removed, how long ads should run before requiring additional payment, or a maximum length for user-contributed audio or video content.

**Note**: The UI for this is will eventually be based on [DurationField](http://code.djangoproject.com/ticket/2443), if and when that gets integrated. Currently it just uses a simple text field to allow entry of seconds as a decimal (float).

# FloatValue #

Accepts and validates a number that is converted to `float` when accessed.

# IntegerValue #

Accepts and validates an integer that is converted to `int` when accessed.

**Note:** there is no `SmallIntegerValue`, since `SmallIntegerField` exists primarily to control column size in the database representation of the model. Since all values are stored using a single model, the value is stored as `CharField(maxlength=255)`, so a `SmallIntegerValue` would provide no advantage whatsoever.

# PercentValue #

Accepts a number from 0 to 100, which is converted to float and divided by 100 when accessed. This enables the administration user to enter a value as a percentage (e.g., 60), while Python code can treat it as a decimal multiple (0.6). Consider the following:

```
class TaxSettings(dbsettings.Group):
    sales_tax = dbsettings.PercentValue()

class Product(models.Model):
    name = models.CharField(maxlength=255)
    price = models.FloatField(max_digits=5, decimal_places=2)

    taxes = TaxSettings()

    def get_sales_tax(self):
        return self.price * Product.taxes.sales_tax
```

**Note**: The range of 0 to 100 probably won't work for all use cases, and thus may be lifted later on.

# PositiveIntegerValue #

Like `IntegerValue`, but with a minimum value of 0.

# StringValue #

Accepts any string up to 255 characters, which is retrieved from the form and from the database as `str` already, so no conversion is necessary when accessing it from Python.

**Note:** This may seem like it would be hardly useful, since strings wouldn't be used in calculations like the other types would be, but it would be very useful for things like email subject lines (and probably others). It was easy enough to implement, anyway.