
Simple configuration
====================

Todo: explain...

In order to easily turn on and off custom install steps, a simple configuration feature was added. It comes in 3 steps:

#. Defining a ``config.txt`` file
#. Running ``./install.bsh --config`` to trigger the config
#. Using ``skip_if_env`` checks

``config.txt``
--------------

The Configuration file is a simple text file, where each line declare an environment variable followed by a string that is presented at configuration time

.. rubric:: Example ``config.txt``

.. code::

   SKIP_FOOBAR Would you like to disable setting up foobar?
   ENABLE_PYTHON Would you like to enable setting up python in ~/mypython?

Todo: show config output, say y to foobar, and n to python

``local_install``
-----------------

Todo: explain...

.. rubric:: ``local_install``

.. code:: bash

   SKIP_FOOBAR=1

``custom.d/<my_plugin>``
------------------------

.. rubric:: Example ``custom.d/my_foobar``

.. code:: bash

   #!/usr/bin/env false

   skip_if_env SKIP_FOOBAR || return 0
   # OR
   # !skip_if_env ENABLE_PYTHON || return 0

   # Start installing stuff here...
