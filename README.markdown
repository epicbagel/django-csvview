A Django View Mixin for turning a ListView (or any view with `get_queryset`) into a highly customizable CSV download utilizing [tablib][].

There are [other good options][option] for turning Django models into CSVs, but this was built for a use case I kept encountering where existing methods weren't flexible enough: with CSVs you often have to match the input format expected by some other system, so automatically generating headers from field names or being limited to only outputting data from your model directly won't cut it. Hence: this app.

### Usage

Simply define a dictionary mapping CSV column headers to attributes on your model (using an OrderedDict if order matters) and then use `CSVViewMixin` and set `csv_mapping` in your view:

    from collections import OrderedDict
    from csvview.views import CSVViewMixin
    from django.views.generic import ListView
    from .models import Foo

    csv_mapping = OrderedDict([
        ('COMPANY', 'company.name'),   
        ('PHONE_NUM', 'contact.phone_number'),
        ('ADDRESS', 'contact.format_address'),
    ])

    class FooCSVView(CSVViewMixin, ListView):
        csv_mapping = csv_mapping
        queryset = Foo.objects.all()

And that's it! Wire up this view to a url, and you'll get a CSV like this:

    COMPANY, PHONE_NUM, ADDRESS
    "Jon's Widget Shop", "913-333-3333", "1234 Main Street, Anywhere, KS 66204"

In the `csv_mapping`: 

- dotted paths are unpacked (so 'company.name' returns `model.company.name`)
- if the result is a callable, it is called (i.e. `model.contact.format_address()`)

Just like with any ListView, you can customize `get_queryset to modify what objects are passed into the CSV.

You can also pass in an optional lambda `csv_post_process`, which when called, will determine whether or not to display the CSV in the row. The following will strip out any rows which contain the string "$0.00"

    class FooCSVView(CSVViewMixin, ListView):
        csv_mapping = csv_mapping
        queryset = Foo.objects.all()
        csv_post_process = lambda *args: "$0.00" not in str(args[1])

### Callable Values

In addition to a string representing a dotted path for your object, you can also use a callable as a value in your mapping dict. The callable is passed two arguments: the object for that row, the header string for that column. 

This is useful when you need to add something to the csv that is either totally unrelated to the actual model for that row, or if you simply just cannot know what the value should be in advance. For example, say you need to add a `FOO_ID` to the csv above. `FOO_ID` is always '37FZ1'. Why? Who knows. Maybe accounting says it has to be there. Rather than messing up your models with random constants like this, you can do this:

    csv_mapping['FOO_ID'] = lambda *args: '37FZ1'

And since the callable is passed the object and the label for the row, you can also use this to calculate values for that row on the fly. 

*TODO I'm having a hard time thinking of a contrived example for this right now, but trust me, this can come in handy*


[tablib]: http://python-tablib.org
[option]: https://github.com/joshourisman/django-tablib


### Thanks

Thanks to [BOLD Internet Solutions][bold] for supporting the development, and allowing the open sourcing, of this package.

[bold]: http://www.bold-is.com/
