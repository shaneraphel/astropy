Decomposing and Composing Units
*******************************

.. _decomposing:

Reducing a Unit to Its Irreducible Parts
========================================

A unit or quantity can be decomposed into its irreducible parts using
the `Unit.decompose() <astropy.units.core.UnitBase.decompose>` or
`Quantity.decompose() <astropy.units.quantity.Quantity.decompose>`
methods.

Examples
--------

.. EXAMPLE START: Reducing a Unit to Its Irreducible Parts

To decompose a unit with :meth:`~astropy.units.core.UnitBase.decompose`::

  >>> from astropy import units as u
  >>> u.Ry
  Unit("Ry")
  >>> u.Ry.decompose()
  Unit("2.17987e-18 m2 kg / s2")

To get the list of units in the decomposition, the
`~astropy.units.core.UnitBase.bases` and `~astropy.units.core.UnitBase.powers`
properties can be used::

  >>> Ry = u.Ry.decompose()
  >>> [unit**power for unit, power in zip(Ry.bases, Ry.powers)]
  [Unit("m2"), Unit("kg"), Unit("1 / s2")]

You can limit the selection of units that you want to decompose by
using the ``bases`` keyword argument::

  >>> u.Ry.decompose(bases=[u.m, u.N])
  Unit("2.17987e-18 N m")

This is also useful to decompose to a particular system. For example,
to decompose the Rydberg unit of energy in terms of `CGS
<https://en.wikipedia.org/wiki/Centimetre-gram-second_system_of_units>`_
units::

  >>> u.Ry.decompose(bases=u.cgs.bases)
  Unit("2.17987e-11 cm2 g / s2")

Finally, if you want to know how a unit was defined::

  >>> u.Ry.represents
  Unit("13.6057 eV")

.. EXAMPLE END

Automatically Composing a Unit into More Complex Units
======================================================

Conversely, a unit may be recomposed back into more complex units
using the :meth:`~astropy.units.core.UnitBase.compose` method. Since there
may be multiple equally good results, a list is always returned.

Examples
--------

.. EXAMPLE START: Recomposing a Unit into More Complex Units

To recompose a unit with :meth:`~astropy.units.core.UnitBase.compose`::

  >>> x = u.Ry.decompose()
  >>> x.compose()
  [Unit("Ry"),
   Unit("2.17987e-62 foe"),
   Unit("2.17987e-18 J"),
   Unit("2.17987e-11 erg"),
   Unit("13.6057 eV")]

Some other interesting examples::

   >>> (u.s ** -1).compose()  # doctest: +SKIP
   [Unit("Bq"), Unit("Hz"), Unit("2.7027e-11 Ci")]

Composition can be combined with :ref:`unit_equivalencies`::

   >>> (u.s ** -1).compose(equivalencies=u.spectral())  # doctest: +SKIP
   [Unit("m"),
    Unit("Hz"),
    Unit("J"),
    Unit("Bq"),
    Unit("3.24078e-17 pc"),
    Unit("1.057e-16 lyr"),
    Unit("6.68459e-12 AU"),
    Unit("1.4378e-09 solRad"),
    Unit("0.01 k"),
    Unit("100 cm"),
    Unit("1e+06 micron"),
    Unit("1e+07 erg"),
    Unit("1e+10 Angstrom"),
    Unit("3.7e+10 Ci"),
    Unit("4.58743e+17 Ry"),
    Unit("6.24151e+18 eV")]

A name does not exist for every arbitrary derived unit
imaginable. In that case, the system will do its best to reduce the
unit to the fewest possible symbols::

   >>> (u.cd * u.sr * u.V * u.s).compose()
   [Unit("Wb lm"), Unit("1e+08 Mx lm")]

.. EXAMPLE END

Converting Between Systems
==========================

Built on top of this functionality is a convenience method to convert
between unit systems.

Examples
--------

.. EXAMPLE START: Converting Between Unit Systems

To convert between unit systems::

   >>> u.Pa.to_system(u.cgs)
   [Unit("10 Ba"), Unit("10 P / s")]

There is also a shorthand for this which only returns the first of
many possible matches::

   >>> u.Pa.cgs
   Unit("10 Ba")

This is equivalent to decomposing into the new system and then
composing into the simplest units possible in that system, though
:meth:`~astropy.units.core.UnitBase.to_system` adds some extra logic to
return the results sorted such that if a simple combination of base
units exists, it will be put sorted to the front::

   >>> unit = u.m**2 / u.s
   >>> unit.decompose(bases=u.cgs.bases)
   Unit("10000 cm2 / s")
   >>> _.compose(units=u.cgs)
   [Unit("10000 St"), Unit("10000 cm2 / s")]
   >>> unit.to_system(u.cgs)
   [Unit("10000 cm2 / s"), Unit("10000 St")]

``to_system`` first calls ``decompose(bases=system.bases)`` and then
``compose(units=system)``.  When ``system`` is a module, ``compose``
uses the ``Unit`` objects defined in that module.  The ``bases``
sequence is used only by ``decompose``.

The public scripts attached to
`astropy issue 19045 <https://github.com/astropy/astropy/issues/19045>`_
were run on Astropy 8.0.1.  Those files define custom units ``myrh``,
``myU``, and ``myG``, and set ``bases = [u.cm, u.s, u.g]``.

.. code-block:: python

    import astropy.units as u
    import custom_astropy_unit_system as mySys

    u.add_enabled_units(mySys)
    dens = u.g * u.cm**-3
    target = mySys.myU**2 / mySys.myrh**2 / mySys.myG
    dens.to(target)  # 1.5890500480699994e+19
    dens.to_system(mySys)
    # UnitsError: Cannot represent unit g / cm3 in terms of the given units

``compose`` then sees only ``myrh``, ``myU``, and ``myG``.  If
``bases`` is instead those three custom units, ``decompose`` raises
``UnitConversionError: Unit kg can not be decomposed into the requested
bases``.  Each irreducible factor must convert onto one requested base
(as ``u.si.bases`` and ``u.cgs.bases`` do).

For an explicit target unit, call ``Unit.to``.

.. EXAMPLE END
