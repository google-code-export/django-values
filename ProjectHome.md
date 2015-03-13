# dbsettings #
## (was django-values) ##

An add-on application for Django that allows placeholders for settings to be defined in Python, while their values are set by staff using an editor while the server is up and running. Many value types are available, and they each map to a native Python type, so model methods and other Python code can access them as standard class attributes.

Much effort has also been made to reduce the overhead of this feature, so that the database is only queried once during each server restart, and only updated when the values themselves are updated.

**NOTE:** This is not intended as a replacement for `settings.py`. This is designed for values that are expected to change based on the needs of the site or its users, so that such changes don't require so much as a restart. `settings.py` is still the place to go for settings that will only vary by project.

One way of looking at it is that `settings.py` is for those things that are required from a technical perspective (database connections, installed applications, avaialble middleware, etc), while `dbsettings` is best for those things that are required due to organizational policy (quotas, minimum requirements, etc). That way, programmers maintain the technical requirements, while administrators maintain organizational policy.

Requirements:
  * [Django](http://www.djangoproject.com/) SVN ([5302 revision](http://code.djangoproject.com/changeset/5302) or later)

Benefits:
  * Placeholders remove specific values from Python code, alleviating maintenance
    * These are defined and instantiated separately, so they can be reused in multiple locations
    * The placeholder may (optionally) include a description of the value
      * This may be used by programmers as a reference to know what the value is for
      * This is also used by the value editor to show the field label
      * If no description is supplied, it is derived from the attribute name (just like model fields)
  * Such values may be accessed from Python code just like any other standard class attribute
    * Settings are read-only when accessed on their classes, to prevent accidental modification by Python code
  * Value editing is relegated to qualified personnel
    * Editor automatically requires staff membership, regardless of anything else
    * An additional permission is added for each models which use these values
      * That permission may then be granted to users or groups, like any other
      * The editor will be customized based on the user's permissions
    * Superusers automatically have permission to edit all values
  * A standard Django model ensures that values are persisted across server restarts
    * All values are stored in a single model, regardless of type, for simplicity
  * Values are retrieved from in-memory cache when accessed, so database interaction is minimal
    * The database is only queried once when loading, and never again until the server is restarted
    * The value editor only makes database updates if the new value is in fact different than the existing
    * All database operations either use the primary key in the `WHERE` clause, or no `WHERE` clause at all, so access is fast
  * 100% contrib app, with no modifications to official Django code
    * Can simply dropped in and used, in any project, with any backend
    * No fancy magic, just the same helper methods Django itself uses (namely `contribute_to_class`)