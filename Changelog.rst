=========
Changelog
=========
1.4.3
-----
DjangoChoices now handles the 'default' choice specially. It allows
to set the default choice on the class level::

    >>> class PizzaTypes(DjangoChoices):
    ...     new_york = ChoiceItem('ny')
    ...     italian = ChoiceItem('it')
    ...
    ...     _default = new_york

    >>> PizzaTypes._default
    ... # 'ny'

The `_default` attribute is underscore-prefixed to prevent nameclashes with
existing `ChoiceItem`s.

Additionally, a shortcut for model fields was added: `as_kwargs()`. Usage::

    class Model(models.Model):
        pizza_type = models.CharField(max_length=10, **PizzaTypes.as_kwargs())

Which is equivalent with::

    class Model(db.Model):
        pizza_type = models.CharField(
            max_length=10, choices=PizzaTypes.choices,
            default=PizzaTypes.new_york, validators=[PizzaTypes.validator]
        )

`as_kwargs` takes an optional `validator` kwarg, whether to include the validators
or not. If you have extra validators, pass `validator=False`.


1.4.2
-----
Added Django 1.9 and Python 3.5 support.

1.4 to 1.4.1
------------
This is a small release that fixes some ugliness. In Django 1.7 and up, the
validator can now be used as documented in the readme. Before this version, the
error::

    ValueError: Cannot serialize: <bound method DjangoChoicesMeta.validator of <class 'choices.models.MyModel.Choices'>>

would be raised, and a workaround was to define a validator function calling the
choices' validator, requiring everything to be defined in the module scope.

This is now fixed by moving to a class based validator which is deconstructible.


1.3 to 1.4
----------
* Added support for upcoming Django 1.9, by preferring stlib SortedDict over
  Django's OrderedDict
* Added pypy to the build matrix
* Added coverage to the Travis set-up
