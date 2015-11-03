Choice items
============

The `ChoiceItem` class is what drives the choices. Each instance
corresponds to a possible choice for your field.


Basic usage
-----------

.. code-block:: python

    class MyChoices(DjangoChoices):
        my_choice = ChoiceItem(1, 'label 1')


The first argument for `ChoiceItem` is the value, as it will be stored
in the database. `ChoiceItem` values can be any type, as long as it matches
the field where the choices are defined, e.g.:

String type:

.. code-block:: python

    class Strings(DjangoChoices):
        one = ChoiceItem('one', 'one')


    class Model1(models.Model):
        field = models.CharField(max_length=10, choices=Strings.choices)


or integer:

.. code-block:: python

    class Ints(DjangoChoices):
        one = ChoiceItem(1, 'one')


    class Model2(models.Model):
        field = models.IntegerField(choices=Ints.choices)


There is also a 'short name'. You can import `C` instead of `ChoiceItem`, if
you're into that.


Labels
------
The second argument to the `ChoiceItem` class is the label.
It's recommended to specify this explicitly if you use
internationalization, e.g.:


.. code-block:: python

    from django.utils.translation import ugettext_lazy as _


    class MyChoices(DjangoChoices):
        one = ChoiceItem(1, _('one'))


If the label is not provided, it will be automatically determined from
the class property, and underscores are translated to spaces. So, the
following example yields:

.. code-block:: python

    >>> class MyChoices(DjangoChoices):
    ...     first_choice = ChoiceItem(1)

    >>> MyChoices.choices
    ((1, 'first choice'),)


Ordering
--------

`ChoiceItem` objects also support ordering. If not provided, the choices are
returned in order of declaration.


.. code-block:: python

    >>> class MyChoices(DjangoChoices):
    ...     first = ChoiceItem(1, order=20)
    ...     second = ChoiceItem(2, order=10)

    >>> MyChoices.choices
    (
        (2, 'second'),
        (1, 'first'),
    )


Values
------
If you really want to use the minimal amount of code, you can leave off the
value as well, and it will be determined from the label.

.. code-block:: python

    >>> class Sample(DjangoChoices):
    ...     OptionA = ChoiceItem()
    ...     OptionB = ChoiceItem()

    >>> Sample.choices
    (
        ('OptionA', 'OptionA'),
        ('OptionB', 'OptionB'),
    )


Default value
-------------

You can specify the default value for the choices with the `_default`
attribute. The underscore serves to prevent nameclashes with choice items that
may be named `default`.

Usage:

.. code-block:: python

    >>> class PizzaTypes(DjangoChoices):
    ...     new_york = ChoiceItem('ny')
    ...     italian = ChoiceItem('it')
    ...
    ...     _default = new_york

    >>> PizzaTypes._default
    'ny'

If not specified, the `_default` is just `None`.

The default value can also be set to any arbitrary value, provided that it's
present in `Choices.values`, e.g.:

.. code-block:: python

    >>> class PizzaTypes(DjangoChoices):
    ...     new_york = ChoiceItem('ny')
    ...     italian = ChoiceItem('it')
    ...
    ...     _default = 'ny'

    >>> PizzaTypes._default
    'ny'

Setting it to a non-existent value will raise a `ValueError`.


`DjangoChoices` class attributes
--------------------------------

The choices class itself has a few useful attributes. Most notably `choices`,
which returns the choices as a tuple.


choices
+++++++

.. code-block:: python

    >>> class Sample(DjangoChoices):
    ...     OptionA = ChoiceItem()
    ...     OptionB = ChoiceItem()

    >>> Sample.choices
    (
        ('OptionA', 'OptionA'),
        ('OptionB', 'OptionB'),
    )


labels
++++++

Returns a dictionary with a mapping from label to value:

.. code-block:: python

    >>> class MyChoices(DjangoChoices):
    ...     first_choice = ChoiceItem(1)
    ...     second_choice = ChoiceItem(2)

    >>> MyChoices.labels
    {'first_choice': 1, 'second_choice': 2}


values
++++++

Returns a dictionary with a mapping from value to label:

.. code-block:: python

    >>> class MyChoices(DjangoChoices):
    ...     first_choice = ChoiceItem(1, 'label 1')
    ...     second_choice = ChoiceItem(2, 'label 2')

    >>> MyChoices.values
    {1: 'label 1', '2': 'label 2'}


validator
+++++++++

Returns a validator that can be used in your model field. This validator checks
that the value passed to the field is indeed a value specified in your choices
class.


.. note::

    This validator had issues in Django 1.7 and up with the new migrations.
    validators have to be deconstructible. This was fixed in the 1.4 release.


Shortcut class methods
----------------------

`as_kwargs(validator=True)`
+++++++++++++++++++++++++++

Using `DjangoChoices` can sometimes get unwieldly. Continuing with the
`PizzaTypes` choices, you might have a model:


.. code-block:: python

    >>> class Pizza(models.Model):
    ...     name = models.CharField(max_length=25)
    ...     pizza_type = models.CharField(
    ...         max_length=10, choices=PizzaTypes.choices,
    ...         default=PizzaTypes.new_york, valiators=[PizzaTypes.validator]
    ...     )

This can be shortcut by using `as_kwargs`:

.. code-block:: python

    >>> class Pizza(models.Model):
    ...     name = models.CharField(max_length=25)
    ...     pizza_type = models.CharField(max_length=10, **PizzaTypes.as_kwargs())


By default, the validator is included. If you wish to specify the validators
explicitly, or not use them at all, you can pass `validator=False` to `as_kwargs`:

.. code-block:: python

    >>> class Pizza(models.Model):
    ...     name = models.CharField(max_length=25)
    ...     pizza_type = models.CharField(max_length=10, **PizzaTypes.as_kwargs(validator=False))
