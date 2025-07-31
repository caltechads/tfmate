Installation
============

This guide covers how to install the ``tfmate`` package and its dependencies.

Prerequisites
-------------

Before installing ``tfmate``, ensure you have:

- Python 3.10 or higher
- `uv <https://docs.astral.sh/uv/>`_ or `pip <https://pip.pypa.io/en/stable/>`_

Installation Methods
--------------------

From PyPI with ``pip``
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    pip install tfmate

From PyPI with ``uv``
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sh -c "$(curl -fsSL https://astral.sh/uv/install)"
    uv tool install tfmate
    # Ensure you have ./local/bin in your PATH, since that's where uv puts the
    # executable
    tfmate --help


From Source
~~~~~~~~~~~

If you want to install from the latest development version:

.. code-block:: bash

    git clone https://github.com/cmalek/tfmate.git
    sh -c "$(curl -fsSL https://astral.sh/uv/install)"
    cd tfmate
    uv tool install .

Verification
------------

After installation, verify that ``tfmate`` is properly installed:

.. code-block:: shell

    tfmate --help


Configuration
-------------

After installation, you may want to configure ``tfmate`` for your specific
environment.  See :doc:`configuration` for detailed configuration options.

Getting Help
------------

If you encounter issues during installation:

1. Check the `GitHub issues <https://github.com/caltechads/tfmate/issues>`_
2. Review the troubleshooting section above
3. Ensure your Python environment meets the prerequisites
4. Try installing in a virtual environment to isolate dependencies