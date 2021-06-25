.. _director_settings:

=================
Director Settings
=================

See :ref:`dovecot_director` for more details.

Director Specific Settings
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _setting-director_consistent_hashing:

``director_consistent_hashing``
-------------------------------

- Default: ``yes`` (v2.3+), ``no`` (v2.2)
- Values: :ref:`boolean`

Enables consistent hashing to director. This reduces users being moved around
when doing backend changes.

.. todo:: Explain consistent hashing


.. _setting-director_mail_servers:

``director_mail_servers``
-------------------------

- Default: <empty>
- Values: :ref:`string`

A space-separated list of Dovecot backends' IP addresses or DNS names. One DNS
entry may contain multiple IP addresses (probably the simplest way to
configure).

Example:

.. code-block:: none

   director_mail_servers = backend1.example.com backend2.example.com


.. _setting-director_servers:

``director_servers``
--------------------

- Default: <empty>
- Values: :ref:`string`

A space-separated list of Dovecot directors' IP addresses or DNS names. One
DNS entry may contain multiple IP addresses (probably the simplest way to
configure).


Related Configuration Options
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``shutdown_clients``
--------------------

For directors, this option should be ``yes``.

By default, all active sessions will be shutdown when director is reloaded or
restarted. Setting this to ``no`` is dangerous on director as existing
sessions are then not killed when director restarts and are then effectively
unmanaged by director until they disconnect and users can end up on multiple
backends at the same time.

See :ref:`setting-shutdown_clients`.


Configuration Example
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: none

  auth_socket_path = director-userdb

  service director {
    fifo_listener login/proxy-notify {
      mode = 0600
      user = $default_login_user
    }
    inet_listener {
      port = 9090
    }
    unix_listener director-userdb {
      mode = 0600
    }
    unix_listener login/director {
      mode = 0666
    }
    unix_listener director-admin {
      mode = 0600
    }
  }
  service ipc {
    unix_listener ipc {
      user = dovecot
    }
  }
  service imap-login {
    executable = imap-login director
  }
  service pop3-login {
    executable = pop3-login director
  }
  service managesieve-login {
    executable = managesieve-login director
  }

These settings don't usually need to be modified, except the TCP port
``9090`` may be changed: it is used for the directors' internal communication
across the network. 

You'll also want to install some kind of health checking script that will
remove unavailable backends from the pool.
:ref:`OX Dovecot Pro <ox_dovecot_pro_releases>` provides this functionality
in the ``dovemon`` monitoring script, which can be found in the OX Dovecot
Pro repository.

.. seealso::

  :ref:`dovecot_director`
  :ref:`director_capacity_sizing`
