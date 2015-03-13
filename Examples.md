# Blog #

In a very simple (and not very likely) example, a blog application could use settings to decide how many comments to allow for a blog entry. The following model could be used to describe this scenario, while allowing the application administrator to change the limit at any time. Only the blog entry model is described here, while a comment model would be included as well.

```
class CommentOptions(dbsettings.Group):
    comment_limit = dbsettings.IntegerValue('Maximum number of comments a blog can receive')

class BlogEntry(models.Model):
    title = models.CharField(maxlength=255)
    body = models.TextField()
	
    options = CommentOptions()
	
    def can_receive_comments(self):
        return self.comments.count() < BlogEntry.options.comment_limit
```

While it's unlikely that a blog application would do so, templates and view code could then use `can_receive_comments` to determine whether to allow users to post comments on a particular entry, based on a value that can be changed at any time.

# Brewery database #

In a considerably more likely scenario, the application is an on-line database of breweries around the country. In among the variety of information that might be included, one useful bit of information might be the number of people a particular brewery employs. Given the number of microbreweries out there, it might also be interesting to see which ones legally qualify as small businesses. The following model provides what's needed.

```
class TaxOptions(dbsettings.Group):
    small_business_employees = dbsettings.IntegerValue('Maximum number of employees for a small business')

class Brewery(models.Model):
    name = models.CharField(maxlength=255)
    website = models.URLField(blank=True)
    employees = models.PositiveIntegerField()

    options = TaxOptions()

    def is_small_business(self):
        return self.employees <= Brewery.options.small_business_employees
```

The value for `small_business_employees` can be retrieved from the U.S. Small Business Administration, and can be updated by staff any time that organization changes it's mind.

# Human resources #

In more complicated situation, a human resources application needs to keep track of employee wages and make sure everyone is making at least the minimum wage defined by government. The mimimum wage value itself does not change for individual employees, but it does change over time, and so is a perfect candidate for run-time configuration. The following model demonstrates how a value reference could be included in code, while allowing the actual value itself to be edited by an administrator.

```
class HROptions(dbsettings.Group):
    minimum_wage = dbsettings.FloatValue()

class EmployeeManager(models.Manager):
    def get_underpaid_employees(self):
        return self.filter(hourly_wage__lt=Employee.options.minimum_wage)

class Employee(models.Model):
    first_name = models.CharField(maxlength=255)
    last_name = models.CharField(maxlength=255)
    hourly_wage = models.FloatField()

    options = HROptions()

    def is_minimum_wage(self):
        return self.hourly_wage == Employee.options.minimum_wage
```

The above model also shows how the value could be used to implement other useful features. The `is_minimum_wage` method could be used in templates to highlight employees that might deserve a raise, while `get_underpaid_employees` returns a list of employees that _need_ a raise in order to satisfy government guidelines. The latter method would likely be used to feed a separate view that managers could use after a wage increase to automatically update all affected employees.