.. title: Fixing Time
.. slug: infra-fixing-time
.. date: 2012-02-19
.. taxonomy: Contributors/Infrastructure

=============
Fixing Time
=============

If time stalls on a VM, set the time with ``date -s`` then run ``ntpdate`` to let
it sync up as normal.

Smallest. SOP. Ever.
