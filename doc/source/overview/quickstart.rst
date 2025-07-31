Quickstart Guide
================

This guide will get you up and running with ``tfmate`` quickly, showing both
the Python client and command-line interface.

Prerequisites
-------------

- Python 3.10 or higher
- Follow the :doc:`/overview/installation` instructions to install ``tfmate``
- A Terraform project to analyze (optional, for configuration analysis)
- AWS credentials configured (optional, for state file access)

Configuration
-------------

Typically the defaults that ship with ``tfmate`` will work. If you need to change those defaults,
you can create a configuration file at ``~/.tfmate.conf``:

You can configure ``tfmate`` using configuration files or environment
variables. See :doc:`/overview/configuration` for more details.

Basic Usage
-----------

Get Help
~~~~~~~~

.. code-block:: bash

    # Show main help
    tfmate --help

    # Show help for specific command groups
    tfmate analyze --help
    tfmate aws --help
    tfmate terraform --help
    tfmate settings --help

Analyze Terraform Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    # Analyze the current directory for Terraform configuration
    tfmate analyze config

    # Analyze a specific directory
    tfmate analyze config --directory ./infrastructure

    # Show detailed provider and backend information
    tfmate analyze config --show-providers --show-backend

List AWS Services
~~~~~~~~~~~~~~~~~

.. code-block:: bash

    # List all available AWS services
    tfmate aws services

    # List only service names
    tfmate aws services --names-only

    # Filter services by name pattern
    tfmate aws services --filter-name "ec2*"

Get Terraform Version
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    # Get Terraform version from state file
    tfmate terraform version

    # Get version from specific directory
    tfmate terraform version --directory ./infrastructure

Show Settings
~~~~~~~~~~~~~

.. code-block:: bash

    # Show current tfmate settings
    tfmate settings

    # Show settings in JSON format
    tfmate --output json settings

    # Show settings with custom configuration file
    tfmate --config-file ./custom.env settings

Output Formats
~~~~~~~~~~~~~~

.. code-block:: bash

    # Use table format (default) for human reading
    tfmate analyze config

    # Use JSON format for scripting
    tfmate --output json analyze config

    # Use text format for simple output
    tfmate --output text aws services --names-only

    # Use text format for settings
    tfmate --output text settings

Next Steps
----------

Now that you have the basics working:

1. **Usage**: See :doc:`/overview/usage` for more advanced features and detailed examples.
2. **Configuration**: See :doc:`/overview/configuration` for configuration options.
3. **Troubleshooting**: See the troubleshooting sections in each guide for common issues.

Getting Help
------------

- Check the full documentation for detailed examples
- Review the troubleshooting sections in each guide
- Report issues on the GitHub repository