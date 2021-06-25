.. _director_administration:

=======================
Director Administration
=======================

Directors can be managed using the ``doveadm director`` commands.
See ``doveadm help director`` man page for the full command parameters.

vhost count
-----------

It's possible to assign different amount of work for different director
servers by changing their vhost count. By default each server has it set
to ``100``. If you want one server to have double the number of users, you can
set its vhost count to ``200``. Or if you want one server to have half the
number of users, you can set its vhost count to ``50``. 

For example, if vhost counts for 3 backends are A=50, B=100, C=200, the
probabilities of backends getting connections are:

====== ===================== ======
   A:   50/(50+100+200) =     14%
   B:   100/(50+100+200) =    29%
   C:   200/(50+100+200) =    57%
====== ===================== ======

Changing the vhost count affects only newly assigned users, so it doesn't have
an immediate effect. Running ``doveadm director flush`` causes the existing
connections to be moved immediately.

The ``doveadm director flush`` command works internally by individually moving
all the users between backends. By default 100 users will be moved in
parallel, but this can be overridden with the ``--max-parallel`` parameter.
Using the ``-F`` parameter immediately reassigns the users to their new hosts
without kicking any existing connections. (The ``-F`` parameter shouldn't be
normally used.)

User Status
-----------

The command ``doveadm director status <username>`` shows diagnostic output of
a users mappings in the director ring.

.. code-block:: none

  doveadm director status <username>

  Current: 10.12.90.17 (expires 2018-02-08 12:20:35)
  Hashed: 10.12.90.17
  Initial config: 10.12.90.17

**Current:** The backend host to which the user is currently mapped onto, or
would be mapped onto, if they connected. The information is looked up in the
shared director user directory. This accounts for, e.g., director tags and user
moves. It shows actual, active mappings.

**Hashed:** The backend host which the user would be mapped onto, if they
connected when the Current host mapping has expired from the user directory.
This accounts for, e.g., which backend hosts are currently known to be usable
by the director ring. It also looks at tags. Does not account for moved users,
as it does not do the lookup in the actual user directory.

**Initial config:** Same as the hashed mapping, but does not use currently
known usable backends or vhost changes made to the running ring. Instead it
maps using the mail backends defined in the Dovecot configuration file.

.. code-block:: none

  # doveadm director ring status
  director ip     port type   last failed     status

**last failed**: Either ``never`` or ``date-time``. Restarting director will
set field to ``never``. Network and protocol failures can cause field value to
change to ``date-time``.

**status**: Values can be ``handshaking``, ``syncing``, and ``synced``. Under
normal operations the value should be ``synced``, other values indicate that
some operation (``handshaking`` / ``syncing``) is currently going on.
``handshaking`` can include number of users received, or users sent.
